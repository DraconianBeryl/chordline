# Choice of Models

While there is a detailed model that's part of the Chordline project, its use its use is optional. Other models can be developed and shared using the same building blocks as Chordline's detailed model.

# Common Details

### Representation
Emotions in Chordline, whatever model they're in, are represented internally as a floating point number in the range \[0.0,1.0].
Interfaces are typical scaled by a factor of 10 to map more naturally to scales that we use in day-to-day life.
### Targeted and Context
Emotions are used primarily in two ways: targeted and untargeted.
Untargeted is describing the current state of the character.
Targeted is describing how the character feels about something else.
Chordline allows nearly anything to be the target of a set of emotions.

Some emotions, such as Trust or Shame, are defined in the context of a person's internal model of some external state.
Chordline does not attempt to replicate those internal models, but it does allow emotions to have a Context in addition to a Target.
As with Target the Context can be nearly anything.
### Abstract Concepts
To aid in using emotions as rich descriptors Chordline has "Concepts".
At their core Concepts are nothing more than a simple label or tag that is used to differentiate otherwise similar emotions.
They are intended for use both as Targets and Contexts.

Examples of Targets include someone having a Love of Sweets, or CARE for Orphans.
An example with both a Target and a Context would be having trust in someone else's ability as Trust with that person as the Target and their ability as the Context. 
### Composability
The emotions in all models are composable in the following ways:

* Dyad: Two emotions that contribute equally to a blended emotion, such as Anticipation + Joy = Hope, or CARE + Love = Familial Love.
	* Formula $$D=\frac{E_1+E_2}{2}$$
	* Back-Propagation, positive $\Delta_D$ $$\Delta_1=\frac{\Delta_D(1-E_1)}{1-D},\quad\Delta_2=\frac{1\Delta_D(1-E_2)}{1-D}$$
	* Back-Propagation, negative $\Delta_D$ $$\Delta_1=\frac{\Delta_DE_1}{D},\quad\Delta_2=\frac{\Delta_DE_2}{D}$$
	* Tension: A measure of how strongly the two are in conflict $$tension=min(E_1,E_2)$$
	* Synergy: $$E_1E_2$$
	* "Diminished Reinforcement", "Combined Presence with Diminished Reinforcement" or "total engagement reduced by overlap": $$(E_1+E_2)-E_1E_2$$
	* "Disparity": $$1-\frac{(E_1+E_2)((E_1+E_2)-E_1E_2)}{2}$$
	* 
 
 * Triad: Three emotions that contribute equally to a blended emotion, such as X, or Y.
	* Formula $$T=\frac{E_1+E_2+E_3}{3}$$
	* Back-Propagation, positive $\Delta_T$ $$\Delta_1=\frac{\Delta_T(1-E_1)}{1-T}\quad\Delta_2=\frac{\Delta_T(1-E_2)}{1-T}\quad\Delta_3=\frac{\Delta_T(1-E_3)}{1-T}$$
	* Back-Propagation, negative $\Delta_T$ $$\Delta_1=\frac{\Delta_TE_1}{T}\quad\Delta_2=\frac{\Delta_TE_2}{T}\quad\Delta_3=\frac{\Delta_TE_3}{T}$$
	* Tension: A measure of how strongly the two are in conflict $$tension=min(E_1,E_2,E_3)$$
	* Standard Deviation
	* Triadic Disparity (average of pairwise Dyadic Disparity) $$1-\frac{(E_1+E_2)((E_1+E_2)-E_1E_2)+(E_2+E_3)((E_2+E_3)-E_1E_2)+(E_1+E_3)((E_1+E_2)-E_1E_3)}{6}$$
	* Cohesion: $$E_1E_2E_3$$
	* Synergy: $$\frac{E_1E_2+E_1E_3+E_2E_3}{3}$$
	* 
* Compose: N emotions that contribute equally to a blended emotion.
	* Formula $$C=\frac{E_1+E_2+\dots+E_n}{n}$$
	* Back-Propagation, positive $\Delta_C$ $$\Delta_1=\frac{\Delta_C(1-E_1)}{1-C}\quad\Delta_2=\frac{\Delta_C(1-E_2)}{1-C}\quad\dots\quad\Delta_n=\frac{\Delta_C(1-E_n)}{1-C}$$
	* Back-Propagation, negative $\Delta_C$ $$\Delta_1=\frac{\Delta_CE_1}{C}\quad\Delta_2=\frac{\Delta_CE_2}{C}\quad\dots\quad\Delta_n=\frac{\Delta_CE_n}{C}$$
	* Tension: A measure of how strongly the two are in conflict $$tension=min(E_1,E_2,\dots,E_n)$$
	* Standard Deviation
	* Cohesion (see above)
	* 
* Weighted Composition: N emotions that contribute unequally to a blended emotion.
	* Formula $$C=\frac{E_1w_1+E_2w_2+\dots+E_nw_n}{w_1+w_2+\dots+w_n}$$
	* Back-Propagation, positive $\Delta_C$ $$\Delta_1=\frac{\Delta_C(1-E_1)}{1-C}\quad\Delta_2=\frac{\Delta_C(1-E_2)}{1-C}\quad\dots\quad\Delta_n=\frac{\Delta_C(1-E_n)}{1-C}$$
	* Back-Propagation, negative $\Delta_C$ $$\Delta_1=\frac{\Delta_CE_1}{C}\quad\Delta_2=\frac{\Delta_CE_2}{C}\quad\dots\quad\Delta_n=\frac{\Delta_CE_n}{C}$$
	* Tension: A measure of how strongly the two are in conflict $$tension=min(E_1,E_2,\dots,E_n)$$
	* Standard Deviation
	* "Amplified Tension", or "Engagement Intensity": $$\frac{(E_1+E_2)((E_1+E_2)-E_1E_2)}{2}$$
	* 
* Conflict Pairs: Two emotions in a tug-of-war relationship.
	* Formula $$P=\frac{1-E_1+E_2}{2}$$
	* Back-Propagation, positive $\Delta_P$ $$\Delta_1=-\Delta_PE_1\quad\Delta_2=\Delta_P(1-E_2)$$
	* Back-Propagation, negative $\Delta_P$ $$\Delta_1=-\Delta_P(1-E_1)\quad\Delta_2=\Delta_PE_2$$
	* Tension: A measure of how strongly the two are in conflict $$tension=min(E_1,E_2)$$
	* 
* Inversion: One emotion inverted numerically.
	* Formula $$I=1-E$$
	* Back-Propagation $$\Delta_E=-\Delta_I$$
	* 
* Tension Pair: A dedicated object for the `tension` method from a Conflict Pair. Useful for making decisions. No back-propagation.
	* Formula $$T=min(E_1,E_2)$$
	* 
* Dominance: A diagnostic object that identifies the dominant emotion and by how much
	* Imbalance $$I_\Delta=abs(E_1-E_2)$$
	* Identity of the largest emotion
	* But see "Disparity" to differentiate e.g. (0.1,0.5) and (0.5,0.9)
	* 
* Alias: just adding an official new name for something else
	* Smart enough to expose all of the same metrics as the original?
	* 
* Synergistic Dyad (same value as the synergy property of a normal Dyad / Cohesive Composition (when more than 2 affects):
	* Formula $$S=E_1E_2$$
	* Back-Propagation, positive $\Delta_S$ $$\Delta_1=\Delta_S\frac{E_1(1-E_1)}{E_1(1-E_1)+E_2(1-E_2)}\quad\Delta_2=\Delta_S\frac{E_2(1-E_2)}{E_1(1-E_1)+E_2(1-E_2)}$$
	* Back-Propagation, negative $\Delta_S$ $$\Delta_1=\Delta_S\frac{E_1^2}{E_1^2+E_2^2}\quad\Delta_2=\Delta_S\frac{E_2^2}{E_1^2+E_2^2}$$
	* n-ary Case: $$\Delta_i=\Delta_S\cdot\frac{E_i^2}{\sum E_n^2}$$
* Sigmoid
	* Formula ($k=12$ is recommended) $$M=\frac{1}{1+e^{-k(E-0.5)}}$$
	* Back-Propagation $$\Delta_E=\frac{1}{k}ln\left(\frac{1-M}{M}\cdot\frac{M+\Delta_M}{1-(M+\Delta_M)}\right)$$
	* 
* Root-Product
	* Formula $$R=\sqrt{E_1E_2}$$
	* Back-Propagation, positive $\Delta_R$ $$s_1=1-E_1\quad s_2=1-E_2$$
	* Back-Propagation, negative $\Delta_R$ $$s_1=E_1\quad s_2=E_2$$
	* Back-Propagation (quadratic, but we always use the positive root), Both: $$S=s_1+s_2\quad r_1=\frac{s_1}{S}\quad r_2=\frac{s_2}{S}\quad \Delta_1=r_1\cdot\Delta_x\quad \Delta_2=r_2\cdot\Delta_x$$ $$\Delta_x=\frac{-(E_1r_2+E_2r_1)+\sqrt{\left(E_1r_2+E_2r_1\right)^2-4\cdot r_1r_2\cdot\left(-2\Delta_R\cdot\sqrt{E_1E_2}-\Delta_R^2\right)}}{2\cdot r_1r_2}$$
	* 

Back-propagation:

* Where possible back-propagation is allowed through compositions, preserving the requested delta as much as possible.
* Back-propagation is allowed only when all component parts allow modification/back-propagation AND no base emotion appears more than once recursively
* 

Sink Emotions:

* For the purposes of modulating compositions or creating buffered deltas there are Sink emotions that have a given fixed value while accepting (and silently discarding) all modifications. For the purposes of back-propagation these don't count as being duplicated no matter how many times they may appear.
* 

Metrics as Emotions:

* Some (all?) of the methods/metrics on the above are available as dedicated objects (e.g. Tension) that can themselves be used in compositions.
* 

### Valence/Activation Values
Each base emotion may be associated with either

1. A vector in Valence-Activation space that is scaled by the emotions value to directly to Valence and Activation values on its own.
2. A transformation matrix that is parameterized by its value and is used only in composition with other emotions.
   In the standard detailed model Anticipation is an example of a base that has a transformation matrix instead of a Valence-Activation vector.

Composite emotions calculate their Valence and Activation values based on the emotions they are composed from.
# The DIY Model
The DIY Model is the simplest model. It starts empty and the author builds whatever they want.

# The Simple Model
The details of the Simple Model are TBD, but the general direction is that they will be an enumerated subset of the possible emotions from the Detailed Model, but without compositing.

# The Detailed Model
## Inspired by Panksepp's Affective Neuroscience and Plutchik's Wheel of Emotions

### Affective Elements
#### Homeostatic Signals:

1. Hunger
2. Thirst
3. Fatigue
4. Pain
5. Temperature Regulation
6. Bladder Fullness
7. Bowel Pressure
8. Sexual Readiness
9. Stress
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

#### Expressive Evolution (category created for Chordline with a high degree of influence from Plutchik):
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

#### Model-Driven Emotional Modifiers (category created for Chordline with influence from Plutchik):
Shortcuts for higher-level emotional drivers that would need be based on mental models of something external to be modeled properly.

1. Surprise (emotional effect now based on the difference between expectation and reality - 0.0 is complete agreement and 1.0 is complete disagreement)
2. Anticipation (emotional effect now based on the expected future - 0.0 is complete disregard for the future and 1.0 is complete regard for the future)
3. Remembrance (emotional effect now based on the remembered past - 0.0 is complete disregard for the past and 1.0 is complete regard for the past)
4. Empathetic (emotional effect based on another's emotions (observed, remembered, anticipated, imagined, etc.) - 0.0 is completely unaffected and 1.0 is completely overwhelmed)

### Combined Emotions

### Homeostatic Signals not part of standard

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