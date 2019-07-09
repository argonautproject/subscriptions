## Argonaut Subscriptions: September 2019 Connectathon

We'll be participating in the Subscription track of the September 2019
Connectathon, testing the draft changes to the Subscriptions framework.  
The track is designed for clients and servers that want to try out a full
specturm of integration from:

1. Patient Encounter notifications to a consumer app via REST-Hook.
2. Group Encounter notifications to an intermediary or clearing house via REST-Hook.
- ? Patient Encounter notifications to an app via Websockets.
- ? Patient Encounter notifications to an app via Server Side Events (SSE).
- ? Define a new topic as a change notification to a resource.
- ? Work with authentication
    - REST-Hook
    - Websockets

For clarity in this document, the following words are defined as follows:

- Client: System or device **receiving** notifications.
- Server: System or device **sending** notifications.
- Trigger Server: System generating messages which **cause** notifications.

Each scenario will include the above three tiers of components.  For each 
scenario, reference implementations for the Server and Client will be provided, 
as well as an implementation of the Trigger Server.  It is intented that an
interested party can replace any portion of the running scenario with their own 
implementation.

- Server Proxy

    This system sits in front of a FHIR R4 Server and intercepts all calls relating 
    to these scenarios.  It handles triggering and sending notifications via the 
    specified channels to clients.  This was done to highlight the areas needed to 
    implement the Server portion of Subscription handling.

    An implementer may choose to take a similar approach (of proxying calls), or may
    use a modified server directly.  Either option should result in the same behavior.

    Link to GitHub Project: 

- Client

    This system creates channel endpoints, generates subscription requests, listens
    for notifications, and provides UI notification to a user.

    Link to GitHub Project:

- Trigger Server

    This system receives information from a client about what type of messages to
    generate, then creates messages and sends them to a FHIR server.

    This component is an artefact of testing.  While source code is provided, a
    typical scenario will be generating events from another source.

    Link to GitHub Project: 

## Scenario 1: Patient Encounter notifications to a consumer app via REST-Hook

#### Subscription information

- Topic: `admission`
- Allowed filters: `Patient`
- Allowed channel types: `REST-Hook`
- Allowed payloads: `Ping`, `ID-only`, `Full`

#### Workflow [(Diagram)](https://github.com/microsoft-healthcare-madison/argo-subscription-docs/blob/master/RegistrationWorkflows/svg/Basic.svg)

1. (Optional) Client requests list of Topic resources from the server.
2. Server responds with list of Topic resources.
3. Client determines which Patient the Subscription will be filtered on.
4. Client creates a REST endpoint to receive notifications on.
5. Client sends Subscription request to Server.
6. Server responds with status: requested.
7. Server tests the REST endpoint.
8. Server updates the Subscription to status: active.
9. Trigger Server sends Encounter for specified patient.
10. Server sends event notification to client REST endpoint.

#### Client Information

- (Optional) Implement Topic discovery
- Determine filter for Encounter based on Patient (id)
- Create a REST endpoint
- Send a Subscription request to the server
- Handle Subscription REST Handshake
- Receive notifications via REST endpoint
- Notify user that an event has occurred.

#### Server Information

- Implement Topic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Accept Encounter resources (to trigger events)
- Filter and match for applicable Subscriptions
- Send event notifications

#### Trigger Information

- Accept a patient id filter parameter
- Generate a Patient record (if it does not exist)
- Generate an Ecounter record for the specified patient
  - Once
  - Periodic


## Scenario 2: Group Encounter notifications to an intermediary or clearing house via REST-Hook

#### Subscription Information

- Topic: `admission`
- Allowed filters: `member-of-group`/`:in`
- Allowed channel types: `REST-Hook`
- Allowed payloads: `Ping`, `ID-only`, `Full`

#### Workflow [(Diagram)](https://github.com/microsoft-healthcare-madison/argo-subscription-docs/blob/master/IntermediaryWorkflows/svg/GenericGroup.svg)


#### Client Information

- (Optional) Implement Topic discovery
- Determine filter for Encounter based on Group Membership
- Create a REST endpoint
- Send a Subscription request to the server
- Handle Subscription REST Handshake
- Receive notifications via REST endpoint
- Notify user that an event has occurred.

#### Intermediary Information

- Implement Topic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Create REST endpoint for Server events
- Subscribe to Server for events
- Listen for event notifications
- Filter and match for applicable Subscriptions
- Send event notifications to Client

#### Server Information

- Implement Topic discovery
- Manage Subscriptions
  - Accept Subscription requests
  - Validate REST-Hook channel endpoints
- Accept Encounter resources
- Accept Group Membership resources/changes
- Filter and match for applicable Subscriptions
- Send event notifications

#### Trigger Information

- Accept patient group membership information
- Generate one or more Patient records for a specified group
- Generate one or more Ecounter records for a relevant patient
  - Once
  - Periodic

## Scenario 3: Patient Encounter notifications to an app via Websockets

#### Subscription Information

- Topic: `admission`
- Allowed filters: `Patient`
- Allowed channel types: `Websocket`
- Allowed payloads: `Ping`, `ID-only`, `Full`

## Potential Scenario: Patient Encounter notifications to an app via Server Side Events (SSE)

#### Subscription Information

- Topic: `admission`
- Allowed filters: `Patient`
- Allowed channel types: `Server-Side-Events`
- Allowed payloads: `Ping`, `ID-only`, `Full`


## Potential Scenario: Define a new topic as a change notification to a resource

#### Basic Information

- Topic: `admission`
- Allowed filters: 

## Potential Scenario: Work with authentication (REST-Hook)

#### Subscription Information

- Topic: `admission`
- Allowed filters: `Patient`
- Allowed channel types: `REST-Hook`
- Allowed payloads: `Ping`, `ID-only`, `Full`

## Potential Scenario: Work with authentication (Websocket)

- Topic: `admission`
- Allowed filters: `Patient`
- Allowed channel types: `Websocket`
- Allowed payloads: `Ping`, `ID-only`, `Full`