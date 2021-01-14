# How to use Cerner's Test Server for SMARTv2

## URLs

### FHIR server base URL

https://fhir-myrecord.stagingcerner.com/beta/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### CapabilityStatement API

BASE_URL/metadata

#### Observation API

BASE_URL/Observation

### Authorization server base URL

https://authorization.cerner.com/tenants/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### Authorize endpoint

AUTH_BASE_URL/protocols/oauth2/profiles/smart-v1/personas/patient/authorize

#### Token endpoint

AUTH_BASE_URL/protocols/oauth2/profiles/smart-v1/token

#### Introspection endpoint

https://authorization.cerner.com/tokeninfo

## Prerequisites

### Register your client with Cerner

It's a hackathon, so contact Max Philips and he will hack your registration into Cerner's system. If you prefer a particular client id, let Max know.

## Scenarios

General notes:

* Cerner's test server requires both old-style and new-style scopes to be requested when generating tokens for this track. That is, you must request a scope of patient/Observation.read as well as any applicable SMARTv2 scopes.
* Cerner's test server does not support writing Observations for this connectathon, but all scopes mentioned on the wiki can still be requested.
* Please set an Accept: application/fhir+json header on all requests to Cerner's FHIR server.

### Scenario 0: Discovery of SMARTv2 Support

SMARTv2 support is advertised in the well-known SMART configuration.

    GET BASE_URL/.well-known/smart-configuration

### Scenario 1: Share access to Observations with SMARTv2 resource-level scopes

Create and use tokens with scopes `patient/Observation.rs` and `patient/Observation.crs` (as mentioned above, this scope can be requested, but not used).

Generate an authorization token with your preferred library or client. Use test patient Nancy Smart 12724066. Her credentials are nancysmart / Cerner01.

Expect 200! Yay!

    GET BASE_URL/Observation?patient={}

Expect 400. No required parameters are set.

    GET BASE_URL/Observation

Expect 403. Insufficient scope.

    GET BASE_URL/Encounter?patient={}

### Scenario 2: Like Scenario 1 but with Vital Signs Observations (SMARTv2 category-level scopes)

Create and use tokens with scopes `patient/Observation.rs?category=http://terminology.hl7.org/CodeSystem/observation-category|vital-signs` and `patient/Observation.crs?category=http://terminology.hl7.org/CodeSystem/observation-category|vital-signs` (as mentioned above, this scope can be requested, but not used).

Generate an authorization token with your preferred library or client. Use test patient Nancy Smart 12724066. Her credentials are nancysmart / Cerner01.

Expect 200! Yay!

    GET BASE_URL/Observation?patient={}&category=vital-signs

Expect 200! Yay!

    GET BASE_URL/Observation?patient={}&category=http://terminology.hl7.org/CodeSystem/observation-category|vital-signs

Expect 422. Need to set a category.

    GET BASE_URL/Observation?patient={}

Expect 422. Need to set the right category.

    GET BASE_URL/Observation?patient={}&category=laboratory

### Scenario 3: Like Scenario 1 but with POST-based "authorize" API call

Unsupported.

### Scenario 4: Like Scenario 1 but with PKCE enabled

Unsupported.

### Scenario 5: Token Introspection

Well-known SMART configuration returns an introspection_endpoint.

    POST https://authorization.cerner.com/tokeninfo
    Content-Type: application/x-www-form-urlencoded

    token=TOKEN
