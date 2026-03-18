# Public App authorization

#### Public App installation flow
Public App installation starts with a user redirection to an app-specific installation URL. Installation URL has to be
generated in the following format:

`https://www.printful.com/oauth/authorize?client_id={clientId}&state={stateValue}&redirect_url={redirectUrl}`

| Parameter    | Description                                                                                                    |
|--------------|----------------------------------------------------------------------------------------------------------------|
| client_id    | the `Client id` of your app                                                                                    |
| state        | use the optional state parameter to help keep track of the customer throughout the installation process        |
| redirect_url | URL to where the customer should be taken after the authorization step. This URL must be registered in the app |

In this URL, the customer is asked to register or log in to an existing account. After authorization, the customer is redirected to the OAuth grant screen. This screen shows information about your app, and the customer is asked to accept the requested scopes. If the customer rejects the app installation, he is redirected back to your provided `redirect_url` with `success=0` parameter. If the customer accepts app installation, he is redirected to your `redirect_url` with `state={stateValue}&code={authorizationCode}&success=1` parameters.

| Parameter | Description                                                               |
|-----------|---------------------------------------------------------------------------|
| state     | state parameter that was provided when installation flow was initialized  |
| code      | authorization code that will be used to request access and refresh tokens |
| success   | flag indicating that authorization flow was successful                    |

After receiving `authorization code`, apps should request `access` and `refresh` tokens. To request these tokens, the app should make a `POST` request to `https://www.printful.com/oauth/token` page with the following parameters:

- grant_type=authorization_code
- client_id={clientId}
- client_secret={clientSecret}
- code={authorizationCode}

If the request is successful, the following response is returned:
```json
{
  "access_token": "smk_GN0I1Os3OdfqzjJnTOWn1wlbqqq2Y2Pc10TS",
  "expires_at": "1562157895",
  "token_type": "bearer",
  "refresh_token": "902LmW0sWNlY-mbrLhb6AgexkI6K4OFo3Hknak8HDhTy_ifB5SdoLH1QgV9"
}
```
| Field         | Description                                                                                                        |
|---------------|--------------------------------------------------------------------------------------------------------------------|
| access_token  | this token should be used to make authorized API calls                                                             |
| expires_at    | timestamp for the date when the `access_token` will expire                                                         |
| token_type    | type of the token. Currently only `bearer`. Use it together with the `access_token` in authorized requests headers |
| refresh_token | a token that can be used to refresh the `access_token` once it expires                                             |

### Refreshing tokens
The `access_token` will expire in **one hour**. When `access_token` expires you must refresh it with the `refresh_token`. After refreshing tokens, you will receive a new set of `access_token` and `refresh_token`. If tokens are not refreshed within **90** days, `refresh_token` will expire, and the app will have to ask the user to reauthorize it. To refresh tokens you must make a `POST` request to `https://www.printful.com/oauth/token` with the parameters:

- grant_type=refresh_token
- client_id=`{clientId}`
- client_secret=`{clientSecret}`
- refresh_token=`{refreshToken}`

The successful refresh request will result in a similar response that you received previously:
```json
{
  "access_token": "smk_GN0I1Os3OdfqzjJnTOWn1wlbqqq2Y2Pc10TS",
  "expires_at": "1562157895",
  "token_type": "bearer",
  "refresh_token": "902LmW0sWNlY-mbrLhb6AgexkI6K4OFo3Hknak8HDhTy_ifB5SdoLH1QgV9"
}
```

