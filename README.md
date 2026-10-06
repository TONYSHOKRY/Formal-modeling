# Formal Modelling of Blinky Block Modular Robotic Systems

This repository contains the executable probabilistic models developed for the
formal-modelling part of the Blinky Block modular robotic system case study.

The models are written in the **PRISM modelling language** and are intended for
analysis with **PRISM** and, where applicable, **Storm**.

This repository accompanies the PhD work on module-based probabilistic modelling,
abstraction, scalability, energy-aware analysis, and architectural adaptation of
distributed modular robotic systems.

---

## 1. Modelling principle

The physical Blinky Block system is represented at the **task and cluster level**.

A cluster is a group of adjacent Blinky Blocks executing the same assigned command.
Instead of creating one complete behavioural model for every physical block, a
cluster is represented by one model module acting as a macro-agent.

Two quantities must therefore be distinguished:

- `N` — number of modelled cluster modules;
- `N_i` — number of physical Blinky Blocks represented by cluster `i`.

For example, a two-cluster configuration may have

```text
N = 2
N_1 = 10
N_2 = 10
```

This means that the model contains two cluster modules, while each cluster
represents ten physical Blinky Blocks.

In the standard model families, `N_i` is a fixed cluster-size parameter used,
among other things, to scale the corresponding energy reward.

In the architectural-extension model `L_2`, the cluster cardinalities become
model variables and can be modified during execution.

---

## 2. Model families

The repository contains several structurally related model families.

### 2.1 Full models — `F_N`

The full models retain the most detailed task-level behaviour used in this work,
including:

- task execution;
- colour selection;
- LED actuation;
- pitch actuation;
- combined LED/pitch actuation;
- sensing;
- hardware-load conditions;
- environment conditions;
- synchronous and staggered scheduling;
- success and task-failure outcomes.

Implemented multi-cluster instances include:

```text
F_2
F_3
F_4
F_5
F_10
```

The one-cluster baseline is denoted:

```text
F_0
```

`F_0` is a retained model identifier and does **not** mean that the model contains
zero clusters.

The multi-cluster models are separately authored executable artefacts following
the same structural replication pattern.

The absence of `F_6`–`F_9` means that executable instances with these identifiers
are not provided. It does not imply an executable generation chain between the
available models.

---

### 2.2 Environment-abstraction models — `E_N`

The environment-abstraction family reduces the explicitly represented sensing
behaviour while retaining the Environment module.

Implemented instances are:

```text
E_2
E_3
E_4
```

Relative to the corresponding full models, the explicit sensing control states
are removed and the associated sensing transitions are simplified.

---

### 2.3 Full-abstraction models — `A_N`

The full-abstraction family provides a coarser behavioural representation.

Implemented paired instances include:

```text
A_2
A_3
A_4
```

A standalone one-cluster abstraction also exists:

```text
A_1
```

The full abstraction merges selected control states and restricts selected
variable domains in order to reduce the represented behavioural state space.

The structural relation between the full and abstract models does **not**
automatically establish simulation, bisimulation, refinement, or property
preservation. Each executable model is therefore analysed independently.

---

### 2.4 Architectural-extension model — `L_2`

`L_2` is an architectural extension of the two-cluster full model `F_2`.

It introduces additional structural-adaptation elements, including:

- cluster-cardinality variables `N1` and `N2`;
- dummy-cluster cardinality `N_dummy`;
- a dummy cluster module;
- adaptation variables;
- structural-adaptation transitions.

Representative actions include:

```text
Resize_clusters
Extend_clusters
activate_dummy_1
deactivate_dummy
```

The model represents several structural adaptation cases.

**Resizing** redistributes represented blocks between existing clusters.

**Extension** increases the represented population by increasing cluster
cardinalities.

**Dummy activation** reallocates part of an existing cluster to an additional
logical cluster module.

`L_2` is therefore **not an abstraction of `F_2`**. It adds model structure,
variables, and adaptation transitions.

---

## 3. Main model components

Depending on the selected model family, the executable models contain some or all
of the following modules:

```text
c1
c2
...
cN

Hardware
Environment
pattern
Task_Controller
Operator
```

The one-cluster baseline uses `BB1` as its cluster-module name and does not contain
the multi-cluster `Operator` module.

A cluster module represents task-level behaviour such as:

- task initiation;
- colour selection;
- sensing;
- pitch actuation;
- LED actuation;
- simultaneous LED/pitch actuation;
- high-, medium-, and low-intensity alternatives;
- success;
- task failure.

Modules may read variables owned by other modules. Synchronisation is realised
using shared PRISM action labels.

---

## 4. Scheduling and synchronisation

The models distinguish two execution modes.

### Synchronous scheduling

Active clusters progress together through shared labels such as:

```text
[step]
```

The participating modules synchronise on the shared label and jointly determine
the global successor state.

### Staggered scheduling

Only the cluster selected by the turn mechanism progresses.

After a cluster terminates its task, turn-handoff actions transfer activity to
another cluster.

Sensing behaviour can similarly synchronise a cluster module with the Environment
module through shared sensing labels.

---

## 5. Probabilistic parameters

The model structure is separated from the numerical calibration of the
probabilistic transitions.

Representative base success-probability parameters include:

```text
q_r_high_c1
q_b_high_c1
q_w_high_c1
q_high_pitch_c1
```

The reference cluster is the feeder-side cluster.

The models also use contextual multipliers such as:

```text
theta
gamma
alpha_i
beta1
beta2
```

Their intended roles are:

| Parameter | Meaning |
|---|---|
| `theta` | motif effect |
| `gamma` | scheduling effect |
| `alpha_i` | cluster-location effect |
| `beta1` | medium-intensity fallback effect |
| `beta2` | low-intensity fallback effect |

The `q` parameters are base success probabilities.

Context-specific branch probabilities are constructed from the base probability
and the multipliers relevant to the corresponding transition.

Numerical parameter values must produce valid probability distributions for all
probabilistic branches.

---

## 6. Energy reward

The executable models contain a PRISM reward structure named:

```text
energy
```

For cluster `i`, its cardinality `N_i` represents the number of physical Blinky
Blocks represented by that cluster.

In the standard models, the cluster cardinality is used to scale the corresponding
energy contribution.

For example:

```text
N_1 = 10
N_2 = 10
```

describes two modelled clusters, each representing ten physical blocks.

The numerical calibration and physical interpretation of the reward values are
treated separately from the structural definition of the model.

---

## 7. Verification properties

The earlier property suite associated with the initial formal-modelling work is
available in the companion repository:

### Formal-methods

https://github.com/TONYSHOKRY/Formal-methods

The general PRISM/PCTL property file is available there as:

```text
General_properties.pctl
```

Repository:

https://github.com/TONYSHOKRY/Formal-methods/blob/main/General_properties.pctl

The `Formal-methods` repository contains the earlier single-Blinky-Block and
two-cluster models together with their property-oriented material.

The present **Formal-modeling** repository contains the wider thesis model
collection and its structural variants.

When reusing a property with another model family, verify that all referenced:

- labels;
- variables;
- control states;
- constants;

exist in the selected model.

The abstraction models do not necessarily expose exactly the same state encoding
as the corresponding full models. Consequently, a property verified on one model
must not be assumed automatically to hold for another structural variant.

---

## 8. Running models with PRISM

The models can be opened using the PRISM graphical interface or analysed from the
command line.

A typical invocation is:

```bash
prism <model-file> <property-file>
```

Symbolic constants can be instantiated using `-const`:

```bash
prism <model-file> <property-file> \
  -const theta=<value>,gamma=<value>,beta1=<value>,beta2=<value>,...
```

The exact constants depend on the selected executable model.

Always inspect the declarations in the corresponding PRISM file before supplying
a valuation.

---

## 9. Running models with Storm

The PRISM-language files can also be analysed using Storm.

A typical invocation is:

```bash
storm --prism <model-file> --build:buildfull
```

Larger models may require a different Storm engine or additional memory options.

The use of PRISM or Storm changes the analysis backend but does not change the
intended model semantics.

Scalability results and engine-specific experiments should always report the
model file, engine, parameter valuation, and relevant execution options.

---

## 10. Relationship between the two repositories

Two public repositories are maintained for different purposes.

### Formal-modeling

https://github.com/TONYSHOKRY/Formal-modeling

This is the main repository for the extended thesis model collection, including:

- multi-cluster full models;
- environment-abstraction models;
- full-abstraction models;
- architectural adaptation;
- scalability-oriented model variants.

### Formal-methods

https://github.com/TONYSHOKRY/Formal-methods

This repository contains the earlier/paper-oriented artefacts, including:

- the single-Blinky-Block model;
- the two-cluster model;
- the general verification-property suite.

For the wider thesis model families, use **Formal-modeling**.

For the earlier property definitions and their examples, consult
**Formal-methods** and adapt the property only after checking its compatibility
with the selected model.

---

## 11. Scope of the model relations

The relations between the models in this repository are **structural modelling
relations**.

In particular, the repository does not claim that:

- `F_N -> E_N` establishes a formal refinement;
- `F_N -> A_N` establishes simulation or bisimulation;
- properties are automatically preserved between model families;
- `L_2` is an abstraction of `F_2`;
- every value of `N` corresponds to an implemented executable model.

Each executable model is analysed independently.

---

## 12. Physical abstraction

The models operate at the discrete task and cluster level.

The following physical quantities are not represented as continuous state
variables in these PRISM models:

- continuous voltage trajectories;
- continuous current trajectories;
- per-block voltage-regime fields;
- detailed electrical transients;
- complete feeder-path dynamics.

Experimental observations of these quantities motivate the abstraction and are
used to inform the numerical calibration of the probabilistic models.

---

## 13. Associated publication

Part of the modelling approach was reported in:

**Antonios Naguib et al.**  
*Module-based Modeling and Assessment of Modular Robots W.R.T Energy Efficiency*  
ECMFA 2026.

The thesis work extends the executable model collection beyond the models used in
the associated publication.

---

## 14. Reproducibility information

For every quantitative verification or scalability experiment, record at least:

- exact model file;
- exact property file;
- PRISM or Storm version;
- complete parameter valuation;
- analysis engine;
- memory-related options when relevant;
- command used to execute the analysis.

Example:

```text
Model:
<model-file>

Properties:
<property-file>

Tool:
PRISM / Storm

Parameters:
theta = ...
gamma = ...
alpha2 = ...
beta1 = ...
beta2 = ...
q_r_high_c1 = ...
q_b_high_c1 = ...
q_w_high_c1 = ...
q_high_pitch_c1 = ...

Engine:
...

Command:
...
```

This information is necessary to reproduce a particular numerical result.

---

## 15. Repository status

This repository accompanies ongoing PhD research.

The executable model files are the authoritative source for the implemented
model structure. Documentation, property suites, and analysis scripts may be
refined as the dissertation and associated reproducibility material are
finalised.
