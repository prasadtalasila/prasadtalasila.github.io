---
hide:
  - navigation
  - toc
---

# Digital Twin Platforms -- Deep Research Report

> Multi-perspective, corpus-grounded research. Method adapted from
> hadufer/claude-storm (MIT) / Stanford OVAL STORM (Shao et al., NAACL 2024).
> Depth: standard - Perspectives: the Platform Builder, the
> Reference-Architecture Scholar, the Skeptic, the Adoption &
> Business-Model Analyst, and the Lifecycle/DevOps Engineer, plus a
> basic fact writer - Citekeys: see References - Date: 2026-08-02

**Disclaimer:** This article has been generated using
[chitragupta](https://prasad.talasila.in/chitragupta).
Despite some potential for hallucination, the ideas communicated in this
tutorial are accurate. Please send your corrections and suggestions to
<prasad.talasila@gmail.com>

## Summary

A digital twin platform is not a digital twin. The Digital Twin
Consortium's glossary separates the two: a twin is a virtual
representation of a real-world entity synchronized at a specified
frequency and fidelity, while a platform is "a set of integrated
services, applications, and other digital twin subsystems that are
designed to be used to implement digital twin systems"
[@noauthor_glossary_nodate-1]. The definitional ground under both terms
remains unsettled -- one review collected 46 distinct definitions of
the twin alone [@vanderhorn_digital_2021] -- and when the strictest
widely used criterion (automated, bidirectional data flow
[@kritzinger_digital_2018]) is applied, many self-described twins turn
out to be models or shadows -- and only five of fourteen graded
open-source frameworks reach full closed-loop digital-twin support
[@gil_survey_2024].

The platform landscape spans commercial cloud offerings (Azure Digital
Twins with its DTDL language, AWS IoT TwinMaker, IBM, Bosch)
[@pfeiffer_modeling_2022; @hakiri_comprehensive_2024], a
foundation-backed open-source ecosystem (Eclipse Ditto, Eclipse BaSyx
and the Asset Administration Shell implementations, INTO-CPS,
iTwin.js), and academic platforms that compose these into fuller
stacks (OpenTwins, KTWIN, DTaaS) [@gil_survey_2024;
@robles_opentwins_2023; @wermann_ktwin_2024; @talasila_composable_2025].
Reference architectures -- ISO 23247's four domains
[@shao_analysis_2023], the Digital Twin Consortium's Platform Stack
with its 62-capability decomposition [@heaton_platform_nodate], and
makeTwin's ten functional modules [@tao_maketwin_2024] -- converge on a
recurring layered anatomy, yet the best empirical study of the field
finds that what real architectures contain (data storage, versioning,
continuous deployment) is not what the standard specifies, and three
standardized functional entities show zero implementation in practice
[@ferko_standardisation_2023].

In practice, none of the platforms examined here ships a whole twin: each occupies one
functional slice whose strengths are the others' gaps, screening for
documentation, maturity, domain fit, and cost eliminated nine of
fourteen candidate frameworks before hands-on evaluation began, and
integration glue -- message
brokers and databases inserted between components that share no
protocol -- is the norm rather than the exception [@gil_survey_2024;
@robles_opentwins_2023]. The lifecycle beyond initial deployment is the
least mature stage: model-driven engineering contributions cluster at
the implementation phase, leaving an operational-phase gap in twin
evolution, simulator maintenance, and drift management
[@beaumont_towards_2025; @alskaif_evolution_2025; @liu_ai_2025].

The economics are equally unsettled. Digital-Twin-as-a-Service and
marketplace models exist in concrete form, but the corpus's digital
twin market forecasts rest on differing, unaudited market definitions
[@saracco_digital_2019; @lim_state---art_2020], a marketplace design
candidly contemplates subsidy dependence
[@noauthor_change2twin_nodate-3], and pilot economics are mixed within
a single project's own reporting: one pilot reports realized time
savings, while another found that a working twin saved no money
because it had to run alongside the traditional methods it was meant
to replace until it earned trust [@noauthor_case_nodate].
The field's flagship institutional report says it plainly: the
publicity around digital twins currently outweighs the evidence base of
success
[@committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024].

## Synthesis Briefing

**Executive summary:** Digital twin platforms are best understood as
metamodels plus plumbing: each platform fixes a description language
and a functional slice (connectivity, asset description, co-simulation,
or 3D), and the working twin emerges only from builder-supplied glue
between slices. The literature's reference architectures and its
implemented platforms have drifted apart -- measurably so -- and every
perspective interviewed for this report converges on the same missing
artifact: standardized, measured, cross-platform evidence (of
performance, of integration cost, of business value) that would let any
of the field's central claims -- reusability, interoperability,
affordability -- be tested rather than asserted.

**Key findings (ranked by reliability):**

1. **No platform examined in this corpus delivers a complete digital
   twin; each covers one functional slice and builders supply the
   glue.** Reliability 9/10 within its scope. Supported by the Platform
   Builder, Scholar, Skeptic, and Basic perspectives; grounded in an
   implemented same-case-study comparison rather than feature sheets
   [@gil_survey_2024; @robles_opentwins_2023;
   @callisto_de_donato_enabling_2024]. The scope caveat matters: the
   principal survey excludes commercial tools, so the strongest
   vendors' offerings are untested here.
2. **The standard-practice mismatch is quantified, in both
   directions.** Data storage appears in 69% of surveyed real
   architectures yet is absent from ISO 23247's functional entities,
   while the standardized plug-and-play, peer interface, and data
   assurance entities showed 0% implementation
   [@ferko_standardisation_2023]. Reliability 9/10 -- the only
   triangulated (SLR + survey + interviews) result in the corpus;
   unchallenged by any perspective.
3. **Terminological inflation is real and measurable at the framework
   level.** Many self-described digital twins are, on inspection,
   models or shadows under the strict bidirectional criterion
   [@kritzinger_digital_2018], and only five of the fourteen graded
   open-source frameworks reach full closed-loop digital-twin support
   [@gil_survey_2024]. Reliability 7/10. Supported by all
   perspectives; the stronger field-wide version ("most platforms are
   shadow platforms") is deliberately not asserted, since the grading
   covers open-source frameworks only and five of fourteen do reach
   the top level.
4. **The operational phase is the least supported lifecycle stage.**
   Model-driven engineering contributions cluster at the
   implementation phase, leaving an acknowledged operational-phase gap
   [@beaumont_towards_2025]; asset drift threatens outright twin
   redevelopment [@alskaif_evolution_2025]; and platform modeling
   languages weaken under evolution scenarios
   [@pfeiffer_modeling_2022]. Adjacent evidence from the AI-simulation
   literature -- where sampled studies use no DT platforms at all and
   only 2 of 22 implement on-demand simulator maintenance -- points
   the same way, though it measures study practice rather than
   platform capability [@liu_ai_2025]. Reliability 8/10; supported by
   the Lifecycle/DevOps and Scholar perspectives.
5. **Platform economics remain unproven.** The corpus's market
   forecasts rest on differing, unaudited market definitions
   [@saracco_digital_2019; @lim_state---art_2020], the one candid
   marketplace design contemplates subsidy dependence
   [@noauthor_change2twin_nodate-3], and pilot-level economic evidence
   is mixed within a single project's own reporting -- realized time
   savings in one pilot, no realized saving in another because the
   twin must first earn trust [@noauthor_case_nodate]. Reliability
   6/10 -- the economics evidence is thin, self-reported, and comes
   largely from one EU project's deliverables.

**Hidden connection:** Three of the perspectives' central gaps are the
same gap. The Builder's unmeasured glue-code cost, the Scholar's
evaluation frameworks that do not operationalize the reference
architectures, and the Skeptic's unvalidated platform proposals all
reduce to the absence of shared measurement infrastructure -- there is
no agreed way to run the same twin on two platforms and compare cost,
fidelity, or latency; the DTaaS survey names the missing metrics
(twinning rate, twin fidelity, synchronization latency) precisely
because nobody uses them yet [@duran_toward_2026]. The Adoption
Analyst's trust-blocked ROI is related but not identical: in
qualification-heavy contexts it is a verification-and-validation
problem that benchmarks alone would not solve [@noauthor_case_nodate].
One honesty note: convergence among perspectives generated for this
report is partly a property of the lenses chosen -- a Skeptic
perspective guarantees an evidence-deficit theme -- and should be
weighed accordingly.

**Actionable insight (for a digital twins researcher):** The
highest-leverage contribution available in this space is a reproducible
cross-platform benchmark: one open exemplar twin, implemented on
several platforms, with standardized performance metrics
[@duran_toward_2026] *and* accounted integration effort -- extending
the same-case-study method of [@gil_survey_2024] from feature grading
to measured cost. A companion experiment -- implementing ISO 23247's
never-implemented functional entities and testing whether they reduce
integration seams -- would connect the benchmark directly to the
standards debate [@ferko_standardisation_2023].

**Frontier question:** Can any platform demonstrate, with measured
evidence rather than architecture diagrams, that it reduces the
lifetime cost of building, evolving, and migrating a digital twin --
and, in qualification-heavy contexts like the additive-manufacturing
pilot, enough to let the twin replace rather than merely accompany the
process it was built to improve?

## 1. What a Digital Twin Platform Is

### 1.1 Twin versus platform

The Digital Twin Consortium's glossary separates two concepts that industrial and academic writing frequently conflate. A digital twin is "a virtual representation of real-world entities and processes, synchronized at a specified frequency and fidelity," whereas a digital twin platform is "a set of integrated services, applications, and other digital twin subsystems that are designed to be used to implement digital twin systems"; in this framing, platforms "provide a purpose-specific foundation for a digital twin system" rather than constituting a twin themselves [@noauthor_glossary_nodate-1]. The distinction is between an instance -- one synchronized virtual counterpart of one physical asset -- and the reusable substrate of data ingestion, modeling, analytics, and lifecycle tooling on which many such instances are built and operated. A comparative review of roughly one hundred deployed digital twins identifies "distinguishing between digital twin (instance) and digital twin platforms" as one of its principal recommendations, and observes that industry currently prefers commercial platforms even as open-source alternatives rise in number and maturity [@parle_comparative_2024]. Practically, the platform view matters because a platform lets an organization create and manage multiple twin instances from shared infrastructure, amortizing engineering effort across assets [@parle_comparative_2024].

### 1.2 A contested definitional landscape

Neither term sits on settled ground. A survey of ten years of literature collected 46 distinct definitions of the digital twin before proposing a consolidated, generalized definition [@vanderhorn_digital_2021]. The situation has not converged since: recent work still notes that "despite various definitions, there is little consensus about what a DT actually is and what it should comprise" [@pfeiffer_towards_2025]. Definitions of the platform inherit this instability, since what counts as a digital twin platform depends directly on what one requires of the twins it hosts.

### 1.3 The integration-level ladder

The most widely used instrument for cutting through this ambiguity classifies systems by the direction and automation of data flow between the physical and virtual objects: a *digital model* has only manual data exchange in both directions; a *digital shadow* adds an automated one-way flow from the physical object to its virtual counterpart; only when data flows automatically in both directions -- the virtual object also acting back on the physical one -- does the system qualify as a *digital twin* [@kritzinger_digital_2018]. Notably, the same review found frequent misuse of the terms, with many self-described twins being models or shadows under this criterion [@kritzinger_digital_2018]. Recent survey work turns this ladder into a grading scheme for platforms and frameworks themselves, from Level 0 (digital-model support only) through Level 1 (digital-shadow support, automated physical-to-digital flow) and Level 2 (the reverse, automated digital-to-physical flow) up to Level 3 (full digital-twin support with closed-loop physical-to-digital-to-physical flow) [@gil_survey_2024]. Applied strictly, the criterion is deflationary, though not uniformly so: in the survey's grading, five of the fourteen open-source frameworks (Eclipse Ditto, SAP's AAS implementation, BaSyx, NOVAAS, and INTO-CPS) do reach full closed-loop support, while the remainder fall short; and for the TerriaJS-based New South Wales twin specifically, the survey notes that its components "mostly behave as Digital Models or Digital Shadows depending on how the data is integrated into the platform" [@gil_survey_2024]. That framework-level result echoes Kritzinger's original literature-level finding of widespread term misuse [@kritzinger_digital_2018].

### 1.4 The platform as a metamodel

A complementary way to understand what a platform *is* comes from modeling-language research: each major platform embeds its own domain-specific language for describing twins -- Microsoft Azure's Digital Twins Definition Language (DTDL), Eclipse's Vortolang, and AWS IoT TwinMaker's API-based scheme -- each with an extractable conceptual metamodel [@pfeiffer_modeling_2022]. Comparison with UML class diagrams shows these languages largely repackage established object-oriented modeling concepts -- "old wine" in "new bottles" -- with only limited genuinely new constructs [@pfeiffer_modeling_2022]. The consequence is architectural: porting a twin between platforms is a model transformation between platform-specific metamodels, demonstrated experimentally via a UML-based pivot from Azure's metamodel to AWS's [@pfeiffer_modeling_2022]. On this view, choosing a platform is choosing a metamodel, and platform lock-in is, at bottom, metamodel lock-in [@pfeiffer_modeling_2022].

## 2. The Platform Landscape

### 2.1 Commercial cloud platforms

The commercial end of the digital twin platform market grew largely out of existing Internet-of-Things offerings. Microsoft Azure was a pioneer in this area with its IoT Hub, which it later adapted into the dedicated Azure Digital Twins service; twins on Azure are described in JSON files conforming to the Digital Twin Definition Language (DTDL), edited through a graphical model editor and connected to devices via the Azure IoT Hub and companion services such as time-series storage and machine learning [@pfeiffer_modeling_2022]. Amazon Web Services offers a comparable service in AWS IoT TwinMaker, but, in contrast to Azure and its DTDL, provides no metamodel of its modeling language [@pfeiffer_modeling_2022]. This heterogeneity has the portability consequences Section 1.4 described: moving models between the two clouds has been demonstrated only as a UML-pivot feasibility experiment [@pfeiffer_modeling_2022]. Beyond these two hyperscalers, comparative surveys tabulate a broader commercial field -- including the IBM digital twin offering, the Bosch IoT Suite (itself built on top of Eclipse IoT open-source components), and AWS IoT TwinMaker -- against feature checklists, revealing uneven coverage of capabilities across vendors [@hakiri_comprehensive_2024]. Dissatisfaction with this state of affairs is itself a driver of new architectures: the makeTwin reference architecture is explicitly motivated by the claim that existing digital twin platforms on the market "have significant limitations" in functionality [@tao_maketwin_2024]. One caveat belongs here rather than buried later: this commercial coverage reflects what the analyzed corpus contains. Prominent industrial vendors absent from that literature -- Siemens and PTC among them -- go undiscussed, and the principal open-source survey explicitly scopes commercial tools out of its evaluation [@gil_survey_2024]. The omission matters, because deployed practice leans the other way: a review of roughly one hundred deployed twins notes an industry preference for commercial platforms [@parle_comparative_2024], so this report's platform-capability findings are strongest for the open-source segment and weakest exactly where adoption is concentrated.

### 2.2 The open-source ecosystem

A substantial open-source ecosystem has developed in parallel, anchored by foundation- and consortium-backed projects. A survey of fourteen open-source digital twin frameworks documents, among others, Eclipse Ditto (Eclipse Foundation and Bosch), Eclipse BaSyx (Eclipse, Bosch, and the Fraunhofer Institute), the AASX Package Explorer from the Industrial Digital Twin Association (IDTA), SAP's I4.0 Asset Administration Shell implementation, and NOVAAS from the NOVA School of Science and Technology, alongside language specifications such as DTDL and Twined, the geospatial TerriaJS platform, the Digital Twin Cities Centre (DTCC) platform from Chalmers, Bentley Systems' iTwin.js, and the INTO-CPS co-simulation toolchain [@gil_survey_2024]. Open-source components also serve as building blocks for higher-level platforms: general-purpose IoT platforms such as ThingsBoard implement much of the required functionality but remain limited in visualization and simulation, and a 3D-simulation extension to ThingsBoard is framed by its authors as only "a first step towards" a digital twin platform [@callisto_de_donato_enabling_2024].

### 2.3 Academic and research platforms

University groups have produced several platforms that compose open-source infrastructure into complete digital twin environments. OpenTwins, from the University of Malaga, takes a compositional approach, assembling Eclipse Ditto with InfluxDB/Telegraf and Kafka into an integrated framework for compositional digital twins [@robles_opentwins_2023]; a subsequent extension re-defines its architecture for distributed digital twins whose components operate on different devices, confronting the synchronization and scalability challenges this raises [@infante_distributed_2025]. KTWIN, from the Federal University of Rio Grande do Sul (UFRGS), is a serverless, Kubernetes-based digital twin platform built around an event-driven architecture [@wermann_ktwin_2024]. The DTaaS platform, developed at Aarhus University with the INTO-CPS Association, organizes digital twin development around reusable assets -- data, models, functions, and tools -- from which composable digital twins are created, supports treating related twins as fleets, and is explicitly intended to provide all organizations an affordable entry point into digital twin engineering [@talasila_realising_2024; @talasila_composable_2025].

### 2.4 A six-category taxonomy as a map

The most systematic map of the open landscape evaluates the fourteen open-source frameworks along ten criteria and sorts them into six groups according to their modeling and technological approach: structured-data/AAS-oriented frameworks (the Ditto/BaSyx/AASX/SAP/NOVAAS cluster), domain-specific frameworks, language specifications (DTDL, Twined), geospatial platforms, 3D-infrastructure platforms (iTwin.js, DTCC), and co-simulation frameworks (INTO-CPS) [@gil_survey_2024]. Five of the fourteen frameworks, selected for maturity, were additionally implemented on a common case study, giving the taxonomy an empirical grounding that feature tables alone lack [@gil_survey_2024]. Read together with the commercial feature comparisons [@hakiri_comprehensive_2024], this taxonomy is the report's working map of just how many different things "digital twin platform software" can mean.

## 3. Reference Architectures and the Standards They Lean On

### 3.1 ISO 23247: a framework standard for manufacturing twins

ISO 23247 organizes a manufacturing digital twin into four interconnected layers of domains: an Observable Manufacturing Domain containing the physical elements, a Device Communication Domain holding the sensor and actuation entities, a Digital Twin Domain containing the twin entities themselves, and a User Domain through which applications such as MES, ERP, and PLM systems interact with the rest [@shao_analysis_2023]. A Cross-system Entity resides across these domains to provide common functionalities such as data translation, data assurance, and security support [@shao_analysis_2023]. The standard's central modeling contribution is the definition of eight types of Observable Manufacturing Elements (personnel, equipment, material, process, facility, environment, product, and supporting document), each digitally represented through seven attribute types of which only the identifier is mandatory [@shao_analysis_2023]. Notably, the layering is not strictly hierarchical: the actuation functional entity in the device communication layer may act on a physical element in concert with either the digital twin entity above it or the user entity one layer further away, supporting both fully automated and semi-automated operation [@shao_analysis_2023]. ISO 23247 deliberately does not standardize information models; it is a framework standard into which other standards -- MTConnect for data collection, STEP for product representation, QIF for quality information -- are plugged when building an actual twin [@shao_analysis_2023]. NIST has elaborated three concrete use-case scenarios, implementable on its Smart Manufacturing Systems Test Bed, that walk through implementing the framework on a real shop floor [@shao_use_2021].

### 3.2 The DTC Platform Stack and capability-driven architecting

The Digital Twin Consortium's Platform Stack Architectural Framework identifies the critical building blocks of a digital twin platform, layering IT/OT infrastructure foundations, virtual representation, a service layer, and an application layer, with security and trustworthiness treated as a dedicated concern [@heaton_platform_nodate]. It explicitly maps itself onto other reference efforts -- RAMI 4.0 (the German Reference Architectural Model for Industrie 4.0), the Digital-Twin-as-a-Service reference model [@aheleroff_digital_2021], the IIC Connectivity Framework, FIWARE, the Asset Administration Shell, and the IBM, Microsoft, and AWS commercial stacks -- positioning itself as an integrative overlay rather than a competitor [@heaton_platform_nodate]. Its companion decomposition breaks digital twin capability into six high-level categories that expand into 62 discrete top-level capabilities, presented as a "Capabilities Periodic Table" [@heaton_platform_nodate]. Practitioner guidance shows the two artifacts being combined to define required capabilities for a given twin [@kayla_digital_2023], and the DISCS project uses the Platform Stack and Periodic Table together to architect a system-of-systems composition of twins [@mckee_discs_2024].

### 3.3 makeTwin and platform reference models beyond the DTC

The makeTwin platform derives its architecture from the five-dimension digital twin model (physical entity, virtual model, data, services, connections) [@tao_five-dimension_2019]. It comprises ten core functional modules -- twinModelBuilder, twinDataProcessor, twinAlgBuilder, twinIoTConnector, twinInteractor, twinSimulator, twinLibrary, twinVisualization, twinSceneTemplate, and twinAppDeployer -- of which eight (all but twinAppDeployer and twinLibrary) interact through customizable APIs [@tao_maketwin_2024]. In parallel, a Digital Twin as a Service (DTaaS) reference model has been proposed for Industry 4.0 that aligns twin capabilities-as-a-service with RAMI 4.0 [@aheleroff_digital_2021]. The continued appearance of new unifying reference models is itself telling: recent work motivates yet another one by the persistent lack of consensus among existing definitions and models, naming the resulting "concept-implementation gap" as the problem to be closed [@pfeiffer_towards_2025].

### 3.4 Standards bridges and the quantified gap with practice

Because platforms speak different description languages, converters have become their own research thread: a DTDL-to-AAS transformation has been implemented and tested [@schmidt_increasing_2023], and a mapping from DTDL into OPC UA has been proposed [@cavalieri_proposal_2023]. Yet there are currently no major implementations for converting twin descriptions between formats; most existing options are one-directional and require manual input [@mattila_interoperability_2025]. Even within a single standard, implementations diverge enough that four open-source reactive (Type 2 -- runtime, API-accessible, as opposed to static file-based shells) AAS implementations have been formally compared [@jacoby_open-source_2023]. The standard-vs-practice gap has been quantified through mixed-methods research combining 29 manufacturing twin architectures drawn from a 140-study systematic review, a 33-respondent survey, and expert interviews [@ferko_standardisation_2023]. The mismatch runs both ways: data storage appears in 69% of surveyed architectures, and DT versioning and continuous deployment also occur in practice, yet none is captured by ISO 23247's functional entities; conversely, the standardized plug-and-play, peer interface, and data assurance functional entities showed 0% implementation, even though experts rate data assurance as essential -- while continuous deployment is judged an implementation rather than an architectural concern [@ferko_standardisation_2023].

## 4. Building on a Platform in Practice

### 4.1 What a platform ships versus what builders add

No current platform delivers a complete digital twin out of the box; each occupies one functional niche whose strengths are precisely the gaps of the others. A survey organizing open-source frameworks into six categories finds, for instance, that structured-data frameworks do not focus sufficiently on simulation models, data analysis, or having the twin fulfill a particular business goal, while the co-simulation category's bi-directional communication "may not be trivial to implement" [@gil_survey_2024]. Even a single "platform" is in practice a constellation of components: running twins on INTO-CPS means combining the INTO-CPS Application, the Maestro co-simulation orchestrator, UniFMU, and the RabbitMQ FMU -- a Functional Mock-up Unit, i.e. a simulation component packaged to the FMI co-simulation standard -- that carries messages from the physical twin into the simulation [@gil_survey_2024]. Part of the stack, however, does come for free: the underlying operating-system, container, and cloud layers already handle isolation, virtualisation, and scalability for the builder, so platform selection can concentrate on twin-specific capabilities such as the FMI-based simulation that IoT-style platforms lack [@talasila_realising_2024].

### 4.2 The documentation gate

Before any integration work begins, candidate platforms must pass a documentation filter, and most do not. The same survey collected 14 frameworks but carried only five into its case study; the discarded candidates fell to a mix of screening reasons -- missing installation documentation for some, immaturity, domain mismatch, GUI-only scope, or licensing cost for others [@gil_survey_2024]. The pool of nominal candidates is far larger than the usable one: a GitHub search on the digital-twin topic returns more than 100 repositories, but many are undocumented or immature [@gil_survey_2024]. A parallel evaluation of open-source implementations likewise filtered out inactive projects and warned that users "are probably not going to find" existing features "because of missing documentation" [@jacoby_open-source_2023].

### 4.3 Glue code as the norm

Once a platform is chosen, builders routinely discover that adjacent components do not connect themselves. The OpenTwins build log is illustrative: Eclipse Ditto provides twin state but no time-series storage of it, so InfluxDB and Telegraf were added; then, because "neither technology implements a broker for the protocols it supports," Apache Kafka was inserted purely to connect Ditto to the storage layer [@robles_opentwins_2023]. The pattern of extending a near-platform into a twin platform recurs elsewhere: ThingsBoard required a 3D-model and 3D-simulation extension before it could approach digital-twin functionality [@callisto_de_donato_enabling_2024].

### 4.4 Modeling-language effort

The twin-description languages platforms ship also impose measurable costs. A controlled comparison of behavior-modeling patterns in DTDL found a quantifiable difference in modeling effort between the closed-model and open-model patterns, and observed more broadly that platform languages cover structural viewpoints while behavioral viewpoints "did not yet receive much attention on these platforms" [@lehner_pattern_2023]. An independent analysis of platform DSLs reached a compatible verdict: the languages are structural in orientation and their weaknesses surface under evolution scenarios, and portability remains speculative -- an Azure-to-AWS model transformation was demonstrated only as a feasibility experiment [@pfeiffer_modeling_2022].

### 4.5 Deployment topology: cloud versus edge

Where the assembled stack runs is a design decision with strategic consequences. Cloud twinning platforms bring "an immediate vendor lock-in," which has motivated architectures built on replaceable middleware -- for example, a swappable Ditto layer -- and open standards [@zech_digital-twins-as--service_2024]. Cloud-centric sensor-to-cloud silos have been criticized on architectural grounds as well [@aguzzi_cloud_2020]. On the other side, edge deployment is required for responsiveness but remains poorly supported: surveyed frameworks treat twins as passive components "without any evaluations of their applicability as independent entities," and edge deployment is "totally unaddressed and not evaluated" in most of them, leaving edge-twin properties underinvestigated [@picone_flexible_2023]. Emerging middleware addresses the gap dynamically, monitoring "cyber-physical entanglement" and migrating twins to different hosting nodes when link latency degrades [@bellavista_entanglement-aware_2024].

### 4.6 Scaling

Scaling a platform built this way is not an afterthought that composes cleanly. Distributing OpenTwins across nodes "poses synchronization and scalability challenges" severe enough to require re-defining the platform itself [@infante_distributed_2025]. Microservice-based twin designs offer a mitigation path, since they inherit mature techniques for migration, replication, and software update from the microservices ecosystem [@bellavista_entanglement-aware_2024], and the container and cloud substrate beneath the platform continues to supply baseline scalability regardless of the twin layer above it [@talasila_realising_2024].

## 5. Lifecycle: DevOps, Evolution, and Operations

### 5.1 TwinOps and DevOps-inspired process proposals

A recurring proposal is to import continuous software-engineering practice into digital twin construction. The earliest articulated variant, TwinOps, is defined as "a process that unifies Model-based Engineering, Digital Twins, and DevOps practice in a uniform workflow" [@hugues_twinops_2020]. In this scheme, model-based code generation drives a single pipeline whose artifacts target simulation testbenches, validation platforms, and the deployed digital twin alike, demonstrated through a combination of AADL architectural models, Modelica physical models, and an IoT platform [@hugues_twinops_2020; @hugues_twinops_2022]. Subsequent work has generalized the idea: model-driven engineering is argued to facilitate DevOps for twins by keeping models as the shared currency between development and operations [@combemale_model-based_2023], and a first systematic analysis of the challenges of model-driven DevOps for digital twins has since been published [@michael_model-driven_2025]. The JuNo-OPS approach makes the process concrete for building-scale twins through three pillars -- an iterative development methodology, a microservice-based architecture, and a DevOps infrastructure -- validated on a multi-functional room and a climatic chamber [@aissat_juno-ops_2024; @aissat_devops_2025; @bordeleau_devops_nodate].

### 5.2 Asset drift: evolution as a first-class requirement

The motivation for these processes is that neither the asset nor the requirements stand still. Digital twins "need to be continuously modified/updated to meet evolving user requirements," and because buildings live for decades, their twins "must be engineered to support evolution" from the outset rather than treated as one-off deliverables [@aissat_juno-ops_2024; @aissat_devops_2025]. The failure mode of ignoring this is explicit: when the actual twin (the physical asset) changes, "the DT may need to be entirely redeveloped in order not to become obsolete," and DevOps-style continuous delivery is proposed precisely to absorb asset and environment change incrementally [@alskaif_evolution_2025]. Platform-level modeling support is currently weak here: studies of digital twin platform modeling languages find them predominantly structural and poorly suited to evolution scenarios [@pfeiffer_modeling_2022; @lehner_pattern_2023].

### 5.3 Concrete evolution mechanisms

Beyond process prescriptions, a smaller body of work supplies mechanisms. Lehner et al. propose fluent APIs for evolving the schemas and models of already-running twins without teardown [@lehner_towards_2021]. The DarTwin notation captures a twin system's purposes and supplies architectural transformations for evolving the system as those purposes change over time [@mertens_continuous_2024]. Kamburjan et al. formalize declarative lifecycle stages for twins and pair them with MAPE-K-style (monitor-analyze-plan-execute over shared knowledge) self-adaptation that reconfigures or replaces digital twin components at runtime [@kamburjan_declarative_2024; @kamburjan_digital_2022]. A complementary line models the physical object's own lifecycle inside the digital twin's lifecycle -- with Bound, Unbound, and Synchronized states realized in the WLDT library -- so that misalignment between twin and asset can be detected and interpreted rather than silently accumulated [@picone_harmonizing_2025]. Maintenance of embedded simulators remains among the least served concerns. A survey of AI simulation by digital twins found on-demand simulator maintenance in only 2 of its 22 sampled studies per its data table (its discussion text says "about 18%"), and separately observed "a complete lack of DT platforms and frameworks" in use across the sample, whether open-source or proprietary [@liu_ai_2025] -- evidence of how study practice runs today and, precisely because no platform was in use, only indirect evidence about platform capability.

### 5.4 CI/CD transfers only partially

Continuous integration and delivery, the machinery underlying DevOps, transfers to cyber-physical settings only in part. A ten-organization empirical study -- starting from the premise, established in prior work, that "existing CI/CD technology cannot be applied to CPSs as is" -- documents the recurring barriers in practice: hardware-in-the-loop stages, simulator mock-ups, and discrepancies between models and physical behavior [@zampetti_continuous_2023]. Digital twin prototypes have been proposed as a partial remedy, standing in for hardware so that embedded software can be tested in agile CI pipelines [@barbie_toward_2024]. GitOps-style declarative operations have likewise been mapped onto digital twin operations, with a proof of concept on the Fischertechnik Training Factory 4.0 [@beaumont_towards_2025], building on earlier GitOps-driven twin platform work [@beetz_gitops_2022]. The stated cost is operational: setting up and maintaining a GitOps platform introduces complexity of its own [@beaumont_towards_2025]. A structural asymmetry -- this report's observation, not the papers' -- also bounds the analogy: Git makes *software* changes easy to revert [@beetz_gitops_2022], but no declarative reconciliation can revert a physical asset, so GitOps-style operations can only ever govern the digital side of a twin.

### 5.5 The operational-phase gap

The aggregate picture is front-loaded. Citing Lehner's mapping study, Beaumont et al. note that most model-driven engineering contributions have "primarily focused on the implementation life cycle of the DT, leaving a gap at the operational phase" [@beaumont_towards_2025]. The mechanisms of Section 5.3 and the GitOps and CI experiments of Section 5.4 are early responses to that gap, but long-running operation -- simulator upkeep, drift management, and safe runtime change -- remains the least mature stage of the digital twin platform lifecycle [@liu_ai_2025; @beaumont_towards_2025; @alskaif_evolution_2025].

## 6. Economics, Adoption, and Trust

### 6.1 Digital Twins as a Service: the business model

The Digital-Twin-as-a-Service (DTaaS) paradigm reframes digital twins from bespoke engineering artefacts into services delivered over cloud infrastructure, an architecture first articulated for Industry 4.0 mass individualization [@aheleroff_digital_2021]. The model has since been elaborated into concrete multi-stakeholder service architectures [@duran_toward_2026], including vendor-independent designs explicitly motivated by the vendor lock-in that arises when open model- and data-exchange infrastructure is lacking [@zech_digital-twins-as--service_2024]. On the open-source side, the DTaaS platform developed at Aarhus University is positioned as "an affordable entry point" into digital twin engineering for all organisations [@talasila_realising_2024], with a follow-on platform design supporting composable digital twins assembled from reusable, shareable assets [@talasila_composable_2025]. The revenue logic underpinning these offerings is consumption-based: pay-as-you-go schemes let users pay for the amount of service actually used, as surveyed in the digital twin business-model literature [@lim_state---art_2020], and on cloud-edge infrastructure the pay-per-use model provides cost efficiency and flexibility because resources scale up and down with digital twin workload demand [@bellavista_exploiting_2024].

### 6.2 Marketplace economics

The EU Change2Twin project offers an unusually candid look at how a digital twin marketplace might sustain itself. Its revenue design enumerates streams including SMEs paying a percentage of services and solutions sold -- "usually 30%", explicitly modelled on the cuts taken by Apple, Google, Amazon, and Steam -- alongside membership fees, SaaS/PaaS/IaaS hosting, public funding, advertisements, and affiliate arrangements [@noauthor_change2twin_nodate-3]. The same document sketches three post-project scenarios: a marketplace that fully funds itself from combined revenue streams; one where only hosting services generate revenue, of interest "only [to] vested parties"; or retreat to a "very low-cost static webpage" carried as a cost by a single party [@noauthor_change2twin_nodate-3]. That the fully self-funding outcome is only one of three contemplated futures signals genuine uncertainty about whether marketplace economics can stand alone.

### 6.3 Divergent market forecasts

Headline market forecasts for digital twins are mutually incompatible by factors of three to five. MarketsandMarkets projected a market of US$15.3 billion in 2023 [@saracco_digital_2019]; Grand View Research forecast growth to US$27.06 billion by 2025, an approximate tenfold increase from US$2.26 billion in 2017 [@lim_state---art_2020]; and a later assessment reported the market "valued at USD$79.16 billion in 2022" with a compound annual growth rate of 27.07% through 2030 [@fitzgerald_potential_2024]. These figures cannot be reconciled by growth alone and are best read as evidence of unsettled market definitions rather than consensus valuation.

### 6.4 The SME asymmetry

Adoption economics are sharply asymmetric by firm size. Digital twin adoption is "easier for large manufacturing companies because of the available workforce, resources and investments than SMEs" [@singh_digital_2023], and small-to-medium manufacturers "almost always cannot afford the higher salaries" of the experienced engineers digital twins require [@grieves_digital_2024]. Tooling choices sharpen the trade-off: commercial software brings stability, support, and out-of-the-box modules but "create[s] some dependencies between manufacturers that can affect negatively the projects at the mid- or long-term", while open source "breaks the barriers of having low budget for SMEs and start-ups" at the cost of stability and support [@gil_survey_2024]. The structural backdrop is a landscape of "vertical and closed silos" built on vendor-specific meta-models [@giulianelli_engineering_2024], against which Gaia-X-style federations propose keeping data "stored decentralized at the respective companies" and exchanged via common standards [@gleich_asset_2024]. Standards conformance itself falls to SMEs: some solution providers deliberately build on proprietary technology, leaving the SME -- for whom digital twins are not core business -- to weigh standards support when selecting a provider [@noauthor_case_nodate], even though SME adoption "is much facilitated by deploying standards" [@noauthor_summary_nodate]. EU initiatives such as HUBCAP respond by offering a cloud-enabled open collaboration platform intended to lower SME barriers to model-based design of cyber-physical systems [@larsen_hubcap_2022; @obaidat_hubcap_2022].

### 6.5 Trust as the ROI blocker

Surveyed implementation barriers rank "bureaucracy, cultural inertia", lack of specialists, unclear value propositions, lack of investment, and "difficulty in setting realistic expectations and trust" among the chief obstacles [@perno_implementation_2022]. A Change2Twin pilot in additive manufacturing makes the economic consequence concrete: the anticipated cost saving "is not achieved, since the digital twin still needs to gain more trust with people", so the twin "will probably coexist next to the traditional qualification methods and therefore not save any money yet" [@noauthor_case_nodate]. That verdict must be read against the same deliverable's other pilots, several of which report first-hand quantified benefits -- a 90% reduction in a major error class, communication time down 20-30%, and elsewhere a 20% increase in accuracy alongside a 20% decrease in installation costs [@noauthor_case_nodate]. The negative result stands out not because it is the only economic verdict but because project deliverables have every incentive to overclaim, and this one didn't: in the qualification-heavy additive-manufacturing context, operator trust in the twin's predictions relative to destructive testing is the binding constraint, and duplicated qualification effort converts that trust deficit directly into an ROI blocker [@noauthor_case_nodate]. A disclosure applies across this section: the marketplace design, the standards guidance, and the pilot verdicts are all deliverables of the same EU project, Change2Twin, and thus correlated rather than independent lines of evidence [@noauthor_change2twin_nodate-3; @noauthor_summary_nodate; @noauthor_case_nodate]. The Digital Twin Consortium has responded with a trust-assurance framework built on "trust vectors" for assuring resilient dynamic systems [@noauthor_assuring_2024], an acknowledgement that until trust is engineered and demonstrable, digital twins supplement rather than replace the processes they were meant to retire.

## 7. Open Problems and the Evidence Gap

### 7.1 The confessed evidence deficit

The most striking feature of the digital twin platform literature is a deficit that the field's own institutions have openly confessed. The U.S. National Academies' foundational report states that "the publicity around digital twins and digital twin solutions currently outweighs the evidence base of success," quoting Mark Girolami, chief scientist of The Alan Turing Institute, that the "Digital Twin evidence base of success and added value is seriously lack[ing]" [@committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024]. The same report is careful not to dismiss the technology outright, acknowledging genuine examples of digital twins providing practical impact and value, but frames the gap between promise and proof as the central problem for the research agenda [@committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024]. A Nature Computational Science editorial repeats the admission verbatim, noting that digital twins "hold immense promise in accelerating scientific discovery, but the publicity currently outweighs the evidence base of success" [@noauthor_rise_2024].

### 7.2 Unvalidated proposals and subjective evaluation

The platform literature itself supplies the mechanism behind this deficit. A systematic review of digital twin middleware "observed that most studies did not validate their proposal and, in some cases, the validation was conducted in closed environments" or on simulated testbeds rather than real deployments [@almeida_middleware_2023]. Where platform evaluation is attempted, existing methods "are mostly based on subjective opinions, simple decision-making methods, and general-purpose criteria and indicators" that may not suit digital twin platforms at all [@tang_evaluation_2025]. A 2026 survey of Digital-Twin-as-a-Service research concludes that the field "lacks metric-oriented approaches that link DTaaS platform design to measurable system behavior," finds "a complete absence of systematic integration and mapping of DT performance metrics," and notes that prior work offers no concrete numerical benchmarks and no standardized communication interfaces; it proposes candidate metrics -- twinning rate, twin fidelity, inference latency, and synchronization latency -- precisely because none are in common use [@duran_toward_2026].

### 7.3 Minimal analytics and simulation

The functional core of the platforms is also less mature than the marketing suggests. A survey of open-source frameworks finds that "the current advances of built-in simulations and data analytics are at a minimum level of development," with analytics support "basic. At maximum, a level 2" on the survey's separate five-level analytics-maturity scale -- meaning integrated dashboards or basic analytic tasks only [@gil_survey_2024]. That survey explicitly scopes out commercial tools, so even this modest picture covers only part of the market [@gil_survey_2024]. Architectural evaluations reach a parallel conclusion: digital twins are still largely modeled as passive entities subordinated to external modules rather than active, independently evaluable components, and edge-oriented deployment remains fragmented and unevaluated [@picone_flexible_2023].

### 7.4 Security as an open threat surface

Security research on digital twins remains predominantly a catalog of threats rather than deployed defenses. A dedicated survey enumerates threats across cloud, fog, and edge infrastructures, ranking rogue devices and components among the most impactful, and argues that confidentiality demands outsized protection "because digital models represent an exact copy of the physical counterparts" -- a breach of the twin discloses the asset itself [@alcaraz_digital_2022].

### 7.5 Blind spots the corpus does not answer

Several questions surveyed for this report simply have no answer in the literature examined. No identified work addresses systematically retiring or decommissioning the twin itself; the nearest treatment considers decommissioning of the physical asset as a life-cycle phase the twin should support, not the twin's own end of life [@fitzgerald_potential_2024]. No study quantifies integration or switching costs -- person-hours, glue code, or migration effort -- for moving between platforms. No per-platform market-share or adoption figures appear in the academic corpus. Claims of composability and reusability, though central to platform value propositions, have not been empirically evaluated at scale. And ISO 23247, the most-cited platform-relevant standard, has not been tested head-to-head outside manufacturing. These absences are themselves findings: they mark the boundary between what the field asserts about digital twin platforms and what it has demonstrated.

## Contradiction Map

- **Conflicts:**
  - *Market size:* the corpus's digital twin forecasts (US$15.3B by 2023 [@saracco_digital_2019]; US$27.06B by 2025, up from US$2.26B in 2017 [@lim_state---art_2020]) differ in vintage, target year, and market definition, and no source audits or reconciles them -- a documented absence of consensus rather than a strict numeric contradiction.
  - *Standards value:* an assertion-versus-evidence tension rather than a symmetric conflict: programmatic guidance asserts SME adoption "is much facilitated by deploying standards" [@noauthor_summary_nodate], while the one empirical study finds practice implements none of several standardized functional entities and standards capture none of several practiced capabilities (storage, versioning, deployment) [@ferko_standardisation_2023].
  - *What counts as a twin platform:* the strict bidirectional criterion [@kritzinger_digital_2018] vs. framework reality: five of fourteen graded open-source frameworks reach full closed-loop support and the rest do not [@gil_survey_2024], with "little consensus about what a DT actually is" persisting [@pfeiffer_towards_2025].
  - *Deployment topology:* a partially reconciled tension: the cloud-centric platform tradition criticized as silo-forming [@aguzzi_cloud_2020] vs. edge deployment required for responsiveness but unevaluated [@picone_flexible_2023], with entanglement-aware middleware migration proposed as the reconciliation [@bellavista_entanglement-aware_2024].
  - *Value narrative:* platforms positioned as affordable entry points [@talasila_realising_2024; @gil_survey_2024] vs. mixed pilot economics within a single project's own reporting -- several quantified positive results alongside one candid no-saving verdict [@noauthor_case_nodate] -- under an institutional judgment that publicity outweighs evidence [@committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024].
- **Strongest / weakest evidence:** Strongest -- the triangulated standard-vs-practice study (140-paper SLR + 33-expert survey + interviews) [@ferko_standardisation_2023] and the implemented same-case-study framework comparison [@gil_survey_2024]. Weakest -- market forecasts differing in scope with unaudited methodologies [@saracco_digital_2019; @lim_state---art_2020], whitepaper alignment claims asserted without validation [@heaton_platform_nodate], and composability/reusability claims never evaluated at scale.
- **Resolving question:** What would the same twin, built on two or more platforms with standardized metrics (twinning rate, fidelity, synchronization latency [@duran_toward_2026]) and accounted integration effort, actually reveal about platform value, lock-in cost, and the usefulness of the standardized-but-unimplemented functional entities?
- **Universal agreement:** the layered platform anatomy (connectivity, data, models, services, applications) recurs across every reference architecture; no platform ships a complete twin; documentation and maintenance quality gate platform selection before features matter; lock-in -- at bottom metamodel lock-in -- is real and motivates open standards, DTaaS, and federation; and open-source platforms and DTaaS offerings are consistently *positioned* as the SME's entry point, though that positioning is a stated aim rather than a measured outcome.
- **Blind spot:** twin retirement/decommissioning; quantified integration and switching costs; per-platform adoption or market-share data; empirical at-scale tests of composability claims; ISO 23247 outside manufacturing.

## Peer Review & Reliability Scorecard

The reconciled result of a four-role independent panel (no reviewer saw
another's critique). One panel run was interrupted by an infrastructure
limit; the three affected reviewers were re-run against the revised
draft.

| Finding | Confidence (1-10) | Why |
|---|---|---|
| No corpus-examined platform ships a complete twin | 9 | Implemented same-case-study comparison, verified quote-level by two reviewers; explicitly scoped to exclude commercial platforms |
| Standard-vs-practice mismatch (69% / 0%) | 9 | Only triangulated mixed-methods result in the corpus; every number re-verified verbatim by two reviewers |
| Operational phase least supported | 8 | Three independent research lines converge; adjacent AI-simulation figures downgraded to indirect evidence after review |
| Terminological inflation (models/shadows sold as twins) | 7 | Rescoped after review: five of fourteen graded frameworks do reach full closed-loop support, so the claim holds for the majority, not the field |
| Platform economics unproven | 6 | Evidence thin, self-reported, and correlated (largely one EU project's deliverables); pilot results mixed, not uniformly negative |

- **Panel verdicts:** domain-accuracy: needs revision (revised) -
  methodology-rigor: needs revision (revised) - clarity-completeness:
  needs revision (revised) - devils-advocate: needs revision (revised).
  All concerns meeting the concession threshold were addressed before
  presenting; none was silently dropped.
- **Concerns addressed** (met the concession threshold): a US$79.16B
  figure cited as a digital twin market valuation was traced to a
  *smart-home* market statistic in the source and removed, along with
  the "forecasts diverge 3-5x" claim built on it; the "most platforms
  are shadow platforms" thesis was rescoped after reviewers showed its
  supporting quote describes one framework (TerriaJS) while five of
  fourteen graded frameworks reach full closed-loop support; an
  attribution to Beaumont et al. of a "cannot git-revert the physical
  asset" limitation was found unsupported in the source and replaced
  with the paper's actual stated limitation; the economics narrative
  was rebalanced after reviewers found the same deliverable reports
  several quantified *positive* pilot results alongside the quoted
  negative one; duplicate citekeys for one paper were unified; the
  documentation-gate claim was corrected to the survey's actual
  multi-reason screening; the liu_ai_2025 maintenance figure was
  restated with its sample size and its internal 18%-vs-9.1%
  inconsistency, and downgraded to indirect evidence about platforms;
  a "concludes" framing of a premise in Zampetti et al. was corrected;
  the report's perspectives are now enumerated in the header; and the
  single-project (Change2Twin) dependence of the economics evidence is
  now disclosed in the text.
- **Concerns logged, not addressed** (below threshold): liu_ai_2025's
  internal 18%-vs-9.1% inconsistency cannot be resolved from this
  corpus, only flagged; commercial platform capability remains
  untested corpus-wide, so capability findings stay scoped to the
  open-source segment; market-forecast methodologies cannot be audited
  from the corpus; minor quotation-mark paraphrases were converted to
  indirect speech rather than re-quoted.
- **Weakest link:** the economics finding (confidence 6): largely one
  EU project's self-reported deliverables. What would verify it:
  independent post-project data on the Change2Twin marketplace's
  survival scenario, and realized (not designed) pricing and margins
  from any operating DTaaS offering.
- **Bias check:** the deliberate Skeptic lens guarantees an
  evidence-deficit theme, and the report's hidden-connection claim is
  qualified accordingly; gil_survey_2024 is the most-load-bearing
  single source (cited in every body section), partially offset by
  ferko_standardisation_2023 and the National Academies report as
  independent anchors.

## References

- **aguzzi_cloud_2020** -- From Cloud to Edge: Seamless Software Migration at the Era of the Web of Things (2020).
- **aheleroff_digital_2021** -- Digital Twin as a Service (DTaaS) in Industry 4.0: An Architecture Reference Model (2021).
- **aissat_devops_2025** -- A devops framework for the systematic engineering and evolution of digital twins for built assets (2025).
- **aissat_juno-ops_2024** -- JuNo-OPS: A DevOps Framework for the Engineering of Digital Twins for Built Assets (2024).
- **alcaraz_digital_2022** -- Digital Twin: A Comprehensive Survey of Security Threats (2022).
- **almeida_middleware_2023** -- Middleware for Digital Twins: A Systematic Mapping Study (2023).
- **alskaif_evolution_2025** -- Evolution at the Core of Digital Twin Engineering (2025).
- **barbie_toward_2024** -- Toward Reproducibility of Digital Twin Research: Exemplified with the PiCar-X (2024).
- **beaumont_towards_2025** -- Towards Automating the Life Cycle Management of Digital Twins (2025).
- **beetz_gitops_2022** -- GitOps: The Evolution of DevOps? (2022).
- **bellavista_entanglement-aware_2024** -- An Entanglement-Aware Middleware for Digital Twins (2024).
- **bellavista_exploiting_2024** -- Exploiting microservices and serverless for Digital Twins in the cloud-to-edge continuum (2024).
- **bordeleau_devops_nodate** -- A DevOps Approach for the Systematic Development and Evolution of Built Assets Digital Twins (n.d.).
- **callisto_de_donato_enabling_2024** -- Enabling 3D Simulation in ThingsBoard: A First Step Towards A Digital Twin Platform (2024).
- **cavalieri_proposal_2023** -- Proposal of Mapping Digital Twins Definition Language to Open Platform Communications Unified Architecture (2023).
- **combemale_model-based_2023** -- Model-Based DevOps: Foundations and Challenges (2023).
- **committee_on_foundational_research_gaps_and_future_directions_for_digital_twins_foundational_2024** -- Foundational Research Gaps and Future Directions for Digital Twins (2024).
- **duran_toward_2026** -- Toward Digital Twin-as-a-Service (DTaaS) Platforms: A Survey on Architecture, Design Requirements, and Performance Metrics (2026).
- **ferko_standardisation_2023** -- Standardisation in Digital Twin Architectures in Manufacturing (2023).
- **fitzgerald_potential_2024** -- The Potential of Digital Twins: Four Industry Perspectives (2024).
- **gil_survey_2024** -- Survey on open‐source digital twin frameworks–A case study approach (2024).
- **giulianelli_engineering_2024** -- Engineering Interoperable Ecosystems of Digital Twins: A Web-based Approach (2024).
- **gleich_asset_2024** -- An Asset Administration Shell-Based Digital Product Passport as a Gaia-X Service (2024).
- **grieves_digital_2024** -- Digital Twins and Their Role in Reengineering Engineering Education (2024).
- **hakiri_comprehensive_2024** -- A comprehensive survey on digital twin for future networks and emerging Internet of Things industry (2024).
- **heaton_platform_nodate** -- Platform Stack Architectural Framework:  An Introductory Guide (n.d.).
- **hugues_twinops_2020** -- TwinOps - DevOps meets model-based engineering and digital twins for the engineering of CPS (2020).
- **hugues_twinops_2022** -- Twinops: Digital twins meets devops (2022).
- **infante_distributed_2025** -- Distributed digital twins on the open-source OpenTwins framework (2025).
- **jacoby_open-source_2023** -- Open-Source Implementations of the Reactive Asset Administration Shell: A Survey (2023).
- **kamburjan_declarative_2024** -- Declarative Lifecycle Management in Digital Twins (2024).
- **kamburjan_digital_2022** -- Digital Twin Reconfiguration Using Asset Models (2022).
- **kayla_digital_2023** -- Digital Twin Platform Stack Architectural Framework by Digital Twin Consortium (2023).
- **kritzinger_digital_2018** -- Digital Twin in manufacturing: A categorical literature review and classification (2018).
- **larsen_hubcap_2022** -- HUBCAP: A Novel Collaborative Approach to Model-Based Design of Cyber-Physical Systems (2022).
- **lehner_pattern_2023** -- A pattern catalog for augmenting Digital Twin models with behavior (2023).
- **lehner_towards_2021** -- Towards Flexible Evolution of Digital Twins with Fluent APIs (2021).
- **lim_state---art_2020** -- A state-of-the-art survey of Digital Twin: techniques, engineering product lifecycle management and business innovation perspectives (2020).
- **liu_ai_2025** -- AI simulation by digital twins: systematic survey, reference framework, and mapping to a standardized architecture (2025).
- **mattila_interoperability_2025** -- Interoperability of Digital Twins for Automation With Digital Twin Schema (2025).
- **mckee_discs_2024** -- DISCS: An Approach for Accelerating the Development of Digital Twins for Smart Cities (2024).
- **mertens_continuous_2024** -- Continuous Evolution of Digital Twins using the DarTwin Notation (2024).
- **michael_model-driven_2025** -- Model-Driven Engineering for Digital Twins: Opportunities and Challenges (2025).
- **noauthor_assuring_2024** -- Assuring Trustworthiness in Dynamic Systems Using Digital Twins and Trust Vectors (2024).
- **noauthor_case_nodate** -- Case studies of digitalization for creating digital twins (n.d.).
- **noauthor_change2twin_nodate-3** -- Change2Twin marketplace design (n.d.).
- **noauthor_glossary_nodate-1** -- Glossary of Digital Twins by Digital Twin Consortium (n.d.).
- **noauthor_rise_2024** -- The rise of digital twins (2024).
- **noauthor_summary_nodate** -- Summary of IoT, and DT Standards (n.d.).
- **obaidat_hubcap_2022** -- HUBCAP: A Novel Collaborative Approach to Model-Based Design of Cyber-Physical Systems (2022).
- **parle_comparative_2024** -- A Comparative Analysis for Harnessing Digital Twin Platforms for Net-Zero Manufacturing (2024).
- **perno_implementation_2022** -- Implementation of digital twins in the process industry: A systematic literature review of enablers and barriers (2022).
- **pfeiffer_modeling_2022** -- Modeling capabilities of digital twin platforms-old wine in new bottles? (2022).
- **pfeiffer_towards_2025** -- Towards a Unifying Reference Model for Digital Twins of Cyber-Physical Systems (2025).
- **picone_flexible_2023** -- A Flexible and Modular Architecture for Edge Digital Twin: Implementation and Evaluation (2023).
- **picone_harmonizing_2025** -- Harmonizing Physical and Digital Twins Lifecycles (2025).
- **robles_opentwins_2023** -- OpenTwins: An open-source framework for the development of next-gen compositional digital twins (2023).
- **saracco_digital_2019** -- Digital Twins: Bridging Physical Space and Cyberspace (2019).
- **schmidt_increasing_2023** -- Increasing Interoperability between Digital Twin Standards and Specifications: Transformation of DTDL to AAS (2023).
- **shao_analysis_2023** -- An Analysis of the New ISO 23247 Series of Standards on Digital Twin Framework for Manufacturing (2023).
- **shao_use_2021** -- Use Case Scenarios for Digital Twin Implementation Based on ISO 23247 (2021).
- **singh_digital_2023** -- Digital Dataspace and Business Ecosystem Growth for Industrial Roll-to-Roll Label Printing Manufacturing: A Case Study (2023).
- **talasila_composable_2025** -- Composable digital twins on Digital Twin as a Service platform (2025).
- **talasila_realising_2024** -- Realising Digital Twins (2024).
- **tang_evaluation_2025** -- Evaluation framework for domain-specific digital twin platforms (2025).
- **tao_five-dimension_2019** -- Five-dimension digital twin model and its ten applications (2019).
- **tao_maketwin_2024** -- makeTwin: A reference architecture for digital twin software platform (2024).
- **vanderhorn_digital_2021** -- Digital Twin: Generalization, characterization and implementation (2021).
- **wermann_ktwin_2024** -- KTWIN: A Serverless Kubernetes-based Digital Twin Platform (2024).
- **zampetti_continuous_2023** -- Continuous Integration and Delivery Practices for Cyber-Physical Systems: An Interview-Based Study (2023).
- **zech_digital-twins-as--service_2024** -- Digital-Twins-as-a-Service in Construction Engineering (2024).
