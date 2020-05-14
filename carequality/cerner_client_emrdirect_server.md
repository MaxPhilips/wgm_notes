# Notes on testing scenario 1 with a Cerner client against EMRDirect server

## Read metadata

```bash
curl --location --request GET 'https://stage.healthtogo.me:8181/fhir/r4/stage/metadata'

      ...
      "security": {
        "extension": [
          {
            "url": "http://fhir-registry.smarthealthit.org/StructureDefinition/oauth-uris",
            "extension": [
              {
                "url": "token",
                "valueUri": "https://stage.healthtogo.me:8181/oauth/stage/token"
              },
              {
                "url": "authorize",
                "valueUri": "https://stage.healthtogo.me:8181/oauth/stage/authz"
              },
              {
                "url": "register",
                "valueUri": "https://stage.healthtogo.me:8181/oauth/stage/register"
              }
            ]
          }
        ],
        ...
```

## Send software statement to /register URI

Generate software statement with the following payload, sign it with Cerner's Carequality private key

```ruby
payload = {
  'iss': 'https://connect.carequality.org/fhir-stu3/1.0.1/Organization/2.16.840.1.113883.3.13.1', # Cerner's entry in Carequality directory
  'sub': 'https://connect.carequality.org/fhir-stu3/1.0.1/Organization/2.16.840.1.113883.3.13.1',
  'aud': 'https://stage.healthtogo.me:8181/oauth/stage/register',
  'exp': Time.now.to_i + 300, # now + 5 min
  'iat': Time.now.to_i,
  'jti': 'random-non-reusable-jwt-id-123',
  'client_name': 'Cerner Client Connectathon May 2020',
  'grant_types': ['client_credentials'],
  'token_endpoint_auth_method': 'private_key_jwt',
  'scope': 'system/Patient.read'
}
```

Send software statement to /register URI

```bash
curl --location --request POST 'https://stage.healthtogo.me:8181/oauth/stage/register' \
--header 'Content-Type: application/json' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "software_statement": "<generated software statement>",
  "certifications": [],
  "udap": 1
}'

201
{
  "client_id": "912c49dc-c6ea-4048-a177-fb33c7f68b52",
  "token_endpoint_auth_method": "private_key_jwt",
  "grant_types": [
      "client_credentials"
  ],
  "client_name": "Cerner Client Connectathon May 2020",
  "software_statement": "<generated software statement>"
}
```

## Use client id to obtain access token

Generate client assertion with the following payload, sign it with Cerner's Carequality private key

```ruby
payload = {
  'iss': '912c49dc-c6ea-4048-a177-fb33c7f68b52',
  'sub': '912c49dc-c6ea-4048-a177-fb33c7f68b52',
  'aud': 'https://stage.healthtogo.me:8181/oauth/stage/token',
  'exp': Time.now.to_i + 300, # now + 5 min
  'iat': Time.now.to_i,
  'jti': 'random-non-reusable-jwt-id-123'
}
```

Send client assertion to /token URI

```bash
curl --location --request POST 'https://stage.healthtogo.me:8181/oauth/stage/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'scope=system/Patient.read' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer' \
--data-urlencode 'client_assertion=<generated client assertion>'

200
{
  "access_token": "<access token>",
  "scope": "system/*.read",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## Read FHIR data

Use access token to retrieve FHIR data

```bash
curl --location --request GET 'https://stage.healthtogo.me:8181/fhir/r4/stage/Patient' \
--header 'Authorization: Bearer <access token>'

200
{
  "resourceType": "Bundle",
  "id": "37066bbb-22ab-438e-a38e-b6df7bbb7735",
  "meta": {
    "lastUpdated": "2020-05-14T23:23:01.625+00:00"
  },
  "type": "searchset",
  "total": 2,
  "link": [
    {
      "relation": "self",
      "url": "https://stage.healthtogo.me:8181/fhir/r4/stage/Patient"
    }
  ],
  "entry": [
    ...
```
