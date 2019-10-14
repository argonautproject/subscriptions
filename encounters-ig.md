## Introduction

The Encounters Implementation Guide is based on FHIR Version R5 and defines the minimum
conformance requirements for supporting patient Encounter notifications.

## Background

Question: what is the context for background?

- Background around Subscription Changes (R4 - R5)

- Background around Encounters and why someone would want notifications

- Background around why there needs to be an Argonaut guide to Encounter Notifications

## Dependency on FHIR R5

- Issues with R4 (implementability)

- Changes in R5:
  - Split into Subscription and Topic
  - Allow servers to support a reduced number of Topics
  - Still generically useful to clients
  - Filters based on Search and existing modifiers
  - Topic filter restrictions, canFilterBy
  - Subscription implementation of filters

Subscriptions in FHIR R4 were designed to be very generic and generally unconcerned with
the internal operations of the server.  While powerful in concept, this design led to 
many servers not implementing Subscriptions.  Even on servers where it was implemented,
it was generally restricted to certain areas or concepts, which were not able to be 
documented with R4 resources.  In practice, this meant that implementing Subscriptions
required working extensively with a specific server implementation to discover and 
coordinate what functionality was usable and in which formats queries needed to be defined.

It was decided that the design of Subscriptions in R4 was not able to move beyond the
issues, and that a redesign was required.  In FHIR R5, Subscriptions have been broken into 
two resources, a Topic and a Subscription, which adress the issues with Subscriptions in 
R4.  Even with the redesign, there is a potential issue with interoperability based on
Topic definitions, which this guide aims to prevent.

With the new design, servers are able to support a reduced set of Topics, with a reduced
set of clearly-defined filters to clients.  This allows clients to be developed generically
while also lowering the burden on server developers.

Topics fall under two categories: conceptual and computable.  There is overlap between
the two (e.g., a server can computably define a new Admission encounter), but the server
is able to use whatever internal implementation makes sense to capture the concept being
expressed.  For example, in the Admissions Topic defined below, two possible computable
definitions are provided (in FHIRPath and using Query), but existing facade servers may
choose to simply implement the Topic as part of an existing HL7 v2 ADT workflow.

Given the goals of the redesign, filtering is based on Search and Search Modifiers.  This
allows for more code reuse (for clients and servers) and lowers the bar for implementation.

## Use case: encounter notifications

- Describe (briefly) use cases for encounters

There are a few key use cases that Argonaut focused on in terms of Encounter notifications:

- Single patient new encounter

This use case covers a system being notified when a particular patient of interest
has started a new encounter.

For example, a consumer device (e.g., phone) receiving notifications that a person is ..? (phrasing)

- Group patient admit (connectathon scenario 2)

This use case covers a system being notified when a patient within a defined group
has started a new encounter.

For example, a physician receiving notifications that one of their patients is being seen
by another physician or organization.

- Discharge use cases
  - Should these mirror admission use cases?  Haven't really discussed.

## Define Canonical Topics (Encounter Start + Encounter End)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Topic xmlns="http://hl7.org/fhir">
	<id value="encounter-start"/>
	<url value="http://argonautproject.org/encounters-ig/Topic/encounter-start"/>
	<title value="encounter-start"/>
	<status value="active"/>
	<resourceTrigger>
		<description value="Beginning of a clinical encounter"/>
		<resourceType value="Encounter"/>
		<queryCriteria>
			<previous value="status:not=in-progress"/>
			<current value="status=in-progress"/>
			<requireBoth value="true"/>
		</queryCriteria>
		<fhirPathCriteria value="%previous.status!='in-progress' and %current.status='in-progress'"/>
	</resourceTrigger>
	<canFilterBy>
		<name value="patient"/>
		<matchType value="=" />
		<matchType value="in" />
		<documentation value="Matching based on the Patient (subject) of an Encounter or based on the Patient's group membership (in)."/>
	</canFilterBy>
</Topic>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Topic xmlns="http://hl7.org/fhir">
	<id value="encounter-end"/>
	<url value="http://argonautproject.org/encounters-ig/Topic/encounter-end"/>
	<title value="encounter-end"/>
	<status value="active"/>
	<resourceTrigger>
		<description value="End of a clinical encounter"/>
		<resourceType value="Encounter"/>
		<queryCriteria>
			<previous value="status=in-progress"/>
			<current value="status:not=in-progress"/>
			<requireBoth value="true"/>
		</queryCriteria>
		<fhirPathCriteria value="%previous.status='in-progress' and %current.status!='in-progress'"/>
	</resourceTrigger>
	<canFilterBy>
		<name value="patient"/>
		<matchType value="=" />
		<matchType value="in" />
		<documentation value="Matching based on the Patient (subject) of an Encounter or based on the Patient's group membership (in)."/>
	</canFilterBy>
</Topic>
```

## Explain Topic derivation

Require servers to support searching on Topic.derivedFromCanonical and/or Topic.derivedFromUri.

Explain that servers are encouraged to add additional filters, but cannot remove existing
ones nor change the 'concept' of a Topic during derivation.

For example:
  - A server wanting to expose the same Topic, but with a different computable definition is OK.
  - A server wanting to expose additional canFilterBy parameters is OK.
  - A server wanting to remove an existing canFilterBy parameter is NOT ok.
  - A server wanting to derive a Topic based on a different resource is NOT ok (e.g., start/end of medication derived from encounter).

## Define bundle used in notifications

- Use of `history` bundle
- Extensions (on meta)
- Contents based on payload

## Define Profile on Subscriptions (channel, payload, etc)

- Must support for channel=`rest-hook`, payload={`empty`, `id-only`}

`Subscription.end` is a required field.  This field is necessary to detect stale subscriptions for removal.  Servers are allowed to determine a reasonable maximum time span, but MUST allow at least thirty one (31) days in the future.  The actual value used should be reasonably tied to the expected duration of the subscription, however it is understood that long-running Subscriptions will need to update this value periodically.  For example, on a server allowing the minumim span of thirty one (31) days, a Subscription created on 15 October 2019 would be allowed have an `end` set for no later than 15 November 2019.

Using a time less than one month is permitted.  For clients that expect to be short-lived, it is reasonable to set a time less than one month in the future (e.g., one hour, one day, one week, etc.).

Servers SHALL support updating the `end` field of an active Subscription so that a Subscription may be extended.  Servers are allowed to determine their maximum future span of time allowed when updating, given that it is also at least thirty one (31) days.

When an expired Subscription is detected, a server may choose to either remove the resource or update the `status` to `off`.

## Worked examples for end-to-end exchange

- Need workflow diagrams with examples here.