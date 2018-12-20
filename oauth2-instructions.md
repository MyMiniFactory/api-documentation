# MyMiniFactory - Authservice

### Description

The Authservice providing OAuth 2.0 solutions for authenticating users using MyMiniFactory credentials.

### Usage

#### Authentication
The client initiates the flow by directing the resource owner's user-agent to the authorization endpoint (i.e. loading the MyMiniFactory's authentication dialog on the user's browser). The client includes its client identifier, requested scope, local state, and a redirection URI to which the authorization server will send the user-agent back once access is granted (or denied).

The base URL is `https://auth.myminifactory.com/web/authorize` where you have to set params.
###### Params:
- `client_id:` Your MyMiniFactory client ID
- `redirect_uri:` Your callback route which handles the authorization code
- `response_type:` It should be `code`
- `state:` Some random string for avoid XSS attack
- `lang:` Language of the site (Optional, by default english or getting data from GEOIP and setting by location) (Supported languages currently: `en`,`hu`,`fr`,`sp`)
- `redirect_type`: If you use our authservice as iframe than add this param with iframe option. (Optional)

###### Example:
- Basic implementation: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=code&state=kjfgierwgn`
- Iframe implementation: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=code&state=kjfgierwgn&redirect_type=iframe`
- Implementation with language: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=code&state=kjfgierwgn&lang=hu`

#### Grant Authorization Code
The authorization code grant type is used to obtain both access tokens and refresh tokens and is optimized for confidential clients. Since this is a redirection-based flow, the client must be capable of interacting with the resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) from the authorization server.

Assuming the resource owner grants access, the authorization server redirects the user-agent back to the client using the redirection URI provided earlier (in the request or during client registration). The redirection URI includes an authorization code and any local state provided by the client earlier.

`http://yoursite.com/callback?code=39835c5e-9576-4b44-b3da-dd9d5a14af94&state=kjfgierwgn`

The client requests an access token from the authorization server's token endpoint by including the authorization code received in the previous step. When making the request, the client authenticates with the authorization server. The client includes the redirection URI used to obtain the authorization code for verification.

```
curl --compressed -v https://auth.myminifactory.com/v1/oauth/tokens \
    -u test_client:test_secret \
    -d "grant_type=authorization_code" \
    -d "code=39835c5e-9576-4b44-b3da-dd9d5a14af94" \
    -d "redirect_uri=http://yoursite.com/callback"
```

The authorization server authenticates the client, validates the authorization code, and ensures that the redirection URI received matches the URI used to redirect the client before. If valid, the authorization server responds back with an access token and, optionally, a refresh token.

```
{
  "user_id": 1,
  "access_token": "00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c",
  "expires_in": 3600,
  "token_type": "Bearer",
  "refresh_token": "6fd8d272-375a-4d8a-8d0f-43367dc8b791"
}
```

If the resource owner denies the access request or if the request fails for reasons other than a missing or invalid redirection URI, the authorization server informs the client by adding the error parameter to the query component of the redirection URI.
`http://yoursite.com/callback?error=access_denied&state=kjfgierwgn`

#### Implicit grant
The implicit grant type is used to obtain access tokens (it does not support the issuance of refresh tokens) and is optimized for public clients known to operate a particular redirection URI. These clients are typically implemented in a browser using a scripting language such as JavaScript.
Since this is a redirection-based flow, the client must be capable of interacting with the resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) from the authorization server.
Unlike the authorization code grant type, in which the client makes separate requests for authorization and for an access token, the client receives the access token as the result of the authorization request.
The implicit grant type does not include client authentication, and relies on the presence of the resource owner and the registration of the redirection URI. Because the access token is encoded into the redirection URI, it may be exposed to the resource owner and other applications residing on the same device.

The client initiates the flow by directing the resource owner's user-agent to the authorization endpoint. The client includes its client identifier, requested scope, local state, and a redirection URI to which the authorization server will send the user-agent back once access is granted (or denied).

The base URL is `https://auth.myminifactory.com/web/authorize` where you have to set params.
###### Params:
- `client_id:` Your MyMiniFactory client ID
- `redirect_uri:` Your callback route which handles the authorization code
- `response_type:` It should be `token`
- `state:` Some random string for avoid XSS attack
- `lang:` Language of the site (Optional, by default english or getting data from GEOIP and setting by location) (Supported languages currently: `en`,`hu`,`fr`,`sp`)
- `redirect_type`: If you use our authservice as iframe than add this param with iframe option. (Optional)

###### Example:
- Basic implementation: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=token&state=kjfgierwgn`
- Iframe implementation: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=token&state=kjfgierwgn&redirect_type=iframe`
- Implementation with language: `https://auth.myminifactory.com/web/authorize?client_id=test_client&redirect_uri=http://yoursite.com/callback&response_type=token&state=kjfgierwgn&lang=hu`

###### Expected response:
When the authservice redirect back to the url it's provide the access token.
`http://yoursite.com/#access_token=4d3999a3-b276-4dc5-92f0-05cb905cd82b&expires_in=21600&state=somestate&token_type=Bearer`

###### Extended implicit grant for mobile users:
(Only for mobile clients)

If you are a mobile user the access token in the implicit grant response have a 10min expires time. The reason is the new secure login for the mobile clients.

The flow:
- You get the access token after the implicit grant
- Revoke this access token by sending device informations about the user. (Example below)
- Get a new access token and refresh it by device informations.

The revoke access token (mobile login) give a new access token wit 2 hour expires time.
```
curl --compressed -v https://auth.myminifactory.com/v1/oauth/mobile/login \
    -d "client_key=YOUR_CLIENT_KEY" \
    -d "access_token=ACCESS_TOKEN" \
    -d 'device_info={"device_id":"ID_OF_THE_DEVICE","manufacturer":"DEVICE","device_model":"MODEL","locale":"LOCALE","user_agent":"USER_AGENT"}'
```
Expected result:
```
{
  "user_id": USER_ID,
  "access_token": NEW_ACCESS_TOKEN,
  "expires_in": 7200,
  "token_type": "Bearer"
}
or
{
  "error": ERROR
}
```

Refresh access token will give a new access token with the same (2 hour) expires time.
```
curl --compressed -v https://auth.myminifactory.com/v1/oauth/mobile/refresh \
    -d "client_key=YOUR_CLIENT_KEY" \
    -d "access_token=OLD_ACCESS_TOKEN" \
    -d "device_id=ID_OF_THE_DEVICE"
```
These data should match with the login device ID and with the latest used access token.

Expected result:
```
{
  "user_id": USER_ID,
  "access_token": NEW_ACCESS_TOKEN,
  "expires_in": 7200,
  "token_type": "Bearer"
}
or
{
  "error": ERROR
}
```


#### Refresh Access Token
If the authorization server issued a refresh token to the client, the client can make a refresh request to the token endpoint in order to refresh the access token.
```
curl --compressed -v https://auth.myminifactory.com/v1/oauth/tokens \
    -u test_client:test_secret \
    -d "grant_type=refresh_token" \
    -d "refresh_token=6fd8d272-375a-4d8a-8d0f-43367dc8b791"
```
The authorization server MUST:

- require client authentication for confidential clients or for any client that was issued client credentials (or with other authentication requirements),
- authenticate the client if client authentication is included and ensure that the refresh token was issued to the authenticated client, and
- validate the refresh token.

If valid and authorized, the authorization server issues an access token.
```
{
  "user_id": 1,
  "access_token": "1f962bd5-7890-435d-b619-584b6aa32e6c",
  "expires_in": 3600,
  "token_type": "Bearer",
  "refresh_token": "3a6b45b8-9d29-4cba-8a1b-0093e8a2b933"
}
```
The authorization server MAY issue a new refresh token, in which case the client MUST discard the old refresh token and replace it with the new refresh token. The authorization server MAY revoke the old refresh token after issuing a new refresh token to the client.

#### Access token introspection
If the authorization server issued a access token or refresh token to the client, the client can make a request to the introspect endpoint in order to learn meta-information about a token.
```
curl --compressed -v https://auth.myminifactory.com/v1/oauth/introspect \
    -u test_client:test_secret \
    -d "token=1f962bd5-7890-435d-b619-584b6aa32e6c" \
    -d "token_type_hint=access_token"
```
The authorization server responds meta-information about a token.
```
{
  "active": true,
  "client_id": "test_client",
  "username": "test@username",
  "token_type": "Bearer",
  "exp": 1454868090
}
```