# Vault Advisor

# Goal

Help security teams make their Vault-managed systems more secure, by validating and suggesting potential configuration changes.

A by-product of this automation should be the ability to scale people processes, to manage more and larger systems efficiently and reliably.

# Scope of Solution

The high level approach will be analyis of the Vault configuration and usage patterns, to find opportunities for improved configuration.

There are many ways in which this can be realized. Two key characteristics of a given solution are:

* **Analysis Approach**
  * **Static Analysis** considers only the Vault policy configuration.
  * **Dynamic Analysis** considers both the (static) Vault policy configuration and the secret access patterns of principals (people, machines and services) recorded in the Vault audit log.

* **Analysis Level**
  * **Validation** of a potential change: Is the change valid, both generally and with the current (or some other) Vault configuration?
  * **Explanation** of a potential change: If the change is valid, what will its effect be? If it is not valid, why not?
  * **Rating** of a potential change: How does the change stack up, against (multiple) metrics and/or against other changes?
  * **Suggestion** of potential changes: A ranked list of potential changes, with explanation and rating (that are of course all valid).

# Roadmap

To get to a Minimum Viable Product in a timely manner, and develop an R&D pipeline, we will look at defining a roadmap with a series of deliverables, that grow the scope of solution over time.

## Strawman Roadmap

* Deliverable 1: Validation and Explanation via Static Analysis.
* Deliverable 2: Add Rating and Suggestion, but still based on Static Analysis.
* Deliverable 3: Add Dynamic Analysis, so that suggestions and ratings consider the pattern of historical use, and not just the policy configuration.

# Design Notes

## Deliverable 1: Validation and Explanation via Static Analysis

Effectively this will build `vault plan`, akin to `tf plan` and `nomad plan`:

* Changes are proposed by users (security team and others with delegated permissions).
* Mulitple changes may be evaluated independently.
* Changes may be chained: Evaluate one change using the successfully application of another change as the starting point.

Techniques to consider for Valildation include:

* SAT-based constraint solving, ala http://emina.github.io/kodkod/

Techniques to consider for Explanation include:

* Constraint solving (see above)
* LIME ([Local Interpretable Model-Agnostic Explanations](https://homes.cs.washington.edu/~marcotcr/blog/lime/))

## Deliverable 2: Rating and Suggestion via Static Analysis

* Ranking v absolute rating?
  * Can rank (relative to one another) even if don't have absolute metrics
* Multiple metrics
  * Attack surface
  * Simplicity
  * Surveyability
* Multi-objective optimization
  * Tuning knobs to choose how to balance competing 

### Change Suggestion Service Architecture

Off-line system with multiple components:

* Repository
  * Same as for `vault plan` in Deliverable 1?
* Proposer
  * Need to identify a short-list of techniques for exploration of 'neighborhood' of current configuration.
* Evaluator
  * Validates and rates

Techniques to consider include:

* memoization, to fingerprint configurations already tried.

## Deliverable 3: Dynamic Analysis

Use techniques such as summarization or sketching to reduce the volume of data that needs to be retained to reason about usage patterns?

Summarization and sketching may also help identify outliers:

* Potential items to drop support for in new proposals?
* Gateway to anomaly detection.

# Relevant Work

## Adjacent Possibilities (Phrase/Concept)

"The next, realistic thing you can do to move towards your goal. The opposite of a distant ideal."

This might be a nice way to express the idea that Vault Advisor lets security team members chip away, interatively and incrementally, at reducing the attack surface of the systems under management.

This was found in a recent blog post by a Bay Area security blogger:
<https://danielmiessler.com/blog/adjacent-possible>.

## Attribute Transformation

[Attribute Transformation for Attribute-Based Access Control (PDF)](http://profsandhu.com/confrnc/misconf/abac17-prosun-pre.pdf)

Policy Attributes = attributes used in authorization policies.

Attribute reduction  

* Contract a large set of assignments of non-policy attributes into a possibly smaller set of policy attribute-value assignments.  
* Useful for
  * abstracting attributes that are too specific for particular types of objects or users
  * designing modular authorization policies
  * modeling hierarchical policies.

Attribute expansion  

* Performing a large set of attribute-value assignments to users or objects from
a possibly smaller set of assignments.
  
Access Control Models

* DAC (Discretionary Access Control) [22] allows resource owners to retain control on their resources by specifying who can or cannot access certain resources.
* MAC (Mandatory Access Control) [22] has been proposed to address inherent limitations of DAC such as Trojan Horses. Mandates access to resources by pre-specified system policies. 

These models emphasize the specific policies of owner-based discretion for DAC and one-directional information flow in a lattice of security labels for MAC.

* RBAC (Role Based Access Control) [21] is a policy neutral, flexible and administrative-friendly model. Notably RBAC is capable of enforcing both DAC and MAC.

MAC is also commonly referred to as LBAC (Lattice-Based Access Control).

Attribute Based Access Control (ABAC) has gained considerable attention from businesses, academia and standard bodies in recent years [11]. ABAC uses attributes on users, objects and possibly other entities (e.g. context, environment, actions) and specifies rules using these attributes to assert who can have which access permissions (e.g. read, write) on which objects. Several attribute based access control models have been proposed in the literature [5, 15, 23, 27]. These models have been proposed for different application domains including Cloud Infrastructure [6, 3, 14, 20, 25]

## Policy Graphs

[Network Policy Whiteboarding and Composition (PDF)](http://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p373.pdf)

Policy Graph Abstraction (PGA)

 * Graphically express network policies and service chain requirements
   * Directed multi-graph
   * Vertices represent groups of network endpoints sharing common properties (expressed as _labels_)
   * Directed edges express intents on communication from source to destination group
 * Different users independently draw policy graphs that can constrain each other.
 * PGA graph captures user intents and invariants
   * facilitates automatic composition of overlapping policies into a coherent policy

[PGA: Using Graphs to Express and Automatically Reconcile Network Policies (PDF)](http://conferences.sigcomm.org/sigcomm/2015/pdf/papers/p29.pdf)

* Policy composition
  * preserve correct _joint intent_ without intervention from human oracle
  * requires capturing _internal intent_ of each policy writer in each policy 
  * graphical model is simple and intutive
* Service Chain Analysis
  * system _must model the behavior of service functions_ for successful composition analysis
  * c.f. Jeff's idea of executing Vault code to validate policies

## Stepwise Refinement

Cocoon: Correct by Construction Networks using Stepwise Refinement

<https://www.usenix.org/system/files/conference/nsdi17/nsdi17-ryzhyk.pdf>

<https://github.com/ryzhyk/cocoon>

* Design and verification of complex networks using stepwise refinement to move from a high-level specification to the final network implementation.
* Specify intermediate design levels in a hierarchical design process that delineates the modularity in complicated systems.
*  Decomposed system cleanly into abstractions for each mechanism.
*  Separates static design from its dynamically changing configuration. Verify former at design time. Check latter at run time using statically defined invariants.

* Roles model arbitrary entities.
* Roles have Policies.
* Assumptions.
* Refinements replace one or more Roles with more detailed implementation.

Use Sentinel to express constraints in Vault configuration, and then verify or possibly generate configuration?

Corral model checker. Boogie DSL.
Non-deterministic simulated execution. Boogie's `havoc` operator.

# Backlog

1. Evaluate more Access Control literature
  * <http://www.codaspy.org/accepted-papers>
  * [ACM Search](http://dl.acm.org/results.cfm?query=Canonical+Completeness+in+Lattice-Based+Languages+for+Attribute-Based+Access+Control&Go.x=0&Go.y=0)
  * Patterns for session-based access control
  * The Traust Authorization Service
  * RestACL: An Access Control Language for RESTful Services
  * On Completeness in Languages for Attribute-Based Access Control
  * Canonical Completeness in Lattice-Based Languages for Attribute-Based Access Control

1. Check PLDI and workshops
  * http://conf.researchr.org/home/pldi-2016
  * http://conf.researchr.org/track/pldi-2016/FMS-2016-papers  
    “While the fields of security and of formal methods/programming languages are thriving areas of computer science, the communities are mostly disjoint, and though there are several formal techniques used for ensuring security, there is no systematic use of emerging powerful formal techniques in security.”  
  * Report on the NSF Workshop on Formal Methods for Security. Stephen Chong, Harvard University and Joshua Guttman

1. Investigate Group Based Policy (GBP)
  * Part of Opendaylight opensource SDN initiative
  * https://wiki.opendaylight.org/view/Group_Policy:Main
  * https://wiki.opendaylight.org/view/Group_Based_Policy_(GBP)/Theory
    * Intent System
    * Promise Theory
    * 20+ year line of research and production systems
