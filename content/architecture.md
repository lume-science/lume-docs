---
title: Architecture
description: >
  How the LUME packages fit together, from the simulator-agnostic base
  classes down to facility-specific staged models.
---

LUME is organized in three tiers.
A simulator-agnostic base package defines the common base and model interface, which is consumed by other parts of the ecosystem such as LUME-pva for EPICS PV serving.
Simulator-specific packages wrap individual codes in a convenient Python layer and adapt them to the LUME interface.
Facility-specific packages are developed by users and can integrate multiple codes into staged pipelines that represent real machines.

## Interface Level

```mermaid
flowchart BT
  base["<b>LUME-Base</b><br/><span style='font-size:0.75em'>LUMEModel &middot; StagedModel<br/>Variables / Actions</span>"]
  pva["<b>LUME-pva</b><br/><span style='font-size:0.75em'>EPICS PV serving</span>"]
  pva -->|uses| base

  classDef pkg fill:#12151d,stroke:#6b84b3,color:#e7e9ee;
  class base,pva pkg;
```

[LUME-Base](https://github.com/lume-science/lume-base) defines the core abstractions shared by every package.
It contains the standard dict-like method of interacting with simulation tools.
The `LUMEModel` interface provides a standard way to expose what users may interact with and how to interact with them in physics simulations with state (through `Variable` objects).
These can be chained using a `StagedModel`.
LUME-pva builds on the `LUMEModel` interface to serve model variables as EPICS PVs.

## Simulation Codes Level

```mermaid
flowchart TB
  base["<b>LUME-Base</b>"]
  base --> impact["<b>LUME-Impact</b>"]
  base --> bmad["<b>LUME-Bmad</b>"]
  base --> cheetah["<b>LUME-Cheetah</b>"]
  base --> torch["<b>LUME-Torch</b>"]
  base --> genesis["<b>LUME-Genesis</b>"]

  classDef pkg fill:#12151d,stroke:#6b84b3,color:#e7e9ee;
  class base,impact,bmad,cheetah,torch,genesis pkg;
```

Each physics simulation tool gets a Python wrapper package (LUME-Impact, LUME-Bmad, LUME-Cheetah, LUME-Torch, and the in-development LUME-Genesis).
These packages define a Python interface for interacting with the codes and also include "batteries-included" `LUMEModel` objects specialized to each code.
These help by automatically generating variables, which can then be extended with custom actions as required, from pre-loaded simulations of user lattices.

## User Implementation Level

```mermaid
flowchart LR
  subgraph s1["Injector stage"]
    direction TB
    impact["<b>LUME-Impact</b>"] --> userinjector["UserInjector"]
  end
  subgraph s2["Linac stage"]
    direction TB
    bmad["<b>LUME-Bmad</b>"] --> userlinac["UserLinac"]
  end
  subgraph s3["FEL stage"]
    direction TB
    genesis["<b>LUME-Genesis</b>"] --> userfel["UserFEL"]
  end
  s1 --> s2 --> s3

  classDef pkg fill:#12151d,stroke:#6b84b3,color:#e7e9ee;
  classDef stage fill:#1e40af,stroke:#6b84b3,color:#e7e9ee;
  class impact,bmad,genesis pkg;
  class userinjector,userlinac,userfel stage;
```

Real machines are modeled by composing the simulator-specific wrapper classes into a `StagedModel`.
Each stage subclasses objects from the LUME package shown above it, and the stages are chained left to right: a `UserInjector` stage (Impact) feeds a `UserLinac` stage (Bmad), which feeds a `UserFEL` stage (Genesis), forming a `UserFacilityModel` that simulates the machine end to end.
Through the `LUMEModel` interface, this chained simulation can be connected to packages like `LUME-pva`, e.g., for controlling the model through EPICS PVs.
