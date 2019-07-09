## Argonaut Subscriptions: September 2019 Connectathon

We'll be participating in the Subscription track of the September 2019
Connectathon, testing the draft changes to the Subscriptions framework.  
The track is designed for clients and servers that want to try out a full
specturm of integration from:

1. Patient Encounter notifications to a consumer app via REST-Hook.
2. Group Encounter notifications to an intermediary or clearing house via REST-Hook.
3. Encounter notifications to a consumer app via Websockets.
- ? Encounter notifications to a consumer app via Server Side Events (SSE).
- ? Define a new topic as a change notification to a resource.
- ? Work with authentication
    - REST-Hook
    - Websockets

For clarity in this document, the following words are defined as follows:

- Client: System or device **receiving** notifications.
- Server: System or device **sending** notifications.
- Trigger Server: System generating messages which **cause** notifications.

Each scenario will include three components which will be provided.  The client and
server components will be of interest for testing, while the Trigger Server is an
artifact needed to run the scenarios.

- Server Proxy

    This system sits in front of a FHIR R4 Server and intercepts all calls relating 
    to these scenarios.  It handles triggering and sending notifications via the 
    specified channels to clients.  This was done to highlight the areas needed to 
    implement the Server portion of Subscription handling.

    Link to GitHub Project: 

- Client

    This system creates channel endpoints, generates subscription requests, listens
    for notifications, and provides UI notification to a user.

    Link to GitHub Project:

- Trigger Server

    This system received information from a client about what type of messages to
    generate, then creates messages and sends them to a FHIR server.

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

- Topic: `admission`
- Allowed filters: `member-of-group`/`:in`
- Allowed channel types: `REST-Hook`
- Allowed payloads: `Ping`, `ID-only`, `Full`


## Scenario 3: Encounter notifications to a consumer app via Websockets

## Potential Scenario: Encounter notifications to a consumer app via Server Side Events (SSE)

## Potential Scenario: Define a new topic as a change notification to a resource

## Potential Scenario: Work with authentication (REST-Hook)

## Potential Scenario: Work with authentication (Websocket)

