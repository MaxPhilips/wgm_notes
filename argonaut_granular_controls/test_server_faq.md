# How to use Cerner's Test Server for Argonaut Granular Controls

## URLs

### FHIR server base URL

https://fhir-ehr.stagingcerner.com/beta/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### CapabilityStatement API

https://fhir-ehr.stagingcerner.com/beta/ec2458f2-1e24-41c8-b71b-0e701af7583d/metadata

#### Observation API

https://fhir-ehr.stagingcerner.com/beta/ec2458f2-1e24-41c8-b71b-0e701af7583d/Observation

### Authorization server base URL

https://authorization.sandboxcerner.com/tenants/ec2458f2-1e24-41c8-b71b-0e701af7583d/

#### Authorize endpoint

https://authorization.sandboxcerner.com/tenants/ec2458f2-1e24-41c8-b71b-0e701af7583d/protocols/oauth2/profiles/smart-v1/personas/provider/authorize

#### Token endpoint

https://authorization.sandboxcerner.com/tenants/ec2458f2-1e24-41c8-b71b-0e701af7583d/protocols/oauth2/profiles/smart-v1/token

## Prerequisites

### Register your client with Cerner

Use [Cerner's Developer Portal](https://code.cerner.com/developer/smart-on-fhir/) to register your client.

Create an account if you do not have one already. After logging in:

* Click "+ New App" in the top right
* Fill out App Name with anything
* Fill out Redirect URI with anything or an applicable redirect URI
* Choose App Type Provider
* Choose FHIR Spec r4
* Choose Authorized yes
* Select Patient Scopes Observation read
* Click "Register"

Note the Client-id provided to you in the registration confirmation message.

After completing the above, contact Max Philips in order to get custom scopes added to your registration. Tell Max which client id needs to be updated.

## Scenarios

General notes:

* Cerner's test server requires both old-style and new-style scopes to be requested when generating tokens for this track
* Cerner's test server does not support writing Observations for this connectathon, but all scopes mentioned on the wiki can still be requested
