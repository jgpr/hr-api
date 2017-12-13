# Raden HR API documentation

Human resource manager api

Table of Contents
=================

* [Table of contents](#table-of-contents)
  * [API basic syntax](#api-basic-syntax)
    * [Eager loading](#eager-loading)
    * [Pagination](#pagination)
    * [Sorting](#sorting)
    * [Filtering](#filtering)
    	* [Custom filters](#custom-filters)
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
  * [Device's API](#devices-api)
    * [Get supported devices list](#get-supported-devices-list)
    * [Device resource](#device-resource)
        * [Create new device](#create-new-device)
        * [Get devices list](#get-devices-list)
        * [Get device by id](#get-device-by-id)
    * [Update device by id](#update-device-by-id)
        * [Raden-T41 Devices API](#raden-t41-devices-api)
            * [Test device connection](#test-device-connection)
            * [Get clockings from device](#get-clockings-from-device)
        * [Clockings resource](#clockings-resource)
            * [create new clocking](#create-new-clocking)
            * [Get clockings list](#get-clockings-list)
            * [Get clocking by id](#get-clocking-by-id)
            * [Update a clocking](#update-a-clocking)
            * [Get clocking reasons list](#get-clocking-reasons-list)


## API basic syntax

### Eager loading

**Simple eager load**

`/clockings?includes[]=user.profile`

Will return a collection of `Clocking`s eager loaded with `User profile`.

**IDs mode**

`/clockings?includes[]=user:ids`

Will return a collection of `Clockings`s eager loaded with the ID of their `User`

**Sideload mode**

`/clockings?includes[]=user:sideload`

Will return a collection of `Clocking`s and a eager loaded collection of their
`User`s in the root scope.

### Pagination

Two parameters are available: `limit` and `page`. `limit` will determine the number of
records per page and `page` will determine the current page.

`/clockings?limit=10&page=3`

Will return clockings number 30-40.

### Sorting

Should be defined as an array of sorting rules. They will be applied in the
order of which they are defined.

**Sorting rules**

Property | Value type | Description
-------- | ---------- | -----------
key | string | The property of the model to sort by
direction | ASC or DESC | Which direction to sort the property by

**Example**

```json
[
    {
        "key": "title",
        "direction": "ASC"
    }, {
        "key": "year",
        "direction": "DESC"
    }
]
```

Will result in the clockings being sorted by title in ascending order and then year
in descending order.

### Filtering

Should be defined as an array of filter groups.

**Filter groups**

Property | Value type | Description
-------- | ---------- | -----------
or | boolean | Should the filters in this group be grouped by logical OR or AND operator
filters | array | Array of filters (see syntax below)

**Filters**

Property | Value type | Description
-------- | ---------- | -----------
key | string | The property of the model to filter by (can also be custom filter)
value | mixed | The value to search for
operator | string | The filter operator to use (see different types below)
not | boolean | Negate the filter

**Operators**

Type | Description | Example
---- | ----------- | -------
ct | String contains | `dm` matches `Bulwark dream` and `dream`
sw | Starts with | `bl` matches `blah` but not `mombo blah`
ew | Ends with | `sh` matches `linux bash` but not `bash linux`
eq | Equals | `bash bash` matches `bash bash` but not `bash`
gt | Greater than | `1548` matches `1600` but not `1400`
gte| Greater than or equalTo | `1548` matches `1548` and above (ony for Laravel 5.4 and above)
lte | Lesser than or equalTo | `1600` matches `1600` and below (ony for Laravel 5.4 and above)
lt | Lesser than | `1600` matches `1548` but not `1700`
in | In array | `['me', 'you']` matches `you` and `me` but not `other`
bt | Between | `[1, 10]` matches `5` and `7` but not `11`

**Special values**

Value | Description
----- | -----------
null (string) | The property will be checked for NULL value
(empty string) | The property will be checked for NULL value

#### Custom filters

Remember our relationship `Clockings n ----- 1 User`. Imagine your want to
filter clockings by `user` name.

```json
[
    {
        "filters": [
            {
                "key": "user.name",
                "value": "Bulwark",
                "operator": "sw"
            }
        ]
    }
]
```

## Authorizing users
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

## Device's API
In time of writing this doc, are API support this kind of devices:
- Raden
    - T41
    - V800
    - RF
        - RF900
        - RF620
        - RF800
        - RF500

#### Get supported devices list
method: `GET`
Requset: `api/v1/ta/devices/groups`
Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "DeviceGroups",
    "data": {
        "device_groups": [
            {
                "id": 2,
                "name": "V800-1",
                "connector": "V800",
                "device_group_id": null,
                "created_at": null,
                "updated_at": null
            },
            {
                "id": 3,
                "name": "T41-1",
                "connector": "RadenT41",
                "device_group_id": 1,
                "created_at": null,
                "updated_at": null
            },
            {
                "id": 4,
                "name": "RF-1",
                "connector": "RadenRF900",
                "device_group_id": 1,
                "created_at": null,
                "updated_at": null
            }
        ]
    },
    "links": {
        "POST": "http://api.jgpr.local/v1/ta/devices/groups"
    },
    "meta": {
        "total": 0,
        "total_in_page": 0,
        "total_pages": 0,
        "current_page": 0,
        "limit": 0,
        "from": 0,
        "to": null,
        "next": null,
        "prev": null
    }
}
```

#### Device resource
###### Create new device

|Parameter	            |Type     | Status	 | Description				|
| --------------------- | -----   | -------- | ------------------------ |
|device	                |array	  |	required |							|
|device.name            |string	  |	required |Minimum: 3				|
|device.address			|string	  |	required |							|
|device.port			|string	  |	optional |		 					|
|device.user_name		|string	  |	optional |							|
|device.password		|string	  |	optional |							|
|device.active			|boolean  |	optional |							|
|device.device_group_id	|numeric  |	required | Valid TA.device_group id	|
|device.description		|string	  |	optional |					  		|

- Mehod: `POST`
- Request: `v1/ta/devices`
```json
{
	"device": {
		"name": "Device-1",
		"address": "192.168.20.13",
		"port": "8080",
		"device_group_id": 1,
		"password": "112",
		"user_name": "admin",
		"description": "Test device"
	}
}
```
- Response:
```json
{
    "status": "CREATED",
    "code": 201,
    "message": "Device created",
    "data": {
        "id": 16,
        "name": "Device-1",
        "address": "192.168.20.13",
        "port": 8080,
        "user_name": "admin",
        "refresh_cycle": null,
        "active": 1,
        "device_group_id": 1,
        "description": "Test device",
        "deleted_at": null,
        "created_at": "2017-12-12 15:46:46",
        "updated_at": "2017-12-12 15:46:46"
    },
    "links": {
        "GET": "http://api.jgpr.local/v1/ta/devices/16",
        "DELETE": "http://api.jgpr.local/v1/ta/devices/16",
        "PUT": "http://api.jgpr.local/v1/ta/devices/16"
    }
}
```

###### Get devices list
- Method: `GET`
- Requset: `v1/ta/devices`
- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "Devices",
    "data": {
        "devices": [
            {
                "id": 13,
                "name": "V800",
                "address": "192.168.20.95",
                "port": 5010,
                "user_name": "admin",
                "refresh_cycle": null,
                "active": 1,
                "device_group_id": 2,
                "description": "12",
                "deleted_at": null,
                "created_at": "2017-11-18 09:45:07",
                "updated_at": "2017-11-18 09:45:07"
            },
            {
                "id": 15,
                "name": "Raden",
                "address": "192.168.20.165",
                "port": 8080,
                "user_name": "admin",
                "refresh_cycle": null,
                "active": 1,
                "device_group_id": 3,
                "description": null,
                "deleted_at": null,
                "created_at": "2017-11-22 14:32:25",
                "updated_at": "2017-12-10 15:29:32"
            }
        ]
    },
    "links": {
        "POST": "http://api.jgpr.local/v1/ta/devices"
    },
    "meta": {
        "total": 0,
        "total_in_page": 0,
        "total_pages": 0,
        "current_page": 0,
        "limit": 0,
        "from": 0,
        "to": null,
        "next": null,
        "prev": null
    }
}
```

###### Get device by id
- Method: `GET`
- Request: `v1/ta/devices/{device_id}`
- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "Device: 16",
    "data": {
        "device": {
            "id": 16,
            "name": "Device-1",
            "address": "192.168.20.13",
            "port": 8080,
            "user_name": "admin",
            "refresh_cycle": null,
            "active": 1,
            "device_group_id": 1,
            "description": "Test device",
            "deleted_at": null,
            "created_at": "2017-12-12 15:46:46",
            "updated_at": "2017-12-12 15:46:46"
        }
    },
    "links": {
        "DELETE": "http://api.jgpr.local/v1/ta/devices/16",
        "PUT": "http://api.jgpr.local/v1/ta/devices/16"
    }
}
```

###### Update device by id
- Method: `PUT`
- Request: `v1/ta/devices/{device_id}`
```json
{
	"name": "Updated-device"
}
```
- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "Device: 16",
    "data": {
        "device": {
            "id": 16,
            "name": "Updated-device",
            "address": "192.168.20.13",
            "port": 8080,
            "user_name": "admin",
            "refresh_cycle": null,
            "active": 1,
            "device_group_id": 1,
            "description": "Test device",
            "deleted_at": null,
            "created_at": "2017-12-12 15:46:46",
            "updated_at": "2017-12-12 15:46:46"
        }
    },
    "links": {
        "DELETE": "http://api.jgpr.local/v1/ta/devices/16",
        "PUT": "http://api.jgpr.local/v1/ta/devices/16"
    }
}
```

#### Raden-T41 Devices API
> Raden-T41 devices use HTTP protocol for connection
> As you know, in HTTP protocols before any interacting
> we must check connection status.
> Also in Raden-T41 API, you must check device connection information
>  before any call

###### Test device connection

- Method: `GET`
- Request: `v1/ta/devices/{device_id}/connect/testConnection`
- Response:
```json
{
    "message": "Ok",
    "data": null,
    "status": 200
}
```

###### Get clockings from device
> Notice: Only new clockings will receive in every call
>
> This method get pure clockings from device and save thous in our DB
> if you want to work with clockings, you must use [this](#get-clockings-list) method
>
> Sended option in requested data, says to device that already received data must be sent or not


- Method: `POST`
- Request: `v1/ta/devices/device_id}/connect/getClockings`
```json
{
	"date_from": "2017-11-01",
	"date_to": "2017-11-13",
	"sended": false
}
```
- Response:
```json
{
    "events": [
        {
            "device_id": 16,
            "user_id": 5,
            "io_id": "e68ebc60",
            "type": 2,
            "reason_id": 1,
            "datetime": "2017-11-01T00:00:12",
            "changed": true,
            "status": "accepted"
        },
        {
            "device_id": 16,
            "user_id": 5,
            "io_id": "31381673",
            "type": 1,
            "reason_id": 1,
            "datetime": "2017-11-01T09:52:04",
            "changed": true,
            "status": "accepted"
        }
    ],
    "message": "Clockings received"
}
```

#### Clockings resource

###### create new clocking

|Parameter    |	Type   |  Status  |	Description      |
| ------------| ------ | -------- | -----------------|
|clocking|	array|	required|-|
|clocking.device_id|	numeric|	optional|	Valid TA.device id|
|clocking.user_id|	numeric|	required|-|
|clocking.io_id|	string|	optional|-|
|clocking.type|	string|	optional|	clock_in, clock_out or attendance|
|clocking.reason_id|	integer|	required|	Valid TA.clocking_reason id|
|clocking.datetime|	date|	required|-|
|clocking.changed|	boolean|	optional|-|
|clocking.status|	string|	optional|	waiting, accepted or rejected|
|clocking.description|	string|	optional|-|

- Method: `POST`
- Request: `v1/ta/clockings`
```json
{
	"clocking": {
		"user_id": 5,
		"reason_id": 1,
		"datetime": "2017-01-01 08:00:00",
		"type": "clock_in"
	}
}
```

- Response:
```json
{
    "status": "CREATED",
    "code": 201,
    "message": "Clocking created",
    "data": {
        "id": 307,
        "device_id": null,
        "user_id": 5,
        "io_id": null,
        "type": "clock_in",
        "reason_id": 1,
        "datetime": "2017-01-01 08:00:00",
        "changed": 0,
        "status": "waiting",
        "description": null,
        "deleted_at": null,
        "created_at": "2017-12-13 12:27:30",
        "updated_at": "2017-12-13 12:27:30"
    },
    "links": {
        "GET": "http://api.jgpr.local/v1/ta/clockings/307",
        "DELETE": "http://api.jgpr.local/v1/ta/clockings/307",
        "PUT": "http://api.jgpr.local/v1/ta/clockings/307"
    }
}
```

###### Get clockings list

- Method: `GET`
- Request: `v1/ta/clockings?includes[]=user.profile&includes[]=reason&limit=1&page=100`
- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "Clockings",
    "data": {
        "clockings": [
            {
                "id": 172,
                "device_id": 14,
                "user_id": 342,
                "io_id": "539a6295",
                "type": "clock_in",
                "reason_id": 1,
                "datetime": "2017-11-06 08:43:15",
                "changed": 1,
                "status": "accepted",
                "description": null,
                "deleted_at": null,
                "created_at": null,
                "updated_at": "2017-12-11 10:07:23",
                "user": {
                    "id": 342,
                    "name": null,
                    "email": null,
                    "deleted_at": null,
                    "created_at": "2017-08-13 13:48:28",
                    "updated_at": "2017-11-23 13:30:17",
                    "profile": {
                        "id": 339,
                        "user_id": 342,
                        "personnel_id": "305",
                        "company_id": 1,
                        "first_name": "خشایار",
                        "last_name": "طهماسبی",
                        "national_code": "0014103532",
                        "father_name": null,
                        "birthday": null,
                        "birth_certificate_number": null,
                        "birth_place": null,
                        "birth_register_place": null,
                        "nationality": null,
                        "married": 0,
                        "sex": "male",
                        "address": null,
                        "education": null,
                        "military": null,
                        "avatar": null,
                        "created_at": "2017-08-13 13:48:28",
                        "updated_at": "2017-11-23 13:29:58",
                        "deleted_at": null
                    }
                },
                "reason": {
                    "id": 1,
                    "name": "تردد عادی",
                    "clocking_reason_id": null,
                    "description": null,
                    "created_at": null,
                    "updated_at": null
                }
            }
        ]
    },
    "links": {
        "POST": "http://api.jgpr.local/v1/ta/clockings"
    },
    "meta": {
        "total": 240,
        "total_in_page": 1,
        "total_pages": 240,
        "current_page": 100,
        "limit": 1,
        "from": 100,
        "to": 101,
        "next": 101,
        "prev": 99
    }
}
```

###### Get clocking by id

- Method: `GET`
- Request: `v1/ta/clockings/{clocking_id}`
- Response
```json
{
    "status": "OK",
    "code": 200,
    "message": "Clocking: 172",
    "data": {
        "clocking": {
            "id": 172,
            "device_id": 14,
            "user_id": 342,
            "io_id": "539a6295",
            "type": "clock_in",
            "reason_id": 1,
            "datetime": "2017-11-06 08:43:15",
            "changed": 1,
            "status": "accepted",
            "description": null,
            "deleted_at": null,
            "created_at": null,
            "updated_at": "2017-12-11 10:07:23"
        }
    },
    "links": {
        "DELETE": "http://api.jgpr.local/v1/ta/clockings/172",
        "PUT": "http://api.jgpr.local/v1/ta/clockings/172"
    }
}
```

###### Update a clocking

- Method: `PUT`
- Request: `v1/ta/clockings/{clocking_id}`
```json
{
	"clocking": {
		"type": "clock_out"
	}
}
```

- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "Clocking updated",
    "data": {
        "id": 307,
        "device_id": null,
        "user_id": 5,
        "io_id": null,
        "type": "clock_out",
        "reason_id": 1,
        "datetime": "2017-01-01 08:00:00",
        "changed": 0,
        "status": "waiting",
        "description": null,
        "deleted_at": null,
        "created_at": "2017-12-13 12:27:30",
        "updated_at": "2017-12-13 12:27:30"
    },
    "links": {
        "GET": "http://api.jgpr.local/v1/ta/clockings/307",
        "DELETE": "http://api.jgpr.local/v1/ta/clockings/307",
        "PUT": "http://api.jgpr.local/v1/ta/clockings/307"
    }
}
```

###### Get clocking reasons list

- Method: `GET`
- Request: `v1/ta/clockings/reasons`
- Response:
```json
{
    "status": "OK",
    "code": 200,
    "message": "ClockingReasons",
    "data": {
        "clocking_reasons": [
            {
                "id": 1,
                "name": "تردد عادی",
                "clocking_reason_id": null,
                "description": null,
                "created_at": null,
                "updated_at": null
            },
            {
                "id": 10,
                "name": "مرخصی ساعتی",
                "clocking_reason_id": null,
                "description": null,
                "created_at": null,
                "updated_at": null
            }, ...
        ]
    },
    "links": {
        "POST": "http://api.jgpr.local/v1/ta/clockings/reasons"
    },
    "meta": {
        "total": 0,
        "total_in_page": 0,
        "total_pages": 0,
        "current_page": 0,
        "limit": 0,
        "from": 0,
        "to": null,
        "next": null,
        "prev": null
    }
}
```