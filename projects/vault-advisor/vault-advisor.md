# Vault Advisor

# Goal

Help security teams make their Vault-managed systems more secure, by validating and suggesting potential configuration changes.

A by-product of this automation should be the ability to scale people processes, to manage more and larger systems efficiently and reliably.

# Relationship to Security Design Principles

Saltzer and Schroeder's 1975 paper [The Protection of Information in Computer Systems](https://www.acsac.org/secshelf/papers/protection_information.pdf) (summary [here](https://courses.cs.washington.edu/courses/cse484/14au/reading/look-at-1975.pdf) and slightly extended [here](https://cryptosmith.com/2013/10/19/security-design-principles/)) proposed 8 design principles for information protection mechanisms:

* **Economy of mechanism**: A simpler design is easier to test and validate.
* **Fail-safe defaults**: Default Deny is the canonical example.
* **Complete mediation**: Ideally, access rights should be validated every time an access occurs. Cached values should be used as little as possible (and we might add, in a principled manner).
* **Open design**: Following Claude Shannon's 1948 dictum: "The enemy knows the system", security system designs should be subject to open review.
* **Separation of privilege**: Prefer multi-person authorization.
* **Least privilege**: Every person and program should operate with as few privileges as possible.
* **Least common mechanism**: Users and programs should not share mechanisms except where absolutely necessary.
* **Psychological acceptability**: The policy interface should reflect the user's mental model of protection. Users won't specify protections correctly if the specification doesn't make sense to them.

They still make an excellent starting point for secure system design, 42 years later.

Several of the principles are inherent in Vault:

* **Economy of mechanism**
  - Vault's APIs and CLI offer a rational set of mechanisms for organizations to build their security workflows on top of.
  - By delegating many hard parts of the problem (such as selection and implementation of crypto algorithms) to Vault, customers are left with a simpler task in reviewing their system security.
* **Fail-safe defaults**
  - Vault's Default Deny approach is the gold-standard for the principle of fail-safe defaults.
* **Open design**
  - Vault's core code is open-source, and the full codebase undergoes periodic 3rd party security audit.

The remaining principles can be supported by systems that use Vault, but *only when correctly configured*. This presents an opportunity for Vault Advisor to add value, by automatically detectiing violations of the design principles in Vault configurations and proposing preferable configurations that reduce the scope of or eliminate those violations.

At a high level, the immediate opportunities for Vault Advisor to help with enforcing the remaining principles are as follows:

* **Complete mediation**
  - Vault Advisor can report on lease use on secret and authentication tokens (TTLs and actual renewal statistics), to identify users and secrets that currently have no TTL or could be configured to renew more frequently.
  - QUESTION: Can a lease have no TTL?
* **Separation of privilege**
  - Vault Advisor can report on use of multi-user authorization (who, when, latency between first and last user)
  - QUESTION: Can we find good heuristics for suggesting cases where multi-user auth should be added? 
* **Least privilege**
  - Vault Advisor can monitor the actual use of secrets versus the secret access granted by policies, and suggest policy re-writes to reduce the access to the minimum required.
* **Least common mechanism**
  - Vault Advisor can identify cases where different policies give access to the same secrets, and suggest policy re-writes to eliminate this sharing.
* **Psychological acceptability**
  - Vault Advisor can hep make changes more acceptable, by offering automatic validation, and if necessary explanantions of the consequences of changes.
  - Vault Advisor should have a measure of the complexity of a proposed change, and if appropriate, offer multiple proposals, at different points in the space of complexity versus reduction in attack surface.

These are discussed in detail later on in this document.

# Scope of Initial Solution

The high level approach will be analyis of the Vault configuration and usage patterns, to find opportunities for improved configuration. But within this, there are many possible solutions. We scope the initial solution as follows:

## Dynamic Analysis

The analysis can consume different inputs:

  * **Static Analysis** considers only the specified Vault configuration.
  * **Dynamic Analysis** considers both the (static) Vault policy configuration and the pattern of actual use of secrets.

We choose to do dynamic analysis from the start, as without information on the actual usage, the ability of the system to guide the operators towards a configuration with minimum attack surface is significantly imparied.

## Capability Modification Only

Vault offers a capability-based security model. The security configuration can be viewed as a set of constraints on (princpal, action, object) triples, where the principal is a user or application, the action is a capability and the object is a secret or set of secrets, referenced by a secret path.

We choose to only propose changes to capabilities in the first instance, and hence not to propose changes to secrets layout or principals.

Changes to secrets layout are the obvious next variable to introduce, but will be more disruptivelikely make the problem more complex. Changes to principals are the most challenging to introduce, since Vault is not the system of record in this area. It is not clear that the upcoming work on Vault identity will change this.

# Prototyping Approach

Vault Advisor will most likely be delivered as a standalone executable, that is securely introduced to the Vault cluster that it will monitor.

The question of how Advisor will be connected to Vault needs to be answered, but should be deferred until we have a good understanding of the information that it requires from Vault, and the effect of latency and reliability of delivery of that information on the results that it can deliver.

Therefore, in the first instance, Advisor will be prototyped as:

* A modified version of Vault (in a separate research github repository) that
  - Tracks the usage statsistics for secrets
  - Dumps the required statistics in plain text when a new API call is made
* A standalone executable that consumes the plain text output from the modified Vault instance.
