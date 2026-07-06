---
hide:
  - navigation
  - toc
---

# Software

In the interest of reproducible research, all implementations of our research work
have been made available as open source software.

See the [Project Portfolio](portfolio.md) for detailed system architecture diagrams and
technology stacks behind the software listed below.

## Digital Twin as a Service (DTaaS)

The DTaaS is a software platform to **Build, Use and Share** digital twins (DTs). DTaaS enables
users to construct digital twins from reusable DT assets (data, models, functions,
and tools), execute them on the platform, and share ready-to-use DTs with other users.
The platform is designed to reduce the effort of building digital twins for
Cyber-Physical Systems, making DT development accessible to a wide range of users
including Small and Medium-sized Enterprises.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| DTaaS | [GitHub](https://github.com/into-cps-association/DTaaS) | [DTaaS v1.0](https://github.com/into-cps-association/DTaaS/releases) | [website](https://into-cps-association.github.io/DTaaS) |

## Simulation Bridge

Simulation Bridge is a Python-based distributed simulation middleware that routes requests between clients and simulation engines via RabbitMQ as a common messaging backbone. It supports multiple protocol adapters (REST, MQTT, RabbitMQ, and in-memory) and includes JWT-secured REST endpoints for streaming real-time results. Simulation agents for MATLAB and SIMUL8 fulfill simulation requests, supporting batch, streaming, and interactive modes.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| Simulation-Bridge | [GitHub](https://github.com/INTO-CPS-Association/simulation-bridge) | [Releases](https://github.com/INTO-CPS-Association/simulation-bridge/releases) | [website](https://github.com/INTO-CPS-Association/simulation-bridge/blob/main/USERGUIDE.md) |

## Plant Controller

Plant Controller is a Raspberry Pi-based edge controller for monitoring and managing plants. It reads soil moisture (Adafruit Seesaw), temperature and humidity (SHT45), and light spectrum (AS7341) sensors at configurable intervals, controls multiple water pumps on daily schedules, stores time-series sensor data in InfluxDB, and receives remote actuation commands via a STOMP message broker. A companion digital twin for the plant setup and the edge device is under development.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| Plant-Controller | [GitHub](https://github.com/into-cps-association/plant-controller) | [Releases](https://github.com/into-cps-association/plant-controller/releases) | [README](https://github.com/into-cps-association/plant-controller#readme) |

## AutolabJS

An automated evaluation platform for programming labs. The software supports programs
in C, C++, Java and Python programming languages. The software is capable of utilizing
different testing strategies (unit, integration, functional tests and I/O tests are
supported).
We also provide a client software named autolabcli which helps users interact with
AutolabJS evaluation server.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| AutolabJS | [GitHub](https://github.com/AutolabJS/AutolabJS) | [AutolabJS v0.4.0](https://github.com/AutolabJS/AutolabJS/releases/tag/autolabjs-v0.4.0) | [wiki](https://github.com/AutolabJS/AutolabJS/wiki/v0.4.0), [website](https://autolabjs.github.io) |
| autolabcli | [GitHub](https://github.com/AutolabJS/autolabcli) | [autolabcli v1.0.0](https://www.npmjs.com/package/@autolabjs/autolabcli) | [wiki](https://github.com/AutolabJS/autolabcli/wiki/Assignment-Submission-using-autolab-CLI), [website](https://autolabjs.github.io/docs/autolabcli/) |

## Social Networks

Observe online social network structure using IRC chat logs, mailing lists etc.
We wish to deduce communication patterns in these online platforms, and also explain
the multiplex nature of online social networks. Determining multiplex network structure
would provide baseline input to normative discussion on optimal structures of multiplex
networks.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| IRCLogParser | [GitHub](https://github.com/prasadtalasila/IRCLogParser) | [IRCLogParser v1.1.1](https://github.com/prasadtalasila/IRCLogParser/releases/tag/v1.1.1) | [website](http://prasadtalasila.github.io/IRCLogParser/), [wiki](https://github.com/prasadtalasila/IRCLogParser/wiki) |
| MLCAT | [GitHub](https://github.com/DeveloperCAP/MLCAT) | [MLCAT v0.1.2](https://github.com/DeveloperCAP/MLCAT/releases/tag/v0.1.2) | [website](http://developercap.github.io/MLCAT/), [wiki](https://github.com/DeveloperCAP/MLCAT/wiki) |

## Protocol Analysis

We are interested in developing tools for sound network measurements. We are developing
a tool to perform custom packet analysis.
GitHub repository containing relevant code is: [BITS-Darshini](https://github.com/prasadtalasila/BITS-Darshini).

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| BITS Darshini | [GitHub](https://github.com/prasadtalasila/BITS-Darshini) | [BITS Darshini v2.1](https://github.com/prasadtalasila/BITS-Darshini/releases/tag/v2.1) | [website](http://prasad.talasila.in/BITS-Darshini/maven), [wiki](https://github.com/prasadtalasila/BITS-Darshini/wiki) |

## Transport Networks

Provide search service for travelers using multi-modal transport service. Here,
multi-modal means multiple means of transport, say, train, bus, flight, boat etc.
In order to provide the multi-modal transport search service, the first step is to
collect schedules of transport operators. The second step is to develop algorithms
that would dynamically search the given schedules for a source-destination pair.
The final step is to develop a web application that would provide search service
to interested travelers.

| Software Name | Source Code | Releases | Documentation |
|-------- |:----------:|:----------:|:----------:|
| Transport Scheduler | [GitHub](https://github.com/prasadtalasila/TransportScheduler) | [Transport Scheduler v0.1.0](https://github.com/prasadtalasila/TransportScheduler/releases/tag/v0.1.0) | [website](http://prasadtalasila.github.io/TransportScheduler), [wiki](https://github.com/prasadtalasila/TransportScheduler/wiki) |
