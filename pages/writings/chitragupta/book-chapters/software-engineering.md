---
hide:
  - navigation
  - toc
---

# Building, Deploying and Operating a Digital Twin: A Software Engineering Approach

**Disclaimer:** This book chapter has been generated using
[chitragupta](https://prasad.talasila.in/chitragupta).
Despite some potential for hallucination, the ideas communicated in this
book chapter are accurate. Please send your corrections and suggestions to
<prasad.talasila@gmail.com>

> **Summary.** Most introductions to digital twins tell you what a digital
> twin *is*. This one assumes you already accept the idea and asks a harder
> question: what does it take to *build* one, *ship* it, and *keep it
> running* for two years without it quietly going wrong? We treat a digital
> twin as what it actually is once you start typing — a small distributed
> system with sensors on one end, models in the middle, and consequences at
> the other end. We build one end-to-end in Python for a heated box, package
> it in containers, decide which parts run on a Raspberry Pi and which run in
> a data centre, wrap it in a CI pipeline that can test something that talks
> to hardware, and then look at the three ways it degrades in production:
> latency, drift, and attackers.

## Prerequisites

You should be comfortable with Python (functions, classes, `numpy`), the
shell, and `git`. You should know roughly what HTTP and a REST API are. Prior
exposure to containers helps but is not assumed — Section 6 introduces what
you need. No control theory, no thermodynamics, and no prior digital twin
knowledge is required; where we need physics we derive the one equation we
use from scratch.

This tutorial is a companion to, not a replacement for, an introduction to
the *concept* of digital twins. If the phrase "digital twin" is entirely new
to you, read a conceptual introduction first and come back; we spend exactly
one section (Section 2) on definitions and then move on to code.

## Learning objectives

By the end of this chapter, you will be able to:

1. **Decompose** a digital twin into named architectural components —
   physical-twin interface, ingestion, storage, models, services,
   presentation, orchestration — and explain what each one owns.
2. **Implement** a working synchronisation loop in Python that reads sensor
   data, advances a calibrated model, computes a residual, and turns that
   residual into a decision, and **quantify** how good it is.
3. **Package and place** a digital twin's components across the edge–cloud
   continuum, justifying each placement with an explicit latency, bandwidth,
   or availability argument rather than a hunch.
4. **Design a CI pipeline** for a digital twin, including the kinds of test
   that ordinary web-application pipelines never need — model-versus-data
   tests, simulated-physical-twin tests, and hardware-in-the-loop tests.
5. **Diagnose** an operational digital twin: distinguish a sensor fault from
   model drift from an attack, and name a mitigation appropriate to each.

## Outline

1. [Motivation: the twin you cannot `pip install`](#1-motivation-the-twin-you-cannot-pip-install)
2. [What you are actually building](#2-what-you-are-actually-building)
3. [A reference architecture you can build from](#3-a-reference-architecture-you-can-build-from)
4. [Worked example 1: a thermal box twin in 150 lines](#4-worked-example-1-a-thermal-box-twin-in-150-lines)
5. [Deciding where the code runs](#5-deciding-where-the-code-runs)
6. [Worked example 2: packaging, configuring, and reusing it](#6-worked-example-2-packaging-configuring-and-reusing-it)
7. [DevOps for digital twins](#7-devops-for-digital-twins)
8. [Operating it: latency, drift, and attackers](#8-operating-it-latency-drift-and-attackers)
9. [Anti-patterns](#9-anti-patterns)
10. [Exercises](#10-exercises)
11. [Where to go next](#11-where-to-go-next)

## 1. Motivation: the twin you cannot `pip install`

Here is a project brief you might plausibly be handed in your first job.

> A company operates 400 shipping containers fitted with refrigeration
> units. Each unit reports its internal temperature, compressor state, and
> door-open events over a cellular link. Compressors fail. When one fails
> mid-voyage the cargo is lost, and nobody notices until the container is
> opened three weeks later. Build us something that notices in an hour.

Notice what this brief does *not* say. It does not say "build us a
dashboard." A dashboard would show you 400 temperature traces, and a human
would have to stare at them. It does not say "build us a threshold alarm"
either — a naive "alert if temperature > 5 °C" fires constantly, because
temperature legitimately rises every time a door opens, and it fires *too
late* on a failing compressor, because by the time the setpoint is missed the
cargo has already been warm for hours.

What the brief actually asks for is a system that knows what each container
*should* be doing, compares that against what it *is* doing, and raises its
hand when the two disagree in a way that physics cannot explain. That is a
digital twin. The definition follows from the requirement rather than the
other way round.

And here is the uncomfortable part: there is no library for this. You cannot
`pip install digital-twin`. What you can install are the *pieces* — an MQTT
client, a time-series database, a numerical integrator, a container runtime —
and the engineering work is in how you assemble, deploy, version, test, and
operate them. Surveys of the open-source landscape find a genuinely useful
set of frameworks, but they differ sharply in which pieces they cover and in
what they assume about your system, so choosing one is itself an
architectural decision rather than a default [@gil_survey_2024]. This is
also, empirically, the part where projects fail: reference models for digital
twins have been proposed repeatedly precisely because there is a persistent
gap between the tidy concept and what people actually implement
[@pfeiffer_towards_2025].

Three properties of the refrigeration brief will recur throughout this
tutorial, and they are what make digital twins a distinct engineering
problem rather than "web development with sensors":

**The interesting logic is a model, not a rule.** Your alerting decision
comes from a physical model of a refrigerated box, which has parameters
(insulation quality, compressor capacity) that differ per container and drift
with age. Models have to be calibrated, revalidated, and versioned like code
— but they are not code, and your ordinary tooling does not know that.

**Time is part of the specification.** A twin that tells you about a
compressor failure four hours late is not a slightly worse twin; for this
brief it is a useless one. Physical and digital sides run on different clocks
with communication delays between them, and that discrepancy is a first-class
design concern rather than an implementation detail
[@frasheri_addressing_2023].

**The consequences are physical.** If your service crashes, a container
warms. If your service is compromised, someone can warm it deliberately.
Digital twins broaden the attack surface of the system they are attached to,
because they add a data path from the outside world into something that used
to be isolated [@de_azambuja_digital_2024].

Everything below is in service of those three properties.

## 2. What you are actually building

### 2.1 Six services, one system

The most useful working definition of a digital twin, for someone about to
write code, is functional rather than philosophical: a digital twin is a
software support system attached to a physical system, providing some
combination of visualisation, monitoring, predictive maintenance, fault
diagnosis, decision support, and reconfiguration [@gomes_digital_2025].

That list is worth internalising, because it doubles as a checklist of what
you might be asked to build, in roughly increasing order of difficulty:

| Service | What it does | Typical output |
|---|---|---|
| Visualisation | Shows current and historical state | A dashboard, a 3D scene |
| Monitoring | Continuously checks properties hold | A boolean per property |
| Predictive maintenance | Estimates time-to-failure | Hours remaining |
| Fault diagnosis | Identifies *which* thing broke | A fault label |
| Decision support | Simulates alternatives before you act | A ranked list of actions |
| Reconfiguration | Changes the physical system automatically | A command |

Two things about this table matter more than they look.

First, **the services share infrastructure but are separate software**. All
six need sensor data, most need a model, and several need history — but a
visualisation service and a reconfiguration service have wildly different
latency, reliability, and safety requirements. Bundling them into one process
is the single most common architectural mistake in student digital twin
projects, and it is the reason the microservice style shows up again and
again in digital twin architectures [@bordeleau_devops_nodate].

Second, **the physical system must work without the twin**. The refrigeration
units kept cargo cold before you arrived and must keep doing so if your
service is down. This is the line between a digital twin and a control
system: a control system is *in* the loop and its failure is the plant's
failure; a twin *supports* the loop and its failure should degrade the
system's intelligence, not its function [@gomes_digital_2025]. When you get
to Section 8 and start reasoning about failure modes, this distinction is the
thing that tells you which failures are catastrophic and which are merely
annoying.

### 2.2 The shadow test

You will constantly need to answer the question "is this thing I built
actually a digital twin?" There is a fast test, and it is about the direction
of automatic data flow. Three levels are usually distinguished
[@kritzinger_digital_2018]:

- **Digital model** — no automatic data flow at all. A CAD file. Someone
  updates it by hand when reality changes.
- **Digital shadow** — automatic flow *from* physical *to* digital. Your
  dashboard updates itself. Nothing flows back automatically.
- **Digital twin** — automatic flow in *both* directions. The digital side
  computes something and that computation changes the physical side without a
  human in between.

The word doing all the work is **automatic**. A dashboard that a human reads,
after which the human phones a technician who changes a setpoint, is a
digital shadow with a human-shaped gap in the return path. That is not a
criticism — a great many valuable industrial systems are shadows, and the
refrigeration brief above is arguably satisfiable by a very good shadow. But
be honest in your architecture documents about which one you built, because
the return path is where all the hard requirements live. Closing the loop
means your software can now break things.

A useful reframing: once the loop is closed, a digital twin *is* a
self-adaptive system. It monitors the physical system, updates an internal
model from observations, decides, and acts — the classic monitor-analyse-plan-
execute-over-knowledge shape, with the model playing the part of the shared
knowledge [@kamburjan_declarative_2024]. Everything the self-adaptive systems
literature knows about stability, oscillation, and conflicting adaptation
loops becomes your problem the moment you close the loop.

### 2.3 Why this is hard as *software*

Let us be precise about what makes a digital twin harder to engineer than a
CRUD application of comparable size. Four things:

1. **Heterogeneous artefacts in one deliverable.** A digital twin bundles
   source code, simulation models (often binary), calibration parameters,
   configuration, and datasets. These have different lifecycles and different
   correctness criteria. Treating them uniformly as "files in a repo" loses
   information; treating them separately fragments your build. A productive
   response is to name the categories explicitly and manage each as a
   first-class, reusable asset — for example splitting a twin's contents into
   *data*, *model*, *function*, and *tool* assets that can be declared once and
   reused across many twins [@talasila_composable_2025].
2. **Development crosses disciplines.** The thermal model is written by
   someone who is not a software engineer; the deployment is written by
   someone who does not know thermodynamics. Digital twin projects for built
   assets routinely involve software engineers alongside domain specialists in
   energy, air quality, and security, and the engineering process has to let
   those groups work independently without stepping on each other
   [@aissat_devops_2025].
3. **You cannot fully test against production.** Your test environment does
   not contain a shipping container. What it contains, if you are doing this
   well, is a *simulated* shipping container — which means part of your test
   infrastructure is itself a model, with its own fidelity question.
4. **Requirements are stated in physical units.** "Detect within one hour"
   and "estimate temperature within 0.5 °C" are the acceptance criteria. Your
   test suite has to assert on numbers with units, and those assertions are
   statistical, not exact.

Hold on to point 3 in particular. It is the reason Section 7 exists.

## 3. A reference architecture you can build from

There is no shortage of reference architectures for digital twins. Systematic
work on architecting them catalogues a large number of proposals and finds
that, beneath the varied vocabulary, they mostly agree on a layered
decomposition [@ferko_architecting_2022], and domain-specific versions add
their own emphases — a predictive-maintenance reference architecture, for
instance, foregrounds the data pipeline and model-serving stages that a
generic diagram glosses over [@van_dinter_reference_2023]. The problem for a
student is that these diagrams are drawn at a level of abstraction where every
box is defensible and none is buildable.

So here is a deliberately concrete version: seven components, each named after
the thing you would actually run. Wherever a real technology makes the layer
easier to picture, it is named. The point is not that you must use these
technologies — it is that you should be able to point at each layer of your
own design and say what plays that role.

```
   +---------------------------------------------------------+
   |  6. Presentation      dashboards, APIs, 3D/AR views      |
   +---------------------------------------------------------+
   |  5. Services          monitoring, diagnosis, what-if,    |
   |                       reconfiguration                    |
   +---------------------------------------------------------+
   |  4. Models            simulation + estimation, behind a  |
   |                       uniform interface                  |
   +---------------------------------------------------------+
   |  3. Storage           time series, model registry,       |
   |                       run artefacts                      |
   +---------------------------------------------------------+
   |  2. Ingestion         broker, schema, buffering, replay  |
   +---------------------------------------------------------+
   |  1. PT interface      sensors, actuators, gateway        |
   +---------------------------------------------------------+
        0. Physical twin

   cross-cutting: orchestration, identity, observability
```

### 3.1 Layer 1: the physical-twin interface

This is the layer everyone underestimates. It owns three responsibilities
that are easy to conflate and painful to separate later:

- **Acquisition** — reading a sensor, at some rate, with some accuracy.
- **Timestamping** — deciding *when* a reading happened, and in whose clock.
- **Actuation** — turning a decision into a physical change, and reporting
  whether it took effect.

Timestamping deserves a paragraph on its own. If your gateway stamps readings
on arrival rather than at the sensor, you have silently folded network delay
into your data, and every downstream model will attribute that delay to
physics. Stamp at the source if you possibly can, carry both stamps if you
can afford the bytes, and treat clock synchronisation as a requirement rather
than an assumption. The gap between when something happened physically and
when the digital side knows about it is a well-studied problem in its own
right, and pretending it is zero is a design decision you should make
consciously, not by accident [@frasheri_addressing_2023]. What you can
usefully sense, at what cost in energy and bandwidth, is likewise a design
space rather than a given [@gomes_sensing_2024].

Actuation deserves one too: **a command is not a state change**. "Set
compressor to 60 %" is a request that may be rejected, delayed, clamped, or
silently ignored by a local safety interlock. Your twin must model the
requested value and the achieved value as different variables, or it will
confidently simulate a world that never happened.

### 3.2 Layer 2: ingestion

Between the physical twin and everything else sits a message broker. Almost
every digital twin architecture has one, and mapping studies of digital twin
middleware find publish/subscribe messaging to be the near-universal
integration mechanism [@almeida_middleware_2023].

For most projects the choice is MQTT, and the reasons are worth knowing
rather than absorbing as folklore. Comparisons of the candidate IoT protocols
— MQTT, CoAP, AMQP, HTTP — line them up along message size, delivery
guarantees, and transport assumptions, and MQTT's combination of tiny
headers, a broker-mediated topic model, and three quality-of-service levels
fits telemetry from constrained devices particularly well
[@naik_choice_2017]. The broader survey literature on MQTT deployments adds
the practical dimension: there are many broker implementations with quite
different feature sets and performance envelopes, so "we use MQTT" does not
by itself pin down behaviour [@mishra_use_2020].

Three ingestion decisions you must make explicitly:

1. **Quality of service.** MQTT's QoS 0 is fire-and-forget; QoS 1 is at-least-
   once; QoS 2 is exactly-once and expensive. Telemetry that feeds a
   statistical model can usually tolerate QoS 0. A command that opens a valve
   cannot.
2. **Buffering and replay.** When the cellular link on a ship drops for six
   hours, does the gateway buffer and replay, or is that data gone? If it
   replays, your consumers must handle out-of-order and duplicate messages.
3. **Schema.** Publish `{"t": 1712.4}` today and you will spend a year
   guessing what `t` meant. Publish a versioned payload with units.

Do not take broker performance on faith, either. Benchmarks built
specifically for publish/subscribe under realistic domain workloads show
substantial differences between popular brokers once you vary message rates,
topic counts, and QoS [@badolato_psmark_2026], and independent functional and
performance evaluations of open-source pub/sub systems reach the same
conclusion from a different angle [@lazidis_publish-subscribe_2022]. If your
twin's latency budget is tight, measure with your own workload.

### 3.3 Layer 3: storage

A digital twin needs at least three distinct stores, and conflating them is a
classic error:

- **A time-series store** for telemetry. Optimised for append-heavy writes and
  range queries. InfluxDB, TimescaleDB, or similar.
- **A model/asset registry** for models, their parameters, and their
  provenance. This is the store that answers "which version of the thermal
  model produced this alert, and what data was it calibrated on?"
- **An artefact store** for run outputs: simulation traces, reports, plots.

The middle one is the one beginners skip, and skipping it is what makes a
two-year-old digital twin unmaintainable. When a regulator asks why the
system recommended a shutdown on 3 March, the answer must be reconstructible.
Proposals for "models-meet-data" repositories in construction engineering
exist precisely because the pairing of a model version with the data version
it was fitted to is the unit that needs storing, not either one alone
[@zech_proposal_2024].

### 3.4 Layer 4: models

Here is the layer that distinguishes a digital twin from every other
distributed system you have built. Two observations shape its design.

**Observation one: you will have more than one model.** Even our small
example has two — a simulator that predicts forward in time, and an estimator
that fuses prediction with measurement. Real systems add data-driven models
for anomaly detection, reduced-order models for speed, and high-fidelity
models for offline analysis. Comprehensive reviews of the modelling side of
digital twins survey this zoo at length: physics-based, data-driven, and
hybrid approaches each carry different data appetites and different validity
conditions [@thelen_comprehensive_2022].

**Observation two: therefore you need a uniform interface.** If each model
exposes a bespoke API, your services become a tangle of adapters. The standard
answer in the cyber-physical systems world is the Functional Mock-up Interface
(FMI), which packages a model as a Functional Mock-up Unit with a common
contract: set inputs and parameters, step time forward, read outputs. Case
studies of digital twin construction routinely use FMI for the co-simulation
side while providing an FMI-*like* Python interface for models that are more
convenient to write natively [@oakes_case_2024], and tooling has grown up
around making FMI-based twins quick to prototype in Python
[@friedrich_cofmpy_2025]. Frameworks that manage a twin's behavioural models
tend to standardise on the same contract for exactly this reason
[@gil_toward_2024].

You do not need FMI for a first project. You *do* need to write your models
behind an interface with that shape:

```python
class Model:
    def set_parameters(self, **params) -> None: ...
    def set_input(self, name: str, value: float) -> None: ...
    def step(self, dt: float) -> None: ...
    def get_output(self, name: str) -> float: ...
```

Four methods. If every model in your system obeys them, you can swap a
physics model for a neural surrogate without touching a single service.

### 3.5 Layer 5: services

Each of the six digital twin services from Section 2.1 becomes one or more
processes here. Some are simple (monitoring: subscribe, compare, publish a
verdict). Some are not: a what-if service must spin up a *copy* of the model,
initialise it to current state, run it faster than real time across several
candidate actions, and rank the outcomes — which means it needs compute
on demand and must never block the real-time path [@frasheri_advanced_2024].

The design pressure this creates is toward service orientation: independent
deployment, independent scaling, explicit interfaces. It is the same pressure
that produced microservices in web systems, and digital twin engineering
frameworks have converged on the same answer, combining a microservice
decomposition with a DevOps toolchain so that separate specialist teams can
develop and deploy the pieces independently [@bordeleau_devops_nodate]. The
services view is strong enough that some authors organise the whole
engineering process around it, describing a twin top-down by the services it
offers rather than bottom-up by the data it connects [@oakes_towards_2024].

### 3.6 Layer 6: presentation

Dashboards, REST APIs, 3D scenes, augmented-reality overlays. Treat
visualisation as a *service consuming the same interfaces as everything else*,
not as a privileged layer with direct database access — otherwise your
dashboard becomes a second, undocumented implementation of your business
logic. Visualisation is a rich topic on its own, spanning plain time-series
plots through to 3D and AR views, and each fidelity level buys a different
kind of insight for a different engineering cost [@bohlbro_visualisation_2024].

### 3.7 The cross-cutting concern: orchestration and lifecycle

Finally, something has to start these components in the right order, restart
them when they crash, inject their configuration, and update them. This is
not a seventh layer so much as a dimension running through all of them, and
Sections 5 through 7 are about it. Its importance grows superlinearly with
the number of twins: one twin can be babysat by a human; four hundred
refrigerated containers cannot, which is why declarative approaches to
digital twin lifecycle management — describe the desired state, let the
platform reconcile — become the only tractable option at fleet scale
[@kamburjan_declarative_2024].

## 4. Worked example 1: a thermal box twin in 150 lines

Time to build one. Our physical twin is deliberately humble: an insulated
plastic box with a resistive heating element, a small fan to keep the air
mixed, and one temperature sensor, all driven by a microcontroller. This is
not a toy chosen for convenience — a laboratory incubator of exactly this
shape, built for tempeh fermentation, has been used as the running case study
for a full digital twin research programme, and the reason is that it is
small enough to understand completely while still exhibiting every hard
problem: calibration, drift, delayed actuation, and unmodelled disturbances
[@gomes_digital_2025].

We build it in five steps. Each step produces something you can run.

### 4.1 Step 1: characterise the physical twin

Before writing any model, write down the interface. Ours:

| Signal | Direction | Type | Rate | Notes |
|---|---|---|---|---|
| `temperature` | PT -> DT | float, °C | 1 Hz | ±0.15 °C noise (datasheet) |
| `heater_on` | PT -> DT | bool | on change | *achieved* state |
| `heater_cmd` | DT -> PT | bool | on demand | *requested* state |
| `lid_open` | PT -> DT | bool | on change | reed switch |

Note the deliberate split between `heater_cmd` and `heater_on`, exactly as
argued in Section 3.1. Note also the sensor noise figure: it comes from the
datasheet, and it will end up in our alarm threshold. **Physical constants
from datasheets are inputs to your software design.** Get used to that.

### 4.2 Step 2: write the smallest model that could work

We need a model that predicts temperature forward in time. Start from an
energy balance rather than from a library. The box holds some amount of
thermal mass; the heater adds energy; energy leaks to the room through the
insulation at a rate proportional to the temperature difference. In symbols,
with `T` the inside temperature, `T_a` the ambient temperature, `P` the
heater power in watts, and `u` the heater state (0 or 1):

```
C * dT/dt  =  P * u  -  (T - T_a) / R
```

Two parameters carry all the physics: `C`, the heat capacity in joules per
kelvin (how much energy it takes to warm the box by one degree), and `R`, the
thermal resistance in kelvin per watt (how well it is insulated). This is a
*lumped-parameter* model: it pretends the entire box is at one uniform
temperature. That is false — the air near the heater is warmer — and being
explicit that it is false is what lets you predict where the model will
break.

Discretise with a forward Euler step of size `dt`:

```
T[k+1] = T[k] + (dt / C) * (P * u[k] - (T[k] - T_a) / R)
```

In code, behind the four-method interface from Section 3.4:

```python
class ThermalBox:
    """Lumped-parameter thermal model of a heated box."""

    def __init__(self, R=0.75, C=800.0, P=40.0, T0=21.0, T_amb=21.0):
        self.R, self.C, self.P = R, C, P      # parameters
        self.T = T0                            # state
        self.T_amb, self.u = T_amb, 0          # inputs

    def set_parameters(self, **params):
        for k, v in params.items():
            setattr(self, k, v)

    def set_input(self, name, value):
        setattr(self, {"heater": "u", "ambient": "T_amb"}[name], value)

    def step(self, dt):
        dTdt = (self.P * self.u - (self.T - self.T_amb) / self.R) / self.C
        self.T += dt * dTdt

    def get_output(self, name):
        return {"temperature": self.T}[name]
```

Twenty lines. Before going further, sanity-check it by hand — this habit will
save you hours. With the heater on forever, `dT/dt` reaches zero when
`P * u = (T - T_a) / R`, so the steady-state rise above ambient is `P * R`.
With `P = 40 W` and `R = 0.75 K/W` that is 30 °C, i.e. the box settles at
51 °C in a 21 °C room. Plausible. The time constant is `tau = R * C =
0.75 * 800 = 600 s`, so it should get roughly two-thirds of the way there in
ten minutes. Also plausible. If your hand-check gives 4000 °C, you have a
units bug, and you have found it in thirty seconds rather than after a day of
debugging a service mesh.

### 4.3 Step 3: calibrate against real data

The default parameters above are guesses. Calibration replaces them with
numbers fitted to *your* box. Run one experiment — a step response — and fit.

The experiment: start at room temperature, lid closed, turn the heater on,
log temperature at 1 Hz for 40 minutes, turn it off, log for another 40. That
second half matters: the cooling curve identifies `R * C` independently of
the heater power, which catches the common failure where `P` is not really 40 W.

The fit is ordinary least squares over two parameters:

```python
import numpy as np
from scipy.optimize import least_squares

def simulate(params, t, u, T0, T_amb, P=40.0):
    R, C = params
    T, out = T0, np.empty_like(t)
    for k in range(len(t)):
        out[k] = T
        dt = t[k + 1] - t[k] if k + 1 < len(t) else 0.0
        T += dt * (P * u[k] - (T - T_amb) / R) / C
    return out

def residuals(params, t, u, T_meas, T_amb):
    return simulate(params, t, u, T_meas[0], T_amb) - T_meas

fit = least_squares(residuals, x0=[0.75, 800.0],
                    bounds=([0.05, 50.0], [10.0, 20000.0]),
                    args=(t, u, T_meas, T_amb))
R_hat, C_hat = fit.x
rmse = np.sqrt(np.mean(fit.fun ** 2))
print(f"R = {R_hat:.3f} K/W, C = {C_hat:.0f} J/K, RMSE = {rmse:.2f} C")
```

On a run of this experiment you might get:

```
R = 0.742 K/W, C = 812 J/K, RMSE = 0.21 C
```

Now stop and read that RMSE, because it is the most important number in the
whole project. **0.21 °C is your model's floor.** No service built on this
model can reliably detect a temperature anomaly smaller than about 0.2 °C,
because the model itself cannot tell that apart from its own error. If the
refrigeration brief had demanded 0.05 °C accuracy, the correct engineering
response would be to go back to the model — add a second thermal mass, model
the sensor's own lag — not to build services on a foundation that cannot
support them.

Three warnings that cost people real time:

1. **Bound your parameters.** Unbounded fits happily return `C = -3` and a
   beautiful RMSE, because a physically absurd parameter set can still match
   40 minutes of data.
2. **Hold out data.** Fit on the heating curve, *evaluate* on the cooling
   curve. A model that fits the data it was trained on is not evidence of
   anything.
3. **Record the validity envelope.** These parameters were identified between
   21 °C and 51 °C with the lid closed. Outside that range they are
   extrapolation. Calibration is only valid under the conditions it was
   performed in, and forgetting this is a standard way to invalidate a
   simulation without noticing [@gomes_digital_2025].

Write the envelope into the parameter file, not into a comment in a notebook
you will lose:

```yaml
# models/thermal_box/params-v3.yaml
model: thermal_box
version: 3
fitted_on: 2026-03-14
dataset: s3://twin-data/box-01/step-response-2026-03-14.parquet
parameters: {R: 0.742, C: 812.0, P: 40.0}
validity: {T_min_c: 21.0, T_max_c: 51.0, lid: closed}
holdout_rmse_c: 0.24
```

That file is an artefact of your build, it has a version, and Section 7 will
put it through CI.

### 4.4 Step 4: close the loop with an estimator

You now have a model that predicts. A prediction alone drifts: integrate for
an hour with a slightly wrong `R` and the predicted temperature wanders away
from reality. The fix is to fuse prediction with measurement, and for a
one-dimensional linear system the standard tool is a scalar Kalman filter.
Strip away the matrix notation and it is four lines of arithmetic:

```python
class ScalarKF:
    """Fuse a model prediction with a noisy measurement."""

    def __init__(self, model, q=1e-4, r=0.0225, p0=1.0):
        self.model = model      # anything with .step()/.get_output()
        self.q = q              # process noise: how wrong the model is
        self.r = r              # measurement noise: sensor variance (0.15^2)
        self.P = p0             # state uncertainty

    def predict(self, dt):
        self.model.step(dt)
        self.P += self.q * dt
        return self.model.get_output("temperature")

    def update(self, z):
        T = self.model.get_output("temperature")
        innovation = z - T                       # the residual
        K = self.P / (self.P + self.r)           # Kalman gain, in [0, 1]
        self.model.T = T + K * innovation
        self.P = (1 - K) * self.P
        return innovation
```

The intuition is entirely contained in `K`. When the model is trusted
(`self.P` small relative to `r`), `K` approaches 0 and measurements are
largely ignored. When the model is distrusted, `K` approaches 1 and the
filter simply believes the sensor. `q` and `r` are how you express that trust:
`r` you get from the datasheet (0.15 °C noise gives variance 0.0225), and `q`
you tune, starting from your calibration RMSE.

And now the synchronisation loop — the actual heart of the digital twin,
which turns out to be about fifteen lines:

```python
def run(broker, model, kf, dt=1.0):
    for msg in broker.subscribe("box-01/telemetry"):
        model.set_input("heater", msg["heater_on"])
        predicted = kf.predict(dt)
        innovation = kf.update(msg["temperature"])
        broker.publish("box-01/twin/state", {
            "t": msg["t"],
            "measured_c": msg["temperature"],
            "predicted_c": predicted,
            "estimated_c": model.get_output("temperature"),
            "residual_c": innovation,
            "model_version": 3,
        })
```

Read the published payload carefully. It carries *four* numbers where a naive
implementation would carry one: what the sensor said, what the model expected,
what the fused best estimate is, and how far apart the first two were. Every
service downstream is built on that fourth number. Publishing the model
version alongside them is not bureaucracy either — it is what makes an alert
from six months ago interpretable.

### 4.5 Step 5: turn the residual into a decision

Here is where the shadow becomes a twin. The residual is a *physics-aware
error signal*: it is near zero whenever reality behaves the way the model
says it should, and it grows whenever something happens that the model does
not know about. Opening the lid is exactly such an event — it violates the
assumptions the model was built under, so the filter can no longer track the
system, and the discrepancy between estimate and measurement opens up
immediately and then closes again once the lid is shut. This effect has been
demonstrated on the real incubator, with the lid opened and closed at logged
times and the residual tracking those events [@gomes_digital_2025].

A workable detector, with the threshold derived rather than guessed:

```python
THRESH_C = 0.5      # ~3x the 0.15 C sensor noise, above the 0.21 C model RMSE
HOLD_S   = 30       # sustained, to reject transients

def monitor(stream, thresh=THRESH_C, hold=HOLD_S):
    breach_started = None
    for s in stream:                       # box-01/twin/state
        if abs(s["residual_c"]) > thresh:
            breach_started = breach_started or s["t"]
            if s["t"] - breach_started >= hold:
                yield {"alarm": "unmodelled_disturbance",
                       "since": breach_started,
                       "residual_c": s["residual_c"]}
        else:
            breach_started = None
```

Two design choices are doing real work here. The threshold is above both the
sensor noise floor *and* the model's RMSE, because an alarm that fires on
model error is worse than useless. The hold time trades detection latency
against false alarms: 30 seconds means you will never detect anything faster
than 30 seconds, and you should check that against the requirement before
shipping.

Then close the loop:

```python
for alarm in monitor(stream):
    if alarm["residual_c"] < 0:            # colder than modelled: heat escaping
        broker.publish("box-01/cmd", {"heater_cmd": True, "reason": alarm})
```

That single `publish` is the difference between a digital shadow and a digital
twin (Section 2.2), and it is also the line that makes your software capable
of breaking a physical thing. Everything in Sections 7 and 8 exists because of
that line.

### 4.6 What you have and have not built

You have a working digital twin: it ingests, models, estimates, monitors,
diagnoses a specific fault class, and reconfigures. On a laptop, in one
process, for one box.

You have not built: anything that survives a restart (the filter state is in
memory), anything that handles two boxes, anything anyone else can deploy,
anything that will still be correct in a year, or anything you can test
without the hardware on your desk. Those five gaps are the rest of this
tutorial.

## 5. Deciding where the code runs

Your laptop process has to become several processes on several machines. The
question "which machine?" has a bad default answer ("the cloud, obviously")
and a good method. Here is the method.

### 5.1 Do the latency arithmetic before you draw the diagram

Suppose our box is part of a laboratory rig where an over-temperature
condition must trigger a heater cut-off **within 200 ms**. That number is the
requirement; everything else is derived from it. Write down every hop and its
cost:

| Hop | Edge deployment | Cloud deployment (p50) | Cloud (p99) |
|---|---|---|---|
| Sensor sample + ADC | 5 ms | 5 ms | 5 ms |
| Gateway serialise + publish | 3 ms | 3 ms | 3 ms |
| Network to broker | 2 ms | 45 ms | 180 ms |
| Broker fan-out | 2 ms | 2 ms | 2 ms |
| Model step + filter update | 1 ms | 1 ms | 1 ms |
| Decision logic | 1 ms | 1 ms | 1 ms |
| Network back | 2 ms | 45 ms | 180 ms |
| Actuator response | 20 ms | 20 ms | 20 ms |
| **Total** | **36 ms** | **122 ms** | **392 ms** |

The cloud deployment meets the requirement on a median day and misses it by
almost a factor of two on a bad one. If you only ever measure p50 you will
ship this and it will fail in the field, intermittently, in a way that is
almost impossible to reproduce. **Budget against your tail, not your median.**

This arithmetic is not a formality; it is the whole design. It tells you that
the *monitoring and reconfiguration* services must run at the edge, and it
says nothing at all about where the what-if service, the dashboard, or the
model-calibration job should run — those have no comparable deadline and
belong wherever compute is cheapest. The general form of this question —
which component goes on which node in a fog or edge topology, subject to
latency, capacity, and cost constraints — is a well-studied optimisation
problem with a large literature of its own [@salaht_overview_2020], and when
the components are microservices with dependencies between them, placement
choices measurably change end-to-end response time [@raza_empowering_2024].
Locality matters especially when the infrastructure spans administrative
domains, where a placement that ignores which components must talk to each
other frequently can be arbitrarily bad [@faticanti_locality-aware_2023].

### 5.2 Four placement heuristics

You will not run an optimiser for a course project. These four rules capture
most of the value:

1. **Latency-critical closed loops go at the edge.** Anything on the path from
   measurement to actuation, with a deadline, runs on hardware you can touch.
   Digital twin architectures designed explicitly for the edge push exactly
   this subset down — a local twin instance that keeps working when the
   uplink does not [@picone_flexible_2023].
2. **Bandwidth reduction goes at the edge.** A 1 kHz vibration sensor produces
   far more data than you want to ship. Compute features locally, ship the
   features. Structural-health-monitoring deployments across civil
   infrastructure are organised precisely as an edge/fog/cloud hierarchy for
   this reason, with each tier reducing what the next has to carry
   [@martin_facilitating_2022]. Increasingly, the inference itself moves
   on-device rather than just the feature extraction
   [@barbone_-device_2026].
3. **Anything needing history, fleet-wide context, or serious compute goes to
   the cloud.** Calibration across 400 containers, what-if studies, model
   retraining. These are batch jobs with elastic demand, which is exactly what
   cloud pricing is good at.
4. **Anything stateful and hard to move stays put — until you have a reason.**
   Moving a running component between tiers is possible, and there is a rich
   line of work on making software migrate seamlessly across the cloud-to-edge
   continuum [@aguzzi_cloud_2020], including orchestrators that treat the
   continuum as one deployment target [@ullah_micado-edge_2021] and runtime
   adaptation of containerised workloads as conditions change
   [@robles-enciso_adapting_2024]. Some visions go further and ask why the
   boundary between clusters should be visible to the application at all
   [@iorio_computing_2023]. Treat all of that as a capability to grow into,
   not a starting point.

The framing that ties these together is the **digital twin continuum**: rather
than "edge twin" versus "cloud twin", a twin's components are distributed
across a spectrum of resources and may relocate as conditions change
[@barbone_digital_2024]. Adaptation can even be driven by resource
availability itself — degrade gracefully when CPU, bandwidth, or battery gets
scarce, rather than failing [@akiki_resources_2025].

### 5.3 Why containers, specifically

Whatever you place where, you need the placed thing to be a self-contained
unit. Three properties make containers the right primitive for digital twins
in particular — not merely the fashionable one.

**Reproducibility.** A digital twin's numerical output depends on library
versions in ways that are genuinely hard to see: a different BLAS, a different
`scipy`, and your calibration lands on slightly different parameters.
Containers pin the entire software environment alongside the code, which is
why they became standard practice in computational science for exactly this
reason [@moreau_containers_2023]. For a system whose output is a *number* that
someone will act on, this is not a convenience.

**Heterogeneity.** Your thermal model may be Python, your co-simulation
orchestrator Java, your dashboard JavaScript, and a vendor's FMU may ship as a
Linux `.so` compiled in 2019. Containers let each carry its own world.

**Uniform deployment target.** The same image runs on a developer laptop, an
edge gateway, and a cluster node. Digital twin platforms lean on this: they
package twins as containers and let Kubernetes handle scheduling, restart, and
scaling [@talasila_realising_2024], and the broader cloud-native ecosystem —
microservice decomposition plus container orchestration — is the assumed
substrate in current surveys [@deng_cloud-native_2024].

A caution before you reach for Kubernetes: it is a large, opinionated system,
and for a single twin on a single gateway it is overhead you cannot justify.
Start with Docker Compose. Move to Kubernetes when you have a real reason —
multiple nodes, autoscaling, rolling updates across a fleet — and note that
even then the day-to-day workflow can stay close to plain Docker if you set it
up thoughtfully [@deslauriers_everyday_2022]. Systematic reviews of
microservice deployment and communication patterns are a good map of the
choices you are signing up for [@karabey_aksakalli_deployment_2021].

## 6. Worked example 2: packaging, configuring, and reusing it

Back to the box. We now turn one laptop process into a deployable system, and
then — the part that actually pays off — we deploy a *second* box without
writing new code.

### 6.1 Split the monolith along service lines

The single `run()` loop from Section 4.4 becomes four containers:

| Container | Responsibility | Placement |
|---|---|---|
| `broker` | MQTT message bus | Edge |
| `twin-core` | Model + estimator, publishes state | Edge |
| `monitor` | Residual -> alarms -> commands | Edge |
| `historian` | Subscribe -> time-series database | Cloud |

The split follows Section 5.1: `twin-core` and `monitor` are on the deadline
path, `historian` is not. It also follows Section 2.1: these are different
services with different failure consequences. If `historian` dies you lose
history; if `monitor` dies you lose safety. Those should not share a process,
and separating them is precisely what lets different people own and redeploy
them independently [@aissat_juno-ops_2024].

A minimal `docker-compose.yml`:

```yaml
services:
  broker:
    image: eclipse-mosquitto:2
    ports: ["1883:1883"]
    volumes: ["./mosquitto.conf:/mosquitto/config/mosquitto.conf:ro"]

  twin-core:
    build: ./twin-core
    depends_on: [broker]
    environment:
      TWIN_ID: box-01
      BROKER_URL: mqtt://broker:1883
      MODEL_PARAMS: /assets/params/box-01-v3.yaml
    volumes: ["./assets:/assets:ro"]
    restart: unless-stopped

  monitor:
    build: ./monitor
    depends_on: [broker]
    environment:
      TWIN_ID: box-01
      BROKER_URL: mqtt://broker:1883
      THRESHOLD_C: "0.5"
      HOLD_S: "30"
    restart: unless-stopped

  historian:
    image: influxdb:2
    volumes: ["tsdb:/var/lib/influxdb2"]

volumes: { tsdb: {} }
```

Three things in that file are worth arguing about, because students routinely
get them wrong.

**The parameters are mounted, not baked.** `box-01-v3.yaml` is data, and it
changes on a different schedule from the code. Baking it into the image means
recalibration requires a rebuild, which means recalibration will not happen.

**The thresholds are environment variables.** They are *policy*, and policy is
tuned in the field by people who will not open a pull request.

**`restart: unless-stopped` is a safety decision, not a convenience.** Read it
alongside Section 2.1: the box must keep heating when the twin is down, so the
twin restarting is fine. If the twin were in the control loop, silent restart
would be exactly the wrong behaviour.

This idea — that a reference architecture becomes useful only when it is
expressed as concrete deployment descriptors rather than a diagram — is worth
taking seriously; work on interoperable architectures for digital-twin-aided
manufacturing delivers them precisely as Dockerfiles, Compose files, and Helm
charts, so that "the architecture" is a thing you can `apply`
[@marosi_interoperable_2022].

### 6.2 The configuration is the twin

Now add the second box. It is the same physical design but a different
physical object, so it has the same code, the same models, and different
parameters. A bad architecture requires a code change. A good one requires a
new configuration file:

```yaml
# twins/box-02.yaml
twin_id: box-02
physical_twin:
  broker_topic_prefix: box-02
  sensors:  [{name: temperature, unit: celsius, noise_std_c: 0.15}]
  actuators: [{name: heater, type: boolean, power_w: 40.0}]
models:
  - {name: thermal, implementation: assets/models/thermal_box:1.4.0,
     parameters: assets/params/box-02-v1.yaml}
services:
  - {name: estimator, implementation: assets/functions/scalar_kf:2.1.0,
     inputs: [thermal, temperature], config: {q: 1.0e-4, r: 0.0225}}
  - {name: monitor, implementation: assets/functions/residual_monitor:1.0.0,
     config: {threshold_c: 0.6, hold_s: 30}}
```

Look at what this file is doing. It names *reusable assets* by version and
composes them into a twin. That is the central idea behind treating a digital
twin as a composition over four categories of reusable asset — data, models,
functions, and tools — declared once in a platform and reused across many
twins by many users [@talasila_composable_2025]. The payoff arrives on twin
number three, not twin number two: your marginal cost per twin becomes one
YAML file and a calibration run.

Note also that `box-02` uses a threshold of 0.6 °C rather than 0.5 °C, because
it sits near a doorway and its residual is noisier. Per-instance policy in a
per-instance config file: exactly the kind of variability that motivates
treating a family of twins as a *product line*, with explicit variation points
rather than copy-pasted repositories [@pfeiffer_towards_2023].

### 6.3 Platforms: what you get by not building this yourself

Everything above — asset storage, configuration, compute provision,
communication, monitoring, lifecycle — is generic. It is not specific to
heated boxes, and building it yourself for every project is waste. That
observation is the entire argument for **digital twin as a service (DTaaS)**
platforms: a shared platform manages assets, composes twins from them, runs
them, and offers the running twins as services to other users
[@talasila_composable_2025]. Reference models for DTaaS in Industry 4.0 set
out the same idea in architectural terms [@aheleroff_digital_2021], and a
recent survey maps the architecture, design requirements, and — usefully for
engineers — the performance metrics by which such platforms should be judged
[@duran_toward_2026].

The concrete open-source landscape is varied, and it repays a look before you
start typing:

- Frameworks built on an IoT digital-twin substrate plus a streaming layer,
  composing twins from smaller twins [@robles_opentwins_2023].
- Platforms built directly on Kubernetes, using serverless primitives so that
  a twin's compute is provisioned only when events arrive
  [@wermann_ktwin_2024].
- Platforms built on Asset Administration Shell middleware, where the
  motivating requirement is that several stakeholders share one twin instance
  as a single source of truth [@zech_digital-twins-as--service_2024].
- Domain platforms that wrap an established workflow — vibration-based
  structural health monitoring, say — around a general twin platform
  [@talasila_digital_2026].
- Large-scale scientific twin infrastructures, where containerised workloads
  and orchestration are what make elastic demand tractable at all
  [@noauthor_intertwin_nodate].
- Comprehensive reference architectures for digital twin software platforms,
  which are useful as checklists even when you do not adopt the platform
  [@tao_maketwin_2024].

Comparative surveys of open-source frameworks, evaluated by building the same
case study on each, are the fastest way to see what the differences actually
cost you [@gil_survey_2024]. And the same containerisation argument recurs in
domains far from ours: cloud-based twins for cognitive robotics use containers
and Kubernetes chiefly so that students and researchers can skip the setup and
get to the content [@niedzwiecki_cloud-based_2024].

**Choosing.** For a course project, build it yourself with Compose — you learn
more, and the assignment is the learning. For a research prototype that must
outlive one student, use a platform. For an industrial deployment, the
decisive questions are usually not technical: who operates it, what happens
when the vendor changes their API, and can you get your models out.

## 7. DevOps for digital twins

Everything so far produced *artefacts*. This section is about the machinery
that turns a change to one of those artefacts into a running change in the
field, safely, repeatedly, without a human remembering the steps. That
machinery is DevOps, and applying it to digital twins is an active enough
research area to have produced named approaches — TwinOps, which fuses
model-based engineering with a DevOps pipeline for cyber-physical systems
[@hugues_twinops_2020; @hugues_twinops_2022], model-based DevOps as a general
programme for Industry 4.0 software [@combemale_model-based_2023], and
JuNo-OPS, a DevOps framework built specifically around microservices for
digital twins of built assets [@aissat_devops_2025].

### 7.1 Why the ordinary pipeline is not enough

A standard web pipeline is: lint, unit test, integration test, build image,
deploy, smoke test. Run that on our thermal box twin and here is what it
fails to catch.

- **A parameter regression.** Someone recalibrates and commits
  `R = 0.42`. Every unit test passes — `R` is a float, the model still steps —
  and the twin now systematically over-predicts temperature. Nothing in the
  ordinary pipeline is capable of noticing.
- **A model-fidelity regression.** Someone replaces the Euler integrator with
  a larger step size for speed. All tests pass. The twin is now less accurate,
  by an amount nobody measured.
- **A physical-interaction regression.** Someone flips a sign in the actuation
  path. Tests pass; the twin now cools when it should heat. You find out from
  the hardware.

The common thread: **the correctness criteria are numerical and physical, and
they live in data, not in code.** Interview studies with practitioners doing
continuous integration for cyber-physical systems find exactly this friction —
the hardware dependency and the difficulty of testing without it are the
recurring obstacles to CI adoption in the field [@zampetti_continuous_2023].
And the organisational half is real too: longitudinal industrial studies of
DevOps adoption catalogue the ways it goes wrong, and most of them are about
process and expectations rather than tooling [@caprarelli_fallacies_2020].

### 7.2 A test pyramid for digital twins

Here is a pyramid adapted to our problem. Each level is cheap enough to run at
a different frequency, and each catches things the level below cannot.

**Level 1 — Unit tests (every commit, seconds).** Ordinary software tests on
ordinary code paths: message parsing, config loading, unit conversion.

```python
def test_step_conserves_equilibrium():
    m = ThermalBox(R=0.742, C=812.0, T0=21.0, T_amb=21.0)
    m.set_input("heater", 0)
    m.step(1.0)
    assert m.get_output("temperature") == pytest.approx(21.0)

def test_steady_state_rise_matches_analytic():
    m = ThermalBox(R=0.742, C=812.0, P=40.0, T0=21.0, T_amb=21.0)
    m.set_input("heater", 1)
    for _ in range(20_000):
        m.step(1.0)
    assert m.get_output("temperature") == pytest.approx(21.0 + 40 * 0.742, abs=0.1)
```

Note the second test: it checks the model against an *analytically derived*
property (steady-state rise is `P * R`), not against a recorded output. Tests
that compare against yesterday's output only tell you the code changed.

**Level 2 — Model-versus-data tests (every commit, seconds).** Check the model
against a stored, versioned reference dataset, and assert on a *physical*
tolerance:

```python
def test_model_tracks_reference_run():
    ds = load("tests/data/box-01-step-response-2026-03-14.parquet")
    m = ThermalBox(**load_params("assets/params/box-01-v3.yaml"))
    pred = simulate_over(ds.t, ds.heater, m, T0=ds.T[0], T_amb=ds.T_amb)
    rmse = float(np.sqrt(np.mean((pred - ds.T) ** 2)))
    assert rmse < 0.30, f"model fidelity regressed: RMSE {rmse:.2f} C"
```

This is the test that catches the parameter regression and the integrator
regression, and it is the one that has no analogue in a web pipeline. Two
notes on making it useful rather than annoying. Set the tolerance from your
*validation* RMSE with headroom (we measured 0.24 °C on holdout, so 0.30 °C
allows normal variation while catching a real regression). And record the
achieved RMSE as a build artefact, so you can plot fidelity over the project's
lifetime — slow degradation across many small commits is otherwise invisible.

**Level 3 — Simulated-physical-twin tests (every commit, minutes).** Replace
the hardware with a *simulator* of the hardware, wire the real twin
containers to it, and test the whole system end to end. This is the level
that resolves the "we cannot test without hardware" complaint, and it is the
reason the idea of a digital twin *prototype* — a simulated stand-in for the
physical system, used inside the CI pipeline for embedded software — is a
serious research topic and not a workaround [@barbie_toward_2024].

Crucially, the simulator must be a *different* model from the twin's, or your
test is circular: a twin tested against its own model will pass whatever it
does. Use a higher-fidelity model, add noise and delay, and inject faults:

```python
def test_lid_open_raises_alarm_within_60s(compose_stack, fake_pt):
    fake_pt.run_to_steady_state(setpoint_c=35.0)
    fake_pt.inject(fault="lid_open", at_s=600)
    alarms = compose_stack.collect("box-01/alarms", until_s=700)
    assert any(a["alarm"] == "unmodelled_disturbance" for a in alarms)
    assert min(a["t"] for a in alarms) - 600 < 60
```

That test asserts on the *requirement* from Section 4.5 — detection latency —
rather than on an implementation detail. Deliberately injecting faults into a
simulation to see how the system responds is standard practice in the twin
world [@frasheri_advanced_2024], and here it becomes a regression test.

**Level 4 — Hardware-in-the-loop (nightly or pre-release, hours).** One real
box, on a bench, wired to a runner. Slow, flaky, and irreplaceable: it is the
only level that tests your driver code, your timing assumptions, and your
actuation path against reality. Run it on a schedule, not on every push, and
treat a failure as blocking for release. Digital twin engineering processes
draw the same distinction, with verification strategies combining
demonstration, testing, modelling, and simulation rather than relying on any
one [@fitzgerald_digital_2024]. Where the twin is being used to verify the
system itself, this integrates naturally with model-based systems engineering
practice [@honcak_mbse_2024].

### 7.3 The pipeline

```yaml
# .github/workflows/twin.yml (abbreviated)
on: [push, pull_request]
jobs:
  fast:
    steps:
      - run: ruff check . && mypy src/
      - run: pytest tests/unit tests/model        # levels 1-2
      - run: python -m tools.validate_assets assets/   # schemas + validity envelopes
  system:
    needs: fast
    steps:
      - run: docker compose -f compose.test.yml up --abort-on-container-exit
      - run: pytest tests/system                  # level 3
  publish:
    needs: system
    if: github.ref == 'refs/heads/main'
    steps:
      - run: docker buildx build --platform linux/amd64,linux/arm64 --push .
      - run: python -m tools.bump_manifest --env staging
```

Two details are digital-twin-specific and easy to miss.

`validate_assets` is a step you will not find in a web pipeline. It checks
that every parameter file declares a validity envelope, that the envelope is
non-degenerate, and that every model referenced by a twin config exists at the
pinned version. Cheap, and it catches a whole class of "works on my twin"
failure.

`--platform linux/amd64,linux/arm64` is there because your edge gateway is
almost certainly ARM and your CI runner is almost certainly x86. Cross-building
is not optional when the deployment target differs from the build host.

### 7.4 Deployment as a reconciled desired state

Do not push updates to gateways. Declare what should be running and let
something reconcile reality to that declaration. Concretely: a git repository
holds a manifest of which twin runs which image at which version with which
parameter file; an agent on each site watches the repository and converges to
it. This is GitOps, and it has been applied to digital twin lifecycle
management specifically, with the argument that Git as the operational centre
gives you version history, branching, and integration with CI/CD for free
[@beaumont_towards_2025].

Why this matters more for twins than for web services:

- **Sites are unreliable and numerous.** Push-based deployment fails when the
  target is offline; pull-based deployment just converges later.
- **You must be able to answer "what is running on box-207?"** With a
  reconciled manifest, the answer is a `git show`.
- **Rollback is a revert.** Given the safety argument in Section 4.5, you want
  rollback to be boring.

At larger scale this generalises: the twin's lifecycle — creation,
configuration, evolution, retirement — is itself something to describe
declaratively rather than script imperatively [@kamburjan_declarative_2024].
And it is worth noticing the reflexive twist that some authors have pursued:
the software development process is itself a system that can be twinned, with
CI/CD telemetry feeding a model of the process [@kimmel_digital_2025].

### 7.5 Versioning things that are not code

Last piece. Your repository holds code, models, parameters, and datasets, and
`git` handles exactly one of those well. A workable convention:

| Artefact | Where it lives | Versioned by | Changes when |
|---|---|---|---|
| Service code | Git | Commit SHA -> image tag | Behaviour changes |
| Model implementation | Git or asset registry | Semantic version | Equations change |
| Parameters | Asset store, pointer in Git | Monotonic integer | Recalibration |
| Datasets | Object store, hash in Git | Content hash | Never (append only) |
| Twin config | Git | Commit SHA | Composition changes |

The rule that makes this work: **an alert must be reconstructible from the
identifiers it carries.** Our `twin/state` payload in Section 4.4 carried
`model_version` for this reason. Make it carry the parameter version and the
image tag too, and any alert in your history can be replayed exactly. This is
also what makes evolution manageable rather than terrifying: a twin is a
long-lived thing whose sensors, models, and purposes change over its life, and
the ability to say precisely what it was at a past moment is the precondition
for changing it safely [@alskaif_evolution_2025].

## 8. Operating it: latency, drift, and attackers

Your twin is deployed. It will now spend years being wrong in three distinct
ways, and telling them apart is the operational skill this section teaches.

### 8.1 Instrument the twin, not just the machine

CPU and memory graphs tell you nothing about whether a digital twin is doing
its job. Four twin-specific metrics do:

1. **Synchronisation lag** — wall-clock age of the newest telemetry the twin
   has processed. Rising lag means the digital and physical sides have drifted
   apart in *time*, which invalidates every decision the twin makes; this
   discrepancy is a first-class concern, not a monitoring nicety
   [@frasheri_addressing_2023].
2. **Residual statistics** — rolling mean and standard deviation of the
   innovation from Section 4.4. Both matter, and they mean different things
   (below).
3. **Coverage** — fraction of expected messages actually received, per sensor.
   Distinguishes "everything is fine" from "the sensor died and the last value
   is being repeated".
4. **Envelope violations** — how often the twin is operating outside the
   validity range recorded during calibration (Section 4.3). A twin running
   outside its envelope is extrapolating, and should say so.

Publish all four. A dashboard showing a beautiful temperature trace from a
sensor that stopped reporting an hour ago is worse than no dashboard.

### 8.2 Reading the residual: three faults, one signal

The residual is the twin's most informative output, and its *shape*
discriminates between fault classes:

| Residual behaviour | Most likely cause | Response |
|---|---|---|
| Step change, then recovery | Unmodelled disturbance (lid opened) | Alarm; expect recovery |
| Step change, no recovery | Sensor fault or actuator fault | Alarm; cross-check coverage |
| Slow monotonic growth over weeks | **Model drift** | Recalibrate |
| Variance grows, mean stays zero | Sensor degradation / added noise | Maintenance |
| Sudden zero variance | Sensor stuck at a constant | Alarm — this is *not* good news |

The third row is the one that quietly ruins projects. Insulation degrades,
heating elements age, a fan collects dust: the physical system slowly stops
matching the parameters you fitted in March. Nothing breaks. The residual mean
creeps from 0.0 to 0.15 to 0.3 °C, and one day your 0.5 °C threshold starts
producing false alarms, and someone raises the threshold to 1.0 °C to make
them stop, and now you cannot detect the fault you built the system for.

The defence is to monitor the residual *mean* on a long timescale and trigger
recalibration when it exceeds a fraction of your alarm threshold — say 0.2 °C
for a 0.5 °C threshold. Automate the recalibration job; do not rely on
somebody remembering. This is the operational face of a deeper question about
when a twin's knowledge still counts as equivalent to the system it
represents, which is what actually licenses acting on the twin's conclusions
[@zhang_knowledge_2024]. The mature version of this idea gives the twin
autonomy over its own upkeep — detecting deviation and adapting its models
without a human trigger [@rivera_forging_2021] — and, where the physical
system is itself reconfigurable, letting the twin drive that reconfiguration
too [@esterle_autonomous_2024].

### 8.3 The attack surface you just created

Return to Section 4.5, and the `publish` that closed the loop. You have built
a network path from a message broker to a heating element. Anyone who can
publish to `box-01/cmd` can turn on the heater.

This is the general shape of the problem: connecting a physical asset to a
digital counterpart with bidirectional flow expands the attack surface of a
system that may previously have been isolated [@de_azambuja_digital_2024].
Comprehensive threat surveys map attacks across the whole stack — the physical
layer, communication, data storage, and the visualisation and access layer
where users and downstream processes connect [@alcaraz_digital_2022], and
recent reviews add the emerging attack classes and open problems
[@jaber_comprehensive_2025; @alhamam_comprehensive_2025].

Rather than repeat a survey, here are five concrete measures for the system we
built, in the order I would implement them:

1. **Authenticate publishers, and separate topics by direction.** Telemetry
   topics are write-only for the gateway and read-only for everyone else.
   Command topics accept writes from exactly one service identity. This is
   not sophisticated; it is the single highest-value hour you will spend.
2. **Distinguish read access from write access per user and per twin.** In a
   multi-stakeholder deployment, some users need to see a twin and others need
   to change it, and enforcing that separation requires real isolation between
   parts of the twin rather than a single credential granting everything
   [@kulik_security_2024].
3. **Treat the network as hostile.** The zero trust model — no implicit trust
   from network location, per-request authorisation, least privilege — maps
   directly onto a twin whose components straddle a factory floor, a gateway,
   and a cloud region [@rose_zero_2020], and there is practical guidance on
   planning a migration towards it rather than attempting it in one step
   [@rose_planning_2022].
4. **Harden the containers.** Non-root users, read-only root filesystems,
   dropped capabilities, pinned base images, scanned dependencies. Container
   isolation is weaker than virtual-machine isolation, and the specific
   weaknesses are well catalogued [@sultan_container_2019].
5. **Bound the actuation.** Rate-limit commands, clamp them to a safe range,
   and keep a hardware interlock that your software cannot override. If a
   compromised twin can only request a 5 % duty cycle change once a minute,
   the blast radius is small.

One last observation that ties this section to the last. Attacks do not only
manifest as breaches; they manifest as *degraded twin metrics*. Experimental
work measuring the impact of cyber-attacks on digital twin metrics shows
familiar symptoms — outdated or inconsistent state, undelivered updates, a
twin that stops receiving data — arising from things like flooding a topic or
downgrading message quality of service, and those look, from a dashboard, very
much like a network problem [@picone_assessing_2026]. Which means your
monitoring from Section 8.1 is also part of your security posture, and an
unexplained rise in synchronisation lag deserves a security hypothesis
alongside the networking one.

## 9. Anti-patterns

A short catalogue of failures I would bet money on seeing in a first project.

**The dashboard that thinks it is a twin.** Telemetry goes into a time-series
database and out to a set of charts, with no model anywhere. It is a digital
shadow (Section 2.2) and it will not detect anything a threshold alarm could
not. Diagnostic question: *where is the prediction that reality is being
compared against?*

**The uncalibrated model.** Textbook parameters, never fitted to the actual
object. It produces a residual that reflects parameter error rather than
physical events. Diagnostic question: *what is the holdout RMSE, and when was
it last measured?*

**The single container.** Ingestion, model, monitoring, and dashboard in one
process, so that a dashboard bug takes down safety monitoring and the
dashboard cannot be redeployed without restarting the estimator. Diagnostic
question: *what happens to actuation when the visualisation crashes?*

**Parameters in the image.** Recalibration requires a rebuild and a redeploy,
so recalibration stops happening, and drift (Section 8.2) goes unaddressed
until the twin is quietly useless.

**The circular test.** The simulated physical twin in CI is the same model the
twin uses, so the system passes every test and fails on hardware. Diagnostic
question: *if the model's equations were wrong, which test would fail?*

**The tuned-away alarm.** Thresholds raised repeatedly to silence false
alarms, until the detector cannot fire. Fix the drift instead, and record
threshold changes in version control so the ratchet is visible.

**No provenance on the alert.** An alert says "anomaly at 14:32" and nothing
else. Six months later, nobody can say which model version or parameter set
produced it, so nobody can determine whether it was right.

## 10. Exercises

Difficulty is marked *easy* (a few minutes, pen and paper), *medium* (an hour
or two at a keyboard), and *hard* (a weekend, or a small group project). **Hints are given for
all exercises; full worked solutions are given for 10.1, 10.4 and 10.7 only** —
the rest are deliberately left open, because the judgement they exercise is
the point.

### Objective 1 — decompose an architecture

**10.1 [easy] Classify the system.** A hospital logs every patient movement
between wards into a database. A simulation of patient flow runs each night on
the previous day's data, and its output is emailed to the bed manager, who
sometimes changes staffing. Is this a digital model, a digital shadow, or a
digital twin? What is the smallest change that would move it up one level?

> **Solution.** It is a *digital shadow*: data flows automatically from the
> physical system into a model, but the return path runs through a human
> reading an email, so nothing flows back automatically. Two things
> disqualify it as a twin: the human in the return path, and the fact that
> the model runs nightly on yesterday's data rather than being synchronised
> with the physical system. The smallest change that makes it a twin is to
> let the simulation's output *automatically* adjust something physical — for
> example, automatically publishing a staffing roster or reserving beds in the
> admissions system — with the human moved from decision-maker to override.
> Note the important consequence: making that change converts a system whose
> worst failure mode is "an email is wrong" into one whose worst failure mode
> is "a ward is understaffed", which is a much larger engineering
> responsibility. Answering "should we?" is as legitimate as answering "how?"

**10.2 [medium] Map the layers.** Take the refrigerated-container brief from
Section 1 and fill in the seven-layer architecture from Section 3 with a
concrete technology or component for each layer. For every layer, write one
sentence stating what breaks if that layer fails.
*Hint: two layers have no acceptable failure mode given the brief. Identify
them and say what redundancy you would add.*

**10.3 [medium] Interface hygiene.** In our thermal box, `heater_cmd` and
`heater_on` are separate signals. Construct a concrete scenario in which
merging them into one signal causes the twin to produce a *wrong* alarm — not
merely a missing one. Then write the test from Section 7.2 that would catch it.
*Hint: think about a local safety interlock that refuses a command, and what
the model then integrates.*

### Objective 2 — implement and quantify a synchronisation loop

**10.4 [easy] Predict the equilibrium.** A box has `R = 1.2 K/W` and
`C = 1500 J/K`, with a 25 W heater, in a 20 °C room. With the heater on
permanently: what is the steady-state temperature, and how long until the box
is within 1 °C of it?

> **Solution.** Steady state is where heat in equals heat out:
> `P = (T - T_a) / R`, so `T = T_a + P * R = 20 + 25 * 1.2 = 50 °C`. The
> time constant is `tau = R * C = 1.2 * 1500 = 1800 s` (30 minutes). The
> response is exponential, `T(t) = 50 - 30 * exp(-t / tau)`, so being within
> 1 °C means `30 * exp(-t / 1800) = 1`, i.e.
> `t = 1800 * ln(30) ~= 1800 * 3.40 ~= 6120 s`, about 102 minutes. Two lessons
> worth extracting: the settling time is roughly `3.4 * tau`, not `tau`; and
> if your step-response calibration experiment (Section 4.3) only runs for
> 40 minutes on *this* box, it never reaches steady state, so `R` and `C`
> become correlated in the fit and both are poorly identified. Design the
> experiment to last at least three or four time constants.

**10.5 [medium] Implement and validate.** Implement `ThermalBox` and `ScalarKF`
from Section 4. Generate synthetic "measurements" from a *second*, different
model — for example a two-mass model with the sensor thermally lagged behind
the air — plus Gaussian noise, and run the estimator against it. Report the
RMSE of the estimate and of the raw prediction.
*Hint: the estimate should beat the prediction. If it does not, your `q`/`r`
ratio is wrong. Try varying `q` over three orders of magnitude and plot RMSE
against it; the curve should be U-shaped, and the minimum tells you what your
model error actually is.*

**10.6 [hard] Detect a fault you did not design for.** Extend your simulated
box with a *heater degradation* fault: `P` decays linearly from 40 W to 30 W
over six simulated hours. Does the detector from Section 4.5 fire? Should it?
Design a second detector that distinguishes heater degradation from insulation
degradation, and justify why your two residual signatures are distinguishable.
*Hint: a failing heater and a leakier box both make the box colder than
predicted. What differs is the dependence on the duty cycle — one shows up
only when the heater is on. Consider correlating the residual with `u`.*

### Objective 3 — place and package

**10.7 [medium] Redo the latency budget.** A robot cell has a twin that must stop
the robot within 100 ms of detecting a collision risk. Measured hops: sensor
2 ms, gateway 2 ms, LAN 1 ms, broker 3 ms, model step 8 ms, decision 1 ms,
actuator 30 ms. The cloud round trip adds 35 ms each way at p50 and 140 ms at
p99. Can this run in the cloud? At the edge? What is the single most valuable
optimisation?

> **Solution.** Edge: `2 + 2 + 1 + 3 + 8 + 1 + 1 + 30 = 48 ms`, comfortably
> inside 100 ms with roughly 2× headroom. Cloud at p50:
> replace the two 1 ms network hops with 35 ms each, giving `48 - 2 + 70 =
> 116 ms` — already over budget on a *good* day. At p99 it is `48 - 2 + 280 =
> 326 ms`, more than three times the budget. Cloud is not viable at any
> percentile, so the placement decision is forced.
> The most valuable optimisation is the actuator's 30 ms, which is 62 % of the
> edge budget — and it is the one most engineers never look at, because it is
> not software. Failing that, the 8 ms model step is the largest software
> term, and a reduced-order model or a cached linearisation would attack it.
> The general lesson: do the arithmetic *before* optimising, or you will spend
> a week shaving the 3 ms broker hop.

**10.8 [medium] Compose a second twin.** Take your Section 6 Compose stack and add
a second box without editing any Python. If you cannot, identify precisely
which hard-coded value blocked you and refactor it into configuration. Then
answer: what is your marginal cost — in files touched — for box number ten?
*Hint: the usual culprits are topic names, parameter file paths, and the
threshold. A good target is one new YAML file and zero code changes.*

### Objective 4 — build the pipeline

**10.9 [medium] Write the fidelity test.** Add the Level 2 test from Section 7.2
to your project: a versioned reference dataset, a tolerance derived from your
holdout RMSE, and a CI job. Then deliberately break it — change `R` by 20 % —
and confirm it fails. Finally, make the job record the achieved RMSE as an
artefact and explain, in two sentences, what you would do with a plot of that
number over 200 commits.
*Hint: the interesting failure is not the 20 % break, which any test catches.
It is twenty commits that each degrade fidelity by 1 %.*

**10.10 [hard] Build the simulated physical twin.** Replace your hardware with
a container that simulates the box and speaks the same MQTT topics, and get
the system test from Section 7.2 passing in CI. Then answer the hard question:
what does this test *not* prove about the real system? Write the list. It
should have at least four entries.
*Hint: consider timing, sensor placement, actuator dynamics, and every
assumption shared between your simulator and your twin's model.*

### Objective 5 — diagnose an operational twin

**10.11 [medium] Triage.** For each observation, give the most likely cause and
the next diagnostic step you would take:
(a) residual mean drifts from 0.0 to 0.4 °C over eight weeks, variance
unchanged; (b) residual jumps to 3 °C and stays there, coverage drops to
zero for one sensor; (c) residual variance triples overnight, mean unchanged;
(d) synchronisation lag climbs steadily from 1 s to 40 s over an hour while
all residuals stay normal; (e) residual variance drops to exactly zero.
*Hint: exactly one of these is more likely to be an attack than a fault, and
one is good news that looks like bad news — or the reverse.*

**10.12 [hard] Threat-model your own twin.** Draw the data flow of your
Section 6 stack and mark every point where an attacker could inject, modify,
suppress, or observe a message. For the three highest-impact points, write the
control you would implement and estimate its cost in engineer-days. Then
answer: which of your controls would have *prevented* the incident, and which
would only have *detected* it?
*Hint: revisit Section 8.3's ordering. Prevention and detection are different
budgets, and a good answer includes at least one control of each kind.*

## 11. Where to go next

Three directions, depending on what you found most interesting.

**If the modelling drew you in**, the next step is co-simulation: coupling
several models with different solvers and time steps into one twin, which is
where the Functional Mock-up Interface earns its complexity, and where
questions of numerical stability at the coupling points become real.

**If the systems side drew you in**, look at scale. Everything here concerned
one twin, or a fleet of identical ones. Twins of *different* systems that must
cooperate — a bridge twin talking to a traffic twin — raise federation and
interoperability problems that are genuinely open [@marah_re-engineering_2025],
and the shift in perspective from individual twins to whole *twinning systems*
is an active reframing of the field [@lugaresi_digital_2025].

**If the assurance side drew you in**, follow the thread from Section 7.2 into
using twins as testing infrastructure — anomaly detection for cyber-physical
systems built on twin-produced expectations is a well-developed line of work
[@xu_traversing_2023] — and into what it takes to trust a twin's conclusions
enough to act on them automatically [@zhang_knowledge_2024].

Whichever you pick, the discipline from Section 4.3 travels with you: write
down what your model is valid for, measure how wrong it is, and never let a
service make a decision finer than that number.

## 12. References

- **aguzzi_cloud_2020** -- From Cloud to Edge: Seamless Software Migration at the Era of the Web of Things (2020).
- **aheleroff_digital_2021** -- Digital Twin as a Service (DTaaS) in Industry 4.0: An Architecture Reference Model (2021).
- **aissat_devops_2025** -- A devops framework for the systematic engineering and evolution of digital twins for built assets (2025).
- **aissat_juno-ops_2024** -- JuNo-OPS: A DevOps Framework for the Engineering of Digital Twins for Built Assets (2024).
- **akiki_resources_2025** -- Resources don't grow on trees: A framework for resource-driven adaptation (2025).
- **alcaraz_digital_2022** -- Digital Twin: A Comprehensive Survey of Security Threats (2022).
- **alhamam_comprehensive_2025** -- A Comprehensive Review on Cybersecurity of Digital Twins Issues, Challenges, and Future Research Directions (2025).
- **almeida_middleware_2023** -- Middleware for Digital Twins: A Systematic Mapping Study (2023).
- **alskaif_evolution_2025** -- Evolution at the Core of Digital Twin Engineering (2025).
- **badolato_psmark_2026** -- PSMark: a distributed IoT benchmark for publish/subscribe under domain-based workloads (2026).
- **barbie_toward_2024** -- Toward Reproducibility of Digital Twin Research: Exemplified with the PiCar-X (2024).
- **barbone_-device_2026** -- On-device AI and digital twins: A synergistic approach to intelligent cyber-physical systems (2026).
- **barbone_digital_2024** -- Digital Twin Continuum: a Key Enabler for Pervasive Cyber-Physical Environments (2024).
- **beaumont_towards_2025** -- Towards Automating the Life Cycle Management of Digital Twins (2025).
- **bohlbro_visualisation_2024** -- Visualisation in a Digital Twin Context (2024).
- **bordeleau_devops_nodate** -- A DevOps Approach for the Systematic Development and Evolution of Built Assets Digital Twins (n.d.).
- **caprarelli_fallacies_2020** -- Fallacies and Pitfalls on the Road to DevOps: A Longitudinal Industrial Study (2020).
- **combemale_model-based_2023** -- Model-Based DevOps: Foundations and Challenges (2023).
- **de_azambuja_digital_2024** -- Digital Twins in Industry 4.0 – Opportunities and challenges related to Cyber Security (2024).
- **deng_cloud-native_2024** -- Cloud-Native Computing: A Survey From the Perspective of Services (2024).
- **deslauriers_everyday_2022** -- Everyday orchestration with Docker on Kubernetes (2022).
- **duran_toward_2026** -- Toward Digital Twin-as-a-Service (DTaaS) Platforms: A Survey on Architecture, Design Requirements, and Performance Metrics (2026).
- **esterle_autonomous_2024** -- Autonomous Reconfiguration Enabled by Digital Twins (2024).
- **faticanti_locality-aware_2023** -- Locality-aware deployment of application microservices for multi-domain fog computing (2023).
- **ferko_architecting_2022** -- Architecting Digital Twins (2022).
- **fitzgerald_digital_2024** -- Digital Twin Engineering Processes (2024).
- **frasheri_addressing_2023** -- Addressing time discrepancy between digital and physical twins (2023).
- **frasheri_advanced_2024** -- Advanced Digital Twin Services (2024).
- **friedrich_cofmpy_2025** -- CoFMPy: A Python Framework for Rapid Prototyping of FMI-based Digital Twins (2025).
- **gil_survey_2024** -- Survey on open-source digital twin frameworks–A case study approach (2024).
- **gil_toward_2024** -- Toward a systematic reporting framework for Digital Twins: a cooperative robotics case study (2024).
- **gomes_digital_2025** -- Digital Twin Tutorial: The Incubator Case Study (2025).
- **gomes_sensing_2024** -- Sensing and Communication of Data from the Physical Twin (2024).
- **honcak_mbse_2024** -- An MBSE approach for Virtual Verification & Validation of Systems with Digital Twins (2024).
- **hugues_twinops_2020** -- TwinOps - DevOps meets model-based engineering and digital twins for the engineering of CPS (2020).
- **hugues_twinops_2022** -- Twinops: Digital twins meets devops (2022).
- **iorio_computing_2023** -- Computing Without Borders: The Way Towards Liquid Computing (2023).
- **jaber_comprehensive_2025** -- A Comprehensive State-of-the-Art Review for Digital Twin: Cybersecurity Perspectives and Open Challenges (2025).
- **kamburjan_declarative_2024** -- Declarative Lifecycle Management in Digital Twins (2024).
- **karabey_aksakalli_deployment_2021** -- Deployment and communication patterns in microservice architectures: A systematic literature review (2021).
- **kimmel_digital_2025** -- Digital Twins for Software Engineering Processes (2025).
- **kritzinger_digital_2018** -- Digital Twin in manufacturing: A categorical literature review and classification (2018).
- **kulik_security_2024** -- Security and Privacy-related Issues in a Digital Twin Context (2024).
- **lazidis_publish-subscribe_2022** -- Publish-Subscribe approaches for the IoT and the cloud: Functional and performance evaluation of open-source systems (2022).
- **lugaresi_digital_2025** -- From Digital Twins to Twinning Systems (2025).
- **marah_re-engineering_2025** -- (Re-)Engineering Digital Twins Towards Federation: Vision and Roadmap (2025).
- **marosi_interoperable_2022** -- Interoperable Data Analytics Reference Architectures Empowering Digital-Twin-Aided Manufacturing (2022).
- **martin_facilitating_2022** -- Facilitating the monitoring and management of structural health in civil infrastructures with an Edge/Fog/Cloud architecture (2022).
- **mishra_use_2020** -- The use of MQTT in M2M and IoT systems: A survey (2020).
- **moreau_containers_2023** -- Containers for computational reproducibility (2023).
- **naik_choice_2017** -- Choice of effective messaging protocols for IoT systems: MQTT, CoAP, AMQP and HTTP (2017).
- **niedzwiecki_cloud-based_2024** -- Cloud-based Digital Twin for Cognitive Robotics (2024).
- **noauthor_intertwin_nodate** -- interTwin: Advancing Scientific Digital Twins through AI, Federated Computing and Data (n.d.).
- **oakes_case_2024** -- Case Studies in Digital Twins (2024).
- **oakes_towards_2024** -- Towards Ontological Service-Driven Engineering of Digital Twins (2024).
- **pfeiffer_towards_2023** -- Towards a Product Line Architecture for Digital Twins (2023).
- **pfeiffer_towards_2025** -- Towards a Unifying Reference Model for Digital Twins of Cyber-Physical Systems (2025).
- **picone_assessing_2026** -- Assessing the Impact of Cybersecurity Attacks on Digital Twin Metrics: An Experimental Study (2026).
- **picone_flexible_2023** -- A Flexible and Modular Architecture for Edge Digital Twin: Implementation and Evaluation (2023).
- **raza_empowering_2024** -- Empowering Microservices: A Deep Dive into Intelligent Application Component Placement for Optimal Response Time (2024).
- **rivera_forging_2021** -- The forging of autonomic and cooperating digital twins (2021).
- **robles-enciso_adapting_2024** -- Adapting Containerized Workloads for the Continuum Computing (2024).
- **robles_opentwins_2023** -- OpenTwins: An open-source framework for the development of next-gen compositional digital twins (2023).
- **rose_planning_2022** -- Planning for a Zero Trust Architecture: A Planning Guide for Federal Administrators (2022).
- **rose_zero_2020** -- Zero Trust Architecture (2020).
- **salaht_overview_2020** -- An Overview of Service Placement Problem in Fog and Edge Computing (2020).
- **sultan_container_2019** -- Container security: Issues, challenges, and the road ahead (2019).
- **talasila_composable_2025** -- Composable digital twins on Digital Twin as a Service platform (2025).
- **talasila_digital_2026** -- A digital twin platform for structural health monitoring (2026).
- **talasila_realising_2024** -- Realising Digital Twins (2024).
- **tao_maketwin_2024** -- makeTwin: A reference architecture for digital twin software platform (2024).
- **thelen_comprehensive_2022** -- A comprehensive review of digital twin — part 1: modeling and twinning enabling technologies (2022).
- **ullah_micado-edge_2021** -- MiCADO-Edge: Towards an Application-level Orchestrator for the Cloud-to-Edge Computing Continuum (2021).
- **van_dinter_reference_2023** -- Reference architecture for digital twin-based predictive maintenance systems (2023).
- **wermann_ktwin_2024** -- KTWIN: A Serverless Kubernetes-based Digital Twin Platform (2024).
- **xu_traversing_2023** -- Traversing the Data Spectrum: Path to Dependable Cyber-Physical Systems through Digital Twins (2023).
- **zampetti_continuous_2023** -- Continuous Integration and Delivery Practices for Cyber-Physical Systems: An Interview-Based Study (2023).
- **zech_digital-twins-as--service_2024** -- Digital-Twins-as-a-Service in Construction Engineering (2024).
- **zech_proposal_2024** -- A Proposal for a Models-Meet-Data Repository for Digital Twins in Construction Engineering (2024).
- **zhang_knowledge_2024** -- Knowledge Equivalence in Digital Twins of Intelligent Systems (2024).
