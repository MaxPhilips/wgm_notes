# How to use Cerner's Test Server for Carequality

Base server URL:  
https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/

Server CapabilityStatement URL:  
https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/metadata

Note: Cerner's server uses **itself** instead of a separate OAuth server for DCR and token generation. Yes, this is a bit goofy. We've exposed a custom request header that can be set on requests to /metadata that causes Carequality OAuth URIs (from Cerner's FHIR server instead of Cerner's Auth server) to override our regular OAuth URIs.

Set the custom request header `Cerner-Override-OAuth-URIs` with a value of `on` to achieve the above. When set, the /metadata response will include:

```json
{
  "url": "http://fhir-registry.smarthealthit.org/StructureDefinition/oauth-uris",
  "extension": [
    {
      "url": "token",
      "valueUri": "https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/Authorization/token"
    },
    {
      "url": "register",
      "valueUri": "https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/Authorization/register"
    }
  ]
}
```

### cURL command

```bash
curl --location --request GET 'https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/metadata' \
--header 'Accept: application/fhir+json' \
--header 'Cerner-Override-OAuth-URIs: on'
```

## Get certificate

Out of band, work with Sequoia Project tech support to receive a reference code and an authorization code that can be exchanged for an X.509 certificate and corresponding private key.

Use the certificate and private key to sign the software statement and client assertion JWTs throughout the rest of the workflow.

## Use software statement JWT to register a client id with Cerner dynamically

Issue a POST request to the server's /register URL, passing:
* software statement JWT

### cURL command

```bash
curl --location --request POST 'https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/Authorization/register' \
--header 'Content-Type: application/fhir+json' \
--data-raw '{
  "software_statement": "<generated JWT>",
  "certifications": [],
  "udap": 1
}'
```

## Use client assertion JWT to get an access token

Issue a POST request to the server's token URL, passing:
* scope: system/Patient.read
* grant_type: client_credentials
* client_assertion_type: urn:ietf:params:oauth:client-assertion-type:jwt-bearer
* client_assertion: your client assertion JWT

### cURL command

```bash
curl -X POST \
  https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/Authorization/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'scope=system%2FPatient.read&grant_type=client_credentials&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=<client assertion JWT>'
```

## Use access token to interact with FHIR resource

Issue a GET request to the server's FHIR Patient resource URL. Cerner's FHIR server requires that consumers request an Accept type of JSON.

### cURL command

```bash
curl -X GET \
  'https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/Patient?_id=1316024' \
  -H 'Accept: application/fhir+json' \
  -H 'Authorization: Bearer <access token received from previous step>' \
```
