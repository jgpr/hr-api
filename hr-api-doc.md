# Raden HR API documentation

Human resource manager api

# Table of contents

* [Authorizing users](#authorizing-users)
   * [Available methods](#available-methods)
      * [POST v1/oauth/token](#post-v1oauthtoken)
      * [GET v1/oauth/tokens](#get-v1oauthtokens)
      * [DELETE v1/oauth/tokens/{token_id}](#delete-v1oauthtokenstoken_id)
      * [GET v1/oauth/personal-access-tokens](#get-v1oauthpersonal-access-tokens)
      * [DELETE v1/oauth/personal-access-tokens/{token_id}](#delete-v1oauthpersonal-access-tokenstoken_id)
      * [POST v1/oauth/token/refresh](#post-v1oauthtokenrefresh)
      * [POST v1/logout](#post-v1logout)
      * [POST v1/password/email](#post-v1passwordemail)
      * [POST v1/password/reset](#post-v1passwordreset)


### Authorizing users
>Raden api uses OAuth2.0 authentication protocol for authenticating clients
>For intracting with API, you need a CLIENT_ID and a CLIENT_SECRET

##### Available methods
|Method|Route|Desc|
| ---- | ---- | ---- |
|v1/oauth/token|POST|Authorize a client to access the user's account|
|v1/oauth/tokens|GET\HEAD|Get all of the authorized tokens for the authenticated user|
|v1/oauth/tokens/{token_id}|DELETE|Delete the given token|
|v1/oauth/personal-access-tokens|GET\HEAD|Get all of the personal access tokens for the authenticated user|
|v1/oauth/personal-access-tokens|POST|Create a new personal access token for the user|
|v1/oauth/personal-access-tokens/{token_id}|DELETE|Delete the given token|
|v1/oauth/token/refresh|POST|Get a fresh transient token cookie for the authenticated user|
|v1/logout|POST|Logout current user|
|v1/password/email|POST|Send a reset link email to given user|
|v1/password/reset|POST|Reset user password|

###### POST v1/oauth/token
With this route you can create and refresh tokens

><span style="color:maroon">Note</span>: each token expires after **10 minutes** and you need to refresh tokens

- Issue new token:
    - Method: `POST`
    - Request: `v1/oauth/token`
    ```json
    {
        "grant_type": "password",
        "client_id": "CLIENT_ID",
        "client_secret": "CLIENT_SECRET",
        "username": "User's email or personnel_id",
        "password": "User's password"
    }
    ```
    - Response: `200 OK`
    ```json
    {
        "type": "bearer",
        "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiI2FiYmQwZWEwM2E0NGQiLCJpYX",
        "refresh_token": "def5020094171aa916cdc71bf4be1f96051e549180b1efa4de7be7",
        "expires_in": 6000
    }
    ```
- Refresh tokens:
    - Method: `POST`
    - Request: `v1/oauth/token`
        ```json
        {
            "grant_type": "refresh_token",
            "refresh_token": "def5020094171aa916cdc646816afa66e1e7171bf4be1f99180b1efa4de7be7",
            "client_id": "CLIENT_ID",
            "client_secret": "CLIENT_SECRET"
        }
        ```
    - Response: `200 OK`
        ```json
        {
            "type": "bearer",
            "access_token": "zIyNDk5OGUxMjg2NmE4NmYwMzJiNDI1ZGY4ZTk0ODIxM2FiYmQpYX",
            "refresh_token": "def5020094171aa916cdc646a3afef51e549180b1efa4de7be7",
            "expires_in": 6000
        }
        ```

###### GET v1/oauth/tokens
Get all active tokens for authorized user
><span style="color:maroon">Note</span>: if you are using password grant this route is not useful

- Get tokens
    - Method: `GET`
    - Request: `v1/oauth/tokens`
    - Response: `200 OK`
    ```json
    [
        {
            "id": "ed9caa67297c3deeb1b4de2b59e25d06cdda4872e1b9228a375320d7c60b73029",
            "user_id": 1,
            "client_id": 2,
            "name": null,
            "scopes": [],
            "revoked": false,
            "created_at": "2017-11-30 12:32:10",
            "updated_at": "2017-11-30 12:32:10",
            "expires_at": "2017-11-30 14:12:10"
        },...
    ]
    ```

###### DELETE v1/oauth/tokens/{token_id}
Delete wanted token

- Delete token
    - Method: `Delete`
    - Request: `v1/oauth/tokens/{token_id}`
    - Response: `200 OK`

###### GET v1/oauth/personal-access-tokens
Get a long live token in simple way. this is useful for simple developing
><span style="color:maroon">Note</span>: this route needs that user already authorized

- Get access token
    - Method: `POST`
    - Request: `v1/oauth/personal-access-tokens`
     ```json
     {
        "name": "your name"
     }
     ```
    - Response: `200 OK`
    ```json
    {
        "accessToken": "eyJ0eXAiOiJKV1QraOhDxb3ZBucZUjJZxHHJRzPLu7c4",
        "token": {
            "id": "4281221e064e5ce8f5b58d413afba8eccfebfb1c17c71b4099ccbe76",
            "user_id": 1,
            "client_id": 1,
            "name": "your name",
            "scopes": [],
            "revoked": false,
            "created_at": "2017-12-11 16:29:13",
            "updated_at": "2017-12-11 16:29:13",
            "expires_at": "2018-12-11 16:29:13"
        }
    }
    ```

###### DELETE v1/oauth/personal-access-tokens/{token_id}
Delete target personal access token

- Delete pernal access token
    - Method: `POST`
    - Request: `v1/oauth/personal-access-tokens/{token_id}`
    - Response: `200 OK`

###### POST v1/oauth/token/refresh
Get a fresh transient token cookie for the authenticated user

- Get fresh token cookie
    - Method: `POST`
    - Request: `v1/oauth/token/refresh`
    ```json
     {
        "grant_type": "refresh_token",
        "refresh_token": "def5020094171aa916cdc646816afa66e1e7171bf4be1f960be7",
        "client_id": "CLIENT_ID",
        "client_secret": "CLIENT_SECRET",
        "scope": "*"
    }
    ```
    - Response: `200 OK, Refreshed`

###### POST v1/logout
Revoke current token

- Logout
    - Method: `POST`
    - Request: `v1/logout`
    - Response: `200 OK`

###### POST v1/password/email
Send a reset link email to given user

- Send reset password mail
    - Method: `POST`
    - Request: `v1/password/email`
    ```json
    {
        "email": "youremail@mailserver.com"
    }
    ```
    - Response: `200 OK`
    ```json
    {
        "msg": "Reset mail sent"
    }
    ```

###### POST v1/password/reset
Reset password with received token

- Reset
    - Method: `POST`
    - Request: `v1/password/reset`
     ```json
     {
        "email": "youremail@mailserver.com",
        "password": "blahblah",
        "password_confirmation": "blahblah",
        "token": "284506b17183c12b700e543df86db6f7db147ae3f5d02ad689f1c5c4b495e293"
    }
     ```
    - Response: `200 OK`
    ```json
    {
        "msg": "Password reset"
    }
    ```
