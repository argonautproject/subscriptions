## Subscribing to events (rather that "search sets")
For many use cases (and much, much more generally), we want to winnow down a firehose of server events into a smaller event stream that a client can subscribe to -- in  way that is scalable and workable across diverse FHIR server architectures. One way to structure this is to imagine event sources and filters, which might be limited and pre-defined (perfectly acceptable for our purposes here) or perhaps created on-the-fly (in the most general case)...

## Two simple use cases: Argonaut ADT

The approach here is motivated by two relatively simple use case cases that we're targeting with the Argonaut 2019 Subscriptions project:

1. Allowing a patient app to subscribe to key encounter events (e.g., admission, discharge) for that patient
2. Allowing a practitioner app to subscribe to key encounter events (e.g., admission, discharge) for any patient "belonging to" that practitioner (e.g., by Patient.generalPractitioner)


## Example! Imagine three resources in a server: one event source + two filters

The event source (`EventDefinition`) here is a named `admission` event that requires no external parameters; the filters (`EventFilter`) each require a single exernally supplied parameter.

```js

// no parameters needed
{
  "resourceType": "EventDefinition",
  "id": "admission",
  "trigger": [{
    "type": "named-event",
    "name": "admission",
    "condition": {
      "expression": "%current.status = 'arrived' and %previous.status != 'arrived'"
    }
  }]
}

// patientId parameter needed
{
  "resourceType": "EventFilter",
  "id": "by-patient",
  "condition": {
    "expression": "%current.subject = %patientId"
  }
}


// practitionerId parameter needed
{
  "resourceType": "EventFilter",
  "id": "by-gp",
  "condition": {
    "expression": "%current.subject.resolve().generalPractitioner = %practitionerId"
  }
}
```

## Now apply them in a a patient-focused Subscription:

```json
{
  "resourceType": "Subscription",
  "eventStream": [
    {"reference": "EventDefinition/admission"},
    {"reference": "EventFilter/by-patient"}],
  "eventStreamParameter": [
    {"name": "patientId", "value": "Patient/123"}]
}
```

## Or for the PCP-based subscription:
```json
{
  "resourceType": "Subscription",
  "eventStream": [
    {"reference": "EventDefinition/admission"},
    {"reference": "EventFilter/by-gp"}],
  "eventStreamParameter": [
    {"name": "practitionerId", "value": "Practitioner/456"}]
}
```

The thing I'm trying to drive toward is a model where an event source emits events into a stream,
composable with filters; so at each step, we know what the stream looks like, and it can be whittled down.
This makes it more clear at least to me how "encounters by patient" and "encounters by pcp" can share their
baseline event definition.

## The Rules

1. Each `EventDefinition` specifies a stream of triggering events (e.g., admission, discharge, or new-result-available), and
also provides a data-driven "condition" that pure-FHIR servers can use to determine when the event triggers
based on CRUD operations. Facade-type FHIR servers can implement a fixed list of event sources by
individually developing support for each and recognizing them by name.

2. Each `EventFilter` provides a way to narrow down a stream of events, and includes a data-driven "condition"
that pure-FHIR servers can use to generically apply the filter. Facade-type FHIR servers can implement a fixed
list of filters by individually developing support for each and recognizing them by name.

3. An `EventDefinition` or an `EventFilter` can be parameterized, e.g., the `by-patient` filter above requires
a `patientId` parameter to be supplied in order to apply the filter.

4. A `Subscription` can include an ordered `eventStream` list of `EventDefinition` (sources) and `EventFilter` (filter) references. The set of events grows each time a source appears in the list (i.e., the set of events up to that point is unioned with the set of events defined by that source), and the set of events is filtered each time a filter appears in the list (i.e., only events where the filter evaluates to `true` pass through).

5. A `Subscription` whose `eventStream` elements require named parameters must include a `eventStreamParameter` array
to supply all required parameters. (If multiple `eventStream` elements require parameters with the same name,
they must also share the same value; using a single list of key-value pairs ensures this.)

## Expressiveness

With this kind of framework, one could accumulate several event types within a single Subscription:

```js
{
  "resourceType": "Subscription",
  "eventStream": [
    {"reference": "EventDefinition/admission"},
    // assuming these exist ;-)
    {"reference": "EventDefinition/transfer"},
    {"reference": "EventDefinition/discharge"},
    {"reference": "EventDefinition/new-lab-result"} 
    {"reference": "EventFilter/by-patient"}],
  "eventStreamParameter": [
    {"name": "patientId", "value": "Patient/123"}],
}
```



One could also define a more pre-coordinated up-front event source:

```json 
{
  "resourceType": "EventDefinition",
  "id": "admission-by-patient",
  "trigger": [{
    "type": "named-event",
    "name": "admission-by-patient",
    "condition": {
      "expression": "%current.subject = %patientId and status = 'arrived' and %previous.status != 'arrived'"
    }
  }]
}
```

... with a single trigger and no downstream filters

```json
{
  "resourceType": "Subscription",
  "eventStream": [
    {"reference": "EventDefinition/admission-by-patient"}],
  "eventStreamParameter": [
    {"name": "patientId", "value": "Patient/123"}]
}
```

## Continuing to explore expressiveness

With this framework we could also generalize beyond admissions to any status change. Note I'm not
suggesting this would be the best way to decompose / model the situation here; just pointing out
that we have a broad design space to explore with this general approch of sources + filters.

```js
// no parameters required
{
  "resourceType": "EventDefinition",
  "id": "encounter-by-any-status-change",
  "trigger": [{
    "type": "named-event",
    "name": "encounter-by-any-status-change",
    "condition": {
      "expression": "%current.status != %previous.status"
    }
  }]
}

// status parameter required
{
  "resourceType": "EventDefinition",
  "id": "encounter-by-specific-status-change",
  "trigger": [{
    "type": "named-event",
    "name": "encounter-by-specific-status-change",
    "condition": {
      "expression": "%current.status = %specifiedStatus and %previous.status != 'arrived'"
    }
  }]
}
```
