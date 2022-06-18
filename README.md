# Argonaut Subscriptions

## Timeline 

- 2019: Jan - April
  - Project selection (selected as a proejct in 2019)
- 2019: April - May
  - Project launch!
  - Develop use cases (cast a wide net)
  - Review work on Subscriptions to-date
  - Define scope (and out-of-scope)
  - Client outreach
  - Birds of feather at HL7 May Working Group Meeting
- 2019: June - Aug
  - Draft spec (Argonaut Encounters IG)
  - Publicly deployed reference implementation(s) that meet the draft spec
    - Hosted: [subscriptions.argo.run](http://subscriptions.argo.run)
    - Client UI: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-client-ui)
    - Client REST Bridge: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-client)
    - Proxy Server: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-server-proxy)
- 2019: Sep - Dec
  - [Connectathon 22](https://confluence.hl7.org/display/FHIR/2019-09+Subscription) - September
- 2020: Jan - Feb
  - Project selection (not a selected project in 2020)
  - [CMS Connectathon 1](https://confluence.hl7.org/display/FHIR/2020-01+Subscriptions) - January
  - [Connectathon 23](https://confluence.hl7.org/display/FHIR/2020-02+Subscriptions+Track) - February
  - Publish Argonaut Encounters IG 1.0
  - Begin work on Backport to R4 IG 1.0
  - Attempt to finalize for ballot
- 2020: Feb - Dec
  - Continue work on R4 Backport IG
  - [Connectathon 24](https://confluence.hl7.org/display/FHIR/2020-05+Subscriptions+Track) - May
  - Focused efforts on R4 Backport IG
  - [Connectathon 25](https://confluence.hl7.org/display/FHIR/2020-09+Subscriptions+Track) - September
- 2021: Jan - Feb
  - Ballot Subscriptions Backport IG [version 0.1.0](http://hl7.org/fhir/uv/subscriptions-backport/2021JAN/)
  - [Connectathon 26](https://confluence.hl7.org/display/FHIR/2021-01+Subscriptions+Track) - January
  - Project selection (selected as project in 2021)
- 2021: March - April
  - Project launch: [Project Home](https://confluence.hl7.org/display/AP/Subscriptions+Home)!
  - Define scope of project for 2021
  - Lobby to add SubscriptionTopic and SubscriptionStatus to R4B (successful)
  - [Connectathon 27](https://confluence.hl7.org/display/FHIR/2021-05+Subscriptions) - May
  - Resolve/apply Backport IG ballot feedback
  - Add event triggers
  - Add notification shapes (notifications can include related resources)
  - Continue focused work on Backport IG in lead up to publication
  - [Connectathon 28](https://confluence.hl7.org/display/FHIR/2021-09+Subscriptions) - September
- 2022
  - `SubscriptionTopic` and `SubscriptionStatus` included in FHIR R4B (https://hl7.org/FHIR/R4B)
  - "Backport" IG supports topic-based subscriptions in FHIR R4B (http://build.fhir.org/ig/HL7/fhir-subscription-backport-ig)
## Overview

## Specifications and Prior work

- Current *R5* Specifications
  - [Subscriptions Framework](http://build.fhir.org/subscriptions.html)
  - [Subscription](http://build.fhir.org/subscription.html)
  - [SubscriptionTopic](http://build.fhir.org/branches/argonaut-subscription/subscriptiontopic.html)
  - [SubscriptionStatus](http://build.fhir.org/branches/argonaut-subscription/subscriptionstatus.html)

- Upcoming Connectathon Tracks
  - [Connectathon 28 (2021-09)](https://confluence.hl7.org/display/FHIR/2021-09+Subscriptions) - HL7 Connectathon Track focusing on the R4 Backport IG and additional resources in R4B.

## Use Cases

- Clinical information has changed
- Scheduling: ( e.g., Argonuat Scheduling Prefetch )  Update provider availability to 3rd parties
- Provider Directory:  Updates to Directory when provider information has changed (such as credentials or qualifications).

## Limitations

## Open Questions

## Resources

=======
# argo-subscription-docs
Home for controlled versions of documents related to Argonaut Subscriptions

## Notes on Workflows:

- Intended for use with: [SequenceDiagram](https://sequencediagram.org/).
- Text files with workflow diagram contents.
- SVG renders of the workflows.

## Notes on Graphs:

- Intended for use with: [Mermaid](http://knsv.github.io/mermaid/#/).
- Text files with graph contents.
- SVG renders of the graphs.

## Client Workflows

- Workflows with client behaviors during Subscription notifications (event processing).

## Error State Workflows

- Workflows with various error states to show handling considerations.

## Intermediary Workflows

- Workflows with registration and event notification when an independent third party is included.

## Missed Event Workflows

- Workflows showing missed events.
- More basic than Error State Workflows, but may handle similar situations.

## Registration Workflows

- Workflows showing the process of creating a Subscription.

## Backporting Changes to R4

- Proposal on how to backport the changes to Subscription slated for R5 to R4: [Backport to R4](backport-to-r4.md)

