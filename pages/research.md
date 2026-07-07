---
hide:
  - navigation
  - toc
---

# Engineering Scalable and Trustworthy Digital Twin Platforms for Cyber-Physical Systems

## Abstract

Digital twins (DTs) are virtual replicas of physical entities referred to as Physical Twins
(PTs). DTs are becoming essential infrastructure for engineering and operating cyber-physical
systems (CPSs) across manufacturing, critical infrastructure, and energy domains. DT research
techniques are being used to create bespoke DTs for the oceans, the earth, and circular
manufacturing systems that are crucial to achieving sustainability goals. However, the effort
needed to build a DT from scratch remains prohibitively high, limiting adoption to
organisations with advanced software engineering resources. This research programme addresses
three interlocking challenges:
(1) the engineering of re-usable, composable DTs and the platforms that manage them throughout
their full life cycle;
(2) the principled integration of heterogeneous simulations into live DTs to enable predictive
and prescriptive capabilities, where the requirement for simulations may arise from a PT
operating in a factory or from a factory linked with its supply chain; and
(3) the establishment of secure, federated sharing of assets and services between organisations.
The unifying goal is an open, platform-level answer to the question: *how can digital twins be
built, operated, and shared efficiently and securely by a wide range of users?* The outcomes are
demonstrated through realistic case studies in structural health monitoring, district cooling,
and autonomous systems, and released as open-source software.

## Motivation

The widespread adoption of sensing, connectivity, and cloud computing has created the
technical prerequisites for digital twins. A DT acts as a persistent, software-based
counterpart to a physical twin (PT), maintaining continuous synchronisation and using
models and simulations to add operational value [[1]](#references). This value takes concrete forms:
predictive maintenance of offshore wind turbines, fault detection in structural health
monitoring [[2]](#references), adaptive control of manufacturing systems [[3]](#references), and management of autonomous
mobile robots.

Despite this potential, implementation remains a major undertaking. A practitioner must
assemble communication infrastructure, domain models, simulation tools, data pipelines,
and visualisation services — and do so repeatedly for each new system. This is especially
burdensome for Small and Medium-sized Enterprises (SMEs), who represent the largest
segment of the industrial base but typically lack the specialised expertise needed to
build DTs from scratch [[4]](#references). The **[Digital Twin as a Service (DTaaS)](https://github.com/into-cps-association/DTaaS)** platform has been
developed to tackle this problem directly, treating re-usability of DT assets (data,
models, functions, and tools) as the primary design principle [[4]](#references), [[5]](#references).

A second class of challenges arises once a DT is operational. Simulations are the engine
of DT intelligence: they support what-if analysis, model validation, and predictive
control. However, the integration of heterogeneous simulation environments into live DT
architectures is poorly supported by current tooling — most DT frameworks treat simulation
as an offline activity, disconnected from the run-time data loop. Enabling bidirectional,
low-latency simulation integration across diverse modelling paradigms requires a dedicated
simulation infrastructure. The *DT Simulation Bridge (DT-SB)* integrates distributed
simulations — from MATLAB dynamics models to AnyLogic agent-based simulations — into live
DTs [[6]](#references), and has also found application in a Manufacturing-as-a-Service ecosystem [[7]](#references).

A third challenge emerges when organisations want to pool DT assets across institutional
or organisational boundaries (dataspaces). DT assets may contain confidential models or
proprietary operational data; sharing them requires not only technical interoperability
but also security guarantees covering confidentiality, integrity, and access control.
Zero-trust security principles and federated identity management have been applied to
cloud platforms [[8]](#references) but have not been systematically integrated into DT platforms. Preliminary
results indicate that a federated DTaaS architecture enforcing secure asset sharing is
both technically feasible and practically useful [[9]](#references). Emerging Industry 5.0 standards further
extend the requirements of DT platforms with the potential integration of AI, cloud
services, and dataspaces [[10]](#references).

**Primary research question:**

> *How can digital twin platforms be engineered so that composable, simulation-capable,
> and securely shareable DTs are accessible to a wide range of users and organisations?*

**Sub-questions guiding the research programme:**

1. How can re-usable DT assets be defined, managed, and composed to reduce the time and
   expertise needed to deploy a DT across diverse application domains?
2. What architectural patterns and interaction protocols are needed to integrate
   heterogeneous, concurrent simulations into live DTs with minimal latency and coupling?
3. What mechanisms enable secure, federated sharing of DT assets and services across
   organisational boundaries while respecting data sovereignty?
4. How do these platform capabilities translate into measurable engineering value in
   real-world applications such as manufacturing supply chains, digital product passports
   (DPP), and autonomous systems?

## Methodology

The research combines platform engineering, formal modelling, and empirical evaluation in a *design science* tradition [[11]](#references): problems are extracted from real projects, artefacts are built to address them, and the artefacts are evaluated in context.

**Platform and Architecture Research.**
The DTaaS platform is the primary research vehicle. Its microservice architecture
separates concerns of asset management, DT life cycle orchestration, execution provisioning,
and user access. New capabilities — semantic lifting of DT configurations, hierarchical
DT composition, and federated asset registries — are designed against a consistent set
of stakeholder requirements drawn from the CP-SENS, SWiM, and Secure Collaboration
projects.

**Simulation Integration.**
The DT-SB is designed as a protocol-agnostic middleware that decouples DTs from specific
simulation tools. DTs managed inside the DTaaS platform, when coupled with distributed
simulators via the DT-SB, can thus achieve scalable, cross-domain operations.

**Security and Federation.**
Federated DT platforms are designed using zero-trust principles: no user, service,
or asset is trusted by default; every interaction is authenticated and authorised.
The secure collaboration architecture builds on OIDC and verifiable credentials to
control asset access across organisational boundaries. Threat modelling follows
established industry standards and is validated against case studies on emergency
management in crowd environments, for instance as part of the Aarhus Festival.

**Case Studies.**
Claims about platform capabilities are validated through a portfolio of case studies
with business partners. Current studies include: structural health monitoring of
bridges and offshore wind turbine foundations (CP-SENS), and district cooling systems in
Asian megacities (SWiM). Each case study contributes reusable assets to the shared DTaaS
asset library, reinforcing the platform's value for subsequent users.

**Open-Source Release.**
All platform components are released as open-source software under the
[INTO-CPS Association](https://into-cps.org), enabling independent validation, community
contributions, and adoption beyond the core research group.

## Expected Outcomes

1. A production-grade, open-source DTaaS platform supporting composable DTs with a
   well-defined asset model, lifecycle management, and scalable cloud deployment,
   validated across different application domains through industrial collaborations
   with regional partners.
2. A DT Simulation Bridge providing multi-protocol, heterogeneous simulation middleware
   that complements the DT and DTaaS platform.
3. A federated security architecture for DTaaS that enforces zero-trust asset sharing
   across organisational boundaries, with demonstrated application to SME industrial
   settings.
4. A body of peer-reviewed publications in journals and conferences (SIMULATION,
   IEEE CPS, ACM MODELS, Springer Engineering of Digital Twins) reporting validated
   results from the above artefacts and case studies.

## References

1. Michael Grieves. *Digital Twin: Manufacturing Excellence through Virtual Factory Replication.* White Paper, 2014.
2. Prasad Talasila, Dmitri Tcherniak, Anders M.D. Jensen, Swarup Mahato, Andreas Schörghofer-Queiroz, Martin D. Ulriksen, Giuseppe Abbiati, Peter Gorm Larsen, and Lars Damkilde. *Structural Health Monitoring of Engineering Structures Using Digital Twins: A Digital Twin Platform Approach.* In Proc. 11th Conference on Experimental Vibration Analysis of Civil Engineering Structures (EVACES 2025), Porto, Portugal, 2–4 July 2025.
3. Fei Tao, Q. Qi, L. Wang, and A.Y.C. Lee. *Digital Twins and Cyber-Physical Systems toward Smart Manufacturing and Industry 4.0: Correlation and Comparison.* *Engineering*, 5(4):653–661, 2019.
4. Prasad Talasila, Cláudio Gomes, Lars B. Vosteen, Hannes Iven, Martin Leucker, Santiago Gil, Peter H. Mikkelsen, Eduard Kamburjan, and Peter G. Larsen. *Composable Digital Twins on Digital Twin as a Service Platform.* *Simulation*, 101(3):287–311, 2025. <https://doi.org/10.1177/00375497241298653>
5. Prasad Talasila, Peter Høgh Mikkelsen, Santiago Gil, and Peter Gorm Larsen. *Realising Digital Twins.* In John Fitzgerald, Cláudio Gomes, and Peter Gorm Larsen (Eds.), *The Engineering of Digital Twins*, pp. 225–256. Springer International Publishing, Cham, 2024.
6. Marco Picone, Samuele Burattini, Marco Melloni, Prasad Talasila, Davide Ziglioli, Matteo Martinelli, Nicola Bicocchi, and Peter Gorm Larsen. *A Multi-Simulation Bridge for IoT Digital Twins.* In *Digital Twins Ecosystems and Applications*, 23rd IEEE International Conference on Pervasive Computing and Communications Workshops (PerCom 2026), Pisa, Italy, 16–20 March 2026.
7. Milan Vathoopan, Prasad Talasila, Jalil Boudjadar, Chresten Larsen, Nicola Bicocchi, Marco Picone, and Marco Melloni. *Orchestrating Distributed Simulations for Circular Manufacturing-as-a-Service Ecosystems.* In Proc. IEEE International Conference on Industrial Technology (ICIT 2026), Monterrey, Mexico, 4–6 March 2026.
8. Parwinder Singh, Michail J. Beliatis, and Mirko Presser. *Enabling Edge-Driven Dataspace Integration through Convergence of Distributed Technologies.* *Internet of Things*, 101087, 2024.
9. Mirgita Frasheri, Prasad Talasila, and Vanessa Scherma. *Towards Federated Digital Twin Platforms.* In Proc. International Workshop on Autonomous System Quality Assurance and Prediction with Digital Twins (ASQAP), co-located with ACM Foundations of Software Engineering 2026, 5–9 July 2026.
10. Parwinder Singh, Jan-Phillip Herrmann, Prasad Talasila, Sven Tackenberg, Michail J. Beliatis, Mirko Presser, and Peter Gorm Larsen. *AI-Human-Centric Industry 5.0: A Multimodal, Service-based, Edge-driven Digital Twin, AI Agents, and IoT Extended Reality.* *IEEE Transactions on Human-Machine Systems*, 2026. *Under Revision.*
11. Alan R. Hevner, Salvatore T. March, Jinsoo Park, and Sudha Ram. *Design Science in Information Systems Research.* *MIS Quarterly*, 28(1):75–105, 2004.
