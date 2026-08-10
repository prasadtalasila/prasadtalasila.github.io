---
hide:
  - navigation
  - toc
---

# Digital Twins in an Ecosystem {#ch:dt-ecosystem}

**Disclaimer:** This thesis chapter has been generated using
[chitragupta](https://prasad.talasila.in/chitragupta).
Despite some potential for hallucination, the ideas communicated in this
thesis chapter are accurate. Please send your corrections and suggestions to
<prasad.talasila@gmail.com>

## From the isolated twin to the ecosystem {#sec:dte-intro}

Almost every canonical account of the digital twin describes a pair.
There is a physical thing, there is a virtual counterpart, and there is
a data connection that keeps the second faithful to the first. The
five-dimension formulation makes this explicit by naming physical
entity, virtual entity, services, data and connections as the
constituents of a twin (Tao et al. 2019), and the classification by
degree of data integration --- digital model, digital shadow, digital
twin --- turns the quality of that single connection into the criterion
that separates a twin from its weaker relatives (Kritzinger et al.
2018). The literature that grew around this pair is large, cross-domain
and, by now, reasonably well mapped (Semeraro et al. 2021; Dalibor et
al. 2022).

The pair, however, is not how twins are encountered in practice once
more than one of them exists. A production machine has a twin; so does
the line it sits in, the factory that contains the line, the product
moving through it, the worker operating it, and the energy contract that
prices its consumption. Each of these was very likely built by a
different team, using a different toolchain, against a different
metamodel, for a different purpose, and --- in the cases that matter
most --- under a different owner. The interesting questions stop being
about how well one twin mirrors one asset and start being about what
happens when many twins have to coexist, exchange information, depend on
one another and be assembled into something larger. That shift is what
this chapter calls the move from the twin to the *ecosystem*.

Several research communities arrived at this observation independently
and at roughly the same time. Ricci et al. named the phenomenon the Web
of Digital Twins and argued that the dominant view in the literature had
been "about the virtualisation of individual physical assets in a
closed-system perspective", proposing instead an open, distributed and
dynamic ecosystem of connected twins acting as an interoperable
service-oriented layer for the applications above it (Ricci et al.
2022). Giulianelli et al. put the engineering consequence bluntly:
ecosystems virtualise "a multitude of heterogeneous but interrelated
physical assets --- possibly belonging to different domains", and at the
centre of the resulting challenge sits the capability of integrating
twins developed with heterogeneous technologies (Giulianelli et al.
2024). Kuruppuarachchi et al. approached it from the
industrial-collaboration side, contrasting the *discrete* twin that "can
create value without depending on other systems" with the *composite*
twin that spans systems and business boundaries, and listing trust,
interoperability, governance, ownership, security and privacy as the
obstacles to the latter (Kuruppuarachchi, Rea, and McGibney 2022).
Michael et al. framed it as a systems-of-systems problem and observed
that research on twins as systems-of-systems "largely ignores reusing
digital twins" (Michael et al. 2022). Healthcare arrived at the same
place by a different road, with the Virtual Human Twin described as "a
distributed and collaborative infrastructure" assembled from
technologies, data and models contributed by many parties (Viceconti et
al. 2024).

The economic pressure behind this is not subtle. Building a single twin
requires marshalling a heterogeneous collection of models, data sources,
tools and services, and coordinating them is itself a substantial
engineering task (Talasila et al. 2024). If every twin is built from
scratch, the cost per twin does not fall, and the technology stays what
Niederer et al. called artisanal rather than industrial (Niederer et al.
2021). Reuse is the only route to scale, and reuse across teams and
organisations is precisely what an ecosystem is for.

### Research question {#sec:dte-rq}

This chapter is organised around one question:

> **RQ:** When digital twins cease to be isolated artefacts and become
> members of an ecosystem, what capabilities must the surrounding
> infrastructure provide --- with respect to *interoperability*,
> *networking*, *collaboration* and *hierarchical composition* --- and
> where do current approaches fall short of providing them?

The four terms in that list are not an arbitrary partition. They
correspond to four distinct relations that can hold between twins, and
the literature treats them as four distinct research programmes with
only partially overlapping membership. Interoperability concerns whether
two twins can *understand* each other. Networking concerns whether they
can *reach* each other, and at what cost in latency, bandwidth and
fidelity. Collaboration concerns whether they *may* exchange anything at
all, given that they answer to different owners with different
interests. Hierarchical composition concerns whether a set of twins can
be treated as a single twin at a coarser granularity, and what that new
twin is entitled to claim about the system it represents.

A recurring argument of the chapter is that these four are not
independent. Interoperability is usually posed as a data-representation
problem, but the choice of representation determines what can be
composed hierarchically; networking is usually posed as a performance
problem, but placement decisions across the edge--cloud continuum decide
which collaborations are even feasible; and collaboration is usually
posed as a trust problem, but the mechanisms that establish trust ---
policy-governed exchange, decentralised identity, provenance --- are
exactly the mechanisms that make a composed twin auditable. Treating
them separately is how ecosystems end up with four partial solutions
that do not fit together.

### A running example {#sec:dte-example}

Abstract arguments about ecosystems are easy to agree with and hard to
test, so the chapter uses one scenario throughout. It is deliberately
mundane, and deliberately spans ownership boundaries.

A contract manufacturer operates a production cell. The cell contains a
six-axis robot supplied by a robot vendor, a conveyor supplied by a
second vendor, and a vision system supplied by a third. The robot vendor
ships a twin of the robot as part of its service contract: it carries a
kinematic and dynamic model, wear estimates and a maintenance-scheduling
service, and the vendor considers the model itself commercially
sensitive. The manufacturer has built its own twin of the cell for
cycle-time optimisation, and its customer --- an automotive tier-one
supplier --- maintains a twin of the product being assembled, for
quality traceability. The building the cell sits in has a twin too,
operated by the facility owner, which prices energy and manages HVAC.
The grid operator would like a twin-level view of the building's
flexible load.

Nothing in this arrangement is exotic; variants of it appear throughout
the manufacturing and value-chain literature (Bhandal 2024; Gleich et
al. 2024; Altamiranda 2024). But every question the chapter asks has a
concrete form in it. Can the manufacturer's cell twin interpret the
robot twin's wear estimate, given that the two were described in
different formats (Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"})? What happens to the cell twin's
cycle-time predictions when the plant network drops for eleven seconds
and the robot twin's state goes stale
(Section [1.4](#sec:dte-network){reference-type="ref"
reference="sec:dte-network"})? On what basis does the robot vendor allow
its twin to answer the manufacturer's queries without disclosing the
model behind them, and what does the tier-one supplier's audit require
(Section [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"})? And is the manufacturer's cell twin ---
which aggregates the robot, conveyor and vision twins --- entitled to
claim it represents the cell, when the robot twin was validated only for
the vendor's own duty cycle
(Section [1.6](#sec:dte-hierarchy){reference-type="ref"
reference="sec:dte-hierarchy"})?

The last question is the one that recurs. It is the easiest to state,
the hardest to answer, and the one on which the practical value of the
other three depends.

### Structure of the chapter

Section [1.2](#sec:dte-terminology){reference-type="ref"
reference="sec:dte-terminology"} takes an inventory of the vocabulary,
because the same phenomenon currently travels under at least eight
names, and the differences between them are sometimes substantive and
sometimes not. Sections [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"}--[1.6](#sec:dte-hierarchy){reference-type="ref"
reference="sec:dte-hierarchy"} treat interoperability, networking,
collaboration and hierarchical composition in turn, each closing with an
assessment of what is settled and what is not.
Section [1.7](#sec:dte-synthesis){reference-type="ref"
reference="sec:dte-synthesis"} synthesises the four into a capability
model that answers the research question directly, and
Section [1.8](#sec:dte-agenda){reference-type="ref"
reference="sec:dte-agenda"} states the open problems that the synthesis
exposes, including three where the literature consulted for this chapter
offers no answer at all.

## What is a digital twin ecosystem? {#sec:dte-terminology}

### An inventory of names

The phenomenon described in
Section [1.1](#sec:dte-intro){reference-type="ref"
reference="sec:dte-intro"} is not short of labels. The following are all
in current use, and the differences between them repay attention.

##### Ecosystem of digital twins.

The most common term, used for a set of twins that are connected,
heterogeneous, and not centrally designed. It carries a deliberate
biological connotation: members join and leave, adapt to one another,
and are not all built by the same party. The healthcare operating suite
of Burattini et al. (2023) and the industrial ecosystem of Martinelli et
al. (2024) both use the word in this sense. An unfortunate collision
arises in environmental applications, where the twinned subject matter
is itself an ecological ecosystem, as in the Mar Menor lagoon platform
(Ye et al. 2024); the word then denotes the referent rather than the
software structure.

##### Web of Digital Twins (WoDT).

A stronger claim than "ecosystem": it commits to openness, to
hypermedia-style discovery, and to the twins forming a service-oriented
layer that applications --- including multi-agent systems --- run on top
of (Ricci et al. 2022). The commitment is architectural, not merely
metaphorical, and it has been carried into concrete toolchains
(Giulianelli et al. 2024, 2025).

##### Digital twin federation.

Emphasises that members retain autonomy and local authority while
participating in a larger whole. Marah and Challenger define it as "a
unified, integrated, interconnected, interoperable, and evolving
ecosystem that couples and comprises a group of digital twins" and
explicitly draw the lineage from the High Level Architecture for
distributed simulation (Marah and Challenger 2025). The term travels
well into settings with real organisational boundaries, such as urban
mobility assessment across municipalities (J. Li et al. 2025) and
federated scientific computing infrastructure ("[interTwin]{.nocase}:
Advancing Scientific Digital Twins Through AI, Federated Computing and
Data," n.d.).

##### Composite digital twin (CDT).

Used where the emphasis is on crossing business boundaries to produce a
single logical twin from parts belonging to different stakeholders, with
the attendant trust and governance requirements (Kuruppuarachchi, Rea,
and McGibney 2022).

##### Digital twin system-of-systems.

The software-engineering framing, in which twins of smaller systems
become parts of, or communicate with, twins of larger systems, and the
classical systems-of-systems concerns about independently developed and
independently evolving constituents apply (Michael et al. 2022;
Altamiranda 2024).

##### System of digital twins and services.

Human et al.'s term for an aggregated set of twins coupled with a
services network, chosen to keep the services --- not only the twins ---
in view as first-class design objects (Human, Basson, and Kruger 2023).

##### Digital twin continuum / fluid computing.

Framings that foreground the deployment substrate: twins spread across a
device--edge--cloud continuum, migrating as conditions change (Barbone
et al. 2024; Bedogni et al. 2025).

##### Digital twin constellation.

Used on the DTaaS platform for the orchestration of components toward
the business goals of a twin (Talasila et al. 2025).

Consortium glossaries have attempted to fix some of this vocabulary
("Glossary of Digital Twins by Digital Twin Consortium" n.d.), and the
Digital Twin Consortium's interoperability framework leans on the older
notions of the digital thread and of system-of-systems to do so
(Budiardjo and Migliori 2021). The attempts have not yet converged, and
the disagreement is not purely verbal: "federation" implies retained
sovereignty, "composite" implies a single resulting twin, and "web"
implies open discovery. A design that satisfies one does not
automatically satisfy the others.

::: {#tab:dte-terms}
  --------------------------------------------------------------------
  **Term**            **Distinctive commitment**   **Representative
                                                   source**
  ------------------- ---------------------------- -------------------
  Ecosystem of DTs    Heterogeneous, not centrally (Burattini et al.
                      designed, members join and   2023)
                      leave                        

  Web of Digital      Open discovery, hypermedia,  (Ricci et al. 2022)
  Twins               consumed by agents           

  DT federation       Members retain local         (Marah and
                      authority and sovereignty    Challenger 2025)

  Composite DT        Parts from several owners    (Kuruppuarachchi,
                      yield one logical twin       Rea, and McGibney
                                                   2022)

  DT                  Independently developed,     (Michael et al.
  system-of-systems   independently evolving       2022)
                      constituents                 

  System of DTs and   Services kept as first-class (Human, Basson, and
  services            design objects               Kruger 2023)

  DT continuum        Deployment substrate spans   (Barbone et al.
                      device, edge and cloud       2024)

  DT constellation    Components orchestrated      (Talasila et al.
                      toward one twin's business   2025)
                      goal                         
  --------------------------------------------------------------------

  : Terms in current use for collections of digital twins, and the
  commitment each one carries beyond "more than one twin".
:::

### A working definition

For the purposes of this chapter, a *digital twin ecosystem* is a set of
digital twins, built and operated by more than one party, that are able
to discover one another, exchange information with defined semantics,
and be combined into larger twins or twin-backed services without being
redesigned.

Three clauses in that definition are doing work. "More than one party"
rules out the case where a single vendor's platform hosts many twins
under one authority --- that is a large deployment, not an ecosystem,
and it does not face the governance problems that dominate
Section [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"}. "Defined semantics" rules out the case
where twins merely exchange bytes; as Combemale et al. observe, if
plug-and-play capability "stays at the technical level" then "we lose
the semantics" of what was exchanged (Combemale et al. 2025). "Without
being redesigned" is the economic clause: an assembly that requires each
participating twin to be rewritten is an integration project, not an
ecosystem.

### Four relations between twins

Given that definition, four relations can hold between two twins $A$ and
$B$, and they map onto the four sections that follow.

1.  **$A$ can interpret $B$'s description and state.** This is
    interoperability, and it decomposes further into technical,
    syntactic, semantic, pragmatic, dynamic and organisational levels, a
    stratification that Combemale et al. adopt from a systematic review
    of the field (Combemale et al. 2025).

2.  **$A$ can reach $B$ within a bounded time.** This is the network
    relation, and it is not a given: twins are real-time-coupled
    artefacts whose usefulness degrades, sometimes discontinuously, when
    the coupling does (Vaezi et al. 2022; Frasheri et al. 2023).

3.  **$A$ is permitted to obtain what it obtains from $B$, and $B$ is
    willing to provide it.** This is the collaboration relation, and it
    is where ownership, intellectual property, privacy and liability
    enter (Kuruppuarachchi, Rea, and McGibney 2022; Michael et al.
    2022).

4.  **$A$ and $B$, together, constitute a twin $C$ of the composite
    physical system.** This is the composition relation, and it is by
    some distance the least well understood of the four (Michael et al.
    2022; Gill et al. 2024).

### A disagreement worth stating early

Before proceeding it is worth naming a fault line that runs through the
whole of what follows. One camp treats ecosystem members as essentially
passive: a twin is a queryable, subscribable representation of an asset,
and the intelligence lives in the applications above the twin layer. The
hierarchical industrial ecosystem of Martinelli et al. (2024) is
explicit about this, describing the ecosystem as an abstraction layer
that decouples applications from the management of physical assets, and
the entity-based FIWARE approach of Franke and Koch (2025) shares the
assumption. The other camp treats ecosystem members as agents: Rivera et
al. argue that realising the vision "will demand DTs with increased
autonomy and enhanced ability to monitor, reason about, and react upon
relevant phenomena" and propose a reference framework for autonomic and
cooperating twins (Rivera et al. 2021); Esterle et al. go further and
consider autonomous cyber-physical systems that meet spontaneously and
use each other's twins to negotiate safe collaboration at runtime
(Esterle et al. 2021); and the Web of Digital Twins is designed
explicitly to be consumed by multi-agent systems (Ricci et al. 2022).

This is not a difference of emphasis. If twins are passive, the
ecosystem's hard problems are representation and plumbing. If twins are
agents, the hard problems include negotiation, conflict resolution and
mutual verification of claims, and none of the standardisation work
discussed in Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"} addresses those. The chapter returns to the
consequence in Section [1.7](#sec:dte-synthesis){reference-type="ref"
reference="sec:dte-synthesis"}.

## Interoperability {#sec:dte-interop}

### Levels, not a level

Interoperability is routinely spoken of as though it were a single
property, and it is not. The stratification used by Combemale et al.,
drawn from a systematic literature review, distinguishes technical,
syntactic, semantic, pragmatic, dynamic and organisational levels
(Combemale et al. 2025). The distinctions matter because different parts
of the literature solve different levels and then claim the word.

At the *technical* level, two twins share a transport: MQTT, HTTP,
OPC UA. This is essentially solved and is not where ecosystems fail. At
the *syntactic* level, they share a serialisation and a schema, so a
message can be parsed. At the *semantic* level, they agree on what the
parsed fields mean --- that this "temperature" is the coolant inlet and
is measured in degrees Celsius at the point of entry. At the *pragmatic*
level, they agree on what an exchange *does*: whether a setpoint message
is advice, a request or a command. At the *dynamic* level, they can
continue to agree while both sides evolve. At the *organisational*
level, the exchange is permitted, accountable and contractually
meaningful.

The bulk of the digital twin standards effort operates at the syntactic
and semantic levels.
Sections [1.4](#sec:dte-network){reference-type="ref"
reference="sec:dte-network"}
and [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"} argue that the dynamic and organisational
levels are where ecosystems actually break.

### The standards landscape

The Digital Twin Consortium's interoperability framework organises the
problem into seven concepts and insists on two requirements that
ecosystems inherit directly: information may originate in multiple
domains and so may carry "disparate syntactic and semantic standards
that must be correlated within a digital thread", and all information
flow must support both unidirectional and bi-directional exchange
(Budiardjo and Migliori 2021). The consortium's platform stack framework
similarly names a distinct service layer for integration,
interoperability and synchronisation (Heaton, n.d.). These are
statements of requirement rather than of mechanism; the mechanisms come
from the standards themselves.

The Asset Administration Shell (AAS) is the most institutionally
committed of these. Its specification adopts the design principle "keep
it simple but do not simplify if it affects interoperability" ("Asset
Administration Shell Part - 1," n.d.), and an ecosystem of open-source
implementations and submodel templates has grown around it (Jacoby et
al. 2023; Zhang et al. 2024). Zhang et al. examine the three AAS types
against requirements extracted from popular digital twin definitions and
conclude that each type contributes to engineering specific twin
components, while leaving open challenges for the implementation of
complete twins (Zhang et al. 2024). Ellwein et al.'s systematic mapping
shows that even the interaction semantics of the AAS --- what an
"active" shell is entitled to do, whether it makes decisions, exercises
skills or negotiates --- differ substantially across the sources that
describe it (Ellwein et al. 2025). A standard whose own community
disagrees about its interaction model is not yet a settled basis for
pragmatic-level interoperability.

Alongside the AAS sit the Digital Twins Definition Language, OPC UA, the
W3C Web of Things Thing Description, and, in manufacturing, the
ISO 23247 series with its reference architecture and its dedicated
Part 4 on information exchange (Larsen, Fitzgerald, and Gomes 2024; Shao
and Helu 2020; Shao, Frechette, and Srinivasan 2023). Surveys of the
standards relevant to twins run to catalogue length ("Summary of IoT,
and DT Standards," n.d.), and new open standards continue to be proposed
("Open Digital Twin Standard," n.d.) --- which is itself the finding:
the landscape is diverging, not converging.

### Three strategies, and the disagreement between them

Faced with plurality, three strategies are visible in the literature,
and they are genuinely in tension.

##### Converge.

Pick one metamodel and make everything speak it. This is the implicit
programme of the AAS community, reinforced by European procurement and
funding practice that increasingly asks proposals to align with relevant
international standards (Zhang et al. 2024). Its attraction is obvious;
its cost is that a single metamodel must serve manufacturing,
healthcare, civil infrastructure and energy, and the evidence that it
can is thin.

##### Bridge.

Accept plurality and build transformations between metamodels. Schmidt
et al. implement and test a generic DTDL-to-AAS mapping, observing first
that the existing standards "are incompatible with each other, i.e.,
they do not have the same (i) syntax, (ii) semantics" (Schmidt et al.
2023). Cavalieri and Salafia map DTDL to OPC UA on the same reasoning
(Cavalieri and Gambadoro 2023). Mattila et al. go a step further and
work backwards from the task: they identify the data fields actually
required for machine-to-machine collaboration, check which description
formats carry them, propose additions where they are missing, and
demonstrate the resulting mapping by controlling an industrial machine
described simultaneously as an AAS document and as a WoT Thing
Description (Mattila et al. 2025). That last piece of work is the most
convincing evidence in the corpus that bridging is tractable, precisely
because it is requirement-driven rather than schema-driven.

##### Leave.

Abandon the industrial metamodels for Web architecture. Ricci et
al. argue for hypermedia-style twins forming an open service layer
(Ricci et al. 2022); Burattini et al. propose a core ontology plus a
Linked Data approach to state management (Burattini et al. 2024); and
Giulianelli et al. turn this into an engineering method and then a
toolchain (Giulianelli et al. 2024, 2025). The strategy inherits the
Web's strengths in discovery and evolution --- and, as the Web of Things
survey documents, also inherits a fragmentation problem of its own that
a decade of standardisation has not resolved (Sciullo et al. 2022).

The three strategies are not reconcilable by simply doing all of them. A
bridge is only worth building if the endpoints are stable, which
converge denies and leave makes irrelevant. The most defensible position
visible in the evidence is that convergence should be pursued *within*
domains, where the modelled concepts genuinely coincide, and bridging
*across* them --- but this is an inference from the pattern of results,
not a claim any single source in this corpus makes.

### Does anyone follow the standards?

A finding that deserves more attention than it usually gets: Ferko et
al. compared digital twin architectures documented in the literature
against the ISO 23247 reference architecture and reported that, from
their own project experience, "standards, and in particular the
ISO 23247 standard, are not completely followed" (Ferko et al. 2023).
This sits uncomfortably beside the analyses that present the same
standard as a generic framework ready for specialisation to new sectors
(Shao, Frechette, and Srinivasan 2023; Shao 2021). Both can be true ---
a standard can be well designed and poorly adopted --- but for the
ecosystem argument the adoption fact dominates the design fact. An
ecosystem built on the assumption that members conform to a reference
architecture will encounter members that do not.

The same gap between specification and practice appears at platform
level. Pfeiffer et al.'s comparison of the modelling capabilities of
commercial twin platforms finds substantial differences in what those
platforms' metamodels can express (Jérôme Pfeiffer et al. 2022), and the
survey of open-source frameworks reaches a similar conclusion by case
study (Gil, Mikkelsen, et al. 2024). Efforts toward a unifying reference
model for twins of cyber-physical systems (Jerome Pfeiffer et al. 2025)
are a response to exactly this fragmentation.

### Semantics: ontologies and knowledge graphs

Semantic-level interoperability in this literature almost always means
ontologies. Steinmetz et al. build twin models supported by knowledge
graphs and note plainly that standards for such modelling are still
lacking (Steinmetz, Schroeder, Sulak, et al. 2022). Kamburjan et al. use
OWL asset models not merely to describe a twin but to drive its
reconfiguration, making the ontology load-bearing rather than
documentary (Kamburjan et al. 2022), and extend the idea to declarative
lifecycle management in which a twin's self-adaptive behaviour must
accommodate physical components that "evolve independently" (Kamburjan
et al. 2024). Semantic reflection lifts runtime state into a knowledge
graph connected to a domain ontology, giving other parties a principled
way to interrogate a twin's current condition rather than a bespoke API
(Kamburjan et al. 2025). Ontology also underpins the cognitive-twin
programme, where it is treated as the basis for unified knowledge
description and for integrating twins across lifecycle phases (Jinzhi et
al. 2022), and service-level description has been given the same
treatment so that twin services can be discovered and reused rather than
rediscovered (Oakes et al. 2024).

The limitation is well known and worth stating: ontologies solve
semantic agreement conditional on the parties having agreed on the
ontology, which relocates the coordination problem rather than
dissolving it. This is why the organisational level, discussed in
Section [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"}, cannot be treated as an afterthought.

### The digital thread: interoperability across time

There is a third axis besides "across twins" and "across levels", and it
is the one industrial practice cares about most: interoperability
*across the lifecycle*. The digital thread names the requirement that
information about an asset --- its design intent, as-built
configuration, service history, and eventual disposal --- remain linked
and interpretable as the asset moves between the organisations
responsible for each phase (Budiardjo and Migliori 2021). Commercial
platform vendors use precisely this framing when they describe twins
spanning an organisation's ecosystem (Khoshkenar and Nassereddine 2024).

For the running example, the thread is what lets the tier-one supplier
trace a quality defect back through the cell twin to the specific robot
trajectory and the specific batch of material. The obstacle is that each
phase typically has its own authoritative representation, and the
mappings between them are lossy. This is acute in civil infrastructure,
where the model architectures used at design time and those used in
operation come from different traditions entirely, and interoperability
solutions have had to be retrofitted (Naderi and Shojaei 2023). Michael
et al. list "different lifecycle representations of the original system"
as a distinct integration challenge for exactly this reason (Michael et
al. 2022), and lifecycle-spanning analytics architectures have been
proposed as a partial answer (Marosi et al. 2022). The IoT-side surveys
make the complementary point that the technical features and
architectural models available for twins differ by lifecycle stage as
well as by domain (Minerva, Lee, and Crespi 2020).

The thread also has a governance face. Once information persists across
lifecycle phases and organisations, provenance becomes a requirement
rather than a nicety --- which is one route by which the digital product
passport work (Gleich et al. 2024) and the dataspace work (Schmidt et
al. 2025) converge on the same infrastructure.

### Behavioural interoperability

Everything above concerns the exchange of *state*. Ecosystems also need
the exchange of *behaviour*: twin $A$'s simulation must be able to be
stepped in concert with twin $B$'s. This is co-simulation, and it has
its own mature standards discussion, including the treatment of hybrid
continuous-time and discrete-event coupling and the research needs that
remain open (Schweiger et al. 2019). Michael et al. list
"interoperability of models and simulation environments" as one of their
fourteen integration challenges (Michael et al. 2022), and it is
arguably the hardest of them: two Functional Mock-up Units may both
conform to FMI and still fail to compose because of algebraic loops,
incompatible step-size policies or mismatched assumptions about their
environment. Tooling has begun to address the mechanics --- CoFMPy
provides unified coordination of co-simulation, algebraic-loop
resolution and distributed data exchange for FMI-based twins (Friedrich
et al. 2025), and Spawn demonstrates automatic run-time coupling of
Modelica and EnergyPlus in the buildings domain (Wetter et al. 2024) ---
but the composition of *validity* claims, as opposed to the composition
of solvers, remains open and is taken up in
Section [1.6](#sec:dte-hierarchy){reference-type="ref"
reference="sec:dte-hierarchy"}.

### Interim assessment

Technical and syntactic interoperability are solved problems dressed up
as open ones. Semantic interoperability has good mechanisms and a
coordination problem. Pragmatic interoperability --- agreeing on what an
exchange commits the parties to --- is barely addressed outside the AAS
interaction-type debate (Ellwein et al. 2025) and Combemale et al.'s
warning about semantics lost to technical plug-and-play (Combemale et
al. 2025). Dynamic interoperability, meaning continued agreement under
independent evolution, is named as a challenge (Combemale et al. 2025;
Michael et al. 2022) and has, in the material surveyed here, no worked
solution.

## Networked digital twins {#sec:dte-network}

### Two readings of "network"

The phrase "digital twin network" is used for two different things, and
conflating them causes confusion.

The first reading, dominant in the software-engineering literature, is
*twins that are networked*: the communication fabric over which
ecosystem members exchange state and invoke each other's services. The
second, dominant in the communications literature, is *twins of
networks*: using a twin to model, predict and optimise a
telecommunications network itself. Wu et al.'s survey defines the
digital twin network in the second sense, as a network that "utilizes
digital twin technology to create the virtual twins of physical objects"
and realises co-evolution between physical and virtual spaces (Wu,
Zhang, and Zhang 2021), and the 6G literature follows suit (Khan et al.
2022; Apostolakis et al. 2023; Hakiri et al. 2024). This chapter is
primarily concerned with the first reading, but the second is not
irrelevant: a twin of the network is one of the more plausible ways to
give an ecosystem the ability to reason about its own communication
substrate.

Vaezi et al. bridge the two readings usefully. Their argument is that
"accurate real-time synchronization between the features at a physical
system and its DT is essential", from which it follows that "appropriate
networking support is a key component to enable future DT development
and applications" (Vaezi et al. 2022). The network is not infrastructure
that a twin happens to sit on; it is a determinant of whether the
artefact is a twin at all.

### Connectivity topologies

The most useful structural vocabulary for networked twins comes from
Schroeder et al., who specify six connectivity topologies: disconnected,
connected, embedded, aggregated, multi-device and combined (Schroeder et
al. 2021). The taxonomy is worth walking through because each topology
carries different ecosystem consequences.

A *disconnected* twin has no live link to its asset and is, in
Kritzinger et al.'s terms, a digital model (Kritzinger et al. 2018); it
can still participate in an ecosystem as a source of design-time
knowledge, but its state claims are not current. A *connected* twin
implements the live link explicitly. An *embedded* twin runs on the
physical device itself, which bounds its computational resources but
makes it robust to network loss. A *multi-device* twin serves several
devices from shared components --- one database, one web service --- so
that "despite having multiple representations, they are hosted on the
same computational resource and therefore can be used in a shared way"
(Schroeder et al. 2021). The *aggregated* topology, where a twin is
composed of other twins, is treated in
Section [1.6](#sec:dte-hierarchy){reference-type="ref"
reference="sec:dte-hierarchy"}. The *combined* topology mixes these.

Two observations follow. First, an ecosystem will contain members in all
six topologies simultaneously, and a member's topology determines what
interaction patterns it can support: an embedded twin on a constrained
device cannot serve arbitrary semantic queries. Second, the topology is
a deployment property, not a modelling property, which means it can
change at runtime --- and the ecosystem's interoperability contracts
must survive that change.

### The network as a first-class design constraint

Ecosystem architectures written from the software side often assume
reachable, low-latency, reliable communication. The literature that has
measured this does not.

Frasheri et al. address the situation directly: what should a twin do
when it and its physical counterpart get out of sync "as a result of
disturbances in the normal operational conditions \... e.g., due to
network degradation or temporary network drop"? Their answer is a
best-effort protocol comprising user notification, deliberate
*degradation of the twin to a digital shadow*, and explicit recovery
mechanisms to re-establish synchronisation (Frasheri et al. 2023). This
is, to my reading, the single most important idea in the networking part
of the ecosystem literature, and it is under-cited. It says that
fidelity is a runtime property that can be lost and regained, that a
twin should know when it has lost it, and --- crucially for ecosystems
--- that a twin's declared fidelity is something its peers must be able
to observe. An ecosystem in which members silently serve stale state as
though it were live is worse than one with no ecosystem at all, because
the staleness propagates through every composition built on top of it.

Upstream of the network sits the sensing layer, whose quality bounds
everything above it (Gomes et al. 2024). Downstream sits the messaging
substrate, where publish/subscribe systems dominate and where
benchmarking under realistic domain-based workloads is only beginning to
be done systematically (Badolato et al. 2026). Between them,
quantitative evidence about ecosystem middleware is scarce: Franke and
Koch's benchmark of FIWARE as a scalable interface for decentralised
twin ecosystems, complete with a candid account of its limitations, is
one of the few studies that measures rather than asserts (Franke and
Koch 2025).

### Placement across the edge--cloud continuum

Where a twin runs is an ecosystem-level decision, not a local one. Early
reference models assumed the cloud (Alam and El Saddik 2017). Current
work assumes a continuum, and treats placement as dynamic.

Bellavista et al. make the strongest version of this argument with an
entanglement-aware middleware: a "highly dynamic and distributed
ecosystem where containerized DTs co-evolve with an orchestration
middleware" that monitors and reconfigures deployments "in light of
application constraints, available resources, and the quality of
cyber-physical entanglement" (Bellavista et al. 2024b). The phrase
"quality of cyber-physical entanglement" is the important one: it makes
the tightness of the twin--asset coupling into a measurable quantity
that the infrastructure optimises, rather than an assumption the
architecture makes. The same group drives the twin lifecycle along the
cloud-to-edge continuum using microservices and serverless techniques
(Bellavista et al. 2024a), and related work frames the whole arrangement
as a continuum (Barbone et al. 2024) or as fluid computing (Bedogni et
al. 2025). Cloud-native platform substrates for many twins exist in open
form (Wermann and Wickboldt 2024), and twins have been distributed
across nodes within a single open-source framework (Infante et al.
2025).

The performance literature supplies the optimisation problems that
placement raises: twin-assisted task offloading with edge collaboration
(Liu et al. 2022), latency-aware and security-aware resource scheduling
for grid applications at the 5G edge (Zhou et al. 2022), and the broader
digital-twin edge-network programme toward 6G (Tang et al. 2022). These
are worth reading with a caution: they typically assume a single
optimising authority with global visibility, which is exactly what a
multi-party ecosystem does not have.

### Twins of networks, and why an ecosystem should want one

The second reading of "digital twin network" deserves more than the
dismissal it usually gets in software-engineering treatments. The
communications community has built a substantial programme around
twinning the network itself: Wu et al. survey the modelling,
communication, computing and data-processing technologies that make a
digital twin network work, and the application scenarios ---
manufacturing, aviation, healthcare, 6G networks, intelligent transport
and urban intelligence --- in which it has been proposed (Wu, Zhang, and
Zhang 2021). The 6G literature treats the twinned network as an
architectural component of the network itself, used for what-if
evaluation of configurations before they are applied to live
infrastructure (Khan et al. 2022; Apostolakis et al. 2023), and the
digital twin edge network programme couples this with multi-access edge
computing so that routing and resource-management decisions can be
studied against the twin rather than the network (Tang et al. 2022).
Comprehensive surveys tie the strand to the industrial IoT more broadly
(Hakiri et al. 2024; Y. Li and Zhang 2024).

The reason this matters for the present argument is C3 from
Section [1.7](#sec:dte-synthesis){reference-type="ref"
reference="sec:dte-synthesis"}.
Section [1.4](#sec:dte-network){reference-type="ref"
reference="sec:dte-network"} argued that an ecosystem member's fidelity
is a function of the communication path to its asset, and that peers
need to observe it. A twin of the ecosystem's own network is one
credible way to supply that observation: it can predict, rather than
merely report, that the path to a given twin is about to degrade, which
is the difference between a composed twin that flags reduced confidence
in advance and one that discovers the problem in its outputs. In the
running example, a twin of the plant network would let the
manufacturer's cell twin anticipate the eleven-second drop instead of
reacting to it.

No source consulted here closes this loop explicitly --- the
network-twin literature and the twin-ecosystem literature are largely
disjoint, citing different venues and different foundational papers.
Connecting them is a modest and, on the evidence assembled here,
unclaimed research opportunity.

### Interim assessment

Networking is the best-instrumented of the four concerns --- it inherits
decades of measurement culture --- and the worst-integrated with the
others. The concepts that matter most for ecosystems are observable
fidelity under degradation (Frasheri et al. 2023) and entanglement
quality as an orchestration signal (Bellavista et al. 2024b). Neither
appears in the interoperability standards discussed in
Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"}: no widely used twin description format has
a field for "how stale is this, and how would you know". That omission
is a concrete, fixable gap, and it is one of the recommendations in
Section [1.8](#sec:dte-agenda){reference-type="ref"
reference="sec:dte-agenda"}.

## Collaborative digital twins {#sec:dte-collab}

### Two senses of collaboration

"Collaborative digital twin" also covers two distinct ideas, and both
belong in this chapter.

The first is *twins that collaborate*: twins of separate assets
cooperating at runtime to achieve something neither could alone. Esterle
et al. describe autonomous cyber-physical systems that meet
spontaneously and use twin models "aiming to improve collaboration and
mutual safety", proposing an architecture for self-integration and
self-improvement at runtime (Esterle et al. 2021), and extend the idea
to autonomous reconfiguration (Esterle, Frasheri, and Larsen 2024).
Rivera et al. supply the reference framework for autonomic and
cooperating twins (Rivera et al. 2021). Gil et al. address the modelling
problem for cooperative robotic systems, extending information-modelling
approaches to include behaviour and using an ontology to express the
semantic relationships between the cooperating parts (Gil et al. 2023).

The second is *people and organisations collaborating through twins*:
the twin as the shared artefact across an organisational boundary. This
is Kuruppuarachchi et al.'s composite twin spanning "systems and
business boundaries" (Kuruppuarachchi, Rea, and McGibney 2022), and it
is where supply chains (Bhandal 2024), cross-municipal urban assessment
(J. Li et al. 2025) and cross-institutional health infrastructure
(Viceconti et al. 2024) live.

The two senses interact badly in one specific way. Runtime collaboration
between twins requires low-friction, low-latency interaction.
Cross-boundary collaboration requires policy checks, provenance and
consent. An ecosystem that supports both has to reconcile a microsecond
budget with a legal review, and the literature has not, in the sources
consulted, produced a design that does so convincingly.

### The non-functional obstacles

Kuruppuarachchi et al.'s list is the best short statement of what stops
collaboration: trust, interoperability, governance, ownership, security
and privacy (Kuruppuarachchi, Rea, and McGibney 2022). Michael et al.'s
longer enumeration of integration challenges reaches the same places
from the engineering side, naming protection of intellectual property,
privacy aspects of data, and rights and roles in the integrated twin as
distinct problems alongside the purely technical ones (Michael et al.
2022). Combemale et al. add prioritisation --- whose objective wins when
two twins disagree --- and place privacy controls and ethics controls in
the same list as the technical challenges rather than in an appendix
(Combemale et al. 2025).

Note what this implies for engineering practice. Intellectual-property
protection is not a matter of encrypting a channel; it is a matter of a
twin being able to answer useful questions about an asset without
revealing the model that answers them. That is a design constraint on
the twin's interface, which means it belongs in the interoperability
discussion of Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"} and is largely absent from it.

### Dataspaces and sovereignty

The most developed institutional answer to cross-organisational exchange
is the dataspace. Schmidt et al. describe dataspaces of the
International Data Spaces family as facilitating "sovereign,
policy-governed data exchange across organizations" and address the
integration of AAS-based twins into them --- reporting that this
integration "remains challenging due to manual processes,
synchronization issues, and varying implementations" (Schmidt et al.
2025). Singh et al. develop the edge-driven, cross-domain version of the
same idea, motivated by the observation that data generated in one
domain is routinely needed in another and that such sharing "across
edges" requires data sovereignty (Singh, Meratnia, et al. 2024; Singh,
Beliatis, and Presser 2024), and connect it to business ecosystem growth
and monetisation (Singh,.., et al. 2024). Gleich et al. give a worked
artefact: an AAS-based Digital Product Passport delivered as a Gaia-X
service, intended to demonstrate "safe and trustworthy data exchange and
collaboration across value chain" (Gleich et al. 2024). Interoperable
analytics reference architectures for twin-aided manufacturing address
the same boundary from the processing side (Marosi et al. 2022).

Dataspaces supply what
Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"} called organisational interoperability:
identity, policy, usage control and auditability. What they do not yet
supply is the pragmatic level. Knowing that party $B$ is permitted to
read a temperature series does not establish what $B$ is entitled to
conclude from it, or what obligation $A$ incurs if the series is wrong.

### Trust, identity and the disappearing perimeter

An ecosystem has no defensible perimeter. Every twin is a networked
service with a data path to a physical asset, which is why the security
surveys treat twins as a distinctive threat class rather than as
ordinary software (Alcaraz and Lopez 2022; Qureshi et al. 2025). Kulik
et al. ground this in the structural fact: twin-enabled systems "depend
on communications and networking, and so face a range of threats"
(Kulik, Kazemi, and Larsen 2024), and the IIoT surveys reach the same
conclusion from the deployment side (Xu et al. 2023).

Zero-trust architecture is the natural fit for a perimeterless ecosystem
(Rose et al. 2020), and its practical precondition in a multi-domain
setting is decentralised identity: authentication and authorisation that
work when resources are shared across a computing continuum spanning
administrative domains (Bernabé Murcia et al. 2025). Permissioned
distributed ledgers appear in the edge-twin literature as a mechanism
for accountable multi-party learning (Lu et al. 2021). Twins also cut
the other way, as instruments *for* security rather than only as targets
(Qureshi et al. 2025).

### Collaborating without sharing

A recurring pattern deserves separate mention because it dissolves
rather than manages the sharing problem: parties collaborate on a model
without exchanging the data that trained it. Lu et al. combine
communication-efficient federated learning with a permissioned
blockchain for twin edge networks (Lu et al. 2021). Kim et al. pose
federated twins as a scheduling problem and solve it with temporal graph
neural networks and deep reinforcement learning (Kim et al. 2025). Marah
and Challenger's federation roadmap positions this within a broader
programme of integration, collaboration and coordination across an
ecosystem of federated twins (Marah and Challenger 2025), and research
infrastructures have adopted federated computing and data as the
organising principle for scientific twins ("[interTwin]{.nocase}:
Advancing Scientific Digital Twins Through AI, Federated Computing and
Data," n.d.).

The limitation is scope. Federated learning solves collaborative
*statistical* model-building. It does nothing for the case where the
shared artefact is a physics-based model, a control decision, or a claim
about the current state of a shared asset --- which is most of what
twins do.

### Humans in the ecosystem

Industry 5.0 framings insist that people are ecosystem participants
rather than users of it. Villani et al. present a twin-driven
human-centric ecosystem on exactly this premise (Villani et al. 2025),
and Tóth et al. propose a collaboration architecture whose stated aim is
"integrating various innovative agents (human, AI, IoT, robot) in a
plant-level collaboration process through a generic semantic definition"
backed by a knowledge graph (Tóth et al. 2023). Human twins are being
built in clinical settings from multimodal data to support practitioner
decision-making (Azevedo et al. 2024), which raises the privacy and
consent questions of Section [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"} in their sharpest form. At the institutional
scale, the social and human dimensions of twin technologies in formal
and informal institutional settings are themselves a research object
(Alexandridis and LaFontaine 2024), and the urban-twin literature has
begun to push back on technocratic framings with a more sceptical
account of what city twins have actually achieved and what remains
conceptually unresolved (Bettencourt 2024). That scepticism is a useful
corrective to the ecosystem enthusiasm of the rest of the field.

### Collaboration at city and sector scale

The manufacturing setting of the running example understates the
governance problem, because a supply chain at least has contracts. Two
settings push further.

The first is the city. Urban twins are proposed as integrations across
mobility, energy, water, buildings and emergency services, each already
owned and operated by a different municipal or private body. Modular,
adaptive architectures have been developed to make such integration
tractable for real-time urban management (Herath et al. 2024), and
methods exist for accelerating twin development specifically in the
smart-city setting (McKee and Dokter 2024). Cross-municipal federation
has been given a functional architecture for mobility assessment (J. Li
et al. 2025), and the value case has been worked through for flexible
building loads coordinated across a network of participants (Reynolds,
Sabri, and Lee 2024). Against this, Bettencourt et al. supply the
necessary corrective: the conceptual challenges for urban twins are not
the same as the technical ones, and the field's achievements to date are
more modest than its rhetoric (Bettencourt 2024). The
institutional-settings literature makes the parallel point that twins
land in formal and informal institutions whose existing decision rights
they do not automatically respect (Alexandridis and LaFontaine 2024).

The second is the research infrastructure, where scientific twins are
being built on federated computing and federated data because no single
institution holds either ("[interTwin]{.nocase}: Advancing Scientific
Digital Twins Through AI, Federated Computing and Data," n.d.), and
where the health-domain equivalent is explicitly framed as a distributed
and collaborative infrastructure assembled from contributed
technologies, data and models (Viceconti et al. 2024).

What both settings share is that the ecosystem's governance predates its
technology. The participants already have decision rights, funding lines
and accountability structures, and the twin ecosystem must fit them
rather than replace them. This is the strongest practical argument for
the federation framing over the composite framing (Marah and Challenger
2025): it is the only one of the terms in
Table [1.1](#tab:dte-terms){reference-type="ref"
reference="tab:dte-terms"} that treats retained local authority as a
design requirement rather than an obstacle.

### The supply side: marketplaces and reuse

If ecosystems are justified by reuse, someone must supply the reusable
parts. The Change2Twin project provides the most concrete evidence
available here: a catalogue of twin enabling technologies documenting
the fragmentation of the tool landscape ("Tools and Libraries
Catalogue," n.d.), a marketplace design ("Change2Twin Marketplace
Design," n.d.), and a review of that marketplace in operation
("Change2Twin Marketplace Review," n.d.). HUBCAP demonstrated the
adjacent case for model-based design of cyber-physical systems,
providing collaboration infrastructure aimed specifically at small and
medium enterprises that cannot absorb the fixed costs alone (Larsen et
al. 2022). Value-creation studies show the same logic at the application
level, with buildings-as-batteries value realised through a
mass-customisation network of participants rather than by any single
twin (Reynolds, Sabri, and Lee 2024).

The commercial platform literature adopts the ecosystem word freely,
typically meaning an organisation-wide deployment connected by a digital
thread (Khoshkenar and Nassereddine 2024; Parle et al. 2024). That is
the weaker sense identified in
Section [1.2](#sec:dte-terminology){reference-type="ref"
reference="sec:dte-terminology"}: one authority, many twins.

### Interim assessment

Collaboration has the clearest problem statement of the four concerns
(Kuruppuarachchi, Rea, and McGibney 2022; Michael et al. 2022) and the
most institutional machinery pointed at it (Schmidt et al. 2025; Gleich
et al. 2024; Singh, Meratnia, et al. 2024). What it lacks is a
connection to the technical layers: dataspace policy is expressed over
datasets, whereas twins expose services, simulations and control
affordances, and no source consulted here expresses usage policy over
those. Section [1.8](#sec:dte-agenda){reference-type="ref"
reference="sec:dte-agenda"} returns to this.

## Hierarchical and composed digital twins {#sec:dte-hierarchy}

### Aggregation as a topology

The hierarchical case has a clean statement in Schroeder et al.'s
aggregated topology: "the digital twin is composed of other digital
twins \... It can be seen as a hierarchical system, where the parent
transparently provides all functionalities of its children, while
possibly adding more functionalities" (Schroeder et al. 2021). Their
worked example is the standard one: the twin of a production line
aggregates the twins of its machines and products, and the aggregation
of production lines yields the twin of the factory. They also note an
attractive practical consequence --- embedded twins with limited
on-device capability can be aggregated into a twin with greater
computational resources --- and a structural one, that a one-to-one
physical--cyber correspondence can be maintained at every level of the
hierarchy simultaneously.

Martinelli et al. build this out into a running industrial ecosystem
"exploiting twin relationships and hierarchies to build a digitalised
replica of the whole manufacturing system structure", with the stated
capabilities of data augmentation, actionability, navigability and
composability, and they measure the resource consumption their hierarchy
imposes (Martinelli et al. 2024). Human et al. approach the same object
as a design problem, giving a six-step framework for a system of
aggregated twins and services running from needs analysis through
physical-system decomposition and service allocation to verification and
validation (Human, Basson, and Kruger 2023). Both are valuable, and both
assume a single designing authority --- which returns the problem to
Section [1.5](#sec:dte-collab){reference-type="ref"
reference="sec:dte-collab"} as soon as the levels of the hierarchy
belong to different owners.

### Horizontal integration versus vertical composition

The most useful analytical distinction comes from Michael et al., who
separate *horizontal integration of digital twin parts* --- assembling
the models, services, data, APIs, access rights and views that make up
one twin at one level --- from *vertical composition of digital twins*
--- embedding a twin into a larger twin (Michael et al. 2022). Their
fourteen challenges divide accordingly, and several are worth restating
because they are usually skipped.

Horizontally, a single asset typically has multiple legitimate views: a
car twin "may feature a driver's view, a maintenance view, an insurance
view, and a producer's view", each with different models, services and
data (Michael et al. 2022). When several twins of the *same* asset
exist, composing them is subject to all the usual difficulties "with the
twist of all DTs of the same original system aspiring to be the true DT
for their respective (potentially overlapping) views". Multi-purpose
twin engineering addresses part of this at design time (Heithoff et al.
2024), and a product-line architecture for twins offers variability
management as an alternative to per-instance composition (Jérôme
Pfeiffer et al. 2023).

Vertically, the challenges are sharper. Michael et al. observe that a
twin being embedded may record data "on different levels of abstraction
or in incompatible granularity", that its services and behaviours "might
contradict the behaviors of the DT it should be embedded in", and that
the models themselves may be incompatible (Michael et al. 2022). To
these they add different lifecycle representations of the original
system, conflicting constraints and requirements, hierarchical
functional abstraction, composition of interfaces both twin-to-twin and
twin-to-CPS, and the interoperability of models and simulation
environments already discussed. Granularity is a first-class design
parameter here, and has been treated as such (Steinmetz, Schroeder,
Rodrigues, et al. 2022), as has the transition from simple to complex
twins across multiple scales and scenarios (Jia, Wang, and Zhang 2022).

Set that list beside Schroeder et al.'s statement that the parent
"transparently provides all functionalities of its children" (Schroeder
et al. 2021) and the tension is plain. Transparent provision presumes
that children's functionalities do not conflict, that their data are
commensurable, and that their behaviours compose. Michael et
al. document that in practice none of these holds (Michael et al. 2022).
The aggregated topology is a description of what one wants, not an
account of how to get it.

### What composition must preserve

The deeper question is what a composed twin is entitled to claim. If
twins $A$ and $B$ are each validated representations of their assets,
the twin $C$ formed from them is not thereby a validated representation
of the composite system. Two reasons stand out.

The first is emergence. Grieves and Vickers framed the twin's purpose
partly as mitigating "unpredictable, undesirable emergent behavior in
complex systems" (Grieves and Vickers 2017). Emergent behaviour is by
construction not present in the constituents, so a composition that
merely federates constituent models cannot exhibit it. Composing twins
to study emergence requires modelling the interactions, not just the
parts --- which is why behavioural interoperability
(Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"}) is load-bearing rather than incidental.

The second is validity. Each constituent twin was validated within an
assumed operating envelope. Composition can move a constituent outside
its envelope without anything in the composed system detecting it.
Combemale et al.'s worked example makes this concrete: two instances of
a room twin come to share a single irrigation tank, which "renders the
current level estimation of the predictive model invalid", forcing
evolution of the predictive model or the extraction of a separate tank
twin that must then be composed with both room twins (Combemale et al.
2025). The National Academies report on foundational research gaps
identifies verification, validation and uncertainty quantification for
twins as a first-order open problem (*Foundational Research Gaps and
Future Directions for Digital Twins* 2024); at ecosystem scale, where
constituents evolve independently and their validation evidence is often
proprietary, the problem is strictly harder and correspondingly less
studied.

No source consulted for this chapter provides a formal account of when
composition preserves validity, or a mechanism by which a composed twin
can compute its own confidence from its constituents'. That is a genuine
gap, and it is stated here as such rather than papered over.

### Composition over the lifecycle, and the reporting precondition

A composed twin is not assembled once. Its constituents are replaced,
upgraded and retired on schedules set by their owners, which makes
composition a lifecycle activity rather than a build step. The
systems-of-systems literature on asset lifecycle management makes this
explicit for capital-intensive industries, where asset twins must
survive decades of component turnover and where SoS concepts are
recruited precisely to handle constituents that change independently
(Altamiranda 2024). The declarative, ontology-driven approach to twin
lifecycle management is a direct response to constituents that "evolve
independently" (Kamburjan et al. 2024), and the multi-scale modelling
work addresses the related problem that a composite may need to be
examined at several granularities in different scenarios (Jia, Wang, and
Zhang 2022).

There is a precondition here that is easy to overlook. A twin can only
be reused in a composition by a party that did not build it if that
party can determine what it does, what it assumes and how it was
validated. Gil et al. argue for exactly this, proposing a systematic
reporting framework for twins and evaluating it on a cooperative
robotics case (Gil, Oakes, et al. 2024); complementary tooling makes
twin structure and provenance continuously visible to stakeholders
rather than documenting it once at handover (Fitzgerald, Gomes, and
Larsen 2024). In the running example, the robot vendor's twin is only
composable into the manufacturer's cell twin if it reports the duty
cycle over which it was validated --- and that report is the same
artefact the manufacturer would need in order to detect, later, that the
cell has drifted outside it. Documentation is usually treated as a
courtesy in this literature. For ecosystems it is a functional
requirement.

### Automating composition

If composition is to happen at ecosystem scale it cannot be manual. Gill
et al. propose a pipeline for automating the composition of twins within
systems-of-systems, and are explicit about the precondition: "a formal
semantic model is necessary, based on domain-specific standards, that
clearly and unambiguously describes all relevant DT information",
supported by a meta-model, with a top-level domain ontology selected to
integrate the constituents and ensure their interoperability (Gill et
al. 2024). Their pipeline includes automated generation of composed twin
behaviour from newly composed twin functions --- the step that
distinguishes genuine composition from mere aggregation of dashboards.

Amadeo et al. approach the same target from the Internet-of-Everything
side, observing that existing twin architectures are application-driven
verticals and that "a general-purpose modeling approach is lacking", and
proposing a user-centric composition model (Amadeo et al. 2024).
Kamburjan et al.'s declarative lifecycle management provides machinery
of a different kind: ontology-driven self-adaptation for twins whose
physical components "evolve independently" (Kamburjan et al. 2024),
which is the dynamic-interoperability problem of
Section [1.3](#sec:dte-interop){reference-type="ref"
reference="sec:dte-interop"} attacked at the composition layer.
Design-pattern catalogues supply the reusable structural vocabulary that
such automation needs to target (Tekinerdogan and Verdouw 2020).

### Platform support for composition

The platform literature has converged on reuse as the mechanism. Van
Schalkwyk and Isaacs argue the industrial case for the composable twin,
which "offers re-use of effort, accelerated time to results, general
applicability, and the dynamic range to address both simple and complex
issues within an enterprise at scale", paired with lean,
minimum-viable-product delivery practice (Schalkwyk and Isaacs 2023).
Talasila et al. give the platform realisation: a Digital Twin as a
Service platform that manages reusable assets --- models, data,
functions and tools --- and creates composable twins from them, then
makes those twins available as a service to other users, with asset
management, storage, compute provisioning, communication, monitoring and
execution as platform responsibilities and with two case studies as
evaluation (Talasila et al. 2025). The same lineage supplies the account
of why building a twin from scratch is expensive enough to justify the
machinery (Talasila et al. 2024). OpenTwins pursues compositional twins
in open source (Robles, Martín, and Díaz 2023) and has been extended to
distributed operation (Infante et al. 2025).

The broader as-a-service framing has an architecture reference model
(Aheleroff et al. 2021), cross-domain instantiations that report reduced
implementation effort and operating cost through open modelling
standards (Zech et al. 2024), arguments for platform openness as an
ecosystem precondition (Grübel et al. 2023), and now a survey that
organises platform requirements into a component-based taxonomy of core
and supportive components with associated performance metrics, while
noting that prior studies "mostly remain domain-specific, thereby
limiting the potential for generalisation" (Duran et al. 2026).

### Interim assessment

Hierarchical composition is where the ecosystem programme is furthest
from delivery. There is a good structural vocabulary (Schroeder et al.
2021; Tekinerdogan and Verdouw 2020), a good design method under single
authority (Human, Basson, and Kruger 2023), running hierarchical systems
(Martinelli et al. 2024), platform machinery for asset reuse (Talasila
et al. 2025; Robles, Martín, and Díaz 2023; Schalkwyk and Isaacs 2023),
and a clear statement of what automation would require (Gill et al.
2024). What is missing is semantics: an account of what a composed twin
means, when composition is legitimate, and how the composite's validity
relates to its constituents'.

## Synthesis: what an ecosystem demands of its infrastructure {#sec:dte-synthesis}

### Answering the research question

Reading the four concerns together, the capabilities an ecosystem
requires of its infrastructure can be stated as eight, each with a
maturity that the preceding sections have argued for.

##### C1 --- Identity and discovery.

Every twin needs a stable, resolvable identity and a discoverable
description, so that peers can find it without out-of-band coordination.
Web and hypermedia approaches provide this natively (Ricci et al. 2022;
Giulianelli et al. 2025); AAS registries provide it within their
ecosystem (Jacoby et al. 2023). Across ecosystems, decentralised
identity is the emerging answer (Bernabé Murcia et al. 2025). *Maturity:
good within ecosystems, weak across them.*

##### C2 --- Semantic description of state and capability.

A peer must be able to determine what a twin represents, what it can be
asked, and what its answers mean. Mechanisms are plentiful (Mattila et
al. 2025; Burattini et al. 2024; Steinmetz, Schroeder, Sulak, et al.
2022; Oakes et al. 2024); agreement on which to use is not (Schmidt et
al. 2023; Ferko et al. 2023). *Maturity: mechanisms good, coordination
poor.*

##### C3 --- Declared and observable fidelity.

A peer must be able to determine how current and how trustworthy a
twin's state is, including under degradation. The concept exists
(Frasheri et al. 2023) and orchestration middleware treats entanglement
quality as a signal (Bellavista et al. 2024b), but no widely used
description format carries it. *Maturity: recognised, not standardised.*

##### C4 --- Bounded, adaptive communication.

Reachability with known latency characteristics, and graceful behaviour
when those characteristics fail (Vaezi et al. 2022; Frasheri et al.
2023; Badolato et al. 2026). *Maturity: good, well instrumented.*

##### C5 --- Placement and orchestration across a continuum.

Deciding where each twin runs, and moving it when conditions change
(Bellavista et al. 2024b, 2024a; Wermann and Wickboldt 2024; Barbone et
al. 2024). *Maturity: good under one authority, unaddressed across
several.*

##### C6 --- Policy, sovereignty and accountability.

Expressing and enforcing what may be shared, with whom, for what
purpose, with an audit trail (Schmidt et al. 2025; Gleich et al. 2024;
Singh, Meratnia, et al. 2024; Rose et al. 2020). *Maturity:
institutionally strong for data, absent for services, simulations and
control affordances.*

##### C7 --- Behavioural composability.

Coupling constituent models so that a composed twin can simulate the
composite system (Schweiger et al. 2019; Friedrich et al. 2025; Wetter
et al. 2024; Gill et al. 2024). *Maturity: solver-level tooling good,
semantics absent.*

##### C8 --- Reusable assets and a supply side.

Models, data, functions and tools that can be found, licensed and
reassembled (Talasila et al. 2025; Schalkwyk and Isaacs 2023;
"Change2Twin Marketplace Review," n.d.; Larsen et al. 2022). *Maturity:
platforms exist, market thin.*

::: {#tab:dte-capabilities}
  ---------------------------------------------------------------------
       **Capability**      **Concern**        **Assessment**
  ---- ------------------- ------------------ -------------------------
  C1   Identity and        Interoperability   Good within an ecosystem,
       discovery                              weak across ecosystems

  C2   Semantic            Interoperability   Mechanisms good,
       description of                         coordination poor
       state and                              
       capability                             

  C3   Declared and        Network /          Recognised, not
       observable fidelity composition        standardised --- falls
                                              between concerns

  C4   Bounded, adaptive   Network            Good, well instrumented
       communication                          

  C5   Placement and       Network /          Good under one authority,
       orchestration       collaboration      unaddressed across
       across a continuum                     several

  C6   Policy, sovereignty Collaboration      Strong for data, absent
       and accountability                     for services and control

  C7   Behavioural         Composition        Solver-level tooling
       composability                          good, semantics absent

  C8   Reusable assets and Composition /      Platforms exist, market
       a supply side       collaboration      thin
  ---------------------------------------------------------------------

  : Capabilities an ecosystem demands of its infrastructure, the concern
  each principally serves, and an assessment of maturity based on the
  evidence reviewed in
  Sections [1.3](#sec:dte-interop){reference-type="ref"
  reference="sec:dte-interop"}--[1.6](#sec:dte-hierarchy){reference-type="ref"
  reference="sec:dte-hierarchy"}.
:::

The direct answer to the research question is therefore that the four
concerns are unevenly served. Networking (C4, C5) is the most mature.
Interoperability (C1, C2) has good mechanisms held back by a
coordination failure that is political rather than technical.
Collaboration (C6) has strong institutional machinery aimed at the wrong
granularity --- datasets rather than twin services. Composition (C7) has
tooling without semantics. And one capability, C3, falls between all
four and is claimed by none of them.

### The couplings between concerns

The claim in Section [1.1.1](#sec:dte-rq){reference-type="ref"
reference="sec:dte-rq"} that the four concerns are not independent can
now be made precise, and three couplings matter most.

*Fidelity couples networking to composition.* A composed twin's
trustworthiness is bounded by the least current of its constituents.
Since currency is a network property (Frasheri et al. 2023) and
composition is a modelling operation (Gill et al. 2024), the composition
layer must be able to read a network-level property that the description
formats do not expose. This is the concrete form of the C3 gap.

*Policy couples collaboration to interoperability.* Michael et al. list
intellectual-property protection and rights and roles as integration
challenges (Michael et al. 2022); dataspaces express policy over data
(Schmidt et al. 2025). Twins expose services and control affordances, so
the policy language and the interface description must be the same
artefact. They currently are not.

*Placement couples networking to collaboration.* Orchestration
middleware reconfigures deployment to preserve entanglement quality
(Bellavista et al. 2024b), but moving a twin can move data across a
jurisdictional or organisational boundary. Sovereignty constraints
(Singh, Meratnia, et al. 2024) are therefore constraints on the
placement optimiser, and none of the offloading and scheduling
formulations reviewed here (Liu et al. 2022; Zhou et al. 2022; Kim et
al. 2025) carries them.

### Passive layer or agent society, revisited

The disagreement flagged in
Section [1.2](#sec:dte-terminology){reference-type="ref"
reference="sec:dte-terminology"} determines which capabilities are
sufficient. Under the passive reading (Martinelli et al. 2024; Franke
and Koch 2025), C1--C8 are close to complete: twins publish,
applications consume, and intelligence is external. Under the agent
reading (Rivera et al. 2021; Esterle et al. 2021; Ricci et al. 2022), at
least two further capabilities are needed --- a means for twins to make
and evaluate commitments to one another, and a means to resolve
conflicting objectives, which Combemale et al. raise as prioritisation
(Combemale et al. 2025). Neither appears in any standard reviewed here.

The choice is not merely academic, because the two readings imply
different description formats. A passive twin needs to describe what it
*is*; an agent twin needs to describe what it will *do*, and under what
conditions it will refuse.

## Open problems {#sec:dte-agenda}

Six problems follow from the synthesis. The first three are supported by
explicit statements in the literature; the last three are gaps where the
corpus consulted for this chapter offers no answer, and they are
labelled as such rather than dressed in a citation that does not support
them.

##### P1 --- A fidelity field in twin descriptions.

Twin description formats should carry machine-readable currency and
confidence, so that peers and compositions can reason about staleness.
The concept is available (Frasheri et al. 2023; Bellavista et al.
2024b); the standards work is not done (Mattila et al. 2025; Zhang et
al. 2024).

##### P2 --- Usage policy over twin services, not only data.

Dataspace machinery should be extended from datasets to service
invocations, simulation runs and control affordances (Schmidt et al.
2025; Michael et al. 2022).

##### P3 --- Sovereignty-aware placement.

Orchestration objectives should include jurisdiction and ownership as
hard constraints alongside latency and cost (Bellavista et al. 2024b;
Singh, Meratnia, et al. 2024).

##### P4 --- Compositional validity (gap).

No source consulted here supplies a formal account of when composing
validated twins yields a valid composite, nor a mechanism for a composed
twin to derive its own uncertainty from its constituents'. Gill et
al. argue that a formal semantic model is necessary (Gill et al. 2024)
and the National Academies report names verification, validation and
uncertainty quantification as foundational gaps (*Foundational Research
Gaps and Future Directions for Digital Twins* 2024), but the
compositional result itself does not exist in this corpus.

##### P5 --- Liability and contract (gap).

The synced corpus contains no treatment of legal liability, contractual
allocation of risk, or regulatory status for cross-organisational twin
ecosystems. Given that composite twins are proposed for safety-relevant
decisions in manufacturing, health and urban management, this absence is
itself a finding.

##### P6 --- Comparative ecosystem benchmarks (gap).

Beyond the FIWARE benchmark (Franke and Koch 2025), the
publish/subscribe workload study (Badolato et al. 2026) and the resource
measurements accompanying one hierarchical prototype (Martinelli et al.
2024), there is little comparative measurement of ecosystem middleware.
Claims about scalability in this field are, for the most part,
architectural rather than empirical.

## Summary {#sec:dte-summary}

This chapter asked what changes when digital twins become members of an
ecosystem rather than isolated artefacts, and specifically what
interoperability, networking, collaboration and hierarchical composition
demand of the surrounding infrastructure.

The answer developed here is that the four demands are unequally met and
badly connected. Interoperability has a rich set of mechanisms ---
description standards, transformations between them, ontologies and
Linked Data --- and a coordination failure about which to use, with
three mutually incompatible strategies (converge, bridge, leave) in
simultaneous pursuit and evidence that the flagship reference
architecture is not in fact followed (Ferko et al. 2023). Networking is
the most mature concern and contributes the chapter's most
under-appreciated idea, that fidelity is a runtime property which can be
lost, observed and recovered (Frasheri et al. 2023). Collaboration has
the clearest problem statement (Kuruppuarachchi, Rea, and McGibney 2022;
Michael et al. 2022) and the most institutional machinery, aimed one
level too low: at datasets rather than at the services, simulations and
control affordances that twins actually expose. Hierarchical composition
has structural vocabulary (Schroeder et al. 2021), running systems
(Martinelli et al. 2024), platform support for asset reuse (Talasila et
al. 2025; Schalkwyk and Isaacs 2023) and a clear statement of what
automation would require (Gill et al. 2024), but no semantics for what a
composed twin is entitled to claim.

Two conclusions follow for the remainder of this thesis. First, the
ecosystem problem is not solved by any single layer: a platform that
gets reuse right but cannot express sovereignty, or a standard that gets
semantics right but cannot express staleness, will not compose into a
working ecosystem. Second, the capability that falls between all four
concerns --- declared, observable fidelity --- is both the cheapest to
fix and the one on which the credibility of every composition rests. It
is taken up in the chapters that follow.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {#refs .references .csl-bib-body .hanging-indent entry-spacing="0"}
::: {#ref-aheleroff_digital_2021 .csl-entry}
Aheleroff, Shohin, Xun Xu, Ray Y. Zhong, and Yuqian Lu. 2021. "Digital
Twin as a Service (DTaaS) in Industry 4.0: An Architecture Reference
Model." *Advanced Engineering Informatics* 47 (January): 101225.
<https://doi.org/10.1016/j.aei.2020.101225>.
:::

::: {#ref-alam_c2ps_2017 .csl-entry}
Alam, Kazi Masudul, and Abdulmotaleb El Saddik. 2017. "C2PS: A Digital
Twin Architecture Reference Model for the Cloud-Based Cyber-Physical
Systems." *IEEE Access* 5: 2050--62.
<https://ieeexplore.ieee.org/abstract/document/7829368/>.
:::

::: {#ref-alcaraz_digital_2022 .csl-entry}
Alcaraz, Cristina, and Javier Lopez. 2022. "Digital Twin: A
Comprehensive Survey of Security Threats." *IEEE Communications Surveys
& Tutorials* 24 (3): 1475--1503.
<https://doi.org/10.1109/COMST.2022.3171465>.
:::

::: {#ref-alexandridis_social_2024 .csl-entry}
Alexandridis, Kostas, and Michael LaFontaine. 2024. "Social and Human
Dimensions of Digital Twin Technologies in Formal and Informal
Institutional Settings." In *Digital Twin: Fundamentals and
Applications*, edited by Soheil Sabri, Kostas Alexandridis, and Newton
Lee, 211--36. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_10>.
:::

::: {#ref-altamiranda_system_2024 .csl-entry}
Altamiranda, Edmary. 2024. "A System of Systems Foundation for Digital
Asset Lifecycle Management." In *Digital Twin: Fundamentals and
Applications*, edited by Soheil Sabri, Kostas Alexandridis, and Newton
Lee, 59--87. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_4>.
:::

::: {#ref-amadeo_composing_2024 .csl-entry}
Amadeo, Marica, Claudio Marche, Giuseppe Ruggeri, Sara Ranjbaran, and
Michele Nitti. 2024. "Composing Digital Twins for Internet of Everything
Applications: A User-Centric Perspective." In *2024 IEEE International
Mediterranean Conference on Communications and Networking (MeditCom)*,
73--78. <https://doi.org/10.1109/MeditCom61057.2024.10621154>.
:::

::: {#ref-apostolakis_digital_2023 .csl-entry}
Apostolakis, Nikolaos, Livia Elena Chatzieleftheriou, Dario Bega, Marco
Gramaglia, and Albert Banchs. 2023. "Digital Twins for Next-Generation
Mobile Networks: Applications and Solutions." *IEEE Communications
Magazine* 61 (11): 80--86. <https://doi.org/10.1109/MCOM.001.2200854>.
:::

::: {#ref-noauthor_asset_nodate .csl-entry}
"Asset Administration Shell Part - 1." n.d.
:::

::: {#ref-azevedo_human_2024 .csl-entry}
Azevedo, Roger, Mary Jean Amon, Mindi Anderson, Sean Mondesire,
Francisco Guido-Sanz, Robert Sottilare, and Megan Wiedbusch. 2024.
"Human Digital Twins to Support Nurse Practitioners' Clinical
Decision-Making Using Multimodal Data: A Theoretical, Methodological,
and Analytical Framework." In *Digital Twin: Fundamentals and
Applications*, edited by Soheil Sabri, Kostas Alexandridis, and Newton
Lee, 149--72. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_7>.
:::

::: {#ref-badolato_psmark_2026 .csl-entry}
Badolato, Christian, Nathan Samson, Houssam Hajj Hassan, Chih-Kai Huang,
Georgios Bouloukakis, Primal Pappachan, and Roberto Yus. 2026. "PSMark:
A Distributed IoT Benchmark for Publish/Subscribe Under Domain-Based
Workloads." In *PerCom 2026-24th IEEE International Conference on
Pervasive Computing and Communications*.
<https://inria.hal.science/hal-05517145/>.
:::

::: {#ref-barbone_digital_2024 .csl-entry}
Barbone, Antonello, Samuele Burattini, Matteo Martinelli, Marco Picone,
Alessandro Ricci, and Antonio Virdis. 2024. "Digital Twin Continuum: A
Key Enabler for Pervasive Cyber-Physical Environments." In *2024 33rd
International Conference on Computer Communications and Networks
(ICCCN)*, 1--9. <https://doi.org/10.1109/ICCCN61486.2024.10637565>.
:::

::: {#ref-bedogni_fluid_2025 .csl-entry}
Bedogni, Luca, Marco Mamei, Marco Picone, Marcello Pietri, and Franco
Zambonelli. 2025. "Fluid Computing & Digital Twins for Intelligent
Interoperability in the IoT Ecosystem." *Future Generation Computer
Systems* 171 (October): 107855.
<https://doi.org/10.1016/j.future.2025.107855>.
:::

::: {#ref-bellavista_exploiting_2024 .csl-entry}
Bellavista, Paolo, Nicola Bicocchi, Mattia Fogli, Carlo Giannelli, Marco
Mamei, and Marco Picone. 2024a. "Exploiting Microservices and Serverless
for Digital Twins in the Cloud-to-Edge Continuum." *Future Generation
Computer Systems* 157 (August): 275--87.
<https://doi.org/10.1016/j.future.2024.03.052>.
:::

::: {#ref-bellavista_entanglement-aware_2024 .csl-entry}
---------. 2024b. "An Entanglement-Aware Middleware for Digital Twins."
*ACM Trans. Internet Things*, October.
<https://doi.org/10.1145/3699520>.
:::

::: {#ref-bernabe_murcia_decentralised_2025 .csl-entry}
Bernabé Murcia, José Manuel, Eduardo Cánovas, Jesús García-Rodríguez,
Alejandro M. Zarca, and Antonio Skarmeta. 2025. "Decentralised Identity
Management Solution for Zero-Trust Multi-Domain Computing Continuum
Frameworks." *Future Generation Computer Systems* 162 (January): 107479.
<https://doi.org/10.1016/j.future.2024.08.003>.
:::

::: {#ref-bettencourt_recent_2024 .csl-entry}
Bettencourt, Luís. 2024. "Recent Achievements and Conceptual Challenges
for Urban Digital Twins." *Nature Computational Science* 4 (March):
150--53. <https://doi.org/10.1038/s43588-024-00604-9>.
:::

::: {#ref-bhandal_conceptualising_2024 .csl-entry}
Bhandal, Rajinder. 2024. "Conceptualising the Application of Digital
Twins in Supply Chain Management: A Path Towards Supply Chain
Resilience." In *Digital Twin: Fundamentals and Applications*, edited by
Soheil Sabri, Kostas Alexandridis, and Newton Lee, 173--89. Cham:
Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_8>.
:::

::: {#ref-budiardjo_digital_2021 .csl-entry}
Budiardjo, Anto, and Doug Migliori. 2021. "Digital Twin System
Interoperability Framework." Tech. rep. Digital Twin Consortium, East
Lansing, Michigan.
<https://www.digitaltwinconsortium.org/pdf/Digital-Twin-System-Interoperability-Framework-12072021.pdf>.
:::

::: {#ref-burattini_ecosystem_2023 .csl-entry}
Burattini, Samuele, Sara Montagna, Angelo Croatti, Nicola Gentili,
Alessandro Ricci, Laura Leonardi, Serafino Pandolfini, and Sofia Tosi.
2023. "An Ecosystem of Digital Twins for Operating Room Management." In
*2023 IEEE 36th International Symposium on Computer-Based Medical
Systems (CBMS)*, 770--75.
<https://doi.org/10.1109/CBMS58004.2023.00317>.
:::

::: {#ref-burattini_towards_2024 .csl-entry}
Burattini, Samuele, Antoine Zimmermann, Marco Picone, and Alessandro
Ricci. 2024. "Towards Linked Data for Ecosystems of Digital Twins." In
*Proceedings of the ACM/IEEE 27th International Conference on Model
Driven Engineering Languages and Systems*, 332--37. MODELS Companion
'24. New York, NY, USA: Association for Computing Machinery.
<https://doi.org/10.1145/3652620.3688245>.
:::

::: {#ref-cavalieri_proposal_2023 .csl-entry}
Cavalieri, Salvatore, and Salvatore Gambadoro. 2023. "Proposal of
Mapping Digital Twins Definition Language to Open Platform
Communications Unified Architecture." *Sensors* 23 (February): 2349.
<https://doi.org/10.3390/s23042349>.
:::

::: {#ref-noauthor_change2twin_nodate-3 .csl-entry}
"Change2Twin Marketplace Design." n.d. Change2Twin project.
<https://www.change2twin.eu/about/deliverables/>.
:::

::: {#ref-noauthor_change2twin_nodate-2 .csl-entry}
"Change2Twin Marketplace Review." n.d. Change2Twin project.
<https://www.change2twin.eu/about/deliverables/>.
:::

::: {#ref-combemale_challenges_2025 .csl-entry}
Combemale, Benoît, Jörg Kienzle, Gunter Mussbacher, Pascal Archambault,
Jean-Michel Bruel, Loli Burgueño, Betty HC Cheng, Loek Cleophas, Gregor
Engels, and Damien Foures. 2025. "On the Challenges of Integrating
Digital Twins." In *2nd International Conference on Engineering Digital
Twins (EDTconf 2025)*. <https://inria.hal.science/hal-05221809/>.
:::

::: {#ref-dalibor_cross-domain_2022 .csl-entry}
Dalibor, Manuela, Nico Jansen, Bernhard Rumpe, David Schmalzing, Louis
Wachtmeister, Manuel Wimmer, and Andreas Wortmann. 2022. "A Cross-Domain
Systematic Mapping Study on Software Engineering for Digital Twins."
*Journal of Systems and Software* 193 (November): 111361.
<https://doi.org/10.1016/j.jss.2022.111361>.
:::

::: {#ref-duran_toward_2026 .csl-entry}
Duran, Kubra, Lal Verda Cakir, Yagmur Yigit, Khayal Huseynov, Sushmitha
Ram Kusu, Mehmet Ali Ertürk, and Berk Canberk. 2026. "Toward Digital
Twin-as-a-Service (DTaaS) Platforms: A Survey on Architecture, Design
Requirements, and Performance Metrics." *IEEE Communications Surveys &
Tutorials* 28: 1845--78. <https://doi.org/10.1109/COMST.2025.3635582>.
:::

::: {#ref-ellwein_rethinking_2025 .csl-entry}
Ellwein, Carsten, David Dietrich, Nicolai Maisch, Rebekka Neumann, Samed
Ajdinović, Armin Lechler, and Andreas Wortmann. 2025. "Rethinking Asset
Administration Shell Communication Types: A Systematic Mapping Study and
Portfolio-Based Classification." *Production Engineering* 20 (December).
<https://doi.org/10.1007/s11740-025-01378-3>.
:::

::: {#ref-esterle_autonomous_2024 .csl-entry}
Esterle, Lukas, Mirgita Frasheri, and Peter Gorm Larsen. 2024.
"Autonomous Reconfiguration Enabled by Digital Twins." In *The
Engineering of Digital Twins*, edited by John Fitzgerald, Cláudio Gomes,
and Peter Gorm Larsen, 345--62. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0_14>.
:::

::: {#ref-esterle_digital_2021 .csl-entry}
Esterle, Lukas, Cláudio Gomes, Mirgita Frasheri, Henrik Ejersbo, Sven
Tomforde, and Peter G. Larsen. 2021. "Digital Twins for Collaboration
and Self-Integration." In *2021 IEEE International Conference on
Autonomic Computing and Self-Organizing Systems Companion (ACSOS-C)*,
172--77. IEEE. <https://ieeexplore.ieee.org/abstract/document/9599295/>.
:::

::: {#ref-ferko_standardisation_2023 .csl-entry}
Ferko, Enxhi, Alessio Bucaioni, Patrizio Pelliccione, and Moris Behnam.
2023. "Standardisation in Digital Twin Architectures in Manufacturing."
In *2023 IEEE 20th International Conference on Software Architecture
(ICSA)*, 70--81. <https://doi.org/10.1109/ICSA56044.2023.00015>.
:::

::: {#ref-fitzgerald_engineering_2024 .csl-entry}
Fitzgerald, John, Cláudio Gomes, and Peter Gorm Larsen, eds. 2024. *The
Engineering of Digital Twins*. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0>.
:::

::: {#ref-committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024 .csl-entry}
*Foundational Research Gaps and Future Directions for Digital Twins*.
2024. Washington, D.C.: National Academies Press.
<https://doi.org/10.17226/26894>.
:::

::: {#ref-franke_fiware_2025 .csl-entry}
Franke, Kai, and Tobias Koch. 2025. "FIWARE as a Scalable Digital Twin
Interface for DT Ecosystems: Benchmark and Limitations." In *2025 IEEE
International Conference on Pervasive Computing and Communications
Workshops and Other Affiliated Events (PerCom Workshops)*, 134--39.
<https://doi.org/10.1109/PerComWorkshops65533.2025.00054>.
:::

::: {#ref-frasheri_addressing_2023 .csl-entry}
Frasheri, Mirgita, Henrik Ejersbo, Casper Thule, Cláudio Gomes, Jakob
Levisen Kvistgaard, Peter Gorm Larsen, and Lukas Esterle. 2023.
"Addressing Time Discrepancy Between Digital and Physical Twins."
*Robotics and Autonomous Systems* 161 (March): 104347.
<https://doi.org/10.1016/j.robot.2022.104347>.
:::

::: {#ref-friedrich_cofmpy_2025 .csl-entry}
Friedrich, Corentin, Andrés Lombana, Jérôme Fasquel, Charlie Schlick,
Nora Bennani, and Mouhcine Mendil. 2025. "CoFMPy: A Python Framework for
Rapid Prototyping of FMI-Based Digital Twins." In *The 2nd International
Conference on Engineering Digital Twins*.
<https://hal.science/hal-05326255/>.
:::

::: {#ref-gil_survey_2024 .csl-entry}
Gil, Santiago, Peter H. Mikkelsen, Cláudio Gomes, and Peter G. Larsen.
2024. "Survey on Open‐source Digital Twin Frameworks--A Case Study
Approach." *Software: Practice and Experience* 54 (6): 929--60.
<https://doi.org/10.1002/spe.3305>.
:::

::: {#ref-gil_modeling_2023 .csl-entry}
Gil, Santiago, Peter H. Mikkelsen, Daniella Tola, Casper Schou, and
Peter G. Larsen. 2023. "A Modeling Approach for Composed Digital Twins
in Cooperative Systems." In *2023 IEEE 28th International Conference on
Emerging Technologies and Factory Automation (ETFA)*, 1--8.
<https://doi.org/10.1109/ETFA54631.2023.10275601>.
:::

::: {#ref-gil_toward_2024 .csl-entry}
Gil, Santiago, Bentley Oakes, Cláudio Gomes, Mirgita Frasheri, and Peter
Larsen. 2024. "Toward a Systematic Reporting Framework for Digital
Twins: A Cooperative Robotics Case Study." *SIMULATION*, August.
<https://doi.org/10.1177/00375497241261406>.
:::

::: {#ref-gill_toward_2024 .csl-entry}
Gill, Milapji Singh, Jingxi Zhang, Andreas Wortmann, and Alexander Fay.
2024. "Toward Automating the Composition of Digital Twins Within
System-of-Systems." In *2024 IEEE 29th International Conference on
Emerging Technologies and Factory Automation (ETFA)*, 1--4.
<https://doi.org/10.1109/ETFA61755.2024.10710740>.
:::

::: {#ref-giulianelli_engineering_2024 .csl-entry}
Giulianelli, Andrea, Samuele Burattini, Andrei Ciortea, and Alessandro
Ricci. 2024. "Engineering Interoperable Ecosystems of Digital Twins: A
Web-Based Approach." In *Proceedings of the ACM/IEEE 27th International
Conference on Model Driven Engineering Languages and Systems*, 476--85.
MODELS Companion '24. New York, NY, USA: Association for Computing
Machinery. <https://doi.org/10.1145/3652620.3688263>.
:::

::: {#ref-giulianelli_hwodt_2025 .csl-entry}
---------. 2025. "HWoDT Framework: A Toolchain to Build Interoperable
Digital Twin Ecosystems." *SoftwareX* 31 (September): 102275.
<https://doi.org/10.1016/j.softx.2025.102275>.
:::

::: {#ref-gleich_asset_2024 .csl-entry}
Gleich, Kevin, Sebastian Behrendt, Moritz Hörger, Martin Benfer, and
Gisela Lanza. 2024. "An Asset Administration Shell-Based Digital Product
Passport as a Gaia-X Service." *Procedia CIRP*, 10th CIRP Conference on
Assembly Technology and Systems (CIRP CATS 2024), 127 (January):
224--29. <https://doi.org/10.1016/j.procir.2024.07.039>.
:::

::: {#ref-noauthor_glossary_nodate-1 .csl-entry}
"Glossary of Digital Twins by Digital Twin Consortium." n.d. *Digital
Twin Consortium*. Accessed May 27, 2024.
<https://www.digitaltwinconsortium.org/glossary/glossary/>.
:::

::: {#ref-gomes_sensing_2024 .csl-entry}
Gomes, Cláudio, Daniel Enrique Lucani Rötter, Alexandros Iosifidis, Hao
Feng, Henrik Ejersbo, and Mirgita Frasheri. 2024. "Sensing and
Communication of Data from the Physical Twin." In *The Engineering of
Digital Twins*, edited by John Fitzgerald, Cláudio Gomes, and Peter Gorm
Larsen, 147--71. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0_7>.
:::

::: {#ref-grieves_digital_2017 .csl-entry}
Grieves, Michael, and John Vickers. 2017. "Digital Twin: Mitigating
Unpredictable, Undesirable Emergent Behavior in Complex Systems." In
*Transdisciplinary Perspectives on Complex Systems: New Findings and
Approaches*, edited by Franz-Josef Kahlen, Shannon Flumerfelt, and
Anabela Alves, 85--113. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-319-38756-7_4>.
:::

::: {#ref-grubel_outlining_2023 .csl-entry}
Grübel, Jascha, Carlos Vivar Rios, Chenyu Zuo, Sabrina Ossey, Robin M.
Franken, Milos Balac, Yanan Xin, Kay W. Axhausen, Martin Raubal, and
Oksana Riba-Grognuz. 2023. "Outlining the Open Digital Twin Platform."
In *2023 IEEE Smart World Congress (SWC)*, 1--3.
<https://doi.org/10.1109/SWC57546.2023.10448743>.
:::

::: {#ref-hakiri_comprehensive_2024 .csl-entry}
Hakiri, Akram, Aniruddha Gokhale, Sadok Ben Yahia, and Nedra Mellouli.
2024. "A Comprehensive Survey on Digital Twin for Future Networks and
Emerging Internet of Things Industry." *Computer Networks* 244 (May):
110350. <https://doi.org/10.1016/j.comnet.2024.110350>.
:::

::: {#ref-heaton_platform_nodate .csl-entry}
Heaton, Linda. n.d. "Platform Stack Architectural Framework: An
Introductory Guide."
:::

::: {#ref-heithoff_model-based_2024 .csl-entry}
Heithoff, Malte, Nico Jansen, Judith Michael, Florian Rademacher, and
Bernhard Rumpe. 2024. "Model-Based Engineering of Multi-Purpose Digital
Twins in Manufacturing." In *Digital Twin: Fundamentals and
Applications*, edited by Soheil Sabri, Kostas Alexandridis, and Newton
Lee, 89--126. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_5>.
:::

::: {#ref-herath_smart_2024 .csl-entry}
Herath, Manoj, Maira Alvi, Roberto Minerva, Hrishikesh Dutta, Noel
Crespi, and Syed Mohsan Raza. 2024. "Smart City Digital Twins: A Modular
and Adaptive Architecture for Real-Time Data-Driven Urban Management."
In *2024 20th International Conference on Network and Service Management
(CNSM)*, 1--7. IEEE.
<https://ieeexplore.ieee.org/abstract/document/10814627/?casa_token=b3lXuilhxVEAAAAA:WzY-AJNhNhitSev00P-EOsrAct0Rzn5kMII2--FmI-MWhs7zhfPeakDTrSPTTaTbbfDKj8c>.
:::

::: {#ref-human_design_2023 .csl-entry}
Human, C., Anton Basson, and Karel Kruger. 2023. "A Design Framework for
a System of Digital Twins and Services." *Computers in Industry* 144
(January): 103796. <https://doi.org/10.1016/j.compind.2022.103796>.
:::

::: {#ref-infante_distributed_2025 .csl-entry}
Infante, Sergio, Julia Robles, Cristian Martín, Bartolomé Rubio, and
Manuel Díaz. 2025. "Distributed Digital Twins on the Open-Source
OpenTwins Framework." *Advanced Engineering Informatics* 64 (March):
102970. <https://doi.org/10.1016/j.aei.2024.102970>.
:::

::: {#ref-noauthor_intertwin_nodate .csl-entry}
"[interTwin]{.nocase}: Advancing Scientific Digital Twins Through AI,
Federated Computing and Data." n.d.
:::

::: {#ref-jacoby_open-source_2023 .csl-entry}
Jacoby, Michael, Michael Baumann, Tino Bischoff, Hans Mees, Jens Müller,
Ljiljana Stojanovic, and Friedrich Volz. 2023. "Open-Source
Implementations of the Reactive Asset Administration Shell: A Survey."
*Sensors* 23 (May): 5229. <https://doi.org/10.3390/s23115229>.
:::

::: {#ref-jia_simple_2022 .csl-entry}
Jia, Wenjie, Wei Wang, and Zhenzu Zhang. 2022. "From Simple Digital Twin
to Complex Digital Twin Part I: A Novel Modeling Method for Multi-Scale
and Multi-Scenario Digital Twin." *Advanced Engineering Informatics* 53
(August): 101706. <https://doi.org/10.1016/j.aei.2022.101706>.
:::

::: {#ref-jinzhi_exploring_2022 .csl-entry}
Jinzhi, Lu, Yang Zhaorui, Xiaochen Zheng, Wang Jian, and Kiritsis
Dimitris. 2022. "Exploring the Concept of Cognitive Digital Twin from
Model-Based Systems Engineering Perspective." *The International Journal
of Advanced Manufacturing Technology* 121 (August).
<https://doi.org/10.1007/s00170-022-09610-5>.
:::

::: {#ref-kamburjan_declarative_2024 .csl-entry}
Kamburjan, Eduard, Nelly Bencomo, Silvia Lizeth Tapia Tarifa, and Einar
Broch Johnsen. 2024. "Declarative Lifecycle Management in Digital
Twins." In *Proceedings of the ACM/IEEE 27th International Conference on
Model Driven Engineering Languages and Systems*, 353--63. Linz Austria:
ACM. <https://doi.org/10.1145/3652620.3688248>.
:::

::: {#ref-kamburjan_digital_2022 .csl-entry}
Kamburjan, Eduard, Vidar Klungre, Rudolf Schlatte, Silvia Lizeth Tapia
Tarifa, David Cameron, and Einar Broch Johnsen. 2022. "Digital Twin
Reconfiguration Using Asset Models." In, 71--88.
<https://doi.org/10.1007/978-3-031-19762-8_6>.
:::

::: {#ref-hinchey_semantic_2025 .csl-entry}
Kamburjan, Eduard, Andrea Pferscher, Rudolf Schlatte, Riccardo Sieve,
Silvia Lizeth Tapia Tarifa, and Einar Broch Johnsen. 2025. "Semantic
Reflection and Digital Twins: A Comprehensive Overview." In *The
Combined Power of Research, Education, and Dissemination*, edited by
Mike Hinchey and Bernhard Steffen, 15240:129--45. Cham: Springer Nature
Switzerland. <https://doi.org/10.1007/978-3-031-73887-6_11>.
:::

::: {#ref-khan_digital-twin-enabled_2022 .csl-entry}
Khan, Latif U., Walid Saad, Dusit Niyato, Zhu Han, and Choong Seon Hong.
2022. "Digital-Twin-Enabled 6G: Vision, Architectural Trends, and Future
Directions." *IEEE Communications Magazine* 60 (1): 74--80.
<https://doi.org/10.1109/MCOM.001.21143>.
:::

::: {#ref-khoshkenar_exploring_2024 .csl-entry}
Khoshkenar, Amin, and Hala Nassereddine. 2024. *Exploring Digital Twin
Platforms Across Industries*. <https://doi.org/10.22260/ISARC2024/0119>.
:::

::: {#ref-kim_federated_2025 .csl-entry}
Kim, Young-Jin, Hanjin Kim, Beomsu Ha, and Won-Tae Kim. 2025. "Federated
Digital Twins: A Scheduling Approach Based on Temporal Graph Neural
Network and Deep Reinforcement Learning." *IEEE Access*.
<https://ieeexplore.ieee.org/abstract/document/10843696/>.
:::

::: {#ref-kritzinger_digital_2018 .csl-entry}
Kritzinger, Werner, Matthias Karner, Georg Traar, Jan Henjes, and
Wilfried Sihn. 2018. "Digital Twin in Manufacturing: A Categorical
Literature Review and Classification." *Ifac-PapersOnline* 51 (11):
1016--22.
<https://www.sciencedirect.com/science/article/pii/S2405896318316021>.
:::

::: {#ref-kulik_security_2024 .csl-entry}
Kulik, Tomas, Zahra Kazemi, and Peter Gorm Larsen. 2024. "Security and
Privacy-Related Issues in a Digital Twin Context." In *The Engineering
of Digital Twins*, edited by John Fitzgerald, Cláudio Gomes, and Peter
Gorm Larsen, 313--44. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0_13>.
:::

::: {#ref-kuruppuarachchi_architecture_2022 .csl-entry}
Kuruppuarachchi, Pasindu, Susan Rea, and Alan McGibney. 2022. "An
Architecture for Composite Digital Twin Enabling Collaborative Digital
Ecosystems." In *2022 IEEE 25th International Conference on Computer
Supported Cooperative Work in Design (CSCWD)*, 980--85. IEEE.
<https://ieeexplore.ieee.org/abstract/document/9776073/>.
:::

::: {#ref-larsen_engineering_2024 .csl-entry}
Larsen, Peter Gorm, John Fitzgerald, and Cláudio Gomes. 2024.
"Engineering Digital Twins for Cyber-Physical Systems." In *The
Engineering of Digital Twins*, edited by John Fitzgerald, Cláudio Gomes,
and Peter Gorm Larsen, 3--17. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0_1>.
:::

::: {#ref-larsen_hubcap_2022 .csl-entry}
Larsen, Peter Gorm, Hugo Daniel Macedo, John Fitzgerald, Holger Pfeifer,
Martin Benedikt, Stefano Tonetta, Angelo Marguglio, et al. 2022.
"HUBCAP: A Novel Collaborative Approach to Model-Based Design
of Cyber-Physical Systems." In *Simulation and Modeling Methodologies,
Technologies and Applications*, edited by Mohammad S. Obaidat, Tuncer
Oren, and Floriano De Rango, 90--110. Cham: Springer International
Publishing. <https://doi.org/10.1007/978-3-030-84811-8_5>.
:::

::: {#ref-li_digital_2025 .csl-entry}
Li, Jingjun, Jascha Grübel, Ali Nadi, Maaike Snelder, Bart van Arem, and
Jie Gao. 2025. "Digital Twin Federation for Urban Mobility Assessment: A
Functional Architecture for Low-Car Transformations in the Netherlands."
<https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5370630>.
:::

::: {#ref-li_digital_2024 .csl-entry}
Li, Yushuai, and Yan Zhang. 2024. "Digital Twin for Industrial
Internet." *Fundamental Research* 4 (1): 21--24.
<https://doi.org/10.1016/j.fmre.2023.01.005>.
:::

::: {#ref-liu_digital-twin-assisted_2022 .csl-entry}
Liu, Tong, Lun Tang, Weili Wang, Qianbin Chen, and Xiaoping Zeng. 2022.
"Digital-Twin-Assisted Task Offloading Based on Edge Collaboration in
the Digital Twin Edge Network." *IEEE Internet of Things Journal* 9 (2):
1427--44. <https://doi.org/10.1109/JIOT.2021.3086961>.
:::

::: {#ref-lu_communication-efficient_2021 .csl-entry}
Lu, Yunlong, Xiaohong Huang, Ke Zhang, Sabita Maharjan, and Yan Zhang.
2021. "Communication-Efficient Federated Learning and Permissioned
Blockchain for Digital Twin Edge Networks." *IEEE Internet of Things
Journal* 8 (4): 2276--88. <https://doi.org/10.1109/JIOT.2020.3015772>.
:::

::: {#ref-marah_re-engineering_2025 .csl-entry}
Marah, Hussein, and Moharram Challenger. 2025. "(Re-)Engineering Digital
Twins Towards Federation: Vision and Roadmap." In *Leveraging
Applications of Formal Methods, Verification and Validation. Software
Engineering Methodologies*, edited by Tiziana Margaria and Bernhard
Steffen, 60--81. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-75387-9_5>.
:::

::: {#ref-marosi_interoperable_2022 .csl-entry}
Marosi, Attila Csaba, Márk Emodi, Ákos Hajnal, Róbert Lovas, Tamás Kiss,
Valerie Poser, Jibinraj Antony, et al. 2022. "Interoperable Data
Analytics Reference Architectures Empowering Digital-Twin-Aided
Manufacturing." *Future Internet* 14 (4): 114.
<https://doi.org/10.3390/fi14040114>.
:::

::: {#ref-martinelli_hierarchical_2024 .csl-entry}
Martinelli, Matteo, Jingxi Zhang, Ann-Kathrin Splettstoßer, Marco
Picone, Marco Lippi, and Andreas Wortmann. 2024. "Hierarchical Digital
Twin Ecosystem for Industrial Manufacturing Scenarios." In *2024 50th
Euromicro Conference on Software Engineering and Advanced Applications
(SEAA)*, 56--63. <https://doi.org/10.1109/SEAA64295.2024.00018>.
:::

::: {#ref-mattila_interoperability_2025 .csl-entry}
Mattila, Joel, Riku Ala-Laurinaho, Juuso Autiosalo, and Kari Tammi.
2025. "Interoperability of Digital Twins for Automation With Digital
Twin Schema." *IEEE Access* 13: 200595--608.
<https://doi.org/10.1109/ACCESS.2025.3633736>.
:::

::: {#ref-mckee_discs_2024 .csl-entry}
McKee, David, and Dennis Dokter. 2024. "DISCS: An Approach for
Accelerating the Development of Digital Twins for Smart Cities." In
*Digital Twin: Fundamentals and Applications*, edited by Soheil Sabri,
Kostas Alexandridis, and Newton Lee, 31--58. Cham: Springer Nature
Switzerland. <https://doi.org/10.1007/978-3-031-67778-6_3>.
:::

::: {#ref-michael_integration_2022 .csl-entry}
Michael, Judith, Jérôme Pfeiffer, Bernhard Rumpe, and Andreas Wortmann.
2022. "Integration Challenges for Digital Twin Systems-of-Systems." In
*Proceedings of the 10th IEEE/ACM International Workshop on Software
Engineering for Systems-of-Systems and Software Ecosystems*, 9--12.
SESoS '22. New York, NY, USA: Association for Computing Machinery.
<https://doi.org/10.1145/3528229.3529384>.
:::

::: {#ref-minerva_digital_2020 .csl-entry}
Minerva, Roberto, Gyu Myoung Lee, and Noel Crespi. 2020. "Digital Twin
in the IoT Context: A Survey on Technical Features, Scenarios, and
Architectural Models." *Proceedings of the IEEE* 108 (10): 1785--1824.
<https://ieeexplore.ieee.org/abstract/document/9120192/>.
:::

::: {#ref-naderi_digital_2023 .csl-entry}
Naderi, Hossein, and Alireza Shojaei. 2023. "Digital Twinning of Civil
Infrastructures: Current State of Model Architectures, Interoperability
Solutions, and Future Prospects." *Automation in Construction* 149
(May): 104785. <https://doi.org/10.1016/j.autcon.2023.104785>.
:::

::: {#ref-niederer_scaling_2021 .csl-entry}
Niederer, Steven A., Michael S. Sacks, Mark Girolami, and Karen Willcox.
2021. "Scaling Digital Twins from the Artisanal to the Industrial."
*Nature Computational Science* 1 (5): 313--20.
<https://doi.org/10.1038/s43588-021-00072-5>.
:::

::: {#ref-oakes_towards_2024 .csl-entry}
Oakes, Bentley, Claudio Gomes, Eduard Kamburjan, Giuseppe Abbiati, Elif
Ecem Bas, and Sebastian Engelsgaard. 2024. "Towards Ontological
Service-Driven Engineering of Digital Twins." In *Proceedings of the
ACM/IEEE 27th International Conference on Model Driven Engineering
Languages and Systems*, 464--69. MODELS Companion '24. New York, NY,
USA: Association for Computing Machinery.
<https://doi.org/10.1145/3652620.3688261>.
:::

::: {#ref-noauthor_open_nodate .csl-entry}
"Open Digital Twin Standard." n.d. Jascha Gruebel.
<https://github.com/odtp-org>.
:::

::: {#ref-parle_comparative_2024 .csl-entry}
Parle, Dattatraya, Gaurav Sharma, Niharika Anand, Noel Padgaonkar, David
Stoddart, and David Malley. 2024. "A Comparative Analysis for Harnessing
Digital Twin Platforms for Net-Zero Manufacturing." *IEEE Access* PP
(August). <https://doi.org/10.1109/ACCESS.2024.3447475>.
:::

::: {#ref-pfeiffer_modeling_2022 .csl-entry}
Pfeiffer, Jérôme, Daniel Lehner, Andreas Wortmann, and Manuel Wimmer.
2022. "Modeling Capabilities of Digital Twin Platforms-Old Wine in New
Bottles?" *J. Object Technol.* 21 (3): 3--1.
<https://raw.githubusercontent.com/awortmann/awortmann.github.io/master/downloads/paper/Modeling_Capabilities_of_Digital_Twin_Platforms_-_Old_Wine_in_New_Bottles.pdf>.
:::

::: {#ref-pfeiffer_towards_2023 .csl-entry}
---------. 2023. "Towards a Product Line Architecture for Digital
Twins." In *2023 IEEE 20th International Conference on Software
Architecture Companion (ICSA-C)*, 187--90.
<https://doi.org/10.1109/ICSA-C57050.2023.00049>.
:::

::: {#ref-pfeiffer_towards_2025 .csl-entry}
Pfeiffer, Jerome, Jingxi Zhang, Benoit Combemale, Judith Michael,
Bernhard Rumpe, Manuel Wimmer, and Andreas Wortmann. 2025. "Towards a
Unifying Reference Model for Digital Twins of Cyber-Physical Systems."
arXiv. <https://doi.org/10.48550/arXiv.2507.04871>.
:::

::: {#ref-qureshi_survey_2025 .csl-entry}
Qureshi, Abdul Rehman, Adrián Asensio, Muhammad Imran, Jordi Garcia, and
Xavi Masip-Bruin. 2025. "A Survey on Security Enhancing Digital Twins:
Models, Applications and Tools." *Computer Communications* 238 (June):
108158. <https://doi.org/10.1016/j.comcom.2025.108158>.
:::

::: {#ref-reynolds_digital_2024 .csl-entry}
Reynolds, John, Soheil Sabri, and Benjamin Lee. 2024. "Digital Twins for
Creating Value Through 'Buildings as Batteries' Using a Mass
Customization Network." In *Digital Twin: Fundamentals and
Applications*, edited by Soheil Sabri, Kostas Alexandridis, and Newton
Lee, 191--209. Cham: Springer Nature Switzerland.
<https://doi.org/10.1007/978-3-031-67778-6_9>.
:::

::: {#ref-ricci_web_2022 .csl-entry}
Ricci, Alessandro, Angelo Croatti, Stefano Mariani, Sara Montagna, and
Marco Picone. 2022. "Web of Digital Twins." *ACM Transactions on
Internet Technology* 22 (4): 1--30. <https://doi.org/10.1145/3507909>.
:::

::: {#ref-rivera_forging_2021 .csl-entry}
Rivera, Luis F., Miguel Jiménez, Norha M. Villegas, Gabriel Tamura, and
Hausi A. Müller. 2021. "The Forging of Autonomic and Cooperating Digital
Twins." *IEEE Internet Computing* 26 (5): 41--49.
<https://ieeexplore.ieee.org/abstract/document/9325563/>.
:::

::: {#ref-robles_opentwins_2023 .csl-entry}
Robles, Julia, Cristian Martín, and Manuel Díaz. 2023. "OpenTwins: An
Open-Source Framework for the Development of Next-Gen Compositional
Digital Twins." *Computers in Industry* 152 (August): 104007.
<https://doi.org/10.1016/j.compind.2023.104007>.
:::

::: {#ref-rose_zero_2020 .csl-entry}
Rose, Scott, Oliver Borchert, Stu Mitchell, and Sean Connelly. 2020.
"Zero Trust Architecture." National Institute of Standards; Technology.
<https://doi.org/10.6028/NIST.SP.800-207>.
:::

::: {#ref-van_schalkwyk_achieving_2023 .csl-entry}
Schalkwyk, Pieter van, and Dan Isaacs. 2023. "Achieving Scale Through
Composable and Lean Digital Twins." In *The Digital Twin*, edited by
Noel Crespi, Adam T. Drobot, and Roberto Minerva, 153--80. Cham:
Springer International Publishing.
<https://doi.org/10.1007/978-3-031-21343-4_6>.
:::

::: {#ref-schmidt_integration_2025 .csl-entry}
Schmidt, Carlos, Friedrich Volz, Ljiljana Stojanovic, and Holger Kett.
2025. "Integration Approaches for Digital Twins in Dataspaces." *Applied
Sciences* 15 (21): 11623. <https://doi.org/10.3390/app152111623>.
:::

::: {#ref-schmidt_increasing_2023 .csl-entry}
Schmidt, Carlos, Friedrich Volz, Ljiljana Stojanovic, and Gerhard
Sutschet. 2023. "Increasing Interoperability Between Digital Twin
Standards and Specifications: Transformation of DTDL to AAS." *Sensors*
23 (September): 7742. <https://doi.org/10.3390/s23187742>.
:::

::: {#ref-schroeder_digital_2021 .csl-entry}
Schroeder, Greyce N., Charles Steinmetz, Ricardo N. Rodrigues, Achim
Rettberg, and Carlos E. Pereira. 2021. "Digital Twin Connectivity
Topologies." *IFAC-PapersOnLine*, 17th IFAC Symposium on Information
Control Problems in Manufacturing INCOM 2021, 54 (1): 737--42.
<https://doi.org/10.1016/j.ifacol.2021.08.086>.
:::

::: {#ref-schweiger_empirical_2019 .csl-entry}
Schweiger, G., C. Gomes, G. Engel, I. Hafner, J. Schoeggl, A. Posch, and
T. Nouidui. 2019. "An Empirical Survey on Co-Simulation: Promising
Standards, Challenges and Research Needs." *Simulation Modelling
Practice and Theory* 95 (September): 148--63.
<https://doi.org/10.1016/j.simpat.2019.05.001>.
:::

::: {#ref-sciullo_survey_2022 .csl-entry}
Sciullo, Luca, Lorenzo Gigli, Federico Montori, Angelo Trotta, and Marco
Felice. 2022. "A Survey on the Web of Things." *IEEE Access* 10
(January): 1--1. <https://doi.org/10.1109/ACCESS.2022.3171575>.
:::

::: {#ref-semeraro_digital_2021 .csl-entry}
Semeraro, Concetta, Mario Lezoche, Hervé Panetto, and Michele Dassisti.
2021. "Digital Twin Paradigm: A Systematic Literature Review."
*Computers in Industry* 130 (September): 103469.
<https://doi.org/10.1016/j.compind.2021.103469>.
:::

::: {#ref-shao_use_2021 .csl-entry}
Shao, Guodong. 2021. "Use Case Scenarios for Digital Twin Implementation
Based on ISO 23247." *NIST*, May.
<https://www.nist.gov/publications/use-case-scenarios-digital-twin-implementation-based-iso-23247>.
:::

::: {#ref-shao_analysis_2023 .csl-entry}
Shao, Guodong, Simon Frechette, and Vijay Srinivasan. 2023. "An Analysis
of the New ISO 23247 Series of Standards on Digital Twin Framework for
Manufacturing." In. American Society of Mechanical Engineers Digital
Collection. <https://doi.org/10.1115/MSEC2023-101127>.
:::

::: {#ref-shao_framework_2020 .csl-entry}
Shao, Guodong, and Moneer Helu. 2020. "Framework for a Digital Twin in
Manufacturing: Scope and Requirements." *Manufacturing Letters* 24
(April): 105--7. <https://doi.org/10.1016/j.mfglet.2020.04.004>.
:::

::: {#ref-singh_enabling_2024 .csl-entry}
Singh, Parwinder, Michail J. Beliatis, and Mirko Presser. 2024.
"Enabling Edge-Driven Dataspace Integration Through Convergence of
Distributed Technologies." *Internet of Things* 25 (April): 101087.
<https://doi.org/10.1016/j.iot.2024.101087>.
:::

::: {#ref-singh_navigating_2024 .csl-entry}
Singh, Parwinder, Nirvana Meratnia, Michail Beliatis, and Mirko Presser.
2024. "Navigating the International Data Space To Build Edge-Driven
Cross-Domain Dataspace Ecosystem: English." In.
:::

::: {#ref-singh_data-driven_2024 .csl-entry}
Singh, Parwinder,..Nidhi, Michail Beliatis, and Mirko Presser. 2024.
"Data-Driven IoT Ecosystem for Cross Business Growth: An Inspiration
Future Internet Model with Dataspace at the Edge." *INTERNET 2024 :*,
International Conference on Evolving Internet - Proceedings, March.
:::

::: {#ref-steinmetz_key-components_2022 .csl-entry}
Steinmetz, Charles, Greyce N. Schroeder, Ricardo N. Rodrigues, Achim
Rettberg, and Carlos E. Pereira. 2022. "Key-Components for Digital Twin
Modeling With Granularity: Use Case Car-as-a-Service." *IEEE
Transactions on Emerging Topics in Computing* 10 (1): 23--33.
<https://doi.org/10.1109/TETC.2021.3131532>.
:::

::: {#ref-steinmetz_methodology_2022 .csl-entry}
Steinmetz, Charles, Greyce N. Schroeder, Adam Sulak, Kaan Tuna, Alecio
Binotto, Achim Rettberg, and Carlos Eduardo Pereira. 2022. "A
Methodology for Creating Semantic Digital Twin Models Supported by
Knowledge Graphs." In *2022 IEEE 27th International Conference on
Emerging Technologies and Factory Automation (ETFA)*, 1--7.
<https://doi.org/10.1109/ETFA52439.2022.9921499>.
:::

::: {#ref-noauthor_summary_nodate .csl-entry}
"Summary of IoT, and DT Standards." n.d. Change2Twin project.
:::

::: {#ref-talasila_composable_2025 .csl-entry}
Talasila, Prasad, Cláudio Gomes, Lars B Vosteen, Hannes Iven, Martin
Leucker, Santiago Gil, Peter H Mikkelsen, Eduard Kamburjan, and Peter G
Larsen. 2025. "Composable Digital Twins on Digital Twin as a Service
Platform." *SIMULATION* 101 (3): 287--311.
<https://doi.org/10.1177/00375497241298653>.
:::

::: {#ref-talasila_realising_2024 .csl-entry}
Talasila, Prasad, Peter Høgh Mikkelsen, Santiago Gil, and Peter Gorm
Larsen. 2024. "Realising Digital Twins." In *The Engineering of Digital
Twins*, edited by John Fitzgerald, Cláudio Gomes, and Peter Gorm Larsen,
225--56. Cham: Springer International Publishing.
<https://doi.org/10.1007/978-3-031-66719-0_11>.
:::

::: {#ref-tang_survey_2022 .csl-entry}
Tang, Fengxiao, Xuehan Chen, Tiago Koketsu Rodrigues, Ming Zhao, and Nei
Kato. 2022. "Survey on Digital Twin Edge Networks (DITEN) Toward 6G."
*IEEE Open Journal of the Communications Society* 3: 1360--81.
<https://doi.org/10.1109/OJCOMS.2022.3197811>.
:::

::: {#ref-tao_five-dimension_2019 .csl-entry}
Tao, Fei, Weiran Liu, Meng Zhang, Tian-liang Hu, Qinglin Qi, He Zhang,
Fangyuan Sui, Tian Wang, Hui Xu, and Zuguang Huang. 2019.
"Five-Dimension Digital Twin Model and Its Ten Applications." *Comput.
Integr. Manuf. Syst* 25 (1): 1--18.
:::

::: {#ref-tekinerdogan_systems_2020 .csl-entry}
Tekinerdogan, Bedir, and Cor Verdouw. 2020. "Systems Architecture Design
Pattern Catalog for Developing Digital Twins." *Sensors* 20 (18): 5103.
<https://doi.org/10.3390/s20185103>.
:::

::: {#ref-noauthor_tools_nodate .csl-entry}
"Tools and Libraries Catalogue." n.d. Change2Twin project.
:::

::: {#ref-toth_human-centric_2023 .csl-entry}
Tóth, Attila, László Nagy, Roderick Kennedy, Belej Bohuš, János Abonyi,
and Tamás Ruppert. 2023. "The Human-Centric Industry 5.0 Collaboration
Architecture." *MethodsX* 11 (December): 102260.
<https://doi.org/10.1016/j.mex.2023.102260>.
:::

::: {#ref-vaezi_digital_2022 .csl-entry}
Vaezi, Mehrad, Kiana Noroozi, Terence D. Todd, Dongmei Zhao, George
Karakostas, Huaqing Wu, and Xuemin Shen. 2022. "Digital Twins from a
Networking Perspective." *IEEE Internet of Things Journal* 9 (23):
23525--44. <https://ieeexplore.ieee.org/abstract/document/9863238/>.
:::

::: {#ref-viceconti_position_2024 .csl-entry}
Viceconti, Marco, Maarten De Vos, Sabato Mellone, and Liesbet Geris.
2024. "Position Paper From the Digital Twins in Healthcare to the
Virtual Human Twin: A Moon-Shot Project for Digital Health Research."
*IEEE Journal of Biomedical and Health Informatics* 28 (1): 491--501.
<https://doi.org/10.1109/JBHI.2023.3323688>.
:::

::: {#ref-villani_digital_2025 .csl-entry}
Villani, Valeria, Marco Picone, Marco Mamei, and Lorenzo Sabattini.
2025. "A Digital Twin Driven Human-Centric Ecosystem for Industry 5.0."
*IEEE Transactions on Automation Science and Engineering* 22:
11291--303. <https://doi.org/10.1109/TASE.2024.3410703>.
:::

::: {#ref-wermann_ktwin_2024 .csl-entry}
Wermann, Alexandre Gustavo, and Juliano Araujo Wickboldt. 2024. "KTWIN:
A Serverless Kubernetes-Based Digital Twin Platform." arXiv.
<https://doi.org/10.48550/arXiv.2408.01635>.
:::

::: {#ref-wetter_spawn_2024 .csl-entry}
Wetter, Michael, Kyle Benne, Hubertus Tummescheit, and Christian
Winther. 2024. "Spawn: Coupling Modelica Buildings Library and
EnergyPlus to Enable New Energy System and Control Applications."
*Journal of Building Performance Simulation* 17 (2): 274--92.
<https://doi.org/10.1080/19401493.2023.2266414>.
:::

::: {#ref-wu_digital_2021 .csl-entry}
Wu, Yiwen, Ke Zhang, and Yan Zhang. 2021. "Digital Twin Networks: A
Survey." *IEEE Internet of Things Journal* 8 (18): 13789--804.
<https://doi.org/10.1109/JIOT.2021.3079510>.
:::

::: {#ref-xu_survey_2023 .csl-entry}
Xu, Hansong, Jun Wu, Qianqian Pan, Xinping Guan, and Mohsen Guizani.
2023. "A Survey on Digital Twin for Industrial Internet of Things:
Applications, Technologies and Tools." *IEEE Communications Surveys &
Tutorials* 25 (4): 2569--98.
<https://doi.org/10.1109/COMST.2023.3297395>.
:::

::: {#ref-ye_advancing_2024 .csl-entry}
Ye, Yu, Aurora González-Vidal, Alejandro Cisterna-García, Angel
Pérez-Ruzafa, Miguel A. Zamora Izquierdo, and Antonio F. Skarmeta. 2024.
"Advancing Towards a Marine Digital Twin Platform: Modeling the Mar
Menor Coastal Lagoon Ecosystem in the South Western Mediterranean."
arXiv. <https://doi.org/10.48550/arXiv.2409.10134>.
:::

::: {#ref-zech_digital-twins-as--service_2024 .csl-entry}
Zech, Philipp, Claudio Nardin, Sashko Ristov, Matthias Flora, and Ruth
Breu. 2024. "Digital-Twins-as-a-Service in Construction Engineering." In
*2024 IEEE 20th International Conference on Automation Science and
Engineering (CASE)*, 3004--10.
<https://doi.org/10.1109/CASE59546.2024.10711409>.
:::

::: {#ref-zhang_digital_2024 .csl-entry}
Zhang, Jingxi, Carsten Ellwein, Malte Heithoff, Judith Michael, and
Andreas Wortmann. 2024. "Digital Twin and the Asset Administration
Shell."
<https://awortmann.github.io/downloads/paper/Digital_twin_and_the_asset_administration_shell.pdf>.
:::

::: {#ref-zhou_secure_2022 .csl-entry}
Zhou, Zhenyu, Zehan Jia, Haijun Liao, Wenbing Lu, Shahid Mumtaz, Mohsen
Guizani, and Muhammad Tariq. 2022. "Secure and Latency-Aware Digital
Twin Assisted Resource Scheduling for 5G Edge Computing-Empowered
Distribution Grids." *IEEE Transactions on Industrial Informatics* 18
(7): 4933--43. <https://doi.org/10.1109/TII.2021.3137349>.
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
