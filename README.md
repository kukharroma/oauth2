# oauth2

To get resources provided by Rest API you should be authorized (except sign up service).<br>
Niffler uses OAuth2 authorization with `resource owner password credentials` grant type.<br>
[See tutorial here](http://websystique.com/spring-security/secure-spring-rest-api-using-oauth2)

In authorization request you have to provide the next parameters in the body:

| Parameter    | Description |
| -------------|------------ | 
| `grant_type`   | Constant value, always `password`|
| `username`     | User login (email)|
| `password`     | User password|

Endpoint URL (for dev env):<br>
POST `http://dev1.niffler.sandbx.co:8010/webapi/oauth/token`

HTTP Request header parameters:

| Parameters   | Value                             | Description |
| -------------|-----------------------------------|-------------|
| `Authorization`| Basic d2ViY2xpZW50Og==          | Base64 encoded string `webclient:`| 
| `Content-Type` | application/x-www-form-urlencoded |            |

`curl webclient@dev1.niffler.sandbx.co:8010/webapi/oauth/token -d grant_type=password -d username=user@example.com -d password=super!secret123`

As response you will get **access** and **refresh** tokens in JSON<br> 
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTY3NjYz......",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiI......",
  "expires_in": 1799,
  "scope": "read write",
  "jti": "9cd2622f-4234-4fdf-b963-d43b726d687e"
}
```

To get resources you have to add `access_token` to `Authorization` header with `Bearer ` prefix

`curl http://dev1.niffler.sandbx.co:8010/webapi/reference/kingdom -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTY3NjYz......"
`


### Refresh Token
Access token are valid during short period of time (30 minutes).
After that you will get **HTTP-401** error:

```json
{
  "error": "invalid_token",
  "error_description": "Access token expired: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTY3NjYz......"
}
``` 

You have 2 options
*  Ask user for login/password and reauthorize session
*  Get new `access_token` using `refresh_token` from the first request (see below):

HTTP Request header are equals to the initial request (` Authorization` and `Content-Type` parameters should be included)  but payload is a little bit different: 

| Parameter    | Description |
| -------------|------------ | 
| `grant_type`   | Constant value, always `refresh_token`|
| `refresh_token`| Refresh token (no additional `Bearer ` is required)|

`curl webclient@dev1.niffler.sandbx.co:8010/webapi/oauth/token -d grant_type=refresh_token -d refresh_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiI......`

In response you can find new access&refresh tokens pare:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTY3Nj....",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiO....",
  "expires_in": 1799,
  "scope": "read write",
  "jti": "e5326a7f-1e49-4257-aaeb-291e00de4d8c"
}
```
