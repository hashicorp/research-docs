
# Vault Best Practices

## Relationship to Security Design Principles

Saltzer and Schroeder's 1975 paper [The Protection of Information in Computer Systems](https://www.acsac.org/secshelf/papers/protection_information.pdf) (summary [here](https://courses.cs.washington.edu/courses/cse484/14au/reading/look-at-1975.pdf) and slightly extended [here](https://cryptosmith.com/2013/10/19/security-design-principles/)) proposed 8 design principles for information protection mechanisms, which make an excellent starting point for secure system design:

* **Economy of mechanism**: A simpler design is easier to test and validate.
* **Fail-safe defaults**: Default Deny is the canonical example.
* **Complete mediation**: Ideally, access rights should be validated every time an access occurs. Cached values should be used as little as possible.
* **Open design**: Following Claude Shannon's 1948 dictum: "The enemy knows the system", security system designs should be subject to open review.
* **Separation of privilege**: Prefer multi-person authorization.
* **Least privilege**: Every person and program should operate with as few privileges as possible.
* **Least common mechanism**: Users and programs should not share mechanisms except where absolutely necessary.
* **Psychological acceptability**: The policy interface should reflect the user's mental model of protection. Users won't specify protections correctly if the specification doesn't make sense to them.

Vault itself adheres to these principles, and using Vault inherently conveys a number of them to applications: 

* **Economy of mechanism**
  - Vault's APIs and CLI offer a rational set of mechanisms for organizations to build their security workflows on top of.
  - By delegating many hard parts of the problem (such as selection and implementation of crypto algorithms) to Vault, customers are left with a simpler task in reviewing their system security.
* **Complete mediation**
  - All requests to Vault are completely mediated.
* **Fail-safe defaults**
  - Vault uses the Default Deny approach to secrets access.
  - Vault uses the most secure setting as the default for all configurable parameters.

* **Open design**
  - Vault's core code is open-source, and the full codebase undergoes periodic 3rd party security audit.

The remaining principles can be supported by systems that use Vault, but *only when correctly configured*. Advisor to add value, by automatically detectiing violations of the design principles in Vault configurations and proposing preferable configurations that reduce the scope of or eliminate those violations.

At a high level, the immediate opportunities for Vault Advisor to help with enforcing the remaining principles are as follows:

Security Principle | Vault Best Practices
--- | ---
Complete Mediation | [Leasing and Renewal](#leasing-and-renewal)
Separation of Privilege | [Multi-Authorization](#multi-authorization)
Least Privilege | [Minimal Root Token Use](#minimal-root-token-use)<br>[Minimized Capabilities](#minimized-capabilities)
Least Common Mechanism | [Context Specific Access](#context-specific-access)
Psychological Acceptability | 

# Best Practices

## Multi-Authorization

Multi-Authorization is a planned Vault feature, likely to be delivered in 2017. It was the subject of the first Vault RFC ([VLT-001: Multi-Authorization](https://docs.google.com/document/d/1hcCRUaYLJZaIrbAEP05xIEALe9e-JIxeEdElyqm5mp4), discoverable from the [Vault RFC list](https://github.com/hashicorp/engineering-docs/blob/master/vault/rfc.md)), but was deferred because Vault lacks a unified notion of identity, across different authN backends. The first steps to resolve this are underway, in the works discussed in RFC [VLT-007: Vault Identity](https://docs.google.com/document/d/1u-Z8BzfFLzLw10VtCVOHPl6B7gO3ecEX9VtW1v0dUpM).

Vault Advisor potential actions

* Recommend capabilities (within policies) that are not currently protected by Multi-Authorization, that would be good candidates to be.
  - Potential heuristics include the scope or sensitivity of an action (should be above a certain threshold

## Minimal Root Token Use

As per [root token generation guide](https://www.vaultproject.io/docs/guides/generate-root.html), it is generally considered a best practice to not persist root tokens. Instead a root token should be generated using Vault's generate-root command only when absolutely necessary.

Vault Advisor potential actions

* Monitor root token generation
  - Summary stats on all incidents of activity: who, when, what did
  - Send alerts: Out of scope of VA, which is an off-line tool, at least initially.
* Detect use of same root token over extended periods of time
  - Indicates persisting of the token. Bad practice.

## Minimized Capabilities

Security Principle: Principle of Least Privilege

Vault Advisor potential actions

* Identify unnecessary access
  - Disparity between access granted and used (per user, machine and app)
  - Propose policy changes to reduce access to that used
  - Propose policy changes to reduce scope of wildcards
  - Possible to eliminate wildcards?
* Offer different choices
  - complexity v. precision trade-off

Version 1: Which capabilities
* v2: [Parameter-Scoped ACLs](https://www.hashicorp.com/blog/vault-0-7/).


## Context Specific Access

Necessary for auditability. Need to know unambiguously who accessed what, how.

Vault Advisor potential actions

* Identify secrets that a user, machine or app has access to in one context as side-effect of access granted in another context.
  - Propose policy changes to spearate them.
* Identify apps using human credentials

## Leasing and Renewal

Vault Advisor potential actions

* Identify secrets not using lease renewal
  - Propose config changes for each one (after modify corresponding application)
  - Eventually manage or integrate with workflow to modify application?

* Monitor lease renewal
  - Histogram and distribution stats for renewal
  - Instances that are being revoked because fail to renew in time
  - Instances in danger of being revoked (not renewing around t/2)
  - Opportunities to reduce lease times/increase renewal rate
  - Number and rate of growth of outstanding leases

## Authentication Standards

Vault Advisor potential actions

* Report authentication history
  - Per user and app
  - machine authentication?
* Recommend instituting timeouts, requiring re-authentication
  - c.f. lease renewal on secrets
  - Look at distribution? How avoid it happening at the worst time?
* Recommend use of multi-factor authentication
* Time-based privilege bracketing
  - Who used crednetials out of hours?

* Roadmap to include configuring dynamic, risk- and/or anomaly-based re-authentication?

## Key Rotation and Rekeying

Vault Advisor potential actions

* Report history
* Recommend when key rotation and rekeying should be applied.
  - c.f. vehicle maintenance schedule

*** Vault master key, PKI backend, transit backend...

## Dynamic Secrets

Vault Advisor potential actions

* Identify secrets that could use dynamic secrets but are not
  - What signals do we have?

## Response Wrapping

Vault Advisor potential actions

* Identify potential applications of response wrapping
  - Secrets accessed exactly once
  - Some condition on secrets accessed repeatedly to indicate good candidates for conversion into wrapped response?

* Recommend tuning of TTL on single
* Report on failures
  - Failed to unwrap in time
  - Multiple unwrap requests (and where from?)

## Use of AppRoles

[AppRole documentation](https://www.vaultproject.io/docs/auth/approle.html)

* Templates for multiple workflows possible with AppRole?

## Revocation

https://www.hashicorp.com/blog/Vault-announcement/

Vault Advisor potential actions

* Report all revocations
  - Summarize by user, time etc.

## Auditing

Vault Advisor potential actions

* Report all changes to auditing
  - Breaks in own connection to Vault (gaps in sequence number on events?)
  - Performance of other audit backends (speed, continuous use)
* Report all sealing/unsealing of Vault
  - Which key shares were used
  - Delay between first and last share being provided
* Recommend changes to master key configuration?
  - More or fewer shares?
  - Higher or lower threshold?

## Use of Transit Backend

Vault Advisor potential actions

* Suggest use of context/per=transaction derived key, if not in use?
  - https://www.hashicorp.com/blog/vault-0-2/
* Performance? (Latency to perform the encryption/decryption)
* Distribution of requests from clients
  - over time
  - over clients
  - -> anomaly detection  

## Disable Unused Backends

Vault Advisor potential actions

* Report on access pattern of each backend
  - Distribution over time
  - Rank by recent usage (user-configurable timeframe)
* Recommend removal of backends not used for certain period of time?

# Backlog

## Vault Documentation

* [Token Auth backend](https://www.vaultproject.io/docs/auth/token.html)
* [RFCs](https://github.com/hashicorp/engineering-docs/blob/master/vault/rfc.md)
  - VLT-003: Request Forwarding: 

## HashiCorp Vault blog posts
https://www.hashicorp.com/blog/vault-category/
Evaluated up to 4/11/17 post by Distil Networks

[Distil Networks](https://www.hashicorp.com/blog/distil-networks-securely-stores-and-manages-all-their-secrets-with-vault-and-consul/)  

* Anything that must/should be Consul backend specific?
* `VAULT_TOKEN="VAULT-TOKEN-HERE"` in client env. Out of scope, presumably.
* Vault cluster config out of scope validation on roadmap?

  - https://www.hashicorp.com/blog/vault-0-6/

* Feature Documentation
  - AppRole https://www.vaultproject.io/docs/auth/approle.html
  - Response Wrapping

  - https://www.hashicorp.com/blog/vault-0-2/
  - https://www.hashicorp.com/blog/vault-0-3/
  - https://www.hashicorp.com/blog/vault-0-4/
  - https://www.hashicorp.com/blog/vault-0-5/
  - https://www.hashicorp.com/blog/vault-0-6/
  - https://www.hashicorp.com/blog/vault-0-6-2/
  - https://www.hashicorp.com/blog/codifying-vault-policies-and-configuration/ **
  - https://www.hashicorp.com/blog/addressing-top-security-threats-with-vault/ 
  - https://www.vaultproject.io/docs/concepts/response-wrapping.html
  - https://www.hashicorp.com/blog/cubbyhole-authentication-principles/
  - https://www.vaultproject.io/docs/concepts/seal.html
  - https://www.vaultproject.io/docs/configuration/index.html
  - https://www.hashicorp.com/blog/using-hashicorps-vault-with-chef/


## HashiCorp Vault Forum Discussions
[Vault Forum](https://groups.google.com/forum/#!forum/vault-tool)

* [AppRole approach vs. Token approach](https://groups.google.com/forum/#!topic/vault-tool/1yf-oqgLnec)
* [AppRole help(!) Unwrapping a secret needed for authentication when youre not authenticated…](https://groups.google.com/forum/#!topic/vault-tool/Y7QaQaBM3fo)
* [large number of leases](https://groups.google.com/forum/#!topic/vault-tool/ULQL8WLXSNY)
* [when to use cubbyhole?](https://groups.google.com/forum/#!topic/vault-tool/-RskTFbrOfg)
* [large number of leases](https://groups.google.com/forum/#!topic/vault-tool/ULQL8WLXSNY)
* [AppRole and upwrapping](https://groups.google.com/forum/#!topic/vault-tool/Y7QaQaBM3fo)
* [AppRole v. token](https://groups.google.com/forum/#!topic/vault-tool/1yf-oqgLnec)
* 
## 3rd Party Blog Posts

Google searches yielding good results:

* Vault AppRole

Posts to review:

* [SeatGeek use of Vault](http://chairnerd.seatgeek.com/secret-management-with-vault/)
* [Kickstarter use of AppRoles](https://kickstarter.engineering/ecs-vault-shhhhh-i-have-a-secret-40e41af42c28)
* [Monitoring ssh brute force attempts with splunk](https://danielmiessler.com/blog/monitoring-ssh-bruteforce-attempts-using-splunk/)


* [Hunting for powershell using heatmaps](https://medium.com/@jshlbrd/hunting-for-powershell-using-heatmaps-69b70151fa5d)


## 3rd Party Solutions

* [Goldfish Vault UI](https://github.com/Caiyeon/goldfish)  
  - [Announcement on Reddit](https://www.reddit.com/r/devops/comments/69ynv4/hashicorp_vault_ui/)

