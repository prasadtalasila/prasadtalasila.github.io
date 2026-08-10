---
hide:
  - navigation
  - toc
---

# Digital Twins for Software Engineers

## Disclaimer

>This book has been generated using
>[chitragupta](https://prasad.talasila.in/chitragupta).
>Despite some potential for hallucination, the ideas communicated in this
>book are accurate. Please send your corrections and suggestions to
><prasad.talasila@gmail.com>

## Audience

Software engineers and programmers who need an overall working knowledge
of digital twins so they can collaborate with modeling experts, business managers,
and customers.

## Demonstrator

The book will be written by using a plant, pot and pump demonstrator where the goal
is to control the soil moisture in the pot.
There is a pre-existing physical twin (system) demo setup that will be used in this book.
The demo detials are available on
[GitHub](https://github.com/INTO-CPS-Association/plant-controller). Also see
[hardware implementation](https://github.com/INTO-CPS-Association/plant-controller/tree/main/docs/pt/controller_3)
and the
[firmware](https://github.com/INTO-CPS-Association/plant-controller/tree/main/pt/controller_3)
for Raspberry Pi-based firmware that samples sensors and controls the pumps
to control soil moisture.
The matching digital twin is not available yet. It needs to be constructed as
a running example in this book along the way.

## Part I — The Big Picture

*So you can hold your own in the kickoff meeting.*

1. [Why Anyone Pays for a Digital Twin: Value, Markets, and Real Deployments](chapters/01-why-anyone-pays-for-a-digital-twin.md)
2. [Twin, Shadow, Model, Simulation: What a Digital Twin Actually Is — and Isn't](chapters/02-twin-shadow-model-simulation.md)
3. [The Anatomy of a Twin: A Reference Architecture in Software Terms](chapters/03-anatomy-of-a-twin.md)

## Part II — Modeling and Simulation Literacy

*Your interface to the modeling experts — enough to ask good questions, not to
build models.*

<ol start="4">
<li><a href="chapters/04-just-enough-modeling.html">Just Enough Modeling: Physics-Based, Data-Driven, and Hybrid Models</a></li>
<li><a href="chapters/05-just-enough-simulation.html">Just Enough Simulation: State, Time, Solvers, and Co-Simulation</a></li>
<li><a href="chapters/06-simulators.html">Simulators: How Each Kind of Model Gets Solved</a></li>
<li><a href="chapters/07-should-you-trust-the-twin.html">Should You Trust the Twin? Calibration, Credibility, and V&amp;V</a></li>
<li><a href="chapters/08-where-ai-fits.html">Where AI Fits: Machine Learning as a Model, a Service, and a Risk</a></li>
</ol>

## Part III — Building the Twin

*Your home turf — the deep part of the book.*

<ol start="9">
<li><a href="chapters/09-connecting-the-physical.html">Connecting the Physical: Sensors, Protocols, and Streaming Data</a></li>
<li><a href="chapters/10-data-engineering-for-twins.html">Data Engineering for Twins: Time Series, Context, and Provenance</a></li>
<li><a href="chapters/11-twin-services.html">Twin Services: Visualization, Monitoring, Prediction, and Decision Support</a></li>
<li><a href="chapters/12-platforms-and-composability.html">Platforms and Composability: Buying, Building, and Assembling from Parts</a></li>
<li><a href="chapters/13-standards-and-open-source.html">Standards and Open Source: What Exists So You Don't Build It Twice</a></li>
<li><a href="chapters/14-running-twins-in-production.html">Running Twins in Production: Deployment, Evolution, and the Twin's Own Lifecycle</a></li>
<li><a href="chapters/15-twins-at-scale.html">Twins at Scale: Ecosystems</a></li>
</ol>
