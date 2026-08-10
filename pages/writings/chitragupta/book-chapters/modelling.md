---
hide:
  - navigation
  - toc
---

# Modelling Techniques for Digital Twins: One Plant Pot, Twelve Ways to Model It

**Disclaimer:** This book chapter has been generated using
[chitragupta](https://prasad.talasila.in/chitragupta).
Despite some potential for hallucination, the ideas communicated in this
book chapter are accurate. Please send your corrections and suggestions to
<prasad.talasila@gmail.com>


> **Summary.** Every digital twin has a model at its heart, but "model" is
> not one thing -- it is a dozen techniques with genuinely different
> assumptions, costs, and failure modes. This tutorial introduces those
> techniques one at a time and applies *every one of them to the same
> physical system*: a single potted plant, watered by a pump, monitored by
> a soil moisture probe. The hardware is real and open -- the INTO-CPS
> Association's `plant-controller` -- and it starts out as something that
> is *not yet a digital twin*. Each technique is a step toward closing that
> loop. Every numerical result quoted here was computed from the parameter
> set in Section 2 and can be reproduced with Python's standard library.

### Learning objectives

By the end of this chapter, you will be able to:

1. **Derive** a first-principles water-balance model of soil in a pot from
   a mass balance, and state the assumptions that make it valid.
2. **Convert** a continuous-time model into an executable one, choose a
   time step defensibly, and predict when a numerical solver will fail --
   both by diverging and by silently missing events.
3. **Fit** a black-box model to logged data by least squares, and
   **diagnose** a badly wrong model whose prediction errors look excellent.
4. **Select** an appropriate modelling technique given a stated purpose, a
   data budget, and a compute budget -- and justify the choice against at
   least one rejected alternative.
5. **Explain** how state estimation lets a twin report a quantity no sensor
   measures, and recognise when a quantity is *not* recoverable from the
   available data.

### Outline

1. [Why the Model Is the Hard Part](#1-why-the-model-is-the-hard-part)
2. [The Running Example: One Plant, One Pot](#2-the-running-example-one-plant-one-pot)
3. [Technique 1: First-Principles Models](#3-technique-1-first-principles-models)
4. [Technique 2: Discretisation and Numerical Solvers](#4-technique-2-discretisation-and-numerical-solvers)
5. [Technique 3: Distributed Models, and Technique 4: Surrogates](#5-technique-3-distributed-models-and-technique-4-surrogates)
6. [Technique 5: Black-Box System Identification](#6-technique-5-black-box-system-identification)
7. [Technique 6: Machine-Learning Models](#7-technique-6-machine-learning-models)
8. [Technique 7: Hybrid (Grey-Box) Models](#8-technique-7-hybrid-grey-box-models)
9. [Technique 8: Probabilistic Models](#9-technique-8-probabilistic-models)
10. [Technique 9: State Estimation and Virtual Sensors](#10-technique-9-state-estimation-and-virtual-sensors)
11. [Technique 10: Discrete-Event Models and Hybrid Automata](#11-technique-10-discrete-event-models-and-hybrid-automata)
12. [Technique 11: Co-Simulation](#12-technique-11-co-simulation)
13. [Technique 12: Degradation Models](#13-technique-12-degradation-models)
14. [Calibration, Validation, and Drift](#14-calibration-validation-and-drift)
15. [Choosing a Technique](#15-choosing-a-technique)
16. [Exercises](#16-exercises)
17. [Hints and Solutions](#17-hints-and-solutions)

## 1. Why the Model Is the Hard Part

A digital twin is usually introduced as three things: a physical asset, a
virtual counterpart, and a two-way flow of data between them. Sensor
readings flow in, and decisions computed on the virtual side flow back out
to change something physical. If data only travels one way -- asset to
computer, full stop -- what you have is a *digital shadow*, not a twin
[@kritzinger_digital_2018].

That definition is easy to state and it hides where the difficulty lives.
Getting sensor readings into a database is a solved problem. Getting
control signals back out is routine. What is *not* routine is the thing in
the middle: the model. The model turns a stream of numbers into an answer
to a question -- how wet is the soil where the roots actually are, will
this plant be short of water by Thursday, is my moisture probe lying to me.
Everything a digital twin is *for* happens inside the model, and that is
where the engineering risk sits.

Newcomers often assume "the model" means one specific thing, usually
whatever they met first: a mesh if they came from mechanical engineering, a
neural network if they came from computer science, differential equations
if they came from physics. In practice a twin of any real asset uses
several models of several kinds, coupled together, and choosing among them
is a genuine engineering decision with consequences
[@thelen_comprehensive_2022]. The choice is not settled by the physics. The
same pot can be modelled with soil-water equations, with a regression
fitted to logged probe readings, or with a combination, and picking one
trades off precision, development cost, and how well the result holds up in
conditions it has never seen [@abbiati_modelling_2024].

There is a second reason to take model selection seriously. A twin built by
hand, by experts, for one high-value asset is achievable today; making that
repeatable across many assets is much harder, and the modelling choices are
where the non-repeatable, artisanal effort concentrates
[@niederer_scaling_2021].

This tutorial fixes one physical system and applies twelve techniques to
it. The system is small enough to fit in your head and cheap enough to
build. Because the asset never changes, every difference you see between
sections is attributable to the modelling technique alone.

## 2. The Running Example: One Plant, One Pot

Our asset is a single potted plant on a shelf, watered automatically. The
hardware is the INTO-CPS Association's open `plant-controller`: a Raspberry
Pi reading Adafruit Seesaw capacitive soil moisture probes through a
TCA9548A I2C multiplexer, together with an SHT45 temperature and humidity
sensor and an AS7341 light-spectrum sensor, driving pumps through relays on
an Automation HAT. Sensors are read every 60 seconds and logged to InfluxDB.

The same pattern appears in the research literature. GreenhouseDT, an
exemplar built at the University of Oslo, pairs exactly this class of
hardware -- light, humidity and moisture sensors with a watering pump --
with a simulation model, an asset model, and a time-series database, and
uses it to study plant health monitoring and model-based control
[@kamburjan_greenhousedt_2024]. That paper positions the greenhouse
alongside a laboratory incubator as the two low-cost teaching exemplars for
digital twins, so the system we are about to model sits in good company.

### 2.1 The system is not yet a digital twin

Here is the most important fact about our starting point, and the reason
this example is worth a whole tutorial.

**The controller waters on a clock, not on moisture.** Its rules are purely
scheduled: Pump 1 fires at 10:15 for 15 seconds, Pump 2 at 14:30 for 6
seconds, Pump 3 at 10:15 for 10 seconds. The three moisture probes are read
every minute and faithfully logged -- and completely ignored when deciding
whether to water.

Data flows one way. In Kritzinger's terms this is a digital *shadow*
[@kritzinger_digital_2018]. It is a perfectly reasonable engineering
artefact, and it is not a digital twin.

So this tutorial has a destination. Each technique below is a step toward
closing that loop, and by Section 15 you will have the ingredients for a
controller that decides when to water based on a model of what is actually
happening in the soil.

### 2.2 The modelling contract

Before writing an equation, state what the model is *for*. This sounds like
bureaucracy and it is not. A model is not a description of reality; it is a
purpose-built approximation, and "is this model good?" is unanswerable
until you have said good *for what* [@gomes_foundational_2024].

> **Purpose.** Predict soil moisture in the root zone up to three days
> ahead, given a watering schedule, to within 0.02 vol/vol. Additionally,
> report root-zone moisture *now*, which no sensor measures directly.
>
> **Inputs.** Pump command $u \in \{0,1\}$; ambient conditions.
>
> **Outputs.** Surface moisture $\theta_s$ (what the probe reads); root-zone
> moisture $\theta_r$ (what the plant experiences).
>
> **Validity envelope.** Indoor conditions, 18--25 °C; one plant, one pot,
> no rain; soil between wilting point and saturation; probe calibrated
> within the last six months.

That last item is the one students skip and engineers regret. Every model
has conditions outside which it quietly stops being true. Writing them down
converts a silent failure into a checkable one.

### 2.3 Two layers, because the probe is in the wrong place

A capacitive probe pushed into a pot measures the soil immediately around
its blade -- the top few centimetres. The plant draws water through roots
distributed lower down. These are not the same soil, and after watering
they are not remotely the same moisture.

So we model the pot as **two layers**: a surface layer that receives the
pump water and loses water by evaporation, and a root layer that loses
water by transpiration. Water moves between them by percolation and
capillary exchange.

This single decision is what makes the pot interesting. The quantity we can
*measure* and the quantity we *care about* are different states of the same
system, and most of Section 10 exists to bridge that gap.

### 2.4 Nominal parameters

| Symbol | Meaning | Value |
|---|---|---|
| $V_s$ | Soil volume, surface layer | 200 mL |
| $V_r$ | Soil volume, root layer | 600 mL |
| $\theta_{fc}$ | Field capacity | 0.28 vol/vol |
| $\theta_{wp}$ | Wilting point | 0.10 vol/vol |
| $K$ | Layer exchange coefficient | 0.02 mL/s per unit $\theta$ |
| $k_d$ | Drainage rate above field capacity | 1/1800 per second |
| $E_s$ | Surface evaporation, unstressed | 10 mL/day |
| $E_r$ | Transpiration, unstressed | 12 mL/day |
| $q_p$ | Pump flow rate | 1.6 mL/s |

Two assumptions are worth flagging, because everything numerical depends on
them. The pump is taken to be a **peristaltic dosing pump at 1.6 mL/s**; a
submersible aquarium pump would deliver perhaps 28 mL/s and flood this pot
in seconds, so if you rebuild this, measure your pump. The layer split and
the exchange coefficient are representative of a fine potting mix rather
than measured from a specific pot.

### 2.5 What arithmetic tells you before any model

In steady state, water in must equal water out. The pot loses
$E_s + E_r = 22$ mL/day when the plant is unstressed. At 1.6 mL/s the pump
must therefore run **13.75 seconds per day**.

Now compare that against the controller's actual schedule:

| Pump | Duration | Delivers | Versus 22 mL/day demand |
|---|---|---|---|
| Pump 1 | 15 s | 24.0 mL/day | surplus, +2.0 |
| Pump 3 | 10 s | 16.0 mL/day | **deficit, -6.0** |
| Pump 2 | 6 s | 9.6 mL/day | **deficit, -12.4** |

You have just discovered, with nothing but arithmetic, that two of the
three shipped schedules under-water their plants. Hold on to the 13.75
seconds; we will check several models against it.

The layer capacities follow directly: the surface layer holds 56 mL at
field capacity and 20 mL at wilting point, the root layer 168 mL and 60 mL.
The pot holds 224 mL when full and 80 mL when the plant dies, so there are
**144 mL of plant-available water** -- about a week's supply, which matches
the intuition that a neglected houseplant survives roughly that long.

## 3. Technique 1: First-Principles Models

The first technique is the one physicists reach for: write down the
governing laws and solve them. Such models are called *first-principles*,
*physics-based*, *mechanistic*, or *white-box*, the last name signalling
that you can see and interpret everything inside
[@thelen_comprehensive_2022].

### 3.1 The lumped-layer assumption

Real soil water movement is a field problem: moisture varies continuously
with depth and position, and the honest description is a partial
differential equation. We are going to refuse to write it, and the refusal
is the most important modelling decision in this tutorial.

Instead we assume each layer has **one moisture value throughout**. This is
the *lumped* assumption, and it turns a PDE in space and time into two
ordinary differential equations in time alone.

Notice how the justification works. We are not claiming the surface layer
is genuinely uniform -- it is not. We are claiming that for a small pot,
over the timescales we care about, the error from treating it as uniform is
small enough given our stated purpose. The two-layer split exists precisely
because the *one*-layer version is not good enough: it would erase the
distinction between what the probe reads and what the roots get. Good
modelling assumptions look like this -- justified by a physical feature of
the problem, and traceable to the contract.

### 3.2 Deriving the equations

Start with the root layer alone, ignoring the surface. Water leaves by
transpiration at a rate that depends on how much is available -- a plant in
dry soil closes its stomata and transpires less:

$$\frac{dW_r}{dt} = -E_r \, f_r(W_r), \qquad
f_r(W_r) = \mathrm{clip}\!\left(\frac{W_r - W_{r,wp}}{W_{r,fc} - W_{r,wp}},\, 0,\, 1\right)$$

The *water stress factor* $f_r$ runs from 0 at wilting point to 1 at field
capacity. This is bookkeeping: rate of storage equals inflow minus outflow.

Now couple the layers. Water exchanges between them by capillary action at
a rate proportional to the moisture difference, and any surface water above
field capacity percolates downward under gravity:

$$
\begin{aligned}
\frac{dW_s}{dt} &= q_p u - K\!\left(\tfrac{W_s}{V_s} - \tfrac{W_r}{V_r}\right) - k_d \max(0, W_s - W_{s,fc}) - E_s f_s \\
\frac{dW_r}{dt} &= K\!\left(\tfrac{W_s}{V_s} - \tfrac{W_r}{V_r}\right) + k_d \max(0, W_s - W_{s,fc}) - k_d \max(0, W_r - W_{r,fc}) - E_r f_r
\end{aligned}
$$

Read each line as an accounting identity. The surface layer gains pump
water and loses to exchange, percolation, and evaporation. The root layer
gains what the surface passes down, loses what drains out the bottom of the
pot, and loses what the plant transpires. Nothing more sophisticated than
conservation of mass is happening, which is characteristic of
first-principles modelling: the equations are usually easy, and knowing
which terms to include is the skill.

In dynamical-systems notation this is $\dot{x} = f(x,u,p)$ with state
$x = [W_s, W_r]^T$, input $u = [u_{\text{pump}}]$, and parameters
$p = [K, k_d, E_s, E_r]^T$. The distinction between *state* (changes as the
system runs), *input* (imposed from outside), and *parameter* (fixed, or
slowly varying) matters in every later section.

### 3.3 What the model tells us before we simulate

Between waterings and below field capacity, both $\max(0,\cdot)$ terms
vanish and both stress factors are linear, so the system is **linear**:
$\dot{x} = Ax + b$ with

$$A = \begin{bmatrix} -K/V_s - a_s & K/V_r \\ K/V_s & -K/V_r - a_r \end{bmatrix} = \begin{bmatrix} -1.0322 \times 10^{-4} & 3.3333 \times 10^{-5} \\ 1.0000 \times 10^{-4} & -3.4619 \times 10^{-5} \end{bmatrix}$$

where $a_s = E_s/(W_{s,fc}-W_{s,wp})$ and similarly for $a_r$. That
linearity is a gift, and Sections 4, 6, and 10 all spend it.

The eigenvalues are $-1.7631 \times 10^{-6}$ and $-1.3607 \times 10^{-4}$
per second, giving time constants of **6.56 days** and **2.04 hours**. Both
are negative, so the pot is stable: left alone, it dries to equilibrium.

Both numbers are physically meaningful. The fast one is water
*redistributing* between the layers after a watering. The slow one is the
whole pot drying out. Together with the 15-second pump pulse, the system
spans three timescales covering a factor of about **37,800** -- and that
spread is about to become our problem.

### 3.4 Strengths and weaknesses

The strengths are considerable. The model extrapolates: it encodes mass
conservation, so it stays plausible in conditions never observed. Its
parameters are interpretable -- if the fitted $E_r$ doubles over a season,
that means the plant is transpiring twice as much, which is what growth
looks like. And it needs very little data: four parameters, estimable from
one drying experiment.

The weaknesses are equally real. Somebody has to *know* the physics, and
for the biology half of this system -- how growth responds to water,
nutrients, and light -- nobody knows it to the precision we know mass
conservation. Where the physics is understood, capturing it faithfully may
be too expensive to run live, the recurring fidelity-versus-cost tension in
digital twin modelling [@thelen_comprehensive_2022]. And the model is only
as good as its assumptions: move the pot outdoors and a model with no term
for rainfall will be confidently, silently wrong.

## 4. Technique 2: Discretisation and Numerical Solvers

The equations describe continuous time. Computers do not do continuous
time. Turning a differential equation into something a processor executes
is itself a modelling step, with its own approximations and its own
spectacular failure modes.

### 4.1 Euler's method

Approximate the derivative by a finite difference over a step $\Delta t$:

$$x_{k+1} = x_k + \Delta t \cdot f(x_k, u_k, p)$$

```python
def step_euler(Ws, Wr, u, dt=60.0):
    Vs, Vr, K, kd, qp = 200., 600., 0.02, 1/1800., 1.6
    Wsfc, Wswp, Wrfc, Wrwp = 56., 20., 168., 60.
    Es, Er = 10./86400, 12./86400
    fs = min(1., max(0., (Ws - Wswp) / (Wsfc - Wswp)))
    fr = min(1., max(0., (Wr - Wrwp) / (Wrfc - Wrwp)))
    exch = K * (Ws/Vs - Wr/Vr)
    perc = kd * max(0., Ws - Wsfc)
    dWs = qp*u - exch - perc - Es*fs
    dWr = exch + perc - kd*max(0., Wr - Wrfc) - Er*fr
    return Ws + dt*dWs, Wr + dt*dWr
```

That is a complete, working digital twin model. Run it in a loop and you
have a simulator.

### 4.2 Failure mode one: instability

Euler is stable only when $\Delta t < 2/|\lambda_{\max}|$. From Section 3.3,
$|\lambda_{\max}| = 1.3607 \times 10^{-4}$, so our limit is
$\Delta t < 14{,}698$ s, or **4.08 hours**.

Simulating ten days of free drying from field capacity, where the true
answer (high-accuracy reference) is $W_r = 83.627$ mL:

| Step size | Euler | RK4 |
|---|---|---|
| 60 s | 83.625 | 83.627 |
| 15 min | 83.598 | 83.627 |
| 1 h | 83.513 | 83.627 |
| 4.00 h | 83.135 | 83.627 |
| **4.44 h** | $-1.5 \times 10^{17}$ | 83.627 |
| 6.00 h | $-7.1 \times 10^{26}$ | $-5.2 \times 10^{37}$ |

The prediction is exact. At 4.00 hours Euler is fine; at 4.44 hours -- just
past the limit -- it returns $-1.5 \times 10^{17}$ mL of water. Nothing
errors out. The simulation runs to completion and returns a number, and the
number is garbage.

Fourth-order Runge--Kutta evaluates the derivative four times per step and
survives to 4.44 hours, failing only at 6. Four times the cost per step is
a bargain when it lets you take steps several times larger.

This matters because the *slow* dynamics tempt you into large steps. You
want to simulate a whole growing season; stepping once per day is very
tempting, and it produces numerical nonsense.

### 4.3 Failure mode two: missing the event entirely

The second failure has no counterpart in most textbook treatments, and on
this system it is the more dangerous one.

The controller samples every 60 seconds. The pumps run for 15, 10, and 6
seconds. A simulation stepping at 60 seconds **cannot represent a 15-second
pump pulse at all**.

Here is a 60-day open-loop simulation under the controller's three real
schedules, reporting root-zone water over the final ten days:

| Step | Pump 1 (15 s) | Pump 3 (10 s) | Pump 2 (6 s) |
|---|---|---|---|
| 60 s | 155.1--198.4 | 155.1--198.4 | 155.1--198.4 |
| 5 s | 154.4--169.7 | 132.5--141.0 | 132.5--141.0 |
| 1 s | 154.4--169.7 | 132.5--141.0 | 103.4--108.4 |

At a 60-second step the three schedules are **indistinguishable**, and the
simulation cheerfully reports all three pots comfortably watered. Correctly
integrated, Pump 2's pot sits at 103--108 mL, drifting toward the 60 mL
wilting point, while Pump 1's sits near field capacity. At a 5-second step
the 10 s and 6 s schedules still collapse together.

Note what makes this worse than divergence: the wrong answer is *plausible*.
Nobody looks at 155--198 mL and suspects a bug. A gardener acting on it
would conclude the schedules are equivalent and keep the one that starves
the plant.

The general rule: **your step must resolve the shortest event that matters,
not just satisfy a stability bound.** Here that is 15 seconds, even though
stability permits four hours. The usual production fix is to handle the
pump analytically -- add its delivered volume as an impulse and integrate
the slow dynamics at 60 s. Doing that agrees with a 1-second reference to
within 0.0007 mL over six hours, and runs sixty times faster.

**The lesson.** Solver and step size are part of your model, not an
implementation detail beneath it. Two engineers with identical equations
and different step sizes have different models. A twin that must keep pace
with a physical asset also has to worry about falling behind, which is a
live concern in real deployments [@frasheri_addressing_2023].

## 5. Technique 3: Distributed Models, and Technique 4: Surrogates

Section 3 bought simplicity by assuming each layer has one moisture value.
What if two layers are not enough?

### 5.1 When the lump breaks

Water does not arrive uniformly. Poured at one spot, it forms a wetting
front that moves down as a front, channels along the pot wall, and may
bypass dry soil entirely through cracks. Roots are denser in some regions
than others. A two-layer model cannot represent any of this: it has exactly
two numbers to describe the whole pot.

The honest response is a **distributed model**: discretise the soil into
many cells and track moisture in each, governed by Richards' equation, the
standard PDE for unsaturated flow in porous media. This captures the
wetting front, depth-dependent gradients, and preferential flow.

It is also expensive. A three-dimensional Richards solver for one pot can
take minutes per simulated day, and it needs soil hydraulic parameters that
are painful to measure. Our contract asks for three-day-ahead predictions
on a Raspberry Pi. A model that takes an hour to predict tomorrow is not a
digital twin component; it is an offline study.

This is the central tension of digital twin modelling. The most accurate
model available is usually far too slow to put in the loop, and practical
twins deliberately use lower fidelity, accepting a quantified accuracy loss
in exchange for keeping up with live data [@thelen_comprehensive_2022].

### 5.2 Surrogates and reduced-order models

The resolution is not to discard the expensive model but to *distil* it. A
**surrogate** is a cheap model fitted to the input-output behaviour of an
expensive one. A **reduced-order model** projects the governing equations
onto a low-dimensional subspace that captures the states actually visited.

The workflow is always three steps, and it recurs across the whole field:

1. **Sample offline.** Run the expensive model across the operating
   envelope -- watering volumes from 5 to 40 mL, initial moisture from
   wilting point to saturation, two or three soil mixes.
2. **Fit.** Build a cheap approximator on those samples.
3. **Deploy online.** Evaluate the cheap model in the live twin.

For our pot, a modest surrogate suffices: run the Richards model over that
envelope and fit a correction mapping our two-layer prediction to the
probe-depth moisture the full model reports. One correction term recovers
most of what the expensive model knew, at the cost of a table lookup.

This is how expensive physics reaches real-time twins in practice. Packaging
a specific set of behaviours extracted from a heavyweight simulation into a
stand-alone executable component is central enough to have earned its own
name, the *executable digital twin* [@hartmann_executable_2022]. Where
models of differing cost exist, a *multi-fidelity* surrogate can combine a
few expensive runs with many cheap ones, getting most of the accuracy for a
fraction of the sampling budget [@torzoni_deep_2023].

The cost is generalisation. A surrogate compresses the expensive model
*over the sampled region*. Ask it about a 200 mL flood when you sampled up
to 40 mL and it will answer -- fluently and wrongly. Record the sampled
envelope alongside the surrogate, and have the twin check inputs against it
rather than trusting the surrogate blindly.

## 6. Technique 5: Black-Box System Identification

Now discard the physics entirely. Suppose you know nothing about soil, but
you have InfluxDB full of probe readings and pump events. Can you build a
predictive model from data alone?

Yes. The classical technique is **system identification**: postulate a
structure with free parameters and fit them to measured sequences
[@thelen_comprehensive_2022].

### 6.1 The ARX model

The standard structure is *autoregressive with exogenous input*. The next
output is a linear combination of recent outputs and recent inputs:

$$\theta_s[k{+}1] = a_1 \theta_s[k] + a_2 \theta_s[k{-}1] + b_1 u[k] + b_2 u[k{-}1] + c$$

Second order, because we know the system has two states. In genuine
black-box practice you would try several orders and choose by
cross-validation.

The elegant part: this is *linear in the unknowns*, so fitting is ordinary
least squares. Stack every sample into $Y = \Phi\theta$ and solve
$\theta = (\Phi^T\Phi)^{-1}\Phi^TY$. One matrix solve, global optimum, no
learning rate.

### 6.2 It works perfectly, until it doesn't

Take an eight-day free-drying experiment -- pump off, starting just below
field capacity -- sampled every 10 minutes. With noise-free data, the fit
returns $a_1 = 1.92054$, $a_2 = -0.92063$. The true values, computed from
the matrix exponential of $A$, are $a_1 = 1.920544$, $a_2 = -0.920627$.
Exact recovery, to five decimals, knowing nothing about soil physics. That
is a genuinely remarkable result and it is why the technique has endured.

Now add measurement noise. Seesaw probes are noisy; a realistic standard
deviation is about 0.01 vol/vol. We will start far below that.

| Probe noise | $a_1$ | $a_2$ | 1-step RMS | Free-run RMS |
|---|---|---|---|---|
| 0 (exact) | 1.92054 | -0.92063 | 0.00000 | 0.00000 |
| **0.0005** | **0.47921** | **0.51897** | 0.00062 | 0.00081 |
| 0.002 | 0.47821 | 0.51847 | 0.00246 | 0.00722 |
| 0.010 (realistic) | 0.45968 | 0.49988 | 0.01222 | 0.03096 |

Look at the second row. The noise is 0.0005 vol/vol -- a *twentieth* of
what a real probe delivers, far below anything achievable in hardware. The
fitted coefficients are nonetheless **completely wrong**: $a_1$ has fallen
from 1.92 to 0.48, and $a_2$ has flipped sign from -0.92 to +0.52.

And both error metrics look excellent. One-step error is 0.00062 vol/vol.
Free-running error -- feeding predictions back, which is what a twin
forecasting three days ahead actually does -- is 0.00081, also tiny.

**Neither metric detects the failure.** This is worse than the usual
warning that one-step error is optimistic; here even the honest free-run
metric passes a structurally broken model. The reason is that the true
signal is a slow smooth decay, and *many* wrong models reproduce a slow
smooth decay. The data does not discriminate.

### 6.3 The diagnostic that does work

If prediction error cannot detect the problem, what can? **Physics.**

Convert the fitted coefficients to poles -- the roots of
$z^2 - a_1 z - a_2 = 0$ -- and then to time constants via
$\tau = \Delta t / (-\ln z)$:

| Model | Pole 1 | Pole 2 |
|---|---|---|
| True | 0.9989, giving 6.56 days | 0.9216, giving 2.04 hours |
| Fitted (noise 0.0005) | 0.9988, giving 5.79 days | **-0.5196** |

The fit recovered the slow mode nearly correctly. But its second pole is
**negative and real**, which corresponds to a mode that flips sign every
sample as it decays. Diffusion cannot do that. No physical process moving
water between two soil layers produces an alternating decay.

So you can reject this model **without any ground truth at all**, purely by
noticing that one of its poles is physically impossible. That is the
practical lesson: validate identified models against physical
plausibility -- pole signs, time constants, steady-state gains -- not only
against prediction error.

### 6.4 A second trap: nonlinearity in the data

Everything above used pure drying data, where the system is genuinely
linear. Refit on twenty days of normal operation, *with* the daily watering
included and no noise whatsoever, and you get $a_1 = 0.76541$,
$a_2 = 0.08362$ -- badly wrong again.

The culprit is the $\max(0, W_s - W_{s,fc})$ percolation term. Right after
watering, the surface exceeds field capacity and the system stops being
linear -- exactly during the most informative part of the record. Fitting a
linear structure to nonlinear data does not fail loudly; it returns
plausible-looking coefficients that are wrong.

### 6.5 What this technique is good for

None of this makes black-box identification bad. It makes it a technique
requiring discipline: excite the system deliberately rather than passively
logging normal operation; sample so consecutive samples genuinely differ;
segment the data so each fit covers one operating regime; and check the
identified dynamics against physical plausibility.

Its advantages are real. It needs no physical understanding, which matters
enormously for assets nobody has modelled. It is fast to build. It adapts
to a changing asset by refitting on recent data. And unlike a neural
network, it yields a model you can *analyse* -- which is exactly what made
the pole diagnostic possible.

## 7. Technique 6: Machine-Learning Models

Machine-learning models extend Section 6: instead of assuming a linear
structure, let a general-purpose learner find the structure. These range
from Gaussian processes and random forests to deep networks, and they earn
their keep where physics-based modelling struggles -- when the mechanism is
poorly understood, or understood but too expensive to simulate
[@thelen_comprehensive_2022].

For the pot, this is not a strawman. The water balance is well understood;
**plant growth is not**. How biomass responds to water, light, and nitrogen
involves photosynthesis, stomatal regulation, and nutrient transport that
no one writes down exactly for a specific houseplant. If we want the twin
to predict growth -- and "a plant controller that nourishes the soil"
implies exactly that -- machine learning stops being optional.

### 7.1 Where it fits on this system

The natural formulation for the water half mirrors ARX. Train a small
network
$\theta_s[k{+}1] = g_\phi(\theta_s[k], \theta_s[k{-}1], u[k], T[k], \mathrm{RH}[k], L[k])$,
now including the temperature, humidity, and light channels the hardware
already logs. A physics model would need explicit evapotranspiration
equations to use those; a network just takes them as features.

Two structural points deserve emphasis. First, training happens **offline**,
once; what runs in the live twin is the trained network doing fast forward
evaluations, not the training loop. Confusing these makes people wildly
overestimate the compute a deployed twin needs.

Second, training on one-step prediction and then running recursively for a
three-day forecast is a known trap -- Section 6.2's problem with more
freedom to fit noise. Errors compound and trajectories drift somewhere
unphysical. Whether a network is trained on one-step or multi-step
objectives materially changes how it behaves when deployed recursively
[@legaard_constructing_2023].

### 7.2 Gaussian processes and the value of "I don't know"

For a system this small, a Gaussian process is often the better choice. A
GP places a distribution over functions and conditions it on data, and its
decisive advantage is a *predictive variance* alongside every prediction --
confidence that correctly collapses far from the training data
[@thelen_comprehensive_2022].

That matters more than it first appears. A twin that says "the soil will be
at 0.19 vol/vol on Thursday, and I have never seen conditions like this" is
far safer driving a pump than one that just says "0.19". The cost is cubic
scaling in sample count, so plain GPs become impractical past a few thousand
points -- though at one sample per minute, a month of data is 43,200 points,
so this bites sooner than you might hope.

### 7.3 The extrapolation problem

Train on a pot that has lived between 0.15 and 0.28 vol/vol, then ask about
0.08 -- below wilting point. The physics model answers correctly: mass
conservation holds, the stress factor clips to zero, transpiration stops.
The network answers confidently and arbitrarily. It has no notion that
water is conserved or that a plant cannot transpire from soil drier than
its wilting point.

This is the fundamental trade. A machine-learning model is only as good as
its training examples and can fail badly on situations unlike anything it
has seen [@thelen_comprehensive_2022]. For a digital twin this matters more
than in typical ML applications, because the situations you most need the
twin to handle -- the failing pump, the forgotten holiday, the plant that
suddenly wilts -- are exactly the ones least represented in data from a
normally-functioning system.

Which motivates the next technique.

## 8. Technique 7: Hybrid (Grey-Box) Models

If physics extrapolates but misses effects, and data-driven models capture
anything but extrapolate terribly, combine them. Hybrid or grey-box models
do exactly that, and they are increasingly the default in serious digital
twin work [@rasheed_digital_2020].

For this system the split is unusually clean, and it is the honest reason
to prefer hybrid here. **Water movement is physics we know. Plant response
is biology we do not.** That is not a contrived division invented for a
tutorial; it is where the knowledge boundary genuinely falls.

### 8.1 Residual learning

Keep the physics model and train a learner to predict its *error*:

$$\theta_s^{\text{hybrid}}[k{+}1] = \underbrace{\theta_s^{\text{physics}}[k{+}1]}_{\text{Section 3}} + \underbrace{\Delta_\phi(\theta_s[k], u[k], T[k], \mathrm{RH}[k], L[k])}_{\text{learned correction}}$$

Our two-layer model assumes evaporation depends only on soil water. In
reality it depends strongly on humidity and air movement. The residual
model learns that dependence from the SHT45 channel without anyone deriving
a vapour-transport equation.

The properties are excellent. The correction is *small* -- a few percent of
the signal -- so it needs far less data than learning the whole dynamics.
Outside the training region the physics term still dominates, so the model
degrades gracefully rather than catastrophically. And the residual is
diagnostic in itself: if $\Delta_\phi$ grows steadily over months, the
physics model is drifting from the asset, which is a maintenance signal
(Section 13).

### 8.2 Physics-informed learning

The reverse direction: keep the network but constrain it with physics. A
*physics-informed* network adds a loss term penalising predictions that
violate known laws. Here the constraint is mass conservation -- water in
minus water out must equal change in storage. If the network predicts the
pot gaining water with the pump off and no condensation, that is
impossible, and the loss says so.

The result generalises better than pure machine learning while staying much
cheaper than full physics simulation, borrowing the physics model's ability
to stay sensible outside the training data and the data-driven model's
ability to run fast [@thelen_comprehensive_2022].

### 8.3 Parameter-varying physics

The third hybrid, and the one most used in practice because it is least
exotic: keep the Section 3 structure exactly, but let a learner supply its
*parameters* as functions of operating condition. Instead of a fixed
$E_r = 12$ mL/day, learn $E_r$ as a function of biomass, light, and
humidity.

This preserves everything good about the white-box model -- interpretability,
extrapolation, small data requirements -- while absorbing effects it could
not capture. The parameters stay physically meaningful, so a learned $E_r$
that has doubled over a season still means "this plant now drinks twice as
much", which no network weight ever means. Section 10 shows what that
enables, and Section 13 shows why you want it.

## 9. Technique 8: Probabilistic Models

Every model so far returns one number. Plants do not behave that way. Two
seedlings from the same packet, in identical pots, on the same shelf, will
differ measurably within weeks -- and no amount of modelling effort removes
that. Biological variability is *irreducible*, not a measurement problem.

A probabilistic model returns a *distribution*, and for a twin whose output
drives a decision, that is often the more useful object.

### 9.1 Where uncertainty comes from

Separating the sources matters, because they are addressed differently
[@thelen_comprehensive_2022-1]:article.md

- **Measurement noise.** The Seesaw probe reports 0.21 when the truth is
  0.205.
- **Parameter uncertainty.** We estimated $K = 0.02$, but the data pins it
  only to roughly 15%.
- **Process noise.** A draught, a sunny afternoon, someone brushing past.
- **Structural uncertainty.** Two layers is not really right, and no
  parameter value fixes that.

The first three are aleatory or parametric and admit routine probabilistic
treatment. The fourth is epistemic, much harder, and routinely ignored --
which is why models are so often overconfident in deployment.

### 9.2 The stochastic pot

Add noise terms:

$$
\begin{aligned}
x_{k+1} &= A_d x_k + B_d u_k + w_k, \qquad w_k \sim \mathcal{N}(0, Q) \\
z_k &= H x_k + v_k, \qquad\qquad\quad\ v_k \sim \mathcal{N}(0, R)
\end{aligned}
$$

with $w_k$ the process noise and $v_k$ the measurement noise. For our probe,
$R = 10^{-4}$ in vol/vol units.

This is not cosmetic. It changes what the model *is*: it now predicts a
distribution. Instead of "0.19 vol/vol on Thursday" it says "0.19 plus or
minus 0.02", and the second statement is the one you can act on. It is also
exactly the structure Section 10 needs.

### 9.3 Bayesian calibration

Given data, Bayesian calibration treats parameters as random variables and
computes a *posterior* rather than a point fit. The output is not
"$K = 0.02$" but a distribution reflecting how much the data actually
constrained it -- which, as Section 14 shows, is much less than you would
hope for some parameters. Propagating that distribution forward gives
honest prediction intervals.

The probabilistic graphical model framework generalises this, representing
the twin's state, parameters, observations, and control inputs as nodes in
a graph whose structure encodes what depends on what, updated as data
arrives [@kapteyn_probabilistic_2021]. Our pot's graph is small, but the
structure scales.

The advantage is that it makes ignorance explicit. Given a bare number, a
decision-maker cannot tell whether it is trustworthy; given a distribution,
they can. Uncertainty quantification is correspondingly treated as central
rather than optional in digital twin practice
[@thelen_comprehensive_2022-1]. The cost is computation -- a full Bayesian
treatment can need thousands of model evaluations, which is often exactly
why Section 5.2's surrogates exist.

## 10. Technique 9: State Estimation and Virtual Sensors

Now we pay off the promise from Section 2.3. The probe measures the surface
layer. The plant lives in the root layer. How wrong is it to use one for
the other?

### 10.1 How badly the probe lies

Water 24 mL into a pot sitting at 0.181 vol/vol, then watch both layers.
The final column expresses the error in root-zone millilitres, and in
multiples of the probe's own noise:

| Time after watering | $\theta_s$ (probe) | $\theta_r$ (truth) | Error if you trust the probe |
|---|---|---|---|
| 1 min | 0.2998 | 0.1814 | 71.1 mL (11.8 sigma) |
| 5 min | 0.2946 | 0.1830 | 67.0 mL (11.2 sigma) |
| 15 min | 0.2849 | 0.1861 | 59.3 mL (9.9 sigma) |
| 1 h | 0.2607 | 0.1933 | 40.4 mL (6.7 sigma) |
| 2 h | 0.2399 | 0.1992 | 24.4 mL (4.1 sigma) |
| 4 h | 0.2187 | 0.2044 | 8.6 mL (1.4 sigma) |
| 8 h | 0.2058 | 0.2050 | 0.4 mL (0.1 sigma) |

Immediately after watering the probe overstates root-zone moisture by up to
**71.9 mL, twelve standard deviations of probe noise**. The water is
sitting in the top of the pot; the roots have not received it yet. Only
after about eight hours, roughly four redistribution time constants, do the
layers agree.

Note the honest flip side: after eight hours the probe is an excellent
proxy. A twin that estimates root-zone moisture earns its keep in the hours
*after* watering, not all the time. Virtual sensing pays off where dynamics
are fast, and that is worth knowing before you build one.

### 10.2 The Kalman filter

For a linear system with Gaussian noise -- exactly Section 9.2 -- the
optimal estimator is the Kalman filter, alternating two steps.

**Predict.** Push the estimate forward and grow the uncertainty:

$$\hat{x}_{k|k-1} = A_d\hat{x}_{k-1} + B_d u_{k-1}, \qquad P_{k|k-1} = A_dP_{k-1}A_d^T + Q$$

**Update.** Correct in proportion to relative confidence:

$$K_k = P_{k|k-1}H^T(HP_{k|k-1}H^T+R)^{-1}, \qquad \hat{x}_k = \hat{x}_{k|k-1} + K_k(z_k - H\hat{x}_{k|k-1})$$

The gain $K_k$ is the whole idea: trust the sensor when it is reliable,
trust the model when it is not, and blend in proportion. Neither the model
blindly nor the noisy sensor blindly [@thelen_comprehensive_2022].

For our pot, $A_d$ over a 60-second step is exactly

$$A_d = \begin{bmatrix} 0.993832 & 0.001992 \\ 0.005975 & 0.997931 \end{bmatrix}$$

whose trace and determinant, 1.991763 and 0.991764, are precisely the ARX
coefficients Section 6.2 recovered from clean data. The same system, seen
two ways.

The remarkable part is the *unmeasured* state. The off-diagonal terms of
$P$ encode the correlation between the layers implied by the model. Because
the model knows water percolates downward at a known rate, an error in
predicted surface moisture is informative about the root layer, and the
filter corrects $W_r$ using a surface measurement.

Over 30 days at a realistic probe noise of 0.01 vol/vol:

| Estimator | RMS error, all times | RMS error, 4 h after watering |
|---|---|---|
| Naive (assume root = surface) | 8.69 mL | 16.25 mL |
| Kalman filter | **5.75 mL** | **11.30 mL** |

A 1.5 times improvement -- real but modest, exactly as Section 10.1
predicted, because most of the time the naive estimate is fine. What the
filter buys you is that it is *never* off by 71 mL, and it tells you which
regime you are in. Real deployments lean on this pattern heavily; a twin of
a floating offshore wind turbine uses the same Kalman-plus-structural-model
combination to estimate tower loads its sensors cannot see
[@branlard_digital_2024]. Our pot does it at houseplant scale.

### 10.3 Joint state and parameter estimation, and an honest failure

A natural extension: append unknown *parameters* to the state and let the
filter estimate them too. Since the system is now nonlinear in the
augmented state, this needs an *extended* Kalman filter, which linearises
about the current estimate at each step.

The parameter worth estimating here is the plant's **water demand**. As the
plant grows, transpiration rises; a fixed schedule that suited a seedling
starves a mature plant. Nothing on the hardware measures demand. If the
twin could infer it, the controller could adapt.

So we simulated 60 days during which true demand rises from 1.0 to 2.15
times the seedling value, and ran an EKF with demand as a third state.

**It does not work well.** The filter tracks demand from 1.00 to only 1.39
while the truth reaches 2.15 -- recovering about 40% of the growth, with an
RMS error of 0.55. Worse, it reports a standard deviation of 0.042,
claiming roughly 13 times more confidence than it deserves. Meanwhile the
root-zone estimate stays excellent: 114.0 mL estimated against 113.6 mL
true at day 55.

The reason is **weak observability**. From one surface probe, "the root
zone is drier than I thought" and "the plant is thirstier than I thought"
explain the same measurements almost equally well. The filter cannot
separate them, so it splits the difference and reports false confidence.

This is not a bug to tune away; it is a property of the sensor placement.
The fix is not a better filter but better *data*: deliberately withhold
water for a few days and measure the drying slope, which depends on demand
and almost nothing else. That is Section 6.5's excitation lesson returning
in a new costume, and it connects directly to the identifiability
discussion in Section 14.1.

Report this kind of result. A twin that silently produces a confident wrong
number is worse than one that reports it cannot tell.

## 11. Technique 10: Discrete-Event Models and Hybrid Automata

Everything so far models the *soil*. But the system also contains a
controller, and a controller is not a differential equation. It is a
switch. Modelling it with continuous mathematics is a category error.

### 11.1 The controller as a state machine

The shipped controller is purely time-driven: at 10:15, turn on Pump 1;
after 15 seconds, turn it off. That is a state machine with two states and
transitions triggered by clock events. Time appears only through event
ordering.

To close the loop, we replace it with moisture-triggered control with
hysteresis: in state `DRY`, run the pump until the probe exceeds
$\theta_{\text{off}}$; in state `WET`, hold off until it falls below
$\theta_{\text{on}}$.

### 11.2 Hybrid automata

The full system is neither purely continuous nor purely discrete. It is
**hybrid**, and the formalism capturing both is the *hybrid automaton*: the
system occupies one of several discrete *modes*, within each mode the
continuous state evolves by mode-specific differential equations, and
crossing a guard triggers a jump [@abbiati_modelling_2024].

Our pot has genuinely physical modes, not just controller states:

| Mode | Continuous dynamics | Guard | Target |
|---|---|---|---|
| `SATURATED` | Free drainage active | $W_s < W_{s,fc}$ | `NORMAL` |
| `NORMAL` | Linear two-layer model | $\theta_r$ below stress threshold | `STRESSED` |
| `STRESSED` | Transpiration reduced by $f_r$ | $\theta_r < \theta_{wp}$ | `WILTING` |
| `WILTING` | Transpiration zero | $\theta_r > \theta_{wp}$ | `STRESSED` |

Simulating the closed loop with thresholds on the surface probe:

| Thresholds | Cycle period | Delivered | Root-zone range |
|---|---|---|---|
| 0.18 / 0.24 | 28.16 h | 6.00 s/day | 0.181--0.193 |
| 0.22 / 0.26 | 14.71 h | 9.30 s/day | 0.222--0.229 |
| 0.24 / 0.28 | 12.71 h | 10.95 s/day | 0.242--0.249 |

Compare against the 13.75 s/day from Section 2.5. None of them reach it,
and the reason is instructive: that figure assumed an *unstressed* plant.
These controllers hold the pot below field capacity, so the stress factors
sit below 1, so the plant actually transpires less than 22 mL/day and needs
less water. The arithmetic was an upper bound, not a target -- and noticing
the discrepancy is how you discover that the plant is being held slightly
dry.

### 11.3 What noise does to a discrete controller

Now add realistic probe noise of 0.01 vol/vol to the 0.22/0.26 controller:

| | Cycle period | Delivered |
|---|---|---|
| Noise-free | 14.71 h | 9.30 s/day |
| Noise 0.01 | **3.04 h** | 11.68 s/day |

The cycle period collapses by nearly a factor of five. Noise repeatedly
carries the reading across the hysteresis band, so the pump switches far
more often than designed -- classic **short-cycling**, which wears out
relays and pumps.

This is a genuinely hybrid phenomenon: it arises from the *interaction*
between a continuous noise process and a discrete switching rule, and
neither model alone predicts it. The fix is also hybrid -- widen the
hysteresis band, filter the measurement, or impose a minimum dwell time in
each mode. Notice that the Kalman filter of Section 10 would supply a
filtered estimate almost for free.

### 11.4 Why the distinction matters

Systems combining continuous and discrete behaviour across engineering
disciplines are the normal case, and the discipline of using several
formalisms together is developed under the name *multi-paradigm modelling*
[@carreira_foundations_2020]. Forcing everything into one formalism means
either a controller described by awkward continuous approximations or soil
physics described by an implausibly large state machine. Neither is a good
trade when you can use the right formalism for each part and define their
interaction carefully.

## 12. Technique 11: Co-Simulation

Section 11 raises a practical question. The soil model might be built in
Modelica by a soil physicist. The growth model might come from an agronomist
in R. The controller is Python on a Raspberry Pi. In a real project these
come from different people, different tools, and different institutions --
and they still have to run as one twin.

### 12.1 The idea

**Co-simulation** couples separately built simulators without merging them.
Each sub-model keeps its implementation private and exposes a small
standard interface -- essentially read a variable, set a variable, advance
by a step [@abbiati_modelling_2024]. A master algorithm sets inputs, steps
each unit, exchanges outputs, and repeats.

The dominant standard is the **Functional Mock-up Interface**, maintained by
the Modelica Association and supported by well over a hundred tools; a model
packaged to it is a *Functional Mock-up Unit* [@abbiati_modelling_2024].
For our pot you would export the soil model as one FMU and the controller
as another, then step them in lockstep.

This is not hypothetical for this hardware. Work on the INTO-CPS
Association's own digital twin platform integrates FMI-based models
alongside machine-learning components in exactly this style
[@infante_integrating_2024], which is precisely the hybrid split Section 8
argued for: physics as an FMU, biology as a learned component, coupled at
runtime. Tooling has become increasingly Python-accessible, lowering the
barrier for this kind of prototyping [@friedrich_cofmpy_2025], and the
approach scales to serious industrial coupling [@wetter_spawn_2024-1].

### 12.2 The catch

Co-simulation looks like free composition and is not. Each unit advances
*independently* over a communication step, during which it does not see the
others change. Inputs are typically held constant across the step, which
introduces error unrelated to either model's own accuracy.

On our pot the consequence should now be familiar. If the communication
step is 60 seconds, the controller cannot react to a threshold crossing
until up to 60 seconds later -- and the pump only runs for 15. Section 4.3's
event-resolution problem returns, now as a *coupling* problem, and it is
why co-simulation masters for systems with short events need either very
small steps or explicit event detection.

Worse, coupling two individually stable simulators can produce an unstable
*combined* simulation, a failure with no counterpart in either model alone.
Numerical stability of the coupling, algebraic loops, and tool
interoperability remain recognised open challenges rather than solved
engineering [@schweiger_empirical_2019].

The lesson generalises: composing models is itself a modelling activity
with its own error sources, and "I validated each component" does not imply
"the assembly is valid".

## 13. Technique 12: Degradation Models

Every model so far assumes fixed parameters. Real systems age. **Degradation
models** describe how parameters evolve over months and years, and they
underpin predictive maintenance -- one of the most commercially important
uses of digital twins [@van_dinter_predictive_2022].

Our pot degrades in two ways, and they behave very differently.

### 13.1 The sensor degrades

Capacitive soil probes corrode. Electrodes oxidise, the protective coating
degrades, and readings drift -- typically reading progressively *lower* over
a season or two. Suppose the drift is about 0.02 vol/vol per year.

The controller holds the *measured* value in its hysteresis band, so drift
moves the *true* moisture the same amount:

| Year | Probe reads | True $\theta_s$ held at | Consequence |
|---|---|---|---|
| 0 | correct | 0.220--0.260 | as designed |
| 1 | 0.02 low | 0.240--0.280 | wetter than intended |
| 2 | 0.04 low | 0.260--0.300 | **above field capacity: waterlogged** |
| 3 | 0.06 low | 0.280--0.320 | **root rot** |

By year two the controller is holding the pot permanently above field
capacity. It is drowning the plant while its logs show a textbook-perfect
0.22--0.26 band. This is the most dangerous kind of degradation: the sensor
that would reveal it is the thing that failed.

Detecting it requires reasoning the raw signal cannot support. Two options
work. First, **redundancy** -- the hardware has three probes, and probes
rarely drift identically; a growing divergence between them is evidence.
Second, **model-based residuals** -- if the twin's physics predicts the pot
should dry from 0.26 to 0.22 in 14 hours and it consistently takes 20, the
model and the sensor disagree, and one of them is wrong. Section 14.3
develops this into a general drift alarm.

### 13.2 The soil degrades

Potting mix settles and compacts, closing the large pores that conduct
water quickly. That reduces the exchange coefficient $K$:

| $K$ | Redistribution time constant |
|---|---|
| 0.020 (fresh) | 2.04 h |
| 0.015 | 2.70 h |
| 0.010 | 4.00 h |
| 0.006 | 6.50 h |

As $K$ falls, water takes longer to reach the roots and the post-watering
transient of Section 10.1 grows both larger and longer. The plant spends
more of each day with water sitting above its roots rather than around
them, even though total water delivered is unchanged.

This one is nicely observable. $K$ is a *physically meaningful* parameter,
so tracking it -- by joint estimation, or by periodically refitting on a
drying experiment -- gives a direct read on soil condition and a repotting
schedule. Extrapolating the trend gives a remaining-useful-life estimate for
the growing medium.

That interpretability is the first-principles model of Section 3 finally
paying off, ten techniques later. A neural network trained on the same data
has no weight meaning "how fast water reaches the roots", so it cannot
support this reasoning at all.

### 13.3 The general pattern

Real degradation modelling for high-value assets is more sophisticated, but
the shape recurs: a physics-based or empirical model of the mechanism,
fused with machine learning that corrects it against observed behaviour,
producing a *probabilistic* remaining-useful-life estimate rather than a
point value [@thelen_augmented_2022]. Because these predictions are
long-horizon and inherently uncertain, Section 9's probabilistic treatment
is essential rather than optional -- the useful output is a distribution
over failure time, not a date [@thelen_probabilistic_2024].

## 14. Calibration, Validation, and Drift

Three cross-cutting activities apply to every technique above, and skipping
them is the commonest way a technically sound model becomes a useless twin.

### 14.1 Calibration

**Calibration** estimates a model's free parameters from measurements. For
our pot the procedure is concrete: water to field capacity, then withhold
water for a week while logging the probe every minute, and solve a
nonlinear least-squares problem for the parameters that best match the
measured drying curve [@gomes_calibration_2024].

Three practical points, all of which this system illustrates.

First, calibrate against a *rich* experiment. A drying curve identifies the
loss terms well and says almost nothing about $k_d$, which only acts above
field capacity. Identifying $k_d$ needs a deliberate over-watering.

Second, some parameters are far better constrained than others. This is
**practical identifiability**, and Section 10.3 met it in its sharpest form:
plant water demand was so weakly observable from a single surface probe
that an otherwise well-behaved filter recovered under half of a doubling.
No estimator fixes that. Only a different experiment does.

Third, calibration compensates only for *parametric* error. If the model
structure is wrong -- two layers where you need five, or a linear fit to
the nonlinear regime of Section 6.4 -- calibration produces a well-fitted
wrong model, which is more dangerous than an obviously bad one.

### 14.2 Validation

**Validation** asks whether the calibrated model is fit for its stated
purpose, and the non-negotiable rule is that it must be evaluated on data
the calibration never saw. For the pot: calibrate on a drying experiment,
then validate on normal watering cycles and on a deliberate over-watering.

Validation must test *the quantity you care about, over the horizon you
care about*. But Section 6.2 is the sobering case: a structurally broken
model passed both one-step *and* free-run tests. Prediction error alone was
not enough, and the check that caught it was physical -- a negative pole in
a diffusive system. Build both kinds of check into your validation.

Finally, validation is bounded by Section 2.2's envelope. A model validated
indoors at 18--25 °C has been shown to work *there*. It has been shown
nothing about a windowsill in August. Establishing credibility for models
used in consequential decisions is an active research problem, not a solved
checklist
[@committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024].

### 14.3 Drift and model updating

A model calibrated once diverges from the asset through wear, repair, or
unanticipated conditions. Production twins need a plan for *updating*, and
the options line up with the techniques above: periodically refit on recent
data; use the estimator of Section 10 to nudge the model toward the sensors
continuously; or run joint state-parameter estimation so parameters track
degradation automatically.

Updating a twin from experimental measurements as the asset changes is an
active area of method development rather than a settled procedure
[@wagner_data-driven_2025], and deciding *when* a twin has diverged enough
to require intervention is a genuine research question in its own right
[@zhang_knowledge_2024].

A monitoring rule you can implement today: track the Kalman filter's
**innovation** -- the difference between each measurement and its
prediction. If the model is right and the noise is characterised correctly,
innovations should be zero-mean and consistent with their predicted
covariance. A persistent bias means the model no longer matches the asset.
That is exactly the drift alarm Section 13.1 needed to catch a corroding
probe, and it comes free from machinery you already have.

## 15. Choosing a Technique

| # | Technique | Needs | Gives you | Fails when |
|---|---|---|---|---|
| 1 | First-principles | Physical understanding | Interpretability, extrapolation, tiny data needs | Mechanism unknown, or too costly to solve |
| 2 | Numerical solvers | A continuous model | An executable model | Step exceeds stability limit, or misses short events |
| 3 | Distributed (PDE) | Geometry, material data, compute | Spatial detail a lump cannot capture | Real-time use; far too slow |
| 4 | Surrogate / ROM | An expensive model to sample | Real-time speed from expensive physics | Inputs stray outside the sampled envelope |
| 5 | System identification | Logged input/output data | Fast models with no physics knowledge | Data poorly excited, noisy, or spanning regimes |
| 6 | Machine learning | Large, representative datasets | Patterns no one can write equations for | Asked about unseen conditions |
| 7 | Hybrid / grey-box | Partial physics plus some data | Graceful failure, good generalisation | Neither ingredient available |
| 8 | Probabilistic | A noise characterisation | Honest confidence, not just a number | Compute budget is tight |
| 9 | State estimation | A model plus live measurements | Unmeasured states; drift correction | The quantity is weakly observable |
| 10 | Hybrid automata | Discrete mode structure | Correct switching, short-cycling, limit cycles | Modes proliferate combinatorially |
| 11 | Co-simulation | Models in incompatible tools | Composition without rewriting | Coupling step too coarse; instability |
| 12 | Degradation | Long-horizon historical data | Predictive maintenance, RUL | Mechanism unknown or unobservable |

Three questions resolve most real choices.

**What is the model for?** Predicting when the pot needs water needs
Sections 3--4 and nothing more. Knowing what the roots experience right now
needs Section 10. Deciding whether to repot needs Section 13. Purpose
determines technique, and a twin with several purposes needs several models
[@gomes_foundational_2024].

**What do you have?** Good physics and little data: go white-box. Poor
physics and abundant data: go black-box. Some of each -- the usual case,
and exactly our pot, where hydrology is known and biology is not -- go
hybrid, which is why most industrial practice has converged there
[@rasheed_digital_2020].

**What is your compute budget?** Our pot's 2-hour and 6.5-day time constants
are forgiving; a Raspberry Pi can run the whole twin. Assets with
millisecond dynamics are not so kind, and that is what Section 5.2's
surrogates are for.

### Closing the loop

Recall where we started: a controller that waters on a clock and ignores its
own sensors. You now have everything needed to fix it. A first-principles
model (Section 3), integrated so it resolves the pump pulse (Section 4), its
parameters calibrated from a drying experiment (Section 14.1), driving a
Kalman filter that reports root-zone moisture rather than surface moisture
(Section 10), feeding a hysteresis controller with a dwell time long enough
to resist short-cycling (Section 11), monitored by innovation-based drift
detection that notices when the probe starts to corrode (Sections 13.1 and
14.3). That is a digital twin, and every piece of it was verified on the way
through.

One last observation. Across all twelve techniques, the recurring failure
was never the mathematics. It was applying a model outside the conditions it
was built for: Euler past its stability limit, a 60-second step against a
15-second pump, a linear fit to data spanning a nonlinear regime, a filter
asked for a quantity its sensor could not observe, a surrogate outside its
sampled envelope. Every one is the *same mistake*. The discipline that
prevents it is the modelling contract of Section 2.2 -- write down what the
model is for and where it is valid, and check inputs against it at runtime.
It is unglamorous, and it is most of what separates a digital twin that
works from one that merely runs.

## 16. Exercises

Exercises marked ($\star$) are straightforward; ($\star\star$) require
combining ideas; ($\star\star\star$) are open-ended. Hints and solutions
follow.

**16.1 ($\star$) Water balance.** Using Section 2.4's parameters, compute
the pump seconds per day needed to hold an unstressed plant in steady state
if you replace the dosing pump with one delivering 4.0 mL/s. Then compute
how long the pot survives from field capacity with the pump disconnected.
State which assumption you are relying on.

**16.2 ($\star$) Implement the model.** Code the Euler stepper from Section
4.1 and reproduce the free-drying reference $W_r = 83.627$ mL after ten
days. Then reproduce the divergence at a 4.44 hour step. *Objective 2.*

**16.3 ($\star\star$) The event you cannot see.** Reproduce Section 4.3's
table. Then implement the impulse shortcut -- add the pump's volume
instantly and integrate at 60 s -- and verify it matches a 1-second
reference to better than 0.001 mL over six hours. Why is this legitimate
here but not if the pump ran for two hours? *Objectives 2, 4.*

**16.4 ($\star\star$) Stability boundary.** Derive the eigenvalues of $A$
symbolically in terms of $K$, $V_s$, $V_r$, $a_s$ and $a_r$. Predict the
Euler stability limit if the pot were divided into a 50 mL surface layer
instead of 200 mL. Verify numerically. What does this say about modelling
thin surface layers? *Objectives 1, 2.*

**16.5 ($\star\star$) The identification trap.** Generate an eight-day
drying curve, add noise of 0.0005 vol/vol, fit the ARX model, and reproduce
the collapse in Section 6.2. Confirm both error metrics look excellent.
Then compute the poles and find the negative one. *Objective 3.*

**16.6 ($\star\star$) Excitation design.** Design a watering profile that
identifies $K$ better than passive logging. Justify it in terms of Section
3.3's time constants, then test it against 16.5's fit. *Objective 3.*

**16.7 ($\star\star$) Virtual sensing.** Implement the Kalman filter of
Section 10.2 and reproduce the 5.75 mL RMS error. Then compare against the
naive estimator in the four hours after watering. How much does the
advantage depend on watering frequency? *Objective 5.*

**16.8 ($\star\star$) Short-cycling.** Reproduce Section 11.3's collapse
from 14.71 h to 3.04 h under probe noise. Then fix it three ways -- widen
the band, filter the measurement, impose a minimum dwell time -- and compare
pump switches per day. Which best preserves the target moisture?
*Objectives 4, 5.*

**16.9 ($\star\star\star$) Detect the drift.** Simulate a probe drifting
0.02 vol/vol per year under closed-loop control. Using only logged data --
no ground truth -- devise a detector that raises an alarm before the pot
goes above field capacity. How early can you catch it, and what is your
false-alarm rate over a year? *Objectives 4, 5.*

**16.10 ($\star\star\star$) Technique selection.** You must twin a
commercial greenhouse: 5,000 plants, moisture probes on 200 of them,
climate control, two years of logs, and a target of predicting crop yield
eight weeks ahead. Choose techniques from Section 15, justify each, and name
one you rejected and why. How does your answer change if you have only two
weeks of data? *Objective 4.*

## 17. Hints and Solutions

**17.1 (full solution).** Demand is $E_s + E_r = 22$ mL/day regardless of
pump. At 4.0 mL/s the pump needs $22/4.0 = 5.5$ s/day. From field capacity
(224 mL) to wilting point (80 mL) there are 144 mL available; at 22 mL/day
that is 6.5 days, but this *overestimates* survival because the stress
factors reduce transpiration as the soil dries, so the true figure is longer
and the plant is suffering for much of it. You are relying on the unstressed
assumption -- which, note, is exactly why Section 11.2's controllers
delivered less than the predicted 13.75 s/day.

**17.2 (hint).** Loop the stepper with `u=0.0` from
`Ws, Wr = 56.0, 168.0`. For the divergence, watch both states: they
oscillate with growing amplitude, and the alternating sign is the signature
of instability rather than inaccuracy -- the same signature as the negative
pole in 16.5, which is not a coincidence.

**17.3 (hint).** The shortcut works because 15 s is far shorter than the
2.04 h redistribution constant, so almost nothing happens *during* the
pulse; only the delivered volume matters. A two-hour pump run is comparable
to the fast time constant, so redistribution during pumping is significant
and the impulse approximation breaks. The general test: an event may be
treated as instantaneous when its duration is small compared with the
fastest dynamics it excites.

**17.4 (hint and partial solution).** The characteristic polynomial is
$\lambda^2 - \mathrm{tr}(A)\lambda + \det(A) = 0$ with
$\mathrm{tr}(A) = -K/V_s - a_s - K/V_r - a_r$. Shrinking $V_s$ to 50 mL
roughly quadruples $K/V_s$, so the fast eigenvalue grows about fourfold and
the Euler limit falls to roughly an hour. The lesson: *thin layers force
small time steps*. This is why modellers often merge a thin surface layer
into its neighbour, or replace it with an algebraic relation -- a
quasi-steady-state approximation trading a little accuracy for a large gain
in allowable step size.

**17.5 (hint).** Reuse the design matrix from Section 6.1 on drying data
only. The poles are roots of $z^2 - a_1z - a_2 = 0$; you should find 0.9988
and -0.5196. Convert with $\tau = \Delta t/(-\ln z)$, which is undefined for
the negative root -- and *that* is the diagnostic. Any real diffusive
process has poles on the positive real axis between 0 and 1.

**17.6 (hint).** You need to excite the fast mode, which passive daily
watering barely touches. A sequence of small waterings spaced near the
2-hour redistribution constant -- rather than one large daily dose -- puts
energy where the fast dynamics live. Contrast with the slow mode, which a
week-long dry-down identifies well. The general principle: excite at
timescales comparable to the dynamics you want to identify.

**17.7 (hint).** Use $A_d$ from Section 10.2, a measurement matrix picking
out the surface state, and $R = 10^{-4}$. The filter's advantage is
concentrated immediately after watering, so watering more often *increases*
the advantage -- a useful counter-intuitive result. If your RMS is much
worse than 5.75 mL, check that you are adding the pump volume to the
predicted state as well as the true one.

**17.8 (hint).** Widening the band preserves pump life but lets moisture
swing further from target. Filtering adds lag, which shifts the effective
thresholds -- measure where the pot actually sits, not where you told it to.
A minimum dwell time is usually the best trade here, because it directly
bounds switching frequency without biasing the setpoint. Note that a
Kalman-filtered estimate (Section 10) gives you the filtering benefit with
far less lag than a moving average.

**17.9 (discussion).** The key insight is that drift is invisible in the
controlled variable by construction -- the controller *forces* the measured
value into its band, so the log looks perfect. You must look at something
the controller does not regulate. Strong answers use: divergence between the
three probes; the *drying time* between waterings, which shortens as the pot
gets genuinely wetter; pump duty cycle, which rises; or Kalman innovations,
which develop a persistent bias. Duty cycle is the most practical -- it is
already logged and needs no extra hardware. Discuss the false-alarm
trade-off honestly: seasonal changes in evaporation also shift duty cycle,
so a detector needs to normalise for temperature and humidity, which the
SHT45 already provides.

**17.10 (discussion).** A defensible answer: hybrid grey-box (technique 7)
for the water and climate balance, since the physics is the same as our pot
and generalises across plants; machine learning (technique 6) for the yield
model, since 5,000 plants over two years is genuinely enough data and nobody
has first-principles yield equations; a surrogate (technique 4) so the twin
runs on greenhouse hardware; probabilistic output (technique 8), because an
eight-week yield forecast without a confidence interval cannot support a
sales commitment; and state estimation (technique 9) to infer moisture for
the 4,800 plants with *no* probe from the 200 that have one -- the
highest-value application in the whole system. Reasonable rejection:
distributed PDE modelling (technique 3) per plant, because 5,000 Richards
solvers cannot run in real time and the spatial detail does not change any
decision. With only two weeks of data the answer inverts: there is nowhere
near enough to learn a yield model spanning a growing season, so lean much
harder on physics, and treat the first season as a calibration campaign --
which is Section 14.1's excitation argument at industrial scale.

## 18. References

- **abbiati_modelling_2024** -- Modelling for Digital Twins (2024).
- **branlard_digital_2024** -- A digital twin solution for floating offshore wind turbines validated using a full-scale prototype (2024).
- **carreira_foundations_2020** -- Foundations of Multi-Paradigm Modelling for Cyber-Physical Systems (2020).
- **committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024** -- Foundational Research Gaps and Future Directions for Digital Twins (2024).
- **frasheri_addressing_2023** -- Addressing time discrepancy between digital and physical twins (2023).
- **friedrich_cofmpy_2025** -- CoFMPy: A Python Framework for Rapid Prototyping of FMI-based Digital Twins (2025).
- **gomes_calibration_2024** -- Calibration of Models for Digital Twins (2024).
- **gomes_foundational_2024** -- Foundational Concepts for Digital Twins of Cyber-Physical Systems (2024).
- **hartmann_executable_2022** -- The Executable Digital Twin: merging the digital and the physics worlds (2022).
- **infante_integrating_2024** -- Integrating FMI and ML/AI models on the open-source digital twin framework OpenTwins (2024).
- **kamburjan_greenhousedt_2024** -- GreenhouseDT: An Exemplar for Digital Twins (2024).
- **kapteyn_probabilistic_2021** -- A probabilistic graphical model foundation for enabling predictive digital twins at scale (2021).
- **kritzinger_digital_2018** -- Digital Twin in manufacturing: A categorical literature review and classification (2018).
- **legaard_constructing_2023** -- Constructing neural network-based models for simulating dynamical systems (2023).
- **niederer_scaling_2021** -- Scaling digital twins from the artisanal to the industrial (2021).
- **rasheed_digital_2020** -- Digital Twin: Values, Challenges and Enablers From a Modeling Perspective (2020).
- **schweiger_empirical_2019** -- An empirical survey on co-simulation: Promising standards, challenges and research needs (2019).
- **thelen_augmented_2022** -- Augmented model-based framework for battery remaining useful life prediction (2022).
- **thelen_comprehensive_2022** -- A comprehensive review of digital twin — part 1: modeling and twinning enabling technologies (2022).
- **thelen_comprehensive_2022-1** -- A comprehensive review of digital twin—part 2: roles of uncertainty quantification and optimization, a battery digital twin, and perspectives (2022).
- **thelen_probabilistic_2024** -- Probabilistic machine learning for battery health diagnostics and prognostics—review and perspectives (2024).
- **torzoni_deep_2023** -- A Deep Neural Network, Multi-fidelity Surrogate Model Approach for Bayesian Model Updating in SHM (2023).
- **van_dinter_predictive_2022** -- Predictive maintenance using digital twins: A systematic literature review (2022).
- **wagner_data-driven_2025** -- Data-Driven Updating of Digital Twins through Experimental Measurements and Parametric Finite Element Model Optimization (2025).
- **wetter_spawn_2024-1** -- Spawn: coupling Modelica Buildings Library and EnergyPlus to enable new energy system and control applications (2024).
- **zhang_knowledge_2024** -- Knowledge Equivalence in Digital Twins of Intelligent Systems (2024).
