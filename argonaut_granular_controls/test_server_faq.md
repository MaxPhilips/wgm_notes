# How to use Cerner's Test Server for Argonaut Granular Controls

## URLs

### FHIR server base URL

https://fhir-ehr.stagingcerner.com/beta/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### CapabilityStatement API

base/metadata

#### Observation API

base/Observation

### Authorization server base URL

https://authorization.sandboxcerner.com/tenants/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### Authorize endpoint

base/protocols/oauth2/profiles/smart-v1/personas/provider/authorize

### Token endpoint

base/protocols/oauth2/profiles/smart-v1/token

## Prerequisites

### Register your client with Cerner

It's a hackathon, so contact Max Philips and he will hack your registration into Cerner's system. If you prefer a particular client id, let Max know.

## Scenarios

General notes:

* Cerner's test server requires both old-style and new-style scopes to be requested when generating tokens for this track
* Cerner's test server does not support writing Observations for this connectathon, but all scopes mentioned on the wiki can still be requested

### Scenario 0: Share access to resources by interaction

Create and use tokens with scopes `patient/Observation.rs` and `patient/Observation.crs` (as mentioned above, this scope can be requested, but not used).

Generate an authorization token with your preferred library or client. Use test patient Nancy SMART 12724066. Her credentials are nancysmart / Cerner01. Other test patients are documented here: https://docs.google.com/document/d/10RnVyF1etl_17pyCyK96tyhUWRbrTyEcqpwzW-Z-Ybs/edit

Expect 200! Yay!

    GET Observation?patient={}

Expect 400. No required parameters are set.

    GET Observation

Expect 403. Insufficient scope.

    GET Encounter?patient={}

### Scenario 1: Share access to data by category

Create and use tokens with scopes `patient/Observation.rs?category=vital-signs` and `patient/Observation.crs?category=vital-signs` (as mentioned above, this scope can be requested, but not used).

Generate an authorization token with your preferred library or client. Use test patient Nancy SMART 12724066. Her credentials are nancysmart / Cerner01. Other test patients are documented here: https://docs.google.com/document/d/10RnVyF1etl_17pyCyK96tyhUWRbrTyEcqpwzW-Z-Ybs/edit

Expect 200! Yay!

    GET Observation?patient={}&category=vital-signs

Expect 200! Yay!

    GET Observation?patient={}&category=http://terminology.hl7.org/CodeSystem/observation-category|vital-signs

Expect 422. Need to set a category.

    GET Observation?patient={}

Expect 422. Need to set the right category.

    GET Observation?patient={}&category=laboratory
