```markdown
# Step 1: Retrieve Authentication Token

## Endpoint

- **Request Method**: POST
- **Endpoint**: `{{base_url}}/securitymanager/api/authentication/login`

## Request Headers

- **Key**: Content-Type
- **Value**: application/json; charset=utf-8

## Request Body

`{"username": "string", "password": "string"}`

## Example Response

```json
{
  "authorized": true,
  "authCode": 0,
  "authStatus": "AUTHORIZED",
  "token": "string",
  "tokenTTL": 1800
}
```

*The value for `token` will be used to authenticate future API calls.*

# Step 2: Use Authentication Token

## Example Endpoint

This example endpoint will retrieve all devices in FireMon

- **Request Method**: GET
- **Endpoint**: `{{base_url}}/securitymanager/api/domain/{{domain_id}}/device`

## Request Headers

- **Key**: Content-Type
- **Value**: application/json; charset=utf-8
- **X-FM-Auth-Token**: `token` value from Step 1
```