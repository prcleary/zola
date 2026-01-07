+++
title = "The Book of Why by Judea Pearl and Dana Mackenzie: book notes"

[extra]
mermaid = true
+++

- not sure it provides its own definition of causality (though does use metaphors about information flowing, or Y "listening to" X), but does critique some previous attempts
- human intelligence makes causal inferences, but statistics has often limited itself to statements about associations ("correlation is not causation"), with honourable exceptions
- "data is dumb": can't answer causal questions alone, e.g. about interventions, counterfactuals; a causal model (from scientific knowledge) is needed too
- artificial intelligence needs to address causal inference (the author started as a computer scientist)
- but mathematical treatment of causal inference has been lacking
- we now have causal diagrams and the *do calculus*, an algebra/notation with symbols representing causality
- fundamentally causal inference is based on an "inference engine": firstly use the causal model (typically a diagram), based on assumptions/knowledge, to decide if a causal question (expressed in do calculus) can be answered; if it can, derive an "estimand" which can be applied to data to give you an answer; if the model is correct and the data is sufficient, then the answer is correct; the inference engine can be automated
- three levels of questions of increasing difficulty that causal inference can answer (the "Ladder of Causation"): observed association, the effect of an intervention, counterfactuals ("what if")
- shows how causal diagrams can explain phenomena like confounding and mediation and explain various apparent paradoxes
- shows how the standard epidemiological definition of confounding is insufficient
- the three junctions of causal diagrams: chains, forks and colliders; these are not necessarily visible from data
- if $A\rightarrow B\rightarrow C$, controlling for B is bad and "prevents information from A getting to C"; same if you control for proxies of B
- the same happens if you control for B when $A\leftarrow B\rightarrow C$, but this is good (controls for confounding; the path via B is a *back-door path*)
- when $A\rightarrow B\leftarrow C$, B is a collider, and controlling for it (e.g. by restriction) then "information starts flowing" (a correlation is induced) even when A and C are independent; this explains a number of paradoxes (e.g. low birth weight, smoking and mortality)
- in a more complex causal model (>3 nodes), deal with confounding by blocking any back-door path (any path from X to Y that begins with an arrow pointing into X)
- sometimes we don't have data on variables that would allow us to block the back-door path
- can use *front-door adjustment* where there is a mediator and a confounder between X and Y: weighted combination of causal effect of X on mediator and causal effect of mediator on Y
- then sets out some axioms for do calculus (to obtain estimands by a process of simplification)
- then introduces *instrumental variables* (with John Snow example; "Water Company" is the instrumental variable), allowing you to use the front-door trick to deconfound the relationship without information on poverty, miasma, etc.
{% mermaid() %}
flowchart LR
A[Water company] --> B[Water Purity] --> C[Cholera]
D[Miasma, Poverty, etc] --> B
D --> C
{% end %}
- I lost the thread a bit from chapter 8 about counterfactuals; it includes a long and involved example about education, experience and salary, using a multi-step algorithm; need to read this chapter and later ones again
- then a chapter on mediation and estimating direct/indirect effects
- a final chapter on implications e.g. for artificial intelligence
- covers his development (and later rejection) of Bayesian networks as causal models
	
