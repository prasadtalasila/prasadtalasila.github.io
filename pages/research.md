---
hide:
  - navigation
  - toc
---

# Engineering Scalable and Trustworthy Digital Twin Platforms for Cyber-Physical Systems

## Abstract

Digital twins (DTs) are becoming essential infrastructure for engineering and operating
cyber-physical systems (CPSs) across manufacturing, civil engineering, and energy.
However, the effort needed to build a DT from scratch remains prohibitively high,
limiting adoption to organisations with advanced software engineering resources.
This research programme addresses three interlocking challenges:
(1) the engineering of re-usable, composable DT assets and the platforms that manage
them throughout their full life cycle;
(2) the principled integration of heterogeneous simulations into live DTs to enable
predictive and prescriptive capabilities; and
(3) the establishment of secure, federated collaboration between organisations sharing
DT assets and services.
The unifying goal is an open, platform-level answer to the question: *how can digital
twins be built, operated, and shared efficiently and securely by a wide range of users?*
The outcomes are demonstrated through realistic case studies in structural health
monitoring, district cooling, and autonomous systems, and released as open-source software.

## Motivation

The widespread adoption of sensing, connectivity, and cloud computing has created the
technical prerequisites for digital twins. A DT acts as a persistent, software-based
counterpart to a physical twin (PT), maintaining continuous synchronisation and using
models and simulations to add operational value. This value takes concrete forms:
predictive maintenance of offshore wind turbines, fault detection in structural health
monitoring, adaptive control of manufacturing systems, and management of autonomous
mobile robots.

Despite this potential, implementation remains a major undertaking. A practitioner must
assemble communication infrastructure, domain models, simulation tools, data pipelines,
and visualisation services — and do so repeatedly for each new system. This is especially
burdensome for Small and Medium-sized Enterprises (SMEs), who represent the largest
segment of the industrial base but typically lack the specialised expertise needed to
build DTs from scratch. The **[Digital Twin as a Service (DTaaS)]()** platform has been
developed to tackle this problem directly, treating re-usability of DT assets (data,
models, functions, and tools) as the primary design principle.

A second class of challenges arises once a DT is operational. Simulations are the engine
of DT intelligence: they support what-if analysis, model validation, and predictive
control. However, the integration of heterogeneous simulation environments into live DT
architectures is poorly supported by current tooling. Most DT frameworks treat simulation
as an offline activity, disconnected from the run-time data loop. Enabling bidirectional,
low-latency simulation integration across diverse modelling paradigms requires a dedicated
architectural element — the *DT Simulation Bridge*.

A third challenge emerges when organisations want to pool DT assets across institutional
or organisational boundaries. DT assets may contain confidential models or proprietary
operational data; sharing them requires not only technical interoperability but also
security guarantees covering confidentiality, integrity, and access control.

**Primary research question:**

> *How can digital twin platforms be engineered so that composable, simulation-capable,
> and securely shareable DTs are accessible to a wide range of users and organisations?*

**Sub-questions guiding the research programme:**

1. How can re-usable DT assets be defined, managed, and composed to reduce the time and
   expertise needed to deploy a DT across diverse application domains?
2. What architectural patterns and interaction protocols are needed to integrate
   heterogeneous, concurrent simulations into live DTs with minimal latency and coupling?
3. How can DevOps and DevSecOps principles be adapted to the hybrid digital-physical
   nature of DT development, specifically through the use of Mock Physical Twins?
4. What mechanisms enable secure, federated sharing of DT assets and services across
   organisational boundaries while respecting data sovereignty?
5. How do these platform capabilities translate into measurable engineering value in
   real-world applications such as structural health monitoring, district cooling,
   and autonomous systems?

## Methodology

The research combines platform engineering, formal modelling, and empirical evaluation in a *design science* tradition: problems are extracted from real projects, artefacts are built to address them, and the artefacts are evaluated in context.

**Platform and Architecture Research.**
The DTaaS platform is the primary research vehicle. Its microservice architecture
separates concerns of asset management, lifecycle orchestration, execution provisioning,
and user access. New capabilities — semantic lifting of DT configurations, hierarchical
DT composition, and federated asset registries — are designed against a consistent set
of stakeholder requirements drawn from the CP-SENS, SWiM, and Secure Collaboration
projects. Architecture decisions are documented using the C4 model and evaluated for
scalability on Kubernetes-hosted deployments.

**Simulation Integration.**
The DT Simulation Bridge is designed as a protocol-agnostic middleware that decouples
DTs from specific simulation tools. Three interaction patterns are supported:
batch simulation (one-shot scenario analysis), streaming simulation (continuous state
exchange), and physical twin simulation (virtual commissioning). Simulation agents for
MATLAB and AnyLogic are implemented and evaluated on commercial hardware.
Protocol overhead is measured empirically using MQTT, AMQP, and HTTP/2 to inform
deployment decisions.

**DevOps for Digital Twins.**
Mock Physical Twins (MockPTs) are formalised as software artefacts that emulate PT
communication interfaces at increasing fidelity levels: from pure interface stubs,
through trace-based replay, to full simulation. This progression allows DT development
to advance independently of hardware availability and provides a structured path for
continuous integration and deployment. The maturity levels are evaluated against the
DTOps framework through experiments in robotic arm commissioning and greenhouse management.

**Security and Federation.**
Federated DT platforms are designed using zero-trust principles: no user, service,
or asset is trusted by default; every interaction is authenticated and authorised.
The secure collaboration architecture builds on OAuth2 and verifiable credentials to
control asset access across organisational boundaries. Threat modelling follows
established industry standards and is validated against the CP-SENS security case
studies involving engineering SMEs.

**Case Studies.**
Claims about platform capabilities are validated through a portfolio of case studies
with engineering partners. Current studies include: structural health monitoring of
bridges and offshore wind turbine foundations (CP-SENS); district cooling systems in
Asian megacities (SWiM); runtime verification of autonomous mobile robots (DS-RT 2024);
and on-demand cardiac DTs demonstrating DevOps workflows. Each case study contributes
reusable assets to the shared DTaaS asset library, reinforcing the platform's value
for subsequent users.

**Open-Source Release.**
All platform components are released as open-source software under the INTO-CPS
Association, enabling independent validation, community contributions, and adoption
beyond the core research group.

## Expected Outcomes

1. A production-grade, open-source DTaaS platform supporting composable DTs with a
   well-defined asset model, lifecycle management, and scalable cloud deployment,
   validated across at least five distinct application domains.
2. A DT Simulation Bridge providing multi-protocol, heterogeneous simulation integration
   with sub-5 ms median overhead, accompanied by agents for at least two major simulation
   environments.
3. A DTOps framework that formally defines MockPT maturity levels and maps them to
   DevOps pipeline stages, enabling continuous integration and deployment of DTs
   independently of hardware availability.
4. A federated security architecture for DTaaS that enforces zero-trust asset sharing
   across organisational boundaries, with demonstrated application to SME industrial
   settings.
5. A body of peer-reviewed publications in journals and conferences (SIMULATION,
   IEEE CPS, ACM MODELS, Springer Engineering of Digital Twins) reporting validated
   results from the above artefacts and case studies.

## Prior Research

### Doctoral Research

My doctoral research is in the broad area of *Network Science*. My work is focused on
structural transformations of multidimensional (multiplex) networks — mathematical
transformations of graphs with multiple edges between vertices. This research has
applications in the domains of virtual networks, multi-modal transportation systems,
and online social networks.

Interested readers can go through the following published material related to my
doctoral thesis:

- [A conceptual summary of the thesis](https://www.dropbox.com/s/09161wst1mgcoek/cell_model.pdf?dl=1)
- [Official synopsis](https://www.dropbox.com/s/yhnse2q0s2w7vfr/synopsis.pdf?dl=1)
  and [thesis defence presentation](https://www.dropbox.com/s/lmppk1legd5yj4h/thesis_pres.pdf?dl=1)
- [Complete doctorate thesis](https://www.dropbox.com/s/nzaaey4010ixlev/thesis.pdf?dl=1)

### Storage Compression Research (2018–2020)

During 2018–20, I worked in the area of storage compression in distributed Internet of
Things (IoT) applications. Research results are available on the [Publications](publications.md) page.
