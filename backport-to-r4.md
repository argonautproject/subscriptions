# Back-porting R5 Subscriptions to R4

The introduction of Topic-based Subscriptions targets FHIR R5, and marks a significant set of improvements to R4 Subscriptions.  Still, given that FHIR R5 will not be published until roughly early 2021 (and implementation will lag further), it's useful to define a standard way to "back-port" R5 Subscriptions functionality to FHIR R4.  This way, implementers invested in R4 can add Topic-based Subscriptions as an "à la carte" feature, gaining value early and laying a clear transition path.

## The (ahem!) Basic approach

We want to support Topic-based Subscriptions in R4 in a manner that's ["simple"](https://www.infoq.com/presentations/Simple-Made-Easy/), even if it's clunky. In other words, we want to avoid complex "overlays" of new subscription behavior on top of existing R4-specified behavior.  For this reason, we avoid trying to re-use (or subtly change the meaning of) FHIR R4 Subscriptions in our back-port. Instead, we rely on FHIR's `Basic` Resource type to convey information about R5 Topics and R5 Subscriptions.

### Representing R5 Topics and R5 Subscriptions

* every FHIR R5 `SubscriptionTopic` is represented in R4 as a "surrogate `Basic` Resource" with a `Basic.code.coding` of `{"system": "http://hl7.org/fhir/resource-types", "code": "R5SubscriptionTopic"}` (TODO: decide the right value for this Coding)

* every FHIR R5 `Subscription` is represented in R4 as a "surrogate `Basic` Resource" with a `Basic.code.coding` of `{"system": "http://hl7.org/fhir/resource-types", "code": "R5Subscription"}` (TODO: decide the right value for this Coding)

* every surrogate `Basic` resource, in addition to a `Basic.code.coding`, contains a "json-embedded-resource" extension with `"url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource"` and a `valueString` containing a JSON-stringified FHIR R5 resource (SubscriptionTopic or Subscription)

* every `Reference.value` pointing to an R5 Subscription or R5 SubscriptionTopic is represented in R4 as a reference to `Basic/:id`

* for every R5 Subscription or R5 SubscriptionTopic, the `Subscription.id` or `SubscriptionTopic.id` of the json-embedded-resource is populated to match the `Basic.id` of its surrogate.

* for every R5 Subscription associated with a single `patient`, the surrogate has its `Basic.subject` populated with a reference to the patient

* fields defined as `integer64` in R5 should be represented as a `string` in Basic (TODO: list fields)

### REST API

Clients interact with the create/update/delete API for `Basic`, interacting with the surrogate resources to manage R5 SubscriptionTopic and R5 Subscription instances. Search requires some additional considerations:

* SubscriptionTopic "search all" is implemented via `GET Basic?code=R5SubscriptionTopic`. Fine-grained search MAY be supported with custom search parameters that index the `valueString` JSON, using definitions from http://build.fhir.org/subscriptiontopic.html#search

* Subscription "search all" is implemented via `GET Basic?code=R5Subscription` or `GET Basic?code=R5Subscription&patient=[id]`. Fine-grained search MAY be supported with custom search parameters that index the `valueString` JSON, using definitions from http://build.fhir.org/subscription.html#search

### Additional Subscription mechanics

The rest of subscription mechanics work as in R5. Namely:

* Channels types and payloads
* Event matching by SubscriptionTopic + filters
* Event notifications with Bundles


## Examples

A "create subscription" request has a `POST` body like:

```json
{
  "resourceType": "Basic",
  "id": "2be70090-1b1d-4651-88a6-f17a0fb8ed9d",
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource",
      "valueString": "{\"resourceType\":\"Subscription\",\"id\":\"2be70090-1b1d-4651-88a6-f17a0fb8ed9d\",\"status\":\"active\",\"topic\":{\"reference\":\"https://server.subscriptions.argo.run/Basic/encounter-start\"},\"end\":\"2020-05-11T21:21:36.047+00:00\",\"reason\":\"Client Testing\",\"filterBy\":[{\"searchParamName\":\"patient\",\"searchModifier\":\"=\",\"value\":\"Patient/48157\"}],\"channelType\":{\"system\":\"http://terminology.hl7.org/CodeSystem/subscription-channel-type\",\"code\":\"rest-hook\",\"display\":\"Rest Hook\"},\"endpoint\":\"https://client.subscriptions.argo.run/Endpoints/bc768dd9-fbaa-445a-88b1-5c7b63b70e9c\",\"heartbeatPeriod\":60,\"contentType\":\"application/fhir+json\",\"content\":\"id-only\"}"
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://hl7.org/fhir/resource-types",
        "code": "R5Subscription"
      }
    ],
    "text": "Backported R5 Subscription"
  }
}
```

Note this subscription points to its SubscriptionTopic as `https://server.subscriptions.argo.run/Basic/encounter-start`, indicating that the contents of `Basic/encounter-start` on the server should look like:

```json
{
    "resourceType": "Basic",
    "id": "encounter-start",
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/json-embedded-resource",
            "valueString": "{\"resourceType\":\"SubscriptionTopic\",\"id\":\"encounter-start\",\"url\":\"http://argonautproject.org/encounters-ig/SubscriptionTopic/encounter-start\",\"version\":\"1.1\",\"title\":\"encounter-start\",\"status\":\"draft\",\"experimental\":true,\"date\":\"2020-05-11\",\"description\":\"Beginning of a clinical encounter\",\"resourceTrigger\":{\"description\":\"Beginning of a clinical encounter\",\"resourceType\":[\"Encounter\"],\"queryCriteria\":{\"previous\":\"status:not=in-progress\",\"current\":\"status:in-progress\",\"requireBoth\":true},\"fhirPathCriteria\":[\"%previous.status!='in-progress' and %current.status='in-progress'\"]},\"canFilterBy\":[{\"searchParamName\":\"patient\",\"searchModifier\":[\"=\",\"in\"],\"documentation\":\"Exact match to a patient resource (reference)\"}]}"
        }
    ],
    "code": {
        "coding": [
            {
                "system": "http://hl7.org/fhir/resource-types",
                "code": "R5SubscriptionTopic"
            }
        ],
        "text": "Backported R5 SubscriptionTopic"
    }
}
```

Note that the `Basic.id` and the json-embedded-resource `SubscriptionTopic.id` here match (both are `1`), as required.

## Managing Permissions with SMART scopes

Permissions to create and read R5 Subscriptions and R5 SubscriptionTopics relies on SMART scopes, including some extension scopes (i.e., scopes that begin with `__`:

* To create R5 Subscription data, a client should request scopes of `patient/Basic.write` (or `user/Basic.write`) and `__patient/R5Subscription.write` (or `__user/R5Subscription.write`)
* To read R5 Subscription data, a client should request scopes of `patient/Basic.read` (or `user/Basic.read`) and `__patient/R5Subscription.read` (or `__user/R5Subscription.read`)
* To read R5 SubscriptionTopic data, a client should request scopes of `patient/Basic.read` (or `user/Basic.read`) and `__patient/R5Subscription.read` (or `__user/R5Subscription.read`)
* (If applicable) to create R5 SubscriptionTopic data, a client should request scopes of `patient/Basic.read` (or `user/Basic.read`) and `__patient/R5SubscriptionTopic.write` (or `__user/R5SubscriptionTopic.write`)
