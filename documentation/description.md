<h1> Elementary</h1>

_An Ontology to Model Electro-mechanical Systems and their Functioning_

[TOC]

## What is this about?

Electro-mechanical systems in industrial applications are inter-connected components that process substance and energy in a coordinated manner. Traditionally, descriptions of such systems, usually diagrams and texts, have been created to solely cater for human understanding. With the age of AI-assisted autonomy dawning on the industry, the question arises as to how we can make this knowledge accessible to also _intelligent artificial agents_. Or, should we just let such agents (including language agents) _discover_ the system on their own and _learn_ to control it? The latter option sounds attractive because it is almost engineering-free. Except that it is riddled with lack of accuracy, reliability, safety, and explainability. Instead of being forced to choose between heavily engineered (and purely reactive) software programs and creative agents based on language/vision models, I advocate making expert-created knowledge encoded in a _machine-understandable_ manner available to both logically reasoning and generative agents. Of course, curation of such knowledge is not so obvious and where possible, it is often expensive. Which is why I show that even basic qualitative descriptions that can be automatically synthesized out of existing sources bring significant benefit to AI agents which are then spared of creatively probabilistic acts. I am quite sure that the astounding capabilities of current day LLMs is on your mind. I will take time to show that I am not talking of knowledge-informed approach as being _disjoint_ from AI methods, but contrarily, that it is a very integral part of AI that is currently missing.

**Elementary** is an ontology that helps model such knowledge. It is based on the Semantic Web standards which has a proven background in industrial applications. The name is inspired by Conan Doyle's fictional character of Sherlock Holmes, who tells his friend Watson that happenings can be explained on the basis of elementary facts (the phrase first appears in the story "A study in Scarlet"). When it comes to the physical world, elementary facts are invaluable in working out mysteries.

In principle, knowledge modeled by Elementary consists of three parts:

![alt text](images/overview.png)

Semantic modeling in classical engineering industrial domains has been more or less focussed around describing only the system design part. This is only a _part_ of the required knowledge. I will explain why we need a much more comprehensive inter-connected knowledge of the system that also inlcudes requirements (why was it built in the first place), its automation (how are the components controlled), and (at least) some idea of the underlying physical principles (why it does what it does).

Further to merely providing a vocabulary to describing the four principal aspects, Elementary also provides concepts to inter-link them:

![alt text](images/overview-relationships.png)

By inter-linking the individual concepts, we allow a system designer to express **how** a system is constructed, **what** is does **physically**, how is this related to the requirements, and finally, the role of the automation system in achieving the goals.

## How should an integrated knowledge look like

Elementary provides the core concepts to describe what you are dealing with (i.e., the electro-mechanical system, its requirements, and therefore, the automation needs). The concepts are equally well applicable to both simple and complex systems. Let us consider a real-life system with medium complexity. The figure below is the schematic of a system called the Air Handling Unit (bear with me on this).
![alt text](images/ahu.png)

Here is a quite good AI-generated description of the figure above:
This Air Handling Unit (AHU) conditions and circulates air through a building by mixing fresh outdoor air with recirculated indoor air. 

1. Air Intake and Mixing

* **Fresh Air Intake**: Outdoor air enters the system through an **Outdoor Damper**, which regulates the volume of incoming air.
* **Air Filtration**: This fresh air passes through a **Filter** to remove dust, debris, and airborne particles.
* **Recirculation**: An **Extract Fan** pulls stale air out of the **Conditioned Space**. A portion of this air is released outside via the **Exhaust Damper**, while the rest is directed downward to mix with the filtered fresh air.

2. Heating and Conditioning

* **Thermal Exchange**: The mixed air passes through a **Heater** (heating coil). 
* **Temperature Control**: A **Valve** modulates the flow of hot water from a boiler into the heater to control how much heat is transferred to the air stream.

3. Supply and Distribution
* **Air Delivery**: The **Supply Fan** draws the heated air from the coil and pushes it forward.
* **Space Heating**: This conditioned, **Heated Air** is delivered directly into 

The topology of this system can be expressed using the Semantic Web language. Here is the description in RDF-Turtle syntax. Do glance through it. Especially to see how it make the knowledge machine-understandable.

```turtle
@prefix rdf:   <http://w3.org> .
@prefix rdfs:  <http://w3.org> .
@prefix brick: <https://brickschema.org> .
@prefix ex:    <http://example.org> .

### --- System & Location Composition ---

ex:Main_AHU a brick:Air_Handling_Unit ;
    rdfs:label "Air Handling Unit" ;
    brick:feeds ex:Office_Zone .

ex:Office_Zone a brick:HVAC_Zone ;
    rdfs:label "Conditioned Space" .


### --- Equipment Components ---

# Dampers
ex:Outdoor_Damper a brick:Outdoor_Damper ;
    rdfs:label "Outdoor Air Damper" ;
    brick:isPartOf ex:Main_AHU .

ex:Exhaust_Damper a brick:Exhaust_Damper ;
    rdfs:label "Exhaust Air Damper" ;
    brick:isPartOf ex:Main_AHU .

# Filter
ex:Air_Filter a brick:Air_Filter ;
    rdfs:label "Fresh Air Filter" ;
    brick:isPartOf ex:Main_AHU .

# Fans
ex:Extract_Fan a brick:Return_Fan ;
    rdfs:label "Extract Air Fan" ;
    brick:isPartOf ex:Main_AHU .

ex:Supply_Fan a brick:Supply_Fan ;
    rdfs:label "Supply Air Fan" ;
    brick:isPartOf ex:Main_AHU .

# Heating Components
ex:Heating_Coil a brick:Heating_Coil ;
    rdfs:label "Hot Water Heating Coil" ;
    brick:isPartOf ex:Main_AHU .

ex:Hot_Water_Valve a brick:Hot_Water_Valve ;
    rdfs:label "Hot Water Control Valve" ;
    brick:controls ex:Heating_Coil .


### --- Airflow and Fluid Topologies ---

# Outside Air Intake and Filtration Path
ex:Outdoor_Damper brick:feedsAir ex:Air_Filter .
ex:Air_Filter     brick:feedsAir ex:Heating_Coil .

# Extract and Recirculation Loop
ex:Office_Zone   brick:feedsAir ex:Extract_Fan .
ex:Extract_Fan   brick:feedsAir ex:Exhaust_Damper , ex:Heating_Coil . # Split path to exhaust and mixing point

# Supply Path
ex:Heating_Coil  brick:feedsAir ex:Supply_Fan .
ex:Supply_Fan    brick:feedsAir ex:Office_Zone .
```

The nice thing about a Knowledge Graph based on a formal ontology is that it can be queried using a structured language. This is what a SPARQL query for listing out all kind of fans and things they feed into looks like:

```sparql
PREFIX rdfs:  <http://w3.org>
PREFIX brick: <https://brickschema.org>

SELECT ?fanName ?fanTypeLabel ?destinationName ?destinationTypeLabel
WHERE {
  # Find any equipment that is a subclass of brick:Fan (captures both Supply and Return fans)
  ?fan a ?fanType .
  ?fanType rdfs:subClassOf* brick:Fan .
  
  # Trace the downstream path where the fan feeds air to another asset or zone
  ?fan brick:feedsAir ?destination .
  ?destination a ?destinationType .
  
  # Fetch human-readable labels for cleaner output
  ?fan rdfs:label ?fanName .
  ?destination rdfs:label ?destinationName .
  
  # Optional: Fetch readable names of the specific Brick classes
  OPTIONAL { ?fanType rdfs:label ?fanTypeLabel }
  OPTIONAL { ?destinationType rdfs:label ?destinationTypeLabel }
}
ORDER BY ?fanName
```

This is really nice, right? But now we want more. For example, how do we know which components are actively responsible for achieving heating in the conditioned space? What role does the exhaust air damper plays on the air pressure in the space? Under a given state, what will happen when the supply fan speed is increased? What is the best way to improve the air quality while not compromising energy consumption? 

#### An Interlude: So, good LLMs and we are done?

Not surprisingly, once we have a validated topological description like the above one, the ever-improving LLMs are impressive in answering such questions. But the well-known trouble is that they are not reliable. Unless of course they have seen each possible combination of configuration and possible states - something which we cant expect to happen unless the system descriptions, run time data, and the underlying physical principles are made available to the LLMs. Moreover, notice that some expert had to validate the above topology description. Though we reduced the required manual engineering, we did not achieve autonomy.

I know that at this point it is **not** conclusive that LLMs on their own cannot do the job. I doubt I would be able to or want to conclude that way. Instead, I would like to show that with rather low effort, we can massively help LLMs and benefit in the larger context.

### System design sounds ok, but why do we need the other parts?

The thought is not surprising and even a fair one. We humans are usually good at figuring out how things work. In most cases just high school science is sufficient. I often use the example of an electric water kettle in my lectures to show how intuitive our reasoning is. But still, quite a few miss out that the kettle's behaviour would be different high up in the mountains. Or would have to learn through observation about how long it takes to boil a liter of water. Or learn through experience what a (particular) kettle does after the water has started boiling? (oh, I have a fancy kettle that keeps the water at 80C for next 1 hour -- discovered that). But you see the point I am trying to make? It is common sense knowledge **plus** experience that makes you aware of how things work (i.e., if you dont read the manuals, which may not have everything, like how long it will take to boil the water).

Kettles apart, when it comes to industrial systems, the sizes and complexities blows up the problem. How does the building's heating/cooling system work? Why does the heating not work when the outdoor temperature is greater than 15C? Was it in the requirements or regulations? Is it working efficiently? Whats wrong with it?

I think it will be easier if I tell you how, other than system design, you can also model requirements, automation, and physics, and interlink the descriptions. Then it will be rather easy to see what it can it do. Either on its own, or when coupled with AI methods.

## Requirements : How can we describe requirementsin a _machine understandable_ manner?

This a can of worms. Mostly because requirements are imprecise and incomplete to begin with and are only alright after the system has been built, operated, and ready to be decomissioned. I am talking of requirements for electro-mechanical systems. So, software engineers, remember the virtual world is **much** more controlled. But my view is a bit too sweeping -- there are domains where requirements are ultra precise. Just take aeronautical or medical domain as an example. Life safety seems to be the driver. They even have arcane formal verifications in place. But thats the Mercedes Benz end of the spectrum. For systems like the one for heating in your building, or the air handling unit in our example, you would be lucky to find any sort of requirement. My approach is: have an ontology to describe requirements and then then have AI agents interpret, question the source, and then synthesise the formal model. More on this a bit later, but first the model.

### Goal-oriented Requirements Engineering

The concept of goal-oriented requirements engineering (GORE) has been around for long. What it proposes is that each requirement be related to the performing system/component and the statement about the _physical_ variables that should be achieved, maintained, or avoided. It does not specify how the automation should act, just what should be accomplished. The part related to physical variables is of importance. GORE provides a grammar to express, dependencies, pre-, and post-conditions of achieving a goal. Not just static, but statement based on temporal logic can be included ("within one hour of starting the pump, the pressure should reach at least 80% of the designed value"). The work of Lamsweerde, who I believe is the fountainhead, is a pleasure to read. Though the formalization and grammar proposed by him is quite extensive, there was never really a progress towards defining a _language_ and a _representation format_. As a result GORE served merely as citation for research work that claim to rein in the wild west of requirements. Here is a picture of GORE in a nutshell:
![alt text](images/gore-nutshell.png)

But now, what if we create a Semantic Web ontology for GORE (i.e, give it both formalization and language) and then let a good LLMs chew human-created requirements and generate the knowledge graph? Would it work? Well I am not the first to attempt the ontology part, aparently there ought to be several ontologies out there like GORO etc. But try locating them. And one more thing, the papers that describe the proposed ontologies actually do not consider integrating system design! I am mean, when you describe what you want your air handling unit to do, you refer to components in the system dont you? Like, "when the outdoor temperature is less than freezing point, ensure that the heating coil is not frosted". So, here is my retake on the GORE ontology (it is part of Elementary, classed under ````RequirementsConcept```` just in case you want to go and browse the ontology right away):

![alt text](images/requirements-concept.png)

If you are, like me, too lazy to load the ontology in [Protege](https://protege.stanford.edu/) and browse, here is a screenshot:
![alt text](images/requirements-entities.png)

### Requirements Specification

I think the _structural_ part related to describing the requirements, goals, and their inter-relationships are not hard to understand. But what is crucial for our vision of achieving autonomous systems is the _goal specification_ part. That is, how do we formally specify a goal in terms of the physical variables and state of the components? This is a rabbit hole and we dont have Alice kinds in engineering. There are tons of standards that address this need: Signal Temporal Language (STL), Temporal Logic of Actions (TLA), Metric Temporal Language (MTL), Timed Computation Tree Logic, KAOS Assertion Language, etc. Each of these have at least something very interesting and unique which will further out objective. But the question is: what kind of tooling is required to integrate their use in to existing engineering practices? We know that most of our requirements are text, and in some cases, augmented with semi-formal diagrams like UML. So, things like STL, TLA, et al are nice, but we hit a roadblock because requirements engineering tools used in practice dont care about formal representation and standard grammar.

#### Ok, now we let loose the LLMs

Trying to goal specifications seemed like a dead end, until quite recently when I realized how good LLMs are getting at reasoning about requirements text. Not surprsing because requirements texts are quite close to be being _controlled natural language_ ([the work](https://attempto.ifi.uzh.ch/site/pubs/papers/doctoral_thesis_kuhn.pdf) by Tobias Kuhn is worth reading). I think this makes it super nice for the attention layers. I took a set of real-life requirements (120 of them of different qualities) and ran them through GPT5.2 and asked it to first translate them to CNL and then after manual inspection ti each of STL, TLA, and MLA. Manual check of the intermediate outputs revealed no errors. I mean, it just worked! The LLM even indicated incomplete and ambiguous input. Take for example:

> Consider this requirement created by human expert. Translate this to controlled natural language: "The air handling unit should respond to changing air demands by considering a combination of signals from the supply pressure sensor and the air flow demand from the VAVs. The AHU should maintain a minimum supply pressure even if VAV demand is low. If the VAV demand goes to zero, then the AHU should go in economy mode"

The proposal for CNL is:

Controlled Natural Language (CNL)
* **Always**, the Air Handling Unit shall calculate its target output based on the supply pressure sensor signal and the VAV air flow demand signal.*
* **Always**, the Air Handling Unit shall maintain the supply pressure at or above the minimum supply pressure threshold.*
* **If** the VAV air flow demand signal is equal to zero, **then** the Air Handling Unit shall enter economy mode **within** \(T\) time units.*


For STL it proposed:

We define the following continuous signals and constants:
\(P_{sup}(t)\): Supply pressure sensor signal at time \(t\)\(D_{vav}(t)\): 
Total VAV air flow demand signal at time \(t\)\(M_{eco}(t)\): 
Binary indicator for economy mode (\(1\) if active, \(0\) otherwise)\(P_{min}\): 
Constant value for the minimum allowable supply pressure
Here are the formal STL formulas:

1. Minimum Pressure Maintenance The system must guarantee that the supply pressure never drops below the minimum safety threshold, regardless of demand.\(\Box \,(P_{sup}(t)\ge P_{min})\)

2. Economy Mode ActivationIf the VAV demand drops to zero, the system must switch to economy mode within a maximum time delay of \(\tau \).\(\Box \,\left((D_{vav}(t)==0)\implies \diamonds _{[0,\tau ]}\,(M_{eco}(t)==1)\right)\)

Additionally, and rightfully, GPT5.2 points out:
>What is the acceptable reaction time delay (\(\tau \)) for entering economy mode? Do you have a specific mathematical function (\(f\)) that combines the sensor signals?

It also outputs the list of variables that I need to bind:

> Supply Air Pressure: Measurable pressure signal from the duct sensor. Airflow Demand: Combined flow request signal from the VAV boxes. Minimum Supply Pressure: Lowest allowable pressure limit for the system. Zero Demand State: Condition where VAV airflow requests equal zero. Active Demand State: Condition where VAV airflow requests exceed zero. Economy Mode: Low-energy operational state of the AHU.

We can now visualize how a model based on GORE would like in the knowledge graph:

![alt text](images/requirements-concept-example.png)

```sparql
@prefix elem: <http://w3id.org/elementary#> .
@prefix ex:  <http://example.org> .
@prefix rdfs: <http://w3.org> .
@prefix xsd:  <http://w3.org> .

### 1. SYSTEM REQUIREMENTS INSTANCES ###

ex:AirConditionedSpace a elem:Context ;
    rdfs:label "Room Comfort" .

ex:OptimizeAHU a elem:Requirement ;
    rdfs:label "Optimize AHU" ;
    elem:addresses ex:AirConditionedSpace ;
    elem:specifies ex:AirflowGoal .

ex:AirflowGoal a elem:Goal ;
    rdfs:label "Airflow" ;
    elem:hasStateSpecification ex:AirflowStringLiteral .

# Represents the literal block linking text, variable, and state
ex:AirflowStringLiteral a elem:StringLiteral ;
    rdfs:label "..." ; 
    elem:refersTo ex:AirPressureVar ;
    elem:physicalProcess ex:RunState .


### 2. SYSTEM DESIGN INSTANCES ###
ex:AHU_System a elem:System ;
    rdfs:label "AHU" ;
    elem:fulfills ex:OptimizeAHU ;
    elem:hasComponent ex:FanComponent .

ex:FanComponent a elem:Component ;
    rdfs:label "Fan" ;
    elem:manages ex:PressurizationMech .


### 3. PHYSICAL MECHANISM & PROCESS INSTANCES ###
ex:PressurizationMech a elem:PhysicalMechanism ;
    rdfs:label "Pressurization" ;
    elem:hasVariable ex:AirPressureVar .

ex:AirPressureVar a elem:PhysicalVariable ;
    rdfs:label "Air Pressure" .
```

So, given an _integrated_ knowledge graph like the one above, it is now easy to logically reason which requirements affect a physical variable, and how the system delegated to fulfill the requirement can potentially achieve the goal. The last part, i.e., how the goal could potentially be achieved, needs reasoning about how the physical mechanism works. Thats where we head to next.

## Physical Mechanism: How do we capture common sense physics behind components and systems?

With mathematical equations, of course. That would be certainly provide much more beyond _common sense_. A mathematical model that represents the near exact functioning of a partcular component would however need parameterization (to capture the components sizing and dynamics). This is more often than not an extensive work. But, for many cases, especially those where only need causal graphs with perhaps a bit of idea on dynamics, we could do with a much more generic or qualitative model of physics. I came up with two ways to do this: (1) describe the physical mechanism in terms of variables and their dependency, and (2) create functional mockup unit of simulation of generic components and describe its interfaces. The latter can capture general dynamic responses. The former can be encoded in to the knowledge graph and does not require any execution engine or solver. Let me briefly describe these two things.

### Qualitative Model of Physical Processes and Mechanisms
Lets resort to visuals again. The key concepts required to describe physics underlying a electro-mechanical component are:
![alt text](images/physical-concepts.png)

There is already some semantics captured by the _has variable_ relationship. Independent variables are those that influence the outcome of the mechanism. Manipulated variable is a specific kind that can be externally manipulated. The outcome is reflected by the change of the dependent variable. Mechanisms can be linked with one another such that "stuff" (a nice term that covers substance, energy, and force) flows between them.

Applying these concepts to a simple process where oil is fired up to heat water in pipes of boiler, the graph would look like:
![alt text](images/physical-concepts-example.png)

Elementary also allows conditions for the mechanism to be expressed. For example:
![alt text](images/physical-concepts-expressivity.png){width=40%}

So far, we have discussed only the physics part. The qualitative model can be "linked" to the structural model by linking the variables to the physical ports where they can be observed or manipulated. Again, example:
![alt text](images/physical-linking-to-component.png){width=20%}

Once you have a component and its underlying physics, you can reason about interaction between connected components by chaining the dependent and independent variables dealing with same stuff. For example:

![alt text](images/physical-concepts-chaining.png){width=40%}

You might have a nagging thought: do you need to do this for every component instance in your system? Well, not, and thats the beauty of the approach. You can describe a _stereotypical_ component using above approach. In fact, a physical mechanism is even more generic -- it can be used by muliple component types. The _stereotypical_ component can be stored in a library and then every real component of that class can be _automatically_ associated to the stereotpye. I have create an extensive library for heating, ventilation, and air-conditioning components. 

## Automation: How can we describe standard control and coordination strategies?