# How to use Cerner's Test Server for Carequality

Base server URL:  
https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/

Server CapabilityStatement URL:  
https://fhir-ehr.stagingcerner.com/beta/0b8a0111-e8e6-4c26-a91c-5069cbc6b1ca/metadata

Note: Cerner's server uses **itself** instead of a separate OAuth server for DCR and token generation. Yes, this is a bit goofy. We've made up a one-off security extension to display /register and /token URLs. This information is available in CapabilityStatement.rest[0].security.extension

```json
{
  "url": "https://fhir-ehr.cerner.com/r4/StructureDefinition/carequality-oauth-uris",
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

Make sure to request JSON format for FHIR resource requests using either:
* ?_format=json
* Accept: application/fhir+json

## Create and sign client assertion JWT

Out of band, work with Sequoia Project tech support to receive a reference code and an authorization code that can be exchanged for an X.509 certificate and corresponding private key.

Use the certificate and private key to create and sign the client assertion JWT. I wrote a ruby script to sign client assertion JWTs: [generate_client_assertion_jwt.rb](generate_client_assertion_jwt.rb)

## Register your client id with Cerner dynamically

Your client id should be the FQDN that your Carequality certificate and private key were issued for. Contact Max Philips and we will register your client id with Cerner manually. We'll need the client id itself and a description of the client.

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

## Use client assertion JWT to interact with /token URL

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
