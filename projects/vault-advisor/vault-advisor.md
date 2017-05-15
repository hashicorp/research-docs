# Vault Advisor

# Goal

Vault Advisor's goal is to help security teams make their Vault-managed systems more secure.

Because it is an automated tool, a by-product should be the ability to scale people processes, to manage more and larger systems efficiently and reliably.

# Approach

The high level approach will be analysis of the Vault configuration and usage patterns, to find opportunities for improved configuration.

Within this, there are many possible solutions. We decompose the problem in a number of ways:

## Best Practice Oriented

We will use the familiar concept of best practices, both in development and the user interface of Advisor. To this end, we will:

* Build a repository of Vault best practices.
* Prioritize best practices for prototyping as Advisor features.
* Present Advisor's recommendations in terms of compliance versus non-compliance to particular best pracitices.
* Allow the operator to enable and disable recommendations on a per-best practice basis.

In this way, a best practice is the Advisor unit of modularity, that we will use to deliver a minimum viable product and then to iteratively add more features.

The ultimate goal should be for the best practice repository to be a shared asset, used by product, engineering, sales and marketing to reason and communicate about Vault's features and value proposition.

Note: We consciously choose not to introduce the concept of bad practices. In the design pattern community, the concept of anti-patterns sprang up (to mean things that look like good patterns, but in fact are not), but it is not clear how much value they really add. Let's see how far we can go without doing the same.

## Dynamic Analysis First

There are two broad choices for the kind of analysis Advisor can perform:

- **Static Analysis** considers only the specified Vault configuration.
- **Dynamic Analysis** considers both the (static) Vault policy configuration and the pattern of actual use of secrets.

We will start with dynamic analysis first, for the following reasons:

* Without information on the actual usage, the ability of the system to guide the operators towards a configuration with minimum attack surface is significantly imparied.
* A lot of value can be delivered based on simple counts and histograms of secrets usage, which should allow us to deliver a Minimum Viable Product more quickly and with lower risk.
* Static analysis, counter-intutively, is more complex both in terms of the tools required and the need to frame the problem correctly to get useful answers. Building simple dynamic analysis first will help us frame the problem better, and give us more time to investigate the state of the art in static analysis.

## Policy Changes First

A Vault configuration reasons about three classes of entity:

* **Principals** Users and applications that are clients of Vault
* **Policies** A set of capabilities
* **Secrets** The sensitive information being managed via Vault

In the first instance, we will restrict Advisor's recommendations to changes to policies, for the following reasons:

* Policies are internal to Vault, and hence can be reconfigured without requiring coordinated changes in external systems.
* Because policies control access to secrets, changing policies alone should be sufficient to achieve important goals such as minimization of privilege and elimination of unnecessary shared privilege.

It is not clear whether changes to principals or secrets should follow next. We defer the decision for now, but note that:

* Both changes to principals and secrets require coordination with external systems
  - The system of record in the case of principals.
  - The applications and users that use them, in the case of secrets
* Therefore, support for workflows that coordinate changes across multiple systems will probably be a part of the solution.
* Vault is uniquely positioned to secure these change authorization workflows, and hence should probably own them in some way.

# Architecture

Advisor Audit Backend
* A Vault Audit Backend that consumes all Vault events and relasy them to the stadalone Advisor executable.
  - Does not send actual secrets to Advisor (only non-sensitive information such as token ids).

Advisor Executable
* Consumes Audit Backend output.
* Computes and store statistics on Vault operations.
* Can be prompted to run some or all of the available best practice analysis modules, and deliver a report.
  - As part of analysis, will use its privileges to scan the current state of the Vault configuration, to provide a snapshot to compare to the pattern of actual usage.

Vault Simulator
* Standalone executable that can drive a Vault instance, to produce audit output to feed to Advisor
  - Required to produce a variety of audit logs embodying different patterns of usage
  - Will be used for regression testing.
  - Possibly also useful as a tool for dialogue with Vault user community, given the impossibility of obtaining real audit logs from external organizations.

# Initial Prototype

To start prototyping, we will address the following best practices:

* Minimal Root Token Use
* Minimized Capabilities

We will not address:

* Support for Vault High-Availability or Multi-Cluster Replication.
* High-Availability of Advisor itself.
* GUI

## Design Notes

* root token initially?
* ability to list every key in the system
* read policies: their definition
* read token information
* read mount tables?

# Backlog

Potential features and directions of investigation. Out of scope for now. Loosley ordered with nearer-term items first.

## Vault Plan

Terraform and Nomad both offer a 'plan' command, that allows the effects of a potential change to be explored before applying it to the system. A similar tool for Vault would be very attractive.

Potential features include:

* Showing what will be broken
  - Which users and applications will no longer be able to perform what actions on secrets, that they can currently.

## Time-Based Analysis

If we store histograms and/or sketches or other summarizations of the history of actual usage, we can offer features such as:

* Showing which policies and secrets are no longer being used in the way they were previously.
  - Globally, or by particular users or applications.
  - Both increase and decrease.
  - Beginning of anomaly detection and alerting.

## Workflow Integration

As noted [above](#policy-changes-first), the ability to securely manage workflows involving 3rd parties will be potentially very useful.

* Systems of record for identity and authN.
* Developers of applications that will be affected by changes to Vault configurations.
