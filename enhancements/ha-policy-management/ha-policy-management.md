title: ha-policy-management
authors:
  - "@nahorigu"
reviewers:
  - "@dmesser"
  - "@dgoodiwn"
approvers:
  - TBD
api-approvers: None
creation-date: 2026-01-29
last-updated: 2026-07-29
tracking-link: N/A
see-also:
  - https://issues.redhat.com/browse/OCPSTRAT-2649
replaces: N/A
superseded-by: N/A
---

# HA Policy Management

## Summary

This enhancement improves guideline compliance checks within the CI process (the Red Hat-internal pipeline for OpenShift) to improve overall HA.
Specifically, it integrates a mechanism to evaluate HA levels based on implementation status and developers' input.
By notifying developers of non-compliant components (via existing testing framwork and Jira),
the management process encourages developers to follow the guidelines.
All data will be stored in Sippy, allowing both developers and partners to grasp the overall HA status early and easily.

## Motivation

A service on an OpenShift cluster relies on multiple components, so overall cluster availability depends on the product of each component's availability.
If any component in the dependency chain lacks high availability (HA), total availability is degraded.

Currently, HA implementation is often left to developers’ discretion, leading to inconsistent or insufficient HA configurations.
Although general guidelines exist ([CONVENTIONS.md](https://github.com/openshift/enhancements/blob/master/CONVENTIONS.md#high-availability)),
they are often overlooked and the overall conformance status is unknown.
As a result, HA levels can be inconsistent across components, especially when new components are added or existing ones undergo major changes.

Therefore, an automated checking process and enforcement mechanism are needed.
This proposal aims to introduce a cluster-wide mechanism to ensure consistent HA implementation across components.

### User Stories

* As an OpenShift Product Manager, I want a clear overview of HA implementation status across components, so I can identify issues affecting overall HA quality earlier.
* As an OpenShift component developer, I want to be alerted to any HA gaps and have the opportunity to explain why HA is lacking, unnecessary, or when it's planned, reducing repetitive queries from end users.
* As a service provider on OpenShift, I want a stable, reliable platform with consistent HA, enabling easier service development without repeatedly consulting each component team about HA status and plans.

### Goals

* Collect HA policy data (defined below) during the CI process and notify component owners of any guideline violations.

### Non-Goals

* Extending HA policy management to cover general guideline compliance beyond HA is also out of scope for now.
* This proposal targets only core and infrastructure-related components, and the other components are out of scope in the first proposal.

## Proposal

### Establish the Tests

* Create a monitortest to collect HA policy information from running OpenShift clusters, then to evaluate whether each component meets the HA policy or not.
* Define the criteria which conditions should be met for each component pass an HA level check for an HA config.
* Define HA configs to define the type of HA feature to be handled (redundancy and health check in the first proposal).
* Define the criteria that must be met to pass the HA level check for each component and for each HA config.

### File Bugs for Violations

* The current status of HA level check is displayed in the dashboard of Sippy.
* If a failure is newly identified, a Jira ticket will need to be filed to track it.
* By tracking issues through Jira ticket statuses, the HA implementation status becomes transparent and can be properly managed.

### Workflow Description

The primary workflow is as follows:
1. Developers create PRs or commit their code.
2. OpenShift CI runs monitortest for ha-policy.
3. Sippy collects and shows test results in the Dashboard.
4. Prow monitors the failed test results, and create Jira tickets if needed.
5. Prow sets labels for tracking on the tickets.
6. Developers fix the failed HA policy check OR provide rationale/plans.

Some details for each step are shown below:
* In step 2, draft PR of the monitortest for ha-policy-check is here: https://github.com/openshift/origin/pull/31449/changes/5f9e5ced830bb85c24ff7d48fef35162dc74ebc3
* In step 4, the description of Jira ticket contains is generated based on the failure message of the monitortest, which typically saying some components in the associated namespace lack one or more HA implementations.
* In step 4, a separate Jira ticket is created for each namespace so it contains multiple failures which belongs to all workloads in the namespace.
  That's useful because namespaces closely align with component boundaries.
* In step 6, the Jira ticket can be handled in one of the following ways.
  With these handling, the component is treated as pass or flake, allowing the release.
  Without any handling, the component is treated as fail (Note that the latter case will be handled as flake in early phase to avoid flood of failures before the process matures).
  * to fix by implementing the specified HA,
  * to declare WONTFIX with some reason,
  * to declare pending with some plan.

Below are the other notes:
* Adding any approved exceptions in the monitortest code, a component can be always treated flake (as permanently approved whitelist).
  Those should likely transition from exceptions to just permanently approved whitelist with a comment explaining why, or a link to the jira that explains.

So the status of test result status is summarized like below:

| State                                                                      | Result |
| ---                                                                        | ---    |
| The component meets HA policy                                              | pass   |
| The component is permanently whitelisted                                   | flake  |
| The Jira ticket is pending (with rationale or plan)                        | flake  |
| No exception/whitelist entry exists, and the violation appears unapproved. | fail (flake in early phase) |

### Roll Out the Process

* Initially, any failures in the monitortest will be tolerated as a flake to avoid mass failures.
* Once the toolset of the new process are prepared and all Jira tickets created in early phase is handled, execute the rollout and start blocking the release based on failed cases.
* Then, new violations will immediately start failing jobs and be timely notified to dev teams.
  This prevents new components from coming in without the HA capability unless someone explicitly approves it, as well as regressions for existing components.

### API Extensions

N/A

### Topology Considerations

No specific changes required

#### Hypershift / Hosted Control Planes

No specific changes required

#### Standalone Clusters

No specific changes required

#### Single-node Deployments or MicroShift

N/A

### Implementation Details/Notes/Constraints

None (HA policy management is implemented in CI process outside OpenShift components)

### Risks and Mitigations

* Risk: Development teams bear the burden of responding to notifications in a timely manner to prioritize and plan the development of HA features.
* Mitigation: During the initial phase, the management process will only issue warnings (flakes) without blocking the actual release process.

### Drawbacks

None

## Alternatives (Not Implemented)

Not known

## Open Questions [optional]

- How to determine the exact coverage of target components of HA policy management?
- How to maintain and publish the result of HA policy management?
- Currently all defined HA configs are healthCheck and redundancy, but is there any other possible HA configs?
- Some of the process could be automated by AI agent.

## Test Plan

We start with limited enforcement (for example only on select components) to verify that the management process works properly, and then gradually expand the scope.

## Graduation Criteria

N/A.

### Dev Preview -> Tech Preview

N/A

### Tech Preview -> GA

N/A

### Removing a deprecated feature

N/A

## Upgrade / Downgrade Strategy

N/A

## Version Skew Strategy

N/A

## Operational Aspects of API Extensions

N/A

## Support Procedures

N/A

## Infrastructure Needed [optional]

N/A
