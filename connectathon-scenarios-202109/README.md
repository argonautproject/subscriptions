## Argonaut Subscriptions: September 2021 Connectathon Scenarios

We'll be participating in the Subscription track of the September 2021
Connectathon, testing the draft changes to the Subscriptions framework.  
The track is designed for clients and servers that want to try out a full
specturm of integration from:

1. [Patient Encounter notifications to a consumer app via REST-Hook](#scenario-1).
2. [Group Encounter notifications to an intermediary or clearing house via REST-Hook](#scenario-2).
- ? Patient Encounter notifications to an app via Websockets.
- ? Patient Encounter notifications to an app via Server Side Events (SSE).
- ? Define a new subscription topic for change notifications to a resource type.
- ? Work with authentication
    - REST-Hook
    - Websockets
- ? Multiple Notifications in a single bundle
- ? Explore graphs of resources on Subscriptions or Subscription Topics

As we are approaching publication for the R4 changes, we will focus on using
the R4 Backport IG during this connectathon.  While R5 testing will be supported,
the focus will be on the Backport IG and resources added to R4B.

For clarity in this document, the following words are defined as follows:

- Client: System or device **receiving** notifications.
- Server: System or device **sending** notifications.
- Trigger: System or device generating messages which **cause** notifications.

Each scenario will include three tiers of components.  Specifically, the scenarios use:
- Server Proxy: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-server-proxy)

    This is a server system (.Net Core CLI with Kestrel Http Server) which sits in front
    of a standard FHIR Server and intercepts all calls relating to the scenarios. It handles 
    processing events and sending notifications via the specified channels to clients. 
    This was done to highlight the areas needed to implement the Server portion of Subscription handling.

    An implementer may choose to take a similar approach (of proxying calls), or may
    use a modified server directly.  Either option should result in the same behavior.

- Client UI: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-client-ui)

    This is a user interface project (React-based Web Application) which walks a user
    through the scenarios.  It includes everything that a client application would need
    to maintain subscriptions.

    Because a Web Application cannot host its own endpoints, while WebSocket scenarios
    interact directly with the Server, REST endpoints are hosted in a Client Host project
    and communication between the UI and Host is handled via WebSockets.


- Client Host: [GitHub](https://github.com/microsoft-healthcare-madison/argonaut-subscription-client)

    This is a server system (.Net Core CLI with Kestrel Http Server) which:
    - hosts the UI (if desired)
    - manages REST endpoints
    - communicates received REST notifications to the Client UI
    - Triggers events on the server (e.g., sends Encounter resources).

    Generally, this project is an artefact of testing.  However, implementers may be 
    interested in details about how the UI interacts with notifications or how REST endpoints
    are managed.


## <a name='scenario-1'>Scenario 1</a>: Single-Patient Encounter notifications via REST-Hook (e.g. to a consumer app)

#### Subscription information

- SubscriptionTopic: [`encounter-start`](https://github.com/argonautproject/subscriptions/blob/master/canonical/subscriptiontopic-encounter-start.json)
- Allowed filter searchParamNames: `patient`
- Allwoed filter searchModifiers: `=`
- Allowed channelTypes: `rest-hook`
- Allowed contents: `empty`, `id-only`

#### Workflow - [Diagram](https://github.com/argonautproject/subscriptions/blob/master/connectathon-scenarios-202109/svg/Scenario1_workflow.svg) - [Simplified Graph](https://github.com/argonautproject/subscriptions/blob/master/connectathon-scenarios-202109/svg/Scenario1_graph.svg)

- Optional SubscriptionTopic Discovery:
    - Client requests all `SubscriptionTopic` resources from the server.
    - Server responds with a bundle of `SubscriptionTopic` resources.
1. Select or Create a Patient:
    - Selecting an existing Patient:
        1. Client asks User for Search Information related to a Patient or information to Create a Patient.
        2. Client asks Server for a list of matching patients.
        3. User selects a patient to use in the Scenario.
    - Creating a Patient:
        1. Client asks Users for Patient Information.
        2. Client Creates Patient record on the Server.
2. Client creates a REST endpoint to receive notifications on.
3. Client sends [Subscription](http://build.fhir.org/subscription-admission.html) request to Server.
4. Server responds with status: `requested`.
5. Server sends a [Handshake bundle](http://build.fhir.org/notification-handshake.html) to the REST endpoint.
6. If successful, Server updates the Subscription to status: `active`.
7. Trigger Server sends Encounter for selected patient.
8. Server sends event notification ([empty](http://build.fhir.org/notification-empty.html), [id-only](http://build.fhir.org/notification-empty.html)) to client REST endpoint.
9. Client Notifies user that a notification has been received.

#### Client Information

- (Optional) Implement SubscriptionTopic discovery
- Either allow searching/selecting or creating a Patient
- Create a REST endpoint
- Send a Subscription request to the server
- Handle Subscription REST Handshake
- Receive notifications via REST endpoint
- Notify user that an event has occurred.
- Additional optional features
  - Request status information via the `$status` operation
  - Request prior event information via the `$events` operation
  - Parse additional content provided in the context of notifications

#### Server Information

- Implement SubscriptionTopic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Have some way of triggering events (e.g., accepting Encounter resources)
- Filter and match for applicable Subscriptions
- Send event notifications
- Additional optional features
  - Implementation of the `$status` operation to provide Subscription Status information
  - Implementation of the `$events` operation to allow query/retrieval of prior events
  - Provide additional content in the context of notifications (e.g., included Patient record)

#### Trigger Information

- Accept a patient id filter parameter
- Generate a Patient record (if it does not exist)
- Generate an Ecounter record for the specified patient
  - Once
  - Periodic

## <a name='scenario-2'>Scenario 2</a>: Group Encounter notifications to an intermediary or clearing house via REST-Hook

#### Subscription Information

- SubscriptionTopic: [`encounter-start`](https://github.com/argonautproject/subscriptions/blob/master/canonical/subscriptiontopic-encounter-start.json)
- Allowed filter searchParamNames: `patient`
- Allwoed filter searchModifiers: `in`
- Allowed channelTypes: `rest-hook`
- Allowed contents: `empty`, `id-only`


#### Workflow [(Diagram)](https://github.com/argonautproject/subscriptions/blob/master/IntermediaryWorkflows/svg/GenericGroup.svg)


#### Client Information

- (Optional) Implement SubscriptionTopic discovery
- Determine filter for Encounter based on Group Membership
- Create a REST endpoint
- Send a Subscription request to the server
- Handle Subscription REST Handshake
- Receive notifications via REST endpoint
- Notify user that an event has occurred.
- Additional optional features
  - Request status information via the `$status` operation
  - Request prior event information via the `$events` operation
  - Parse additional content provided in the context of notifications

#### Intermediary Information

- Implement SubscriptionTopic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Create REST endpoint for Server events
- Subscribe to Server for events
- Listen for event notifications
- Filter and match for applicable Subscriptions
- Send event notifications to Client

#### Server Information

- Implement SubscriptionTopic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Accept Encounter resources
- Accept Group Membership resources/changes
- Filter and match for applicable Subscriptions
- Send event notifications
- Additional optional features
  - Implementation of the `$status` operation to provide Subscription Status information
  - Implementation of the `$events` operation to allow query/retrieval of prior events
  - Provide additional content in the context of notifications (e.g., included Patient record)

#### Trigger Information

- Accept patient group membership information
- Generate one or more Patient records for a specified group
- Generate one or more Ecounter records for a relevant patient
  - Once
  - Periodic


## Optional Scenario: Patient Encounter notifications to an app via Websockets

#### Info

Use the existing Patient scenario, but use the `websocket` channel instead of `rest-hook`.

Changes to the scenario:
- Allowed channelType: `websocket`
- Allowed contents: `empty`, `id-only`, `full-resource`
- Requred support (client and server) of the `$get-ws-binding-token` operation

#### Goals

Test the definitions of the `websocket` channel in a well-understood context.


## Potential Scenario: Additional Security considerations

#### Info
Use an existing scenario, but add additional security to notifications e.g.:
- Encrypted contents
- SMART Backend Service Authentication

#### Goals

- Test various security procedures to build a set of recommendations
- Test ways to add required information to a subscription
  - Note: considering a proposal for some sort of `$secret` operation to allow clients to provide secrets


## Potential Scenario: Patient notifications for new Communication or Task steps

#### Info

Request from Patient Empowerment: can Subscriptions be used to notify patients that they have something
'new' in the context of Communication or Task steps.

#### Goals

- Define and test a SubscriptionTopic to cover the requested areas.
- Test the feasibility of the approach.

## Potential Scenario: Patient Encounter notifications to an app via Server Side Events (SSE)

#### Subscription Information

#### Info

Use the existing Patient scenario, but with a custom channel implementing Server Side Events instead of `rest-hook`.

Changes to the scenario:
- Allowed channelType: `sse` (?)
- Allowed contents: `empty`, `id-only`, `full-resource`

#### Goals

Test Server Side Events to see if they provide a good compromise between security, performance, and usability.  
Specifically, clients should be able to remain authenticated via standard RESTful mechanisms, but 
still have the performance of websockets.  Usability is a large unknown.

