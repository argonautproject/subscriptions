# Argonaut Subscriptions

## Overview

## Specifications and Prior work

 - [FHIR Resource Subscription](http://build.fhir.org/subscription.html) - The subscription resource is used to define a push-based subscription from a server to another system. Once a subscription is registered with the server, the server checks every resource that is created or updated, and if the resource matches the given criteria, it sends a message on the defined "channel" so that another system can take an appropriate action
 - [210901 FHIR Subscriptions Connectathon Track](http://wiki.hl7.org/index.php?title=210901_FHIR_Subscriptions) - Latest of several HL7 Connectathons to advance the maturity of the interactions and resources associated with FHIR Subscriptions.
## Use Cases
These specifications are designed to support sharing any data that can be represented in FHIR. This means they should be useful for such diverse systems as:

## Limitations

## Open Questions

## Resources
 - [Overview Presentation](https://docs.google.com/presentation/d/14ZHmam9hwz6-SsCG1YqUIQnJ56bvSqEatebltgEVR6c/edit?usp=sharing)
 - [Client and Server Implementations](./implementations.md)
 - [Argonaut Project: Bulk Data Export Security Risk Assessment Report](./security-risk-assessment-report.pdf)
 - [Discussion Group (FHIR Zulip "Bulk Data" Track)](https://chat.fhir.org/#narrow/stream/bulk.20data)
