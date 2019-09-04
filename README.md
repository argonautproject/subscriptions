# Argonaut Subscriptions

## Timeline

- April Launch!
- April - May
  - Develop use cases (cast a wide net)
  - Review work on Subscriptions to-date
  - Define scope (and out-of-scope)
  - Client outreach
  - Birds of feather at HL7 May Working Group Meeting
- June - Aug
  - Draft spec
  - Publicly deployed reference implementation(s) that meet the draft spec
- Sep - Dec
  - Connectathon at HL7 September Working Group Meeting
  - Publish Argonaut Subscriptions IG 1.0


## Overview

## Specifications and Prior work

 - [FHIR Resource Subscription](http://build.fhir.org/subscription.html) - The subscription resource is used to define a push-based subscription from a server to another system. Once a subscription is registered with the server, the server checks every resource that is created or updated, and if the resource matches the given criteria, it sends a message on the defined "channel" so that another system can take an appropriate action
 - [210901 FHIR Subscriptions Connectathon Track](http://wiki.hl7.org/index.php?title=210901_FHIR_Subscriptions) - Latest of several HL7 Connectathons to advance the maturity of the interactions and resources associated with FHIR Subscriptions.
## Use Cases

- Clinical information has changed
- Scheduling: ( e.g., Argonuat Scheduling Prefetch )  Update provider availability to 3rd parties
- Provider Directory:  Updates to Directory when provider information has changed (such as credentials or qualifications).

## Limitations

## Open Questions

## Resources

- Notes & slides on Google Drive ([folder](https://drive.google.com/drive/folders/1I8zbQ1Yz3T9IwSumEmStRIkqSSI6iODm))
- Realtime chat on [Zulip](https://chat.fhir.org/#narrow/stream/argonaut)
- Spec development on [GitHub](#)
- Granular decisions via [GitHub Issues](https://github.com/argonautproject/subscriptions/issues) + PRs
- Calls every two weeks (or more often, if needed)

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

