# Overview
## Design Influences from Human Models

Chordline’s approach to Affective Elements is informed by established psychological, counseling, and physiological models of emotion and affect.
These bodies of work provided valuable guidance in identifying patterns of human experience and in highlighting the complexity of emotional interaction.

At the same time, Chordline does not attempt to reproduce any of these models directly.
The goals of a system intended for use in interactive media differ significantly from those of clinical or scientific frameworks. Fidelity to lived human psychology is therefore balanced against the need for clarity, consistency, and usability within a design context.
Where divergences occur, they reflect the intent to create a system that is both expressive for authors and computationally tractable for games.
## Nature of Affective Elements

Within Chordline, _Affective Elements_ are the fundamental units for representing emotion and affect. Each element holds a value that quantifies the intensity of a state, providing a consistent foundation for modeling preference, decision-making, and relational dynamics.

These elements are abstract by design.
They are not tied to a single psychological theory or taxonomy, but instead act as placeholders that can represent primary emotions, composite states, physiological pressure, a complex mental model, or anything else that a designer finds useful.
This abstraction ensures flexibility: authors may align them with familiar emotional categories, culturally specific concepts, or novel constructs suited to the needs of their project.

The intent is not to replicate the full richness of human psychology but to offer a structured, tractable system for interactive media.
By reducing affect to quantifiable values, Chordline enables emotional complexity to be expressed consistently and manipulated reliably within the context of a game or simulation.
## Storage and Value Range

Affective Element values are stored internally as floating point numbers within the range `[0,1]`.
This normalized range ensures consistency across all uses of the system, allowing different elements and composites to be compared and combined without ambiguity.

For user-facing interfaces, such as authoring tools and debugging outputs, values are scaled to the range `[0,10]`.
This convention is intended for human convenience, providing a scale that is easier to read and reason about while preserving the precision of the internal representation.

Composite Affective Elements do not store or cache values. Instead, they are evaluated as _read-through_ expressions: their value is computed dynamically from their constituent elements each time it is accessed.
This approach guarantees that composite results always reflect the current state of their parts, without risk of becoming outdated.
## Composability

Affective Elements can be combined into composites, allowing more complex emotional patterns to be represented.
Beyond serving as a direct combination of their sources, composites typically expose additional properties that give quantitative insight into the relationship between the contributing elements.
These properties describe qualities that emerge from the interaction itself, rather than from the individual parts, and help capture the nuance of complex affective states.

Many composites are also _write-through_.
In these cases, assigning a value to the composite distributes that change back into its constituent elements according to the rules of the composition.
This makes composites not only useful for observing combined affective states but also practical tools for adjusting related elements together in a controlled and coherent manner.
## Standard Uses in Core

Within the Chordline Core, Affective Elements provide the foundation for several key systems.
They are central to the representation of preferences, shaping how entities evaluate options and form inclinations.
They also interact directly with Chordline’s Event system — the narrative building blocks of the framework — by serving as both inputs and outcomes of Events.
This allows affective change to be expressed as part of a story’s progression rather than as isolated data updates.

In addition, Affective Elements support relationship modeling, where multiple values combine to capture the evolving dynamics between entities.
Through these roles, they act as the connective tissue linking emotional state, narrative structure, and systemic response.
By maintaining a consistent numerical form, they ensure that affective information can be used seamlessly across the different parts of Core while remaining adaptable to a project’s specific design goals.
## Selector Functions

Selector functions provide a mechanism for choosing among Affective Elements based on Context.
They allow the system to determine which affective values are relevant in a given situation, and how those values should be retrieved or combined.

Because Context can shift depending on narrative, entity, target, or scope, selectors play an important role in making Affective Elements responsive to circumstance.
They act as the bridge between raw affective data and the situational logic that determines how that data is used.

Details of selector design and usage extend beyond the scope of this overview and are addressed in a later section.
Here, it is sufficient to note that selectors are included in this document because their behavior is inseparable from how Affective Elements function within Chordline.
## Relationship to Plugins

Affective Elements are extended and adapted through plugins, which provide standardized ways of enriching their behavior beyond the Core.
Planned “standard library” plugins will introduce additional composites, new forms of derived properties, and specialized mechanisms for interacting with affective data.
These plugins build on the same foundational structures, ensuring consistency while expanding expressive range.

Chordline’s own _standard model_ for emotions — the framework of affects and composites described in this document — is itself delivered as an optional plugin.
This design choice emphasizes that no single emotional model is mandatory.
Authors may adopt the standard model, customize it, or replace it entirely with alternatives, depending on the goals of their project.

By situating this functionality in plugins rather than Core, Chordline maintains a balance between stability and flexibility.
Core defines the essential rules and guarantees, while plugins allow authors to adopt or omit features according to the needs of their projects.
This modular approach ensures that Affective Elements can serve both minimal implementations and richly elaborated emotional systems without conflict.

# Core Affective Elements and Compositions
## Introduction
This section describes the guarantees provided by **Core**, the foundational layer of Chordline that defines how Affective Elements and their Compositions function before any plugins are added.
Core ensures that every project has access to a minimal but complete set of tools for representing and manipulating affect, with well-defined behaviors and consistent logical interfaces.

At the base level, Core guarantees that every Affect stores a value within the range `[0,1]`, supports both reading and writing of that value, and participates in an observer pattern to notify when its value changes.
On top of these base elements, Core provides a set of standardized Compositions that define how multiple Affects can be related, blended, or transformed.
These Compositions include both general-purpose forms, such as weighted averages, and specialized patterns designed to capture particular relationships between affects.

The material in this section is limited to Core itself.
More elaborate models, additional Compositions, and higher-level emotional structures are delivered as plugins, building on the guarantees defined here.
## Base Affective Elements in Core
Core defines a minimal but complete set of guarantees for individual Affective Elements, ensuring that every project has a consistent and predictable foundation.
### Creation and Identity
Affective Elements are created explicitly and may be referenced by Compositions through object reference rather than name.
Each Affect can be given a **name**, which serves as a self-identification label but is not required to be globally unique.
In most implementations, Affects will be stored in a multi-level associative array on an entity, keyed by **Name**, **Target**, and **Context** (where both Target and Context may be empty).
### Access Methods
Values cannot be directly read or written from the stored field. Instead, access occurs through defined methods:

- **User-Level Access** (values are scaled to `[0,10]` for human readability):
    
    - Retrieve the primary value.
        
    - Retrieve secondary metrics or properties (Name, Target, Context) in addition to the primary value.
        
    - Adjust the value by a delta.
        
    - Set the value to an exact number (implemented internally via a delta adjustment).
        
- **System-Level Access** (values remain in `[0,1]`):
    
    - Retrieve the primary value.
        
    - Retrieve secondary metrics.
        
    - Adjust the value by a delta.
        

All setter-style methods accept any numeric float as input.
Values are clamped to the valid range.
### Write Semantics
Base Affects themselves do not support write-through behavior; this applies only to Compositions.
All direct updates (through user- or system-level APIs) are clamped as necessary before being applied.
### Observer and Notification Pattern
Affects implement a notification system to propagate changes. Every update generates a notification event, regardless of whether the change was redundant or due to clamping.
Notifications include:

- Unique ID (base update ID with qualifiers as it propagates).
    
- Propagation path.
    
- Before value.
    
- After value.
    
- Delta.
    
- Wall-clock timestamp.
    
- Simulation timestamp.

Observation uses a singleton registry that allows listeners to subscribe at multiple granularities:

- To a single Affect object.
    
- To an Affect owner.
    
- To a target.
    
- To a context.
    
- To all Affects globally.
### Back-Propagation Rules
When a Composition distributes changes back into its components, Core enforces strict safeguards:

- Back-propagation is only allowed if all components permit modification or propagation.
    
- No base Affect may appear more than once in the component list.
    
- For these rules, Read-Only compositions always count as unique, even if repeated.
## Compositions in Core
Compositions define structured ways of relating, blending, or transforming Affective Elements.
They extend the capabilities of base Affects by enabling authors to express more complex dynamics such as interactions, contrasts, or emergent qualities.
Core provides a guaranteed set of Compositions that are always available, ensuring that every project has consistent tools for constructing higher-level affective structures.

Each Composition type is defined by:

- **Purpose**: the kind of affective relationship it was designed to model.
    
- **Mathematical Model**: the precise method by which the Composition produces its value.
    
- **Write-Through Behavior**: the rules governing how changes propagate back into constituent Affects.
    
- **Properties**: any additional metrics that provide quantitative insight into the relationship between composing Affects.
    
- **Example**: a simple illustration of its use.

This section will describe each Composition type provided by Core in turn, following this structure.

Note that it is intended to be easy to identify the Affect associated with metrics like min and max, but the method for doing so hasn't been determined yet.

### Alias

The **Alias** composition creates a secondary reference to an existing Affect.
It provides the same **Target** and **Context** as the base Affect but assigns a distinct **Name** at the object level.

An Alias is a complete proxy:

- Value access and secondary metrics are transparently forwarded to the base Affect.
    
- Modifications applied through the Alias are passed directly to the base Affect.
    
- All notification and observer behaviors are identical to interacting with the base Affect itself.
    

In every respect except for its Name, the Alias behaves exactly as if the base Affect were being accessed directly.
This makes it useful for cases where a single affective state needs to be referenced under different labels without duplication.

#### Example
```json
Attraction = Alias(Desire)
```
### Inversion
The **Inversion** composition produces an Affect whose value is the complement of a base Affect.
Let the base Affect be denoted as $A_B$​, and the inverted Affect as $A_I$​.
The relationship between them is:$$A_I=1−A_B$$​
When a delta $\Delta_I$​ is applied to the inverted Affect, the corresponding delta applied to the base Affect, $\Delta_B$​, is the negative of that value:$$\Delta_B=−\Delta_I$$​
All notifications generated by an Inversion report the inverted values as they would normally be read, without exposing the base Affect’s raw values.
This ensures consistency in how observers perceive changes, regardless of whether the update originated from $A_B$​ or $A_I$​.

#### Example
```json
Mistrust = Inversion(Trust)
```

### Simple Composition
The **Simple Composition** combines $n$ base Affects of equal weight into a single composite value.
With $n$ base Affects, $A_1,A_2,\dots,A_n$​, and the composite Affect $A_C$​.
The composite value is the arithmetic mean of its base values:$$A_C=\frac{1}{n}\sum_{i=1}^{n}A_i$$
#### Write-through Behavior
When a $\Delta_C$ is applied to the composite, it is redistributed to the base Affects in proportion to the “headroom” or “footroom” of each base, depending on whether the delta is positive or negative.
This ensures that no base Affect is pushed outside the valid $[0,1]$ range while keeping the composite consistent.

For a **positive delta**:$$\Delta_i = \frac{\Delta_C \cdot (1 - A_i)}{1 - A_C}$$
For a **negative delta**:$$\Delta_i = \frac{\Delta_C \cdot A_i}{A_C}$$
#### Additional Metrics
In addition to the composite value, Simple Compositions provide several derived metrics:

- Min:  $$Min=\min(A_1,A_2,A_3)$$
    
- Max: $$Max=\max(A_1,A_2,A_3)$$
        
- **Standard Deviation**: the statistical standard deviation of the base values.$$\sigma = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (A_i - A_C)^2}$$
    
- **Disparity**: **Standard Deviation** scaled from the range $[0,0.5]$ to $[0,1]$:$$Disparity = 2\sigma$$
    
- **Cohesion**: the product of all base values.
  Can be seen as the aggregate at its weakest, with destructive interference between the base affects.
  Is strictly $\le min$, and almost always below it.$$Cohesion = \prod_{i=1}^{n} A_i$$
    
- **Coverage**: the inverse of the product of the inverses of the base values.
  Mirror of Cohesion, but not its inverse.
  Can be seen as the aggregate at its strongest, with constructive reinforcement between the base affects.
  Is strictly $\ge max$, and almost always above it.$$Coverage = 1 - \prod_{i=1}^n (1 - A_i)$$
  
    

#### Example
```json
Discomfort = Compose(Hunger, Thirst, Fatigue, Pain, HeatStress, ColdStress)
```
### Weighted Composition
The **Weighted Composition** combines $n$ base Affects with individual weights.
Let the base Affects be $A_1, A_2, \dots, A_n$​, the weights $w_1, w_2, \dots, w_n$​ (not necessarily normalized), and the composite Affect $A_C$​.
The composite value is the weighted average with internal normalization:$$A_C=\frac{\sum_{i=1}^{n} w_i A_i}{\sum_{i=1}^{n} w_i}$$
#### Write-through Behavior

When a delta $\Delta_C$​ is applied to the composite, redistribution to base Affects uses the same headroom/footroom rule as Simple Composition.

For a positive delta: $$\Delta_i=\frac{\Delta_C\,(1-A_i)}{1-A_C}$$

For a negative delta:$$\Delta_i=\frac{\Delta_C\,A_i}{A_C}$$

#### Additional Metrics

- Min:  $$Min=\min(A_1,A_2,A_3)$$
    
- Max: $$Max=\max(A_1,A_2,A_3)$$
        
- **Weighted Standard Deviation** (population form, using the same, not-necessarily-normalized weights):$$\sigma_w=\sqrt{\frac{\sum_{i=1}^{n} w_i\,(A_i-A_C)^2}{\sum_{i=1}^{n} w_i}}$$
    
- **Weighted Disparity**: **Weighted Standard Deviation** scaled from the range $[0,0.5]$ to $[0,1]$:$$Disparity_w = 2\sigma_w$$
    
#### Example
```json
Blend = WeightedCompose({ Sorrow: 2.0, Anger: 1.0, Fear: 0.5 })
```

#### Potential Developments
- **Weight spread diagnostics (not yet part of Core):** candidate scalars to characterize how concentrated vs. uniform the weights are, such as normalized entropy of the normalized weights or a concentration index (e.g., sum of squared normalized weights).
    
- **Changeable weights:** support for updating weights after construction (e.g., to reflect shifting narrative emphasis or learned preferences).

### Dyad
The **Dyad** composition combines two base Affects with equal weight.
Let the base Affects be $A_1$​ and $A_2$​, and the composite Affect be $A_D$​.
The composite value is their arithmetic mean:
$$A_D = \frac{A_1 + A_2}{2}$$

#### Write-through Behavior
When a delta $\Delta_D$​ is applied to the composite, redistribution to the base Affects follows the same headroom/footroom rule used in Simple Composition.

For a **positive delta**:$$\Delta_i = \frac{\Delta_D \cdot (1 - A_i)}{1 - A_D}$$

For a **negative delta**:$$\Delta_i = \frac{\Delta_D \cdot A_i}{A_D}$$
#### Additional Metrics

- Min:  $$Min=\min(A_1,A_2,A_3)$$
    
- Max: $$Max=\max(A_1,A_2,A_3)$$
        
- **Cohesion** (also exposed under the name Synergy): the product of the two base values.
  See the description under **Simple Composition** for more details.$$Cohesion = A_1 \cdot A_2$$
    
- **Coverage**:
  See the description under **Simple Composition** for more details.$$Coverage = 1 - (1 - A_1) \cdot (1 - A_2) = (A_1 + A_2) - (A_1 \cdot A_2)$$
    
- **Standard Deviation**: the statistical standard deviation of the base values.$$\sigma = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (A_i - A_C)^2}$$
    
- **Disparity**: **Standard Deviation** scaled from the range $[0,0.5]$ to $[0,1]$:$$Disparity = 2\sigma$$
    
#### Example
```json
Hope = Dyad(Anticipation, Happiness)
```

### Triad

The **Triad** composition combines three base Affects with equal weight.
Let the base Affects be $A_1, A_2, A_3$​, and the composite Affect be $A_T$​.
The composite value is the arithmetic mean:$$A_T=\frac{A_1+A_2+A_3}{3}$$

#### Write-through Behavior
When a delta $\Delta_T$​ is applied to the composite, redistribution to the bases follows the headroom/footroom rule (as with Simple Composition).

For a positive delta: $$\Delta_i=\frac{\Delta_T\,(1-A_i)}{1-A_T}\qquad (i\in\{1,2,3\})$$

For a negative delta: $$\Delta_i=\frac{\Delta_T\,A_i}{A_T}\qquad (i\in\{1,2,3\})$$

#### Additional Metrics

- Min:  $$Min=\min(A_1,A_2,A_3)$$
    
- Max: $$Max=\max(A_1,A_2,A_3)$$
    
- Population Standard Deviation: $$\sigma=\sqrt{\frac{(A_1-A_T)^2+(A_2-A_T)^2+(A_3-A_T)^2}{3}}$$
    
- Disparity (scaled spread): $$Disparity=2\sigma$$
    
- Coverage:
  See the description under **Simple Composition** for more details. $$Coverage=1-(1-A_1)(1-A_2)(1-A_3)$$
    
- Cohesion:
  See the description under **Simple Composition** for more details. $$Cohesion=A_1A_2A_3$$
    

#### Potential Developments

- Triadic Synergy (average of pairwise products) — under consideration for future inclusion.
    

#### Example
```json
EngagedDelight = Triad(Surprise, Joy, Interest)
```

### Conflict Pair

The **Conflict Pair** composition models two Affects placed in narrative opposition, representing a tug-of-war dynamic.
Let the base Affects be $A_L$ (the “low” or diminishing side) and $A_H$​ (the “high” or reinforcing side), and let the composite be $A_V$​.
The composite value is defined as:$$A_V = \frac{1 - A_L + A_H}{2}$$

This formulation ensures that as $A_L$​ increases, $A_V$​ decreases, while increases in $A_H​$ drive $A_V$​ upward. The result is always constrained to the range $[0,1]$.

#### Write-through Behavior

When a delta $\Delta_V$​ is applied to the composite, it is redistributed between $A_L$​ and $A_H$​ using slack-aware portioning.
Both base Affects are always adjusted unless one has reached a limit of 0 or 1.

For a positive delta:$$\Delta_L = -\Delta_V \cdot A_L \qquad \Delta_H = \Delta_V \cdot (1 - A_H)$$

For a negative delta:$$\Delta_L = -\Delta_V \cdot (1 - A_L) \qquad \Delta_H = \Delta_V \cdot A_H$$

#### Additional Metrics

- Min:$$Min = \min(A_L, A_H)$$
    
- Max:$$Max = \max(A_L, A_H)$$
    
- Population Standard Deviation:$$\sigma = \sqrt{\frac{(A_L - A_V)^2 + (A_H - A_V)^2}{2}}$$
    
- Disparity:$$Disparity = 2\sigma$$
    

#### Tension Delta
In addition to ordinary deltas, Conflict Pairs support a special **Tension Delta**, which applies a dyadic-style adjustment directly to the base Affects.
The requested delta magnitude is preserved across $A_L$​ and $A_H$​, but the resulting changes in the composite value $A_V$​ and in derived metrics (such as Min or Disparity) may not correspond to the requested delta.

This mechanism allows designers to manipulate the underlying tension between the two sides directly, without necessarily steering the composite toward or away from one side.

#### Example
```json
Hesitance = ConflictPair(Attraction, Anxiety)
```
This could be used to model a character's hesitance to ask someone out on a date, with high values representing reluctance to take the risk and low values being more willing to make the attempt.
Rather than looking for this value to reach $0$, which will only happen when **Attraction** is $1$ and **Anxiety** is $0$, the author should choose some threshold, such as $0.25$, which indicates that **Attraction** is a certain amount higher than **Anxiety** ($0.5$ in this case).
The **Tension** could then be used to color the scene, and for the value $0.25$ it will be in the range $[0,0.5]$.
With a high **Tension** the character might be bashful, have trouble making eye contact, and perhaps stammer or talk too fast, but with low **Tension** the character might be more confident.

N.B. It's worth mentioning that this example is intentionally framing **Attraction** as reducing **Hesitance**, which is counter to how it's most often presented in games - e.g. increase **Attraction** to increase **Boldness** to ask them out on a date.
The choice is really up to the author and how they want to think about things or present them to the player.
Mechanically **Boldness** with the base affects reversed is identical to **Hesitance**, only with inverted values, but for both of them their **Tension** metric represents the conflict between the two affects, and so even with **Boldness** a high **Tension** would be the indicator to show nervousness indicators during the "Ask them out" scene.
From this perspective **Hesitance** is more aligned with **Tension** than **Boldness** would be.

### Tension Pair

The **Tension Pair** composition models the strain between two Affects when their joint presence is itself the most salient feature.
Let the base Affects be $A_1$​ and $A_2$​, and the composite Affect be $A_T$​.
The composite’s value is defined as the minimum of the two bases:$$A_T = \min(A_1, A_2)$$

#### Write-through Behavior

When a delta $\Delta_T$​ is applied to the Tension Pair, it is propagated to the base Affects using the same **Tension Delta** mechanism described for Conflict Pairs.
This ensures that the adjustment reflects changes in the underlying relationship rather than attempting to preserve a target composite value.

#### Example
```json
DifficultyStudying = Tension(Drive, Play)
```

### Synergistic Dyad

The **Synergistic Dyad** promotes the pairwise **synergy** of two Affects to be the primary value.
Let the base Affects be $A_1$​ and $A_2$​, and the composite Affect be $A_S$​, with the value relationship as:$$A_S = A_1 \cdot A_2$$

#### Write-through Behavior

When a delta $\Delta_S$​ is applied to the composite, redistribution to the bases depends on the sign of $\Delta_S$​.

Positive delta:$$\Delta_1=\Delta_S\frac{A_1(1-A_1)}{A_1(1-A_1)+A_2(1-A_2)}\qquad \Delta_2=\Delta_S\frac{A_2(1-A_2)}{A_1(1-A_1)+A_2(1-A_2)}$$

Negative delta:$$\Delta_1=\Delta_S\frac{A_1^2}{A_1^2+A_2^2}\qquad \Delta_2=\Delta_S\frac{A_2^2}{A_1^2+A_2^2}$$

#### Additional Metrics

- Min:$$Min=\min(A_1,A_2)$$
    
- Max:$$Max=\max(A_1,A_2)$$
    
- Mean:$$Mean=\tfrac{A_1+A_2}{2}$$
    

Note: When both inputs are the same Affect (i.e., $A_1=A_2$​), the composition acts as a nonlinear scaling of that Affect (very flat at low values, with increasing nuance toward high values).

Example
```json
Devotion = Synergy(Love, Trust)
```

### Sigmoid
The **Sigmoid** composition applies a logistic transform to a single base Affect, normalized so that inputs $A_B\in[0,1]$ map to outputs $A_S\in[0,1]$ with midpoint at $A_B=0.5$ and tunable steepness $k>0$.
Let
- $A_B$​ be the base Affect,
    
- $A_S$​ be the sigmoid-transformed Affect,
    
- $k$ be the slope/steepness parameter,
    
- $\sigma(x)=\dfrac{1}{1+e^{-x}}$.
    

Define the normalization constants:$$L=\sigma(-0.5k),\quad U=\sigma(0.5k),\quad R=U-L$$
    

Primary value (symbols introduced above, then the formula):$$A_S=\dfrac{\sigma\!\big(k(A_B-0.5)\big)-L}{R}$$
    

This yields $A_S=0$ at $A_B=0$ and $A_S=1$ at $A_B=1$, and is strictly increasing in $A_B$​ for $k>0$.

#### Write‑through Behavior (inverse transform as a delta)

For a requested change $\Delta_S$​ to the sigmoid output (i.e., new value $A_S+\Delta_S$​), the corresponding delta to apply to the base $A_B$​ is:$$\Delta_B=\frac{1}{k}\ln\!\left(\frac{A_SR+L}{1-(A_SR+L)}\cdot\frac{1-((A_S+\Delta_S)R+L)}{(A_S+\Delta_S)R+L}\right)$$`
    
#### Example
```
Burnout = Sigmoid( Stress, 12 )
```

### Root-Product
The **Root-Product** composition blends two Affects via the geometric mean. Let the base Affects be $A_1$​ and $A_2$​, and the composite Affect be $A_R$​.
The value relationship is:$$A_R=\sqrt{A_1A_2}$$

#### Write-through Behavior
Let a requested change to the composite be $\Delta_R$​. Define per-base “slack” terms $s_1,s_2$​ according to the sign of $\Delta_R$​:

- Positive $\Delta_R$​:$$s_1=1-A_1\qquad s_2=1-A_2$$
    
- Negative $\Delta_R$​:$$s_1=A_1\qquad s_2=A_2$$
    

Normalize these to obtain update ratios:$$S=s_1+s_2\qquad r_1=\frac{s_1}{S}\qquad r_2=\frac{s_2}{S}$$

Solve the quadratic constraint  $$(A_1+\Delta_1)(A_2+\Delta_2)=(A_R+\Delta_R)^2$$
under $$\Delta_1=r_1\Delta_x,\ \Delta_2=r_2\Delta_x$$
Use the positive root:$$\Delta_x=\frac{-(A_1r_2+A_2r_1)+\sqrt{(A_1r_2+A_2r_1)^2-4\,r_1r_2\left(-2\Delta_R\sqrt{A_1A_2}-\Delta_R^2\right)}}{2\,r_1r_2}$$

Apply to the bases: $$\Delta_1=r_1\,\Delta_x\qquad \Delta_2=r_2\,\Delta_x$$

#### Additional Metrics

- Min: $$Min=\min(A_1,A_2)$$
    
- Max: $$Max=\max(A_1,A_2)$$
    
- Mean: $$Mean=\tfrac{A_1+A_2}{2}$$
    
- Synergy (product): $$Synergy=A_1A_2$$
    

#### Example
```json
Affection = RootProduct(Care, Love)
```



# Common Details
### Overview
Chordline's _Emotional Model_ is designed for incremental adoption and utility across the entire range from a few specialized emotions to full dynamic emotional modeling.
### Note on Naming
As part of the support for the more detailed models, the building blocks of the system are called _Affective Elements_, reflecting the fact that they may be representing something that isn't technically an _Emotion_ but may contribute to emotions or emotional processing.
An _affect_ is a general term for a "felt state", e.g. from a biological, psychological, or computational source.
An _Emotion_ is a recognized and labeled "affective episode" and in human psychology is typically a complex, event-linked, and cognitively-processed blend of affects.
### Representation
Affective Elements in Chordline, whatever model they're in, are represented internally as a floating point number in the range \[0.0,1.0].
Interfaces are typical scaled by a factor of 10 to map more naturally to scales that we use in day-to-day life.
In order to support global performance enhancement via a C++ implementation plugin, all affects and compositions will be individual objects.
An emotional model will take the form of an encapsulating class with a constructor that initializes everything in the model for a given feeler/target.
Encapsulated objects are available for custom composition.
### Targeted and Context
Affective Elements are used primarily in two ways: targeted and untargeted.
Untargeted is describing the current state of the character.
Targeted is describing how the character feels about something else.
Note that there is a semantic difference between an Untargeted Affective Element (current mood) and an Affective Element that targets _Self_.
An Affective Element targeting _Self_ is how that entity feels about itself in general, which is not necessarily linked to their current mood.
Chordline allows nearly anything to be the target of a set of Affective Elements.
Some complex emotions, such as Trust or Shame, are defined in the context of a person's internal model of some external state.
Chordline does not attempt to replicate those internal models, but it does allow Affective Elements to have a Context in addition to a Target.
As with Target the Context can be nearly anything.
### Abstract Concepts
To aid in using Affective Elements as rich descriptors Chordline has "Concepts".
At their core Concepts are nothing more than a simple label or tag that is used to differentiate otherwise similar affects.
They are intended for use both as Targets and Contexts.

Examples of Targets include someone having a Love of Sweets, or CARE for Orphans.
An example with both a Target and a Context would be having trust in someone else's ability as Trust with that person as the Target and their ability as the Context. 
### Entity Interactions
Each Affective Element, including composite, has a hook point in the update logic that passes the element's identity, the current value, the requested delta, and the target of the affect (if any) to the owning entity.
(maybe also needs to pass the source of the delta? this implies that this is given (or at least can be given) when changes are requested)
The Entity has a registry of functions associated with Affective Elements and/or Targets that it will then pass the information through.
Each of these functions (if any) can modify the requested delta, which is then returned to the element to be applied.
The purpose of this is to allow for entity-specific saturation and/or desensitization for some or all Affective Elements, possibly only in some contexts.
(not sure how this interacts with global default emotional interactions or cultural overlays, etc)
### Composability
The Affective Elements in all models are composable in the following ways:

* Dyad: Two affects that contribute equally to a blended affect, such as Anticipation + Joy = Hope, or CARE + Love = Familial Love.
	* Formula:
	  $$D=\frac{E_1+E_2}{2}$$
	* Back-Propagation, positive $\Delta_D$
	  $$\Delta_1=\frac{\Delta_D(1-E_1)}{1-D},\quad\Delta_2=\frac{1\Delta_D(1-E_2)}{1-D}$$
	* Back-Propagation, negative $\Delta_D$
	  $$\Delta_1=\frac{\Delta_DE_1}{D},\quad\Delta_2=\frac{\Delta_DE_2}{D}$$
	* Tension: A measure of how strongly the two are in conflict:
	  $$tension=min(E_1,E_2)$$
	* Synergy:
	  $$E_1E_2$$
	* "Diminished Reinforcement", "Combined Presence with Diminished Reinforcement" or "total engagement reduced by overlap":
	  $$(E_1+E_2)-E_1E_2$$
	* ~~"Disparity":~~
	  $$1-\frac{(E_1+E_2)((E_1+E_2)-E_1E_2)}{2}$$
* Triad: Three affects that contribute equally to a blended affect, such as X, or Y.
	* Formula:
	  $$T=\frac{E_1+E_2+E_3}{3}$$
	* Back-Propagation, positive $\Delta_T$
	  $$\Delta_1=\frac{\Delta_T(1-E_1)}{1-T}\quad\Delta_2=\frac{\Delta_T(1-E_2)}{1-T}\quad\Delta_3=\frac{\Delta_T(1-E_3)}{1-T}$$
	* Back-Propagation, negative $\Delta_T$
	  $$\Delta_1=\frac{\Delta_TE_1}{T}\quad\Delta_2=\frac{\Delta_TE_2}{T}\quad\Delta_3=\frac{\Delta_TE_3}{T}$$
	* Tension: A measure of how strongly the two are in conflict:
	  $$tension=min(E_1,E_2,E_3)$$
	* Standard Deviation
	* Triadic Disparity (average of pairwise Dyadic Disparity)
	  $$1-\frac{(E_1+E_2)((E_1+E_2)-E_1E_2)+(E_2+E_3)((E_2+E_3)-E_1E_2)+(E_1+E_3)((E_1+E_2)-E_1E_3)}{6}$$
	* Cohesion:
	  $$E_1E_2E_3$$
	* Synergy:
	  $$\frac{E_1E_2+E_1E_3+E_2E_3}{3}$$
	* 
* Compose: N affects that contribute equally to a blended affect.
	* Formula:
	  $$A_C=\frac{E_1+E_2+\dots+E_n}{n}$$
	* Back-Propagation, positive $\Delta_C$
	  $$\Delta_1=\frac{\Delta_C(1-E_1)}{1-A_C}\quad\Delta_2=\frac{\Delta_C(1-E_2)}{1-A_C}\quad\dots\quad\Delta_n=\frac{\Delta_C(1-E_n)}{1-A_C}$$
	* Back-Propagation, negative $\Delta_C$
	  $$\Delta_1=\frac{\Delta_CE_1}{A_C}\quad\Delta_2=\frac{\Delta_CE_2}{A_C}\quad\dots\quad\Delta_n=\frac{\Delta_CE_n}{A_C}$$
	* Tension: A measure of how strongly the two are in conflict
	  $$tension=min(E_1,E_2,\dots,E_n)$$
	* Standard Deviation
	* Cohesion (see above)
	* 
* Weighted Composition: N affects that contribute unequally to a blended affect.
	* Formula:
	  $$C=\frac{E_1w_1+E_2w_2+\dots+E_nw_n}{w_1+w_2+\dots+w_n}$$
	* Back-Propagation, positive $\Delta_C$
	  $$\Delta_1=\frac{\Delta_C(1-E_1)}{1-C}\quad\Delta_2=\frac{\Delta_C(1-E_2)}{1-C}\quad\dots\quad\Delta_n=\frac{\Delta_C(1-E_n)}{1-C}$$
	* Back-Propagation, negative $\Delta_C$
	  $$\Delta_1=\frac{\Delta_CE_1}{C}\quad\Delta_2=\frac{\Delta_CE_2}{C}\quad\dots\quad\Delta_n=\frac{\Delta_CE_n}{C}$$
	* Tension: A measure of how strongly the two are in conflict:
	  $$tension=min(E_1,E_2,\dots,E_n)$$
	* Standard Deviation
	* Weights may be changed dynamically
	* 
* Conflict Pairs: Two affects in a tug-of-war relationship.
	* Formula:
	  $$P=\frac{1-E_1+E_2}{2}$$
	* Back-Propagation, positive $\Delta_P$
	  $$\Delta_1=-\Delta_PE_1\quad\Delta_2=\Delta_P(1-E_2)$$
	* Back-Propagation, negative $\Delta_P$
	  $$\Delta_1=-\Delta_P(1-E_1)\quad\Delta_2=\Delta_PE_2$$
	* Tension: A measure of how strongly the two are in conflict:
	  $$tension=min(E_1,E_2)$$
	* Tension Delta:
	  Distributes requested delta, positive or negative, to both components based on slack using the Dyad formula.
	  Effect on the value will usually be less than the requested delta.
	  Effect on tension will usually be less than the requested delta.
	* 
* Tension Pair: A dedicated object for the `tension` method from a Conflict Pair. Useful for making decisions.
	* Formula:
	  $$T=min(E_1,E_2)$$
	* Back-Propagation following the Tension Delta method of a Conflict Pair (does not necessarily preserve requested delta)
* Synergistic Dyad (same value as the synergy property of a normal Dyad / Cohesive Composition (when more than 2 affects):
	* Formula:
	  $$S=E_1E_2$$
	* Back-Propagation, positive $\Delta_S$
	  $$\Delta_1=\Delta_S\frac{E_1(1-E_1)}{E_1(1-E_1)+E_2(1-E_2)}\quad\Delta_2=\Delta_S\frac{E_2(1-E_2)}{E_1(1-E_1)+E_2(1-E_2)}$$
	* Back-Propagation, negative $\Delta_S$
	  $$\Delta_1=\Delta_S\frac{E_1^2}{E_1^2+E_2^2}\quad\Delta_2=\Delta_S\frac{E_2^2}{E_1^2+E_2^2}$$
	* n-ary Case:
	  $$\Delta_i=\Delta_S\cdot\frac{E_i^2}{\sum E_n^2}$$
* Sigmoid
	* Formula ($k=12$ is recommended):
	  Exact fit:$$\sigma(x)=\frac{1}{1+e^x}\quad L=\sigma(-0.5k)\quad U=\sigma(0.5k)\quad R=U-L\quad A_\sigma=\frac{\sigma(k(A_B-0.5))-L}{R}$$
	  Approximate fit (worse with lower k):$$M=\frac{1}{1+e^{-k(E-0.5)}}$$
	* Back-Propagation:
	  Exact fit:$$\Delta_E=\frac{1}{k}ln\left(\frac{MR+L}{1-(MR+L)}\cdot\frac{1-((M+\Delta_M)R+L)}{(M+\Delta_M)R+L}\right)$$
	  Approximate fit:$$\Delta_E=\frac{1}{k}ln\left(\frac{1-M}{M}\cdot\frac{M+\Delta_M}{1-(M+\Delta_M)}\right)$$
	* 
* Root-Product
	* Formula:
	  $$R=\sqrt{E_1E_2}$$
	* Back-Propagation, positive $\Delta_R$
	  $$s_1=1-E_1\quad s_2=1-E_2$$
	* Back-Propagation, negative $\Delta_R$
	  $$s_1=E_1\quad s_2=E_2$$
	* Back-Propagation (quadratic, but we always use the positive root), Both:
	  $$S=s_1+s_2\quad r_1=\frac{s_1}{S}\quad r_2=\frac{s_2}{S}\quad \Delta_1=r_1\cdot\Delta_x\quad \Delta_2=r_2\cdot\Delta_x$$
	  $$\Delta_x=\frac{-(E_1r_2+E_2r_1)+\sqrt{\left(E_1r_2+E_2r_1\right)^2-4\cdot r_1r_2\cdot\left(-2\Delta_R\cdot\sqrt{E_1E_2}-\Delta_R^2\right)}}{2\cdot r_1r_2}$$
	* 
* Conditional
	* Constructed from a function and two affects
	* effectively `f() ? A : B`
	* Exposes value() of the object as its value
	* Exposes the object that was selected()
	* 
* Dominance: A diagnostic object that identifies the dominant affect and by how much
	* Imbalance:
	  $$I_\Delta=abs(E_1-E_2)$$
	    
	* Identity of the largest affect
	* But see "Disparity" to differentiate e.g. (0.1,0.5) and (0.5,0.9)
	* 
Back-propagation (general):

* Where possible back-propagation is allowed through compositions, preserving the requested delta as much as possible.
* Back-propagation is allowed only when all component parts allow modification/back-propagation AND no base Affect appears more than once recursively
* 

Read-Only:

* A single-affect composition that acts like it accepts updates but will silently discard them.
* All multi-affect compositions support having sources marked as read-only, causing them to behave that way within the composition object, but optimized.

Local Buffer:

* All multi-affect compositions can have any of the components designated as a local buffer (with an initial value), which behaves the same as a simple affect without having to create the anonymous object.

Sink Affects:

* For the purposes of modulating compositions or creating buffered deltas there are Sink Affects that have a given fixed value while accepting (and silently discarding) all modifications. For the purposes of back-propagation these don't count as being duplicated no matter how many times they may appear.
* 

Metrics as Affects:

* Some (all?) of the methods/metrics on the above are available as dedicated objects (e.g. Tension) that can themselves be used in compositions.
* 

Observer/Notification Patterns:

* All Affects can be observed to be notified about changes.
* The first call into the system will set the base change ID, all propagations will add qualifiers to it
* Propagation path is part of the event
* Observation of a composite will cause it to observe its components
* Start value, final value, and delta will be explicit data in the event
* Timestamps, both wall-clock and simulation, will be in the event
* Multiple granularity levels, not all will apply to all objects:
	* Global
	* Affect Owner
	* Affect Target (implies target knows who is feeling about them, which wasn't previously an explicit expectation)
	* Individual Affect
### Possible Gatekeeping Patterns

* Security gating PLAY / Joy
* Shame/Fear/Stress Suppressing Expression of Anger or LUST
* CARE Inhibiting or Overriding Disgust or Anger
* Empathy Biasing Expressive Output
* Fatigue / Pain Blocking PLAY / SEEKING
* Model-Based Mismatch Filtering - High Surprise or Low Anticipation may prevent the internal model from accepting or escalating other affects.
* Social Modulation Gate
* Threshold Activation Gate
* Inhibition by Competition - One affect is not allowed to increase if another mutually exclusive state is active
### Deferred to Plugins

* Sustained Pressure - Plugin applies recurring deltas while source is high (plus reasonable variations)
* Threshold Triggers - Porcelain objects using the observer hooks core feature (triggers an arbitrary event, watch out for flapping)
* Periodic Stabilized Coupling - Mutual reduction in conflicting affects. Core method with automatic periodic invocation via plugin
* Delayed Onset Influence - Plugin tracks duration above/below threshold (and then triggers an arbitrary event)
* Decay-Weighted Echo Influence - Requires tracking past peaks or smoothing history (a spike in A triggers a delta in B that then decays over time)
* Bidirectional Influence - can be constructed using others, especially threshold triggers, sustained pressure, etc. using a chain of events
* Resource-Based Suppression
	* Dynamic Ceiling - The maximum allowed value of affect B is reduced based on A
	* Delta Inhibition - Inputs that would normally raise B have their magnitude reduced based on A
	* Absolute Lockout - With A above a certain threshold, B cannot be increased at all
* Separate Mood from Baseline, with automatic adjustment between them
	* Note that composites don't decay (towards baseline); they reflect the changes caused by the decay of their bases.
* Affect Decay - A scalar affective element naturally diminishes in value over time if not reinforced
	* "His anger faded over the next few hours."
	* Common in short-lived emotions like Fear, Surprise, Embarrassment
* Affect Inertia / Persistence - An element maintains its value unless actively countered - resistance to decay
	* "She doesn't let go of things easily."
	* Strong in Sorrow, CARE, Stress, chronic Pain
	* May be scalar: decay coefficient less than 1.0
* Decay Acceleration / Compression - Decay rate increases over time or under specific conditions.
	* "Once she started to calm down, it passed quickly."
	* Fear, Surprise, Panic -> fast dropoff after peak
* Oscillation / Rhythmic Regulation - Affect naturally cycles, either due to internal rhythm or contextual reset.
	* "He always has ups and downs."
	* Mood swings, fatigue/recover loops, some CARE/LUST expressions
* Rebound Effects - A rapid or extreme drop in one affect causes a compensatory rise in another.
	* "After the fear subsided, she became euphoric." ("WE SURVIVED!")
	* Often modeled in mood disorders; can create dramatic whiplash moments
* Affective Incompatibility (Simultaneity Exclusion) - Some Affective Elements are conceptually or biologically unlikely to co-occur at high intensity.
* Valence Opposition - Elements at opposite ends of the circumplex naturally attenuate one another.
* Homeostatic Reprioritization - Strong signals from one homeostatic axis suppress others to conserve attention or energy
* Resource Competition - Two Affective Elements compete for the same attentional or expressive bandwidth
* Decay-Inhibiting Conflict - One affect's presence slows or halts the natural decay of another
* 
### Valence/Activation Values
Each base affect may be associated with either

1. A vector in Valence-Activation space that is scaled by the affect's value to directly to Valence and Activation values on its own.
2. A transformation matrix that is parameterized by its value and is used only in composition with other affects.
   In the standard detailed model Anticipation is an example of a base that has a transformation matrix instead of a Valence-Activation vector.

Composite affects calculate their Valence and Activation values based on the affects they are composed from.
# The DIY Model
The DIY Model is the simplest model. It starts empty and the author builds whatever they want.

# The Simple Model
The details of the Simple Model are TBD, but the general direction is that they will be an enumerated subset of the possible affects from the Detailed Model, but without compositing.

# The Detailed Model
## Inspired by Panksepp's Affective Neuroscience and Plutchik's Wheel of Emotions

### Affective Elements
#### Regulatory Affects:

1. Hunger
2. Thirst
3. Fatigue
4. Pain
5. Bladder Fullness
6. Bowel Pressure
7. Sexual Readiness
8. Stress
9. Pleasure
10. Heat Stress
11. Cold Stress
#### Primal Affects (primarily from Panksepp, with additions from other neuroscience):
Moods/Motivations observed by neuroscience in mammals.

1. Drive (SEEKING)
2. Loss (GRIEF/PANIC)
3. RAGE
4. Fear (FEAR)
5. LUST
6. CARE
7. PLAY
8. Security (added by Chordline)
9. Disgust (added by Chordilne)

#### Social Affects (category created for Chordline with a high degree of influence from Plutchik):
The expression of core moods/motivations filtered through cognition, social context, and/or abstract interpretation.

1. Drive -> Interest/Curiosity
2. Loss -> Sorrow
3. RAGE -> Anger
4. FEAR -> Anxiety
5. LUST -> Desire / Attraction
6. CARE -> Love
7. PLAY -> Joy / Happiness
8. Security -> Trust
9. Disgust -> Revulsion

#### Reflective Affects (category created for Chordline with influence from Plutchik):
Shortcuts for higher-level affectual drivers that would need be based on mental models of something external to be modeled properly.

1. Surprise (affect now based on the difference between expectation and reality - 0.0 is complete agreement and 1.0 is complete disagreement)
2. Familiarity (affect now based on the perceived completeness of the mental model - 0.0 is no mental model and 1.0 is complete confidence)
3. Anticipation (affect now based on the expected future - 0.0 is complete disregard for the future and 1.0 is complete regard for the future)
4. Remembrance (affect now based on the remembered past - 0.0 is complete disregard for the past and 1.0 is complete regard for the past)
5. Empathetic (affect based on another's expressed emotions (observed, remembered, anticipated, imagined, etc.) - 0.0 is completely unaffected and 1.0 is completely overwhelmed)

### Combined Affects
### Physiological or Conceptual Roots

In human physiology some of the Primal Affects and Regulatory Affects share mental circuitry and are therefore related to each other in real humans in ways that aren't currently accounted for in the model.
This section does not enumerate these relationships; it merely lists the kinds of relationships that may exist.

1. Shared Origin - Multiple Affective Elements arise from the same biological or neurological system (e.g. CARE and GRIEF from attachment circuitry)
2. Neural Competition - Certain Elements suppress or gate one another due to shared neural substrates
3. 
### Primal vs. Social Affects

Many composites are formed from a blend of related Primal/Social pair, where the weights on the blend are expected to change over the course of the narrative.
Composites following this pattern have helper methods to simplify the process, automatically creating a weighted dyad between that pair and using that in the composition with other elements.
### Simple Inversions

1. Fatigue -> Energy
2. Thirst -> Hydration
3. Hunger -> Satiation
4. SECURITY -> Vulnerability
5. SEEKING -> Apathy
6. Anxiety -> Confidence
7. Sorrow -> Peace
8. Trust -> Suspicion
### Regulatory Affects not part of standard

1. Itch/Surface Irritation
2. Vestibular Distress
3. Air Hunger
4. Energy Availability
5. Hormonal Disruption
6. Immune Challenge
7. Reproductive Readiness (wanting children)
8. Addiction Craving
9. Blood Sugar Variability
10. Sensory Overload
11. Restlessness / Tonic Drive
12. Satiation / Fullness
13. Recovery Load  ("Pressure to reduce exertion following recent stress/injury")

# References

1. https://en.wikipedia.org/wiki/Emotion_classification#Plutchik's_wheel_of_emotions
2. https://en.wikipedia.org/wiki/File:Plutchik_Dyads.svg
3. https://en.wikipedia.org/wiki/Jaak_Panksepp#Primary_affective_systems
4. https://en.wikipedia.org/wiki/Emotion_classification#Circumplex_model
5. https://en.wikipedia.org/wiki/Emotion_classification#The_Book_of_Human_Emotions
6. https://en.wikipedia.org/wiki/Affect_(psychology)
7. 