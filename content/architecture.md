---
title: Architecture
description: >
  How the LUME packages fit together, from the simulator-agnostic base
  classes down to facility-specific staged models.
---

LUME is organized in three tiers. A simulator-agnostic base package defines
the common model interface, simulator-specific packages adapt individual
codes to that interface, and facility-specific packages compose those models
into staged pipelines for real machines.

```mermaid
flowchart TB
  subgraph tier1["Simulator Agnostic"]
    direction LR
    base["<b>LUME-Base</b><br/>LUMEModel<br/>StagedModel(LUMEModel)<br/>Variables / Actions"]
    pva["<b>LUME-pva</b><br/>EPICS PV serving"]
    base --> pva
  end

  subgraph tier2["Facility Agnostic"]
    impact["<b>LUME-Impact</b>"]
    bmad["<b>LUME-Bmad</b>"]
    cheetah["<b>LUME-Cheetah</b>"]
    torch["<b>LUME-Torch</b>"]
    genesis["<b>LUME-Genesis</b>"]
  end

  subgraph tier3["Facility Specific"]
    direction LR
    userinjector["UserInjector"] --> userlinac["UserLinac"] --> userfel["UserFEL"]
  end

  base --> impact
  base --> bmad
  base --> cheetah
  base --> torch
  base --> genesis

  impact --> userinjector
  bmad --> userlinac
  genesis --> userfel

  classDef pkg fill:#12151d,stroke:#6b84b3,color:#e7e9ee;
  classDef dev fill:#12151d,stroke:#6b84b3,stroke-dasharray:6 4,color:#e7e9ee;
  classDef stage fill:#1e40af,stroke:#6b84b3,color:#e7e9ee;
  class base,pva,impact,bmad,cheetah,torch pkg;
  class genesis dev;
  class userinjector,userlinac,userfel stage;

  linkStyle 1,2,8,9,10 stroke:#4ade80,stroke-width:2px;
```

## The tiers

**Simulator agnostic.** [LUME-Base](https://github.com/lume-science/lume-base)
defines the core abstractions shared by every package: `LUMEModel`, the
`StagedModel` composition of models, and the variable/action interface.
LUME-pva builds on it to serve model variables as EPICS PVs.

**Facility agnostic.** Each simulation code gets a thin adapter package
(LUME-Impact, LUME-Bmad, LUME-Cheetah, LUME-Torch, and the in-development
LUME-Genesis) whose model class subclasses `LumeModel`. These packages know
about their simulator but nothing about any particular machine.

**Facility specific.** Real machines are modeled by composing the adapters
into a `StagedModel`. For example, a facility might chain a `UserInjector`
stage (Impact), a `UserLinac` stage (Bmad), and a `UserFEL` stage (Genesis)
into a `UserFacilityModel` that simulates the machine end to end.
