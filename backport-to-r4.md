# Back-porting R5 Subscriptions to R4

The introduction of Topic-based Subscriptions targets FHIR R5, and marks a significant set of improvements to R4 Subscriptions.  Still, given that FHIR R5 will not be published until roughly early 2021 (and implementation will lag further), it's useful to define a standard way to "back-port" R5 Subscriptions functionality to FHIR R4.  This way, implementers invested in R4 can add Topic-based Subscriptions as an "à la carte" feature, gaining value early and laying a clear transition path.

## The (ahem!) Basic approach

We want to support Topic-based Subscriptions in R4 in a manner that's ["simple"](https://www.infoq.com/presentations/Simple-Made-Easy/), even if it's clunky. In other words, we want to avoid complex "overlays" of new subscription behavior on top of existing R4-specified behavior.  For this reason, we avoid trying to re-use (or subtly change the meaning of) FHIR R4 Subscriptions in our back-port. Instead, we rely on FHIR's `Basic` Resource type to convey information about R5 Topics and R5 Subscriptions.

### Representing R5 Topics and R5 Subscriptions

* every FHIR R5 `Topic` is represented in R4 as a "surrogate `Basic` Resource" with a `Basic.code.coding` of `{"system": "http://hl7.org/fhir/resource-types", "code": "R5Topic"}` (TODO: decide the right value for this Coding)

* every FHIR R5 `Subscription` is represented in R4 as a "surrogate `Basic` Resource" with a `Basic.code.coding` of `{"system": "http://hl7.org/fhir/resource-types", "code": "R5Subscription"}` (TODO: decide the right value for this Coding)

* every surrogate `Basic` resource, in addition to a `Basic.code.coding`, contains a "json-embedded-resource" extension with `"url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource"` and a `valueString` containing a JSON-stringified FHIR R5 resource (Topic or Subscription)

* every `Reference.value` pointing to an R5 Subscription or R5 Topic is represented in R4 as a reference to `Basic/:id`

* for every R5 Subscription or R5 Topic, the `Subscription.id` or `Topic.id` of the json-embedded-resource is populated to match the `Basic.id` of its surrogate.

* for every R5 Subscription associated with a single `patient`, the surrogate has its `Basic.subject` populated with a reference to the patient

### REST API

Clients interact with the create/update/delete API for `Basic`, interacting with the surrogate resources to manage R5 Topic and R5 Subscription instances. Search requires some additional considerations:

* Topic "search all" is implemented via `GET Basic?code=R5Topic`. Fine-grained search MAY be supported with custom search parameters that index the `valueString` JSON, using definitions from http://build.fhir.org/topic.html#search

* Subscription "search all" is implemented via `GET Basic?code=R5Subscription` or `GET Basic?code=R5Subscription&patient=[id]`. Fine-grained search MAY be supported with custom search parameters that index the `valueString` JSON, using definitions from http://build.fhir.org/subscription.html#search

### Additional Subscription mechanics

The rest of subscription mechanics work as in R5. Namely:

* Channels types and payloads
* Event matching by Topic + filters
* Event notifications with Bundles


## Examples

A "create subscription" request has a `POST` body like:

```json
{
  "resourceType": "Basic",
  "code": {
    "coding": [
      {
        "system": "http://hl7.org/fhir/resource-types",
        "code": "R5Subscription"
      }
    ]
  },
  "subject": {
    "reference": "Patient/58"
  },
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource",
      "valueString": "{\"resourceType\":\"Subscription\",\"channel\":{\"endpoint\":\"https://client.subscriptions.argo.run/Endpoints/b962286c-d1c6-4b13-a409-8df2a7208fb8\",\"header\":[],\"heartbeatPeriod\":60,\"payload\":{\"content\":\"id-only\",\"contentType\":\"application/fhir+json\"},\"type\":{\"coding\":[{\"code\":\"rest-hook\",\"display\":\"Rest Hook\",\"system\":\"http://terminology.hl7.org/CodeSystem/subscription-channel-type\",\"userSelected\":false}],\"text\":\"REST Hook\"}},\"end\":\"2019-09-20T15:21:28.427Z\",\"eventCount\":0,\"filterBy\":[{\"matchType\":\"=\",\"name\":\"patient\",\"value\":\"Patient/58\"}],\"reason\":\"Client Testing\",\"status\":\"requested\",\"topic\":{\"reference\":\"https://server-r4.subscriptions.argo.run/Basic/1\"}}"
    }
  ]
}
```

Note this subscription points to its Topic as `https://server-r4.subscriptions.argo.run/Basic/1`, indicating that the contents of `Basic/1` on the server should look like:

```json
{
  "resourceType": "Basic",
  "id": 1,
  "code": {
    "coding": [
      {
        "system": "http://hl7.org/fhir/resource-types",
        "code": "R5Topic"
      }
    ]
  },
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource",
      "valueString": "{\"resourceType\":\"Topic\",\"canFilterBy\":[{\"documentation\":\"Exact match to a patient resource (reference)\",\"matchType\":[\"=\",\"in\",\"not-in\"],\"name\":\"patient\"}],\"date\":\"2019-08-01\",\"description\":\"Admission Topic for testing framework and behavior\",\"experimental\":true,\"resourceTrigger\":{\"description\":\"Beginning of a clinical encounter\",\"fhirPathCriteria\":\"%previous.status!='in-progress' and %current.status='in-progress'\",\"queryCriteria\":{\"current\":\"status:in-progress\",\"previous\":\"status:not=in-progress\",\"requireBoth\":true},\"resourceType\":[\"Encounter\"]},\"status\":\"draft\",\"title\":\"admission\",\"url\":\"http://argonautproject.org/subscription-ig/Topic/admission\",\"version\":\"0.4\",\"id\":\"1\"}"
    }
  ]
}
```

Note that the `Basic.id` and the json-embedded-resource `Topic.id` here match (both are `1`), as required.
