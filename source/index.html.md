---
title: ox API 0.1.0
language_tabs:
  - shell: Shell
  - http: HTTP
toc_footers: []
includes:
  - schemas
  - errors
search: true
highlight_theme: darkula
headingLevel: 2

---


# Introduction

Welcome to the Signal Biometrics `ox` API documentation. This should provide you with all the info you need to collect, store, retrieve and expose time series data

From a high-level perspective, the API allows you to apply CRUD operations to entities using the RESTful connector that you want. The entities you can interact with are:
 - Beacon (an emitting device)
 - Client (an API client)
 - Event (a datapoint)
 - Org (an organisation)
 - Sensor (a sensing device)

Our production URL is  
[https://api.signal.bio/](https://api.signal.bio)

Our staging URL is  
[https://api.stage.signal.bio/](https://api.stage.signal.bio)

You can report issues at  
[api.support@signal.bio](mailto:api.support@signal.bio)


# Requests

Almost all requests _need_ to include a valid JWT token. See the [authentication](#authentication) section for more details about this

## Headers
This depends on the endpoint, but as a general rule if the request du not create or modify entities, you will not need to include a header (beside `Authorization`). If you are passing along a JSON payload however, remember to set the `Content-type` header as well

|Header|Content|
|-|-|
|Authorization|bearer _{jwt}_|
|Content-type|application/json|

<aside class="notice">You must replace <code>{headers}</code> with valid headers</aside>

## Payload & params
As a general rule, endpoints that create a single entity will expect a JSON, while those creating many entities will expect an array of these objects

Endpoints that modify entities will either require an `id` URL parameter and a payload (single), or a JSON consisting of the `ids` to modify as properties and their changes as an object. Have a look at the [updateManyBeacon](updatemanybeacon) for an example of this

Endpoints that do not require a payload (read & delete operations) will only require parameters passed in the URL, with multiple values separated by commas

<aside class="notice">You must replace parameters in <code>{brackets}</code> with your own</aside>

# Responses

Our API responses are JSON objects with the results (or error) assigned to the `data` property. Our structure liberally adds a few extra properties to the [JSend](https://labs.omniti.com/labs/jsend) "specs":

> Response JSON

```json
{
  "program": "ox",
  "version": "0.0.1",
  "datetime": "2017-10-11T13:10:11.669Z",
  "timestamp": 1507727411669,
  "code":200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "status": "success",
    "message": "OK",
    "code": 200
  }
}
```

|Property|Usage|
|-|-|
|`program`|The name of the API|
|`version`|The version of the API|
|`datetime`|When the response was sent (i.e. `_new Date`)|
|`timestamp`|When the response was sent (i.e. `Date.now()`)|
|`code`|HTTP status of the response|
|`status`|_success_ or _failure_ or _error_|
|`message`|Additional API message|
|`data`|The error or result payload|

## Success

If the request succeed with a `200`, the response `data` property will be populated according to the HTTP operation requested by the client and the number of entities

|Verb|Scope|Type|Content|
|-|-|-|-|
|POST|Single|Object|{id, info}|
|POST|Collection|Array|[{id, info}]|
|GET|Single|Object|{entity}|
|GET|Collection|Array|[{entity}]|
|PATCH|Single|Object|{id, info}|
|PATCH|Collection|Array|[{id, info}]|
|DELETE|Single|Object|{id, info}|
|DELETE|Collection|Array|[{id, info}]|

## Failure

If the request fails witha `4xx` or `5xx`, the response `data` property will be be the same regardless of the HTTP operation but will change depending on the number of entities

|Verb|Scope|Type|Content|
|-|-|-|-|
|\*|Single|Object|{id, info}|
|\*|Collection|Array|[{id, info}]|


# Authentication

> Header example

```
Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

For all API request, `ox` expects a [JSON Web Token](https://jwt.io/introduction/) to be included in a header that looks like the one on the right

The `org` entity is are the root of our authentication scheme. At this time, only Signal can create new `org`. Beside this, entities have the right to create, modify or delete entities with a higher tier then themselves. The hiearchy looks like this:
|Entity|Owner|Tier|
|-|-|-|
|Org|Signal Biometrics|Tier 0|
|Client|Org|Tier 1|
|Beacon|Org|Tier 1|
|Sensor|Client|Tier 2|
|Event|Client|Tier 2|

> Code samples

```shell--http
# 1. create a client
curl -X POST -H "Content-Type: application/json" -d '{client_entity_payload}' -H "{headers}" https://api.signal.bio/client
# returns response.body.data.client_api_key
```

```shell--http
# 2. retrieve a token
curl https://api.signal.bio/client/token/{parent_org_id}/{client_id}/{client_api_key}
# returns response.body.data.token
```

```shell--http
# 3. authorize a request with a header 
curl -H "Authorization: bearer {jwt}" https://api.signal.bio/{endpoint}
```

Since the API is meant to be accessed programatically, we are not relying on a user/password scheme to grant access to our ressources. Rather, the authentication of a client happens in three steps:
1. An `org` create a client and get an API key
1. The client uses this API key to retrieve a token
1. The client uses this token to authenticate its requests

The token expires, but the client do not. Typically, clients will be devices, user accounts interfaces or applications and will be given the cleartext `client_api_key` as en environement variable or as part of a configuration object

<aside class="warning">Remember that <code>/client</code> is a protected endpoint</aside>

# /

`GET /`

Providing that the request is authorized and that the API is reachable, the root path will respond to a `GET` request with a  `200 — OK`. You can use it to confirm that your token is valid and that the host is reachable

# /beacon

`/beacon`

Beacons collect the readings of sensors, called events. Beacons are typically bluetooth-enabled devices that are connected to the internet and relay back events to our realtime database

When a beacon is added, it is also added to our realtime database and watch both by a server service and our app instances. When the `beacon_last_event` field is updated, this change is broadcasted to all connected & authenticated devices

<h2 id="beacon_create-one">Create One</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/beacon -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/beacon HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /beacon`

*Add a new beacon*

> Body parameter

```json
{
  "parent_org": 1234123412341234,
  "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA",
  "beacon_env": "iOS 9.0",
  "beacon_type": "iphone",
  "beacon_version": "1.1.1/34",
  "beacon_last_event": {
      "event_sensor": 2345234523452345,
      "event_reading": 105,
      "event_type": "heart_rate",
      "event_timestamp": 1509108148219
  }
}
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:28:34.716Z",
  "timestamp": 1517326114716,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5681589568667648"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Beacon](#beacon-schema)|Beacon object that needs to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Beacon](#beacon-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_create-many">Create Many</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/beacon -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/beacon HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /beacon`

*Add many beacons*

> Body parameter

```json
[{
  "parent_org": 1234123412341234,
  "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA",
  "beacon_env": "iOS 9.0",
  "beacon_type": "iphone",
  "beacon_version": "1.1.1/34",
  "beacon_last_event": {
      "event_sensor": 2345234523452345,
      "event_reading": 105,
      "event_type": "heart_rate",
      "event_timestamp": 1509108148219
  }
},
{
  "parent_org": 4321432143214321,
  "beacon_id": "06D44FE3-014D-B0FE-A81F-14037E8701AA",
  "beacon_env": "iOS 8.0",
  "beacon_type": "iphone",
  "beacon_version": "2.1.1",
  "beacon_last_event": {}
}]
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:30:38.308Z",
  "timestamp": 1517326238308,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Beacons](#beacon-schema)|Array of beacon objects that need to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Beacon](#beacon-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_retrieve-one-by-id">Retrieve One by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/beacon/{id} -H "{headers}"
```

```http
GET https://api.signal.io/beacon/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /beacon/{id}`

*Retrieve a beacon by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:49:38.744Z",
  "timestamp": 1517327378744,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "beacon_version": "123",
    "beacon_env": "baz",
    "beacon_type": "roz",
    "beacon_last_event": {
      "baa": "zzz"
    },
    "created_at": "2018-01-16T23:25:25.685Z",
    "parent_org": {
      "name": "1234123412340000",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412340000"
      ]
    },
    "modified_at": "2018-01-16T23:25:25.685Z",
    "beacon_id": "bar"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Beacon entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_retrieve-many-by-id">Retrieve Many by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/beacon/{id},{id},{id} -H "{headers}"
```

```http
GET https://api.signal.io/beacon/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /beacon/{id},{id},{id}`

*Retrieve many beacons by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:52:55.965Z",
  "timestamp": 1517327575965,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": [
    {
      "beacon_env": "iOS 9.0",
      "beacon_type": "iphone",
      "beacon_last_event": {
        "event_type": "heart_rate",
        "event_sensor": 2345234523452345,
        "event_timestamp": 1509108148219,
        "event_reading": 105
      },
      "created_at": "2018-01-30T15:30:38.181Z",
      "parent_org": {
        "id": "1234123412341234",
        "kind": "Org",
        "path": [
          "Org",
          "1234123412341234"
        ]
      },
      "modified_at": "2018-01-30T15:30:38.181Z",
      "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA",
      "beacon_version": "1.1.1/34"
    },
    {
      "beacon_version": "1.1.1/34",
      "beacon_env": "iOS 9.0",
      "beacon_type": "iphone",
      "beacon_last_event": {
        "event_type": "heart_rate",
        "event_sensor": 2345234523452345,
        "event_timestamp": 1509108148219,
        "event_reading": 105
      },
      "created_at": "2018-01-30T15:30:38.181Z",
      "parent_org": {
        "id": "1234123412341234",
        "kind": "Org",
        "path": [
          "Org",
          "1234123412341234"
        ]
      },
      "modified_at": "2018-01-30T15:30:38.182Z",
      "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA"
    }
  ]
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Beacon entity ids separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_delete-one-by-id">Delete One by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/beacon/{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/beacon/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /beacon/{id}`

*Delete a beacon by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:55:13.871Z",
  "timestamp": 1517327713871,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Beacon entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_delete-many-id">Delete Many by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/beacon/{id},{id},{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/beacon/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /beacon/{id},{id},{id}`

*Delete many beacons by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:57:57.367Z",
  "timestamp": 1517327877367,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Beacon entity id separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_update-one-by-id">Update One by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/beacon -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/beacon HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /beacon`

*Update an existing beacon*

> Body parameter

```json
{ 
    "5655778694266880": {
        "beacon_env": "iOS 10.0",
        "beacon_type": "iphone",
        "beacon_version": "2.2.0/5"
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Beacon](#beacon-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="beacon_update-many-by-id">Update Many by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/beacon -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/beacon HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /beacon`

*Update many existing beacons*

> Body parameter

```json
{   
    "5655778694266880": {
        "beacon_env": "iOS 10.0",
        "beacon_type": "iphone",
        "beacon_version": "2.2.0/5"
    },
    "5092828740845568": {
        "beacon_env": "iOS 11.0",
        "beacon_type": "ipad",
        "beacon_version": "1.2.0"
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880",
      "5092828740845568"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Beacon](#beacon-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>


# /client

`/client`

Clients are service accounts with access to the API. These accounts can be used on devices or as components for any authflow that involve access to the API. More precisely, the clients are used to issue and validate API keys, which in turn are used to sign and generate access tokens. They are also used to establish permission on our different ressources

<h2 id="client_create-one">Create One</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/client -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/client HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /client`

*Add a new client*

> Body parameter

```json
{
  "parent_org": 1234123412341234,
  "parent_user": 4321432143214321,
  "client_name": "My first client",
  "client_id": "ipad/123",
}
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:28:34.716Z",
  "timestamp": 1517326114716,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5681589568667648"
    ],
    "info": "done",
    "client_api_key": "0dbe990f688799287c3673382a9d4460cd343f2511a0cff8a26bf088a726c805"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Client](#client-schema)|Client object that needs to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Client](#client-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="notice">The clear text API key is returned in <code>data.client_api_key</code></aside>

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_create-many">Create Many</h2>

<aside class="warning">Not implemented</aside>

> Code samples

```shell
curl -X POST https://api.signal.bio/client -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/client HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /client`

*Add many clients*

> Body parameter

```json
[{
  "parent_org": 1234123412341234,
  "parent_user": 4321432143214321,
  "client_name": "My first client",
  "client_id": "ipad/123",
},{
  "parent_org": 1234123412341234,
  "parent_user": 4321432143214321,
  "client_name": "My second client",
  "client_id": "ipod/777",
}
]
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:30:38.308Z",
  "timestamp": 1517326238308,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Clients](#client-schema)|Array of client objects that need to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Client](#client-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">Not implemented</aside>

<h2 id="client_issue-token">Issue Token</h2>

> Code samples

```shell
curl https://api.signal.bio/client/token/{parent_org}/{client_id}/{client_api_key} -H "{headers}"
```

```http
GET https://api.signal.io/client/token/{parent_org}/{client_id}/{client_api_key} HTTPS/1.1
Host: api.signal.bio
```

`GET /client/tokent/{parent_org}/{client_id}/{client_api_key}`

*Retrieve a JSON Web Token signed with an API key*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:49:38.744Z",
  "timestamp": 1517327378744,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "parent_user": {
      "id": "4321432143214321",
      "kind": "User",
      "path": [
        "User",
        "4321432143214321"
      ]
    },
    "client_secret": "$2a$10$L/GOsc5v4lw/iUzuYDOUH.zBEMb1fbYrFlJJXtLTYC78lG.1nl.Fa",
    "client_id": "baz",
    "created_at": "2018-02-01T12:58:37.574Z",
    "parent_org": {
      "id": "1234123412341234",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412341234"
      ]
    },
    "modified_at": "2018-02-01T12:58:37.579Z",
    "client_name": "booboo",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXJlbnRfb3JnIjp7ImlkIjoiMTIzNDEyMzQxMjM0MTIzNCIsImtpbmQiOiJPcmciLCJwYXRoIjpbIk9yZyIsIjEyMzQxMjM0MTIzNDEyMzQiXX0sInBhcmVudF91c2VyIjp7ImlkIjoiNDMyMTQzMjE0MzIxNDMyMSIsImtpbmQiOiJVc2VyIiwicGF0aCI6WyJVc2VyIiwiNDMyMTQzMjE0MzIxNDMyMSJdfSwiY2xpZW50X2lkIjoiYmF6IiwiY2xpZW50X25hbWUiOiJib29ib28iLCJpYXQiOjE1MTc1MDgxMDMsImV4cCI6MTUxNzc2NzMwM30.iizTopa0Zr0ddcmi8Htu4dcN_lpFrTdzoblDyt33bls"
  },
  "created_at": "2018-01-16T23:25:25.685Z",
  "modified_at": "2018-05-17T11:36:55.907Z",
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Client entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|403|[Forbidden](#errors)|The API key does not match the one stored|None|
|404|[Not Found](#errors)|The client was not found|None|

<aside class="notice">The JWT in <code>data.token</code> his used to authenticate our requests</aside>

<aside class="success">This endpoint does not require authentication</aside>

<h2 id="client_retrieve-one-by-id">Retrieve One by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/client/{id} -H "{headers}"
```

```http
GET https://api.signal.io/client/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /client/{id}`

*Retrieve a client by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:49:38.744Z",
  "timestamp": 1517327378744,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "client_id": "ipad/123",
    "client_name": "My first client",
    "parent_org": {
      "name": "1234123412341234",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412341234"
        ]
    },
    "parent_user": {
      "name": "4321432143214321",
      "kind": "Org",
      "path": [
        "Org",
        "4321432143214321"
        ]
    },
  },
  "created_at": "2018-01-16T23:25:25.685Z",
  "modified_at": "2018-05-17T11:36:55.907Z",
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Client entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_retrieve-many-by-id">Retrieve Many by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/client/{id},{id},{id} -H "{headers}"
```

```http
GET https://api.signal.io/client/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /client/{id},{id},{id}`

*Retrieve many clients by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:52:55.965Z",
  "timestamp": 1517327575965,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": [{
    "client_id": "ipad/123",
    "client_name": "My first client",
    "parent_org": {
      "name": "1234123412341234",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412341234"
        ]
    },
  },
  {
    "client_id": "iphone/777",
    "client_name": "My second client",
    "parent_org": {
      "name": "1234123412341234",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412341234"
        ]
    }
  }]
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Client entity ids separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_delete-one-by-id">Delete One by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/client/{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/client/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /client/{id}`

*Delete a client by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:55:13.871Z",
  "timestamp": 1517327713871,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Client entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_delete-many-id">Delete Many by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/client/{id},{id},{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/client/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /client/{id},{id},{id}`

*Delete many clients by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:57:57.367Z",
  "timestamp": 1517327877367,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Client entity id separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_update-one-by-id">Update One by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/client -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/client HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /client`

*Update an existing client*

> Body parameter

```json
{ 
    "5655778694266880": {
        "client_name": "new device name",
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Client](#client-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="client_update-many-by-id">Update Many by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/client -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/client HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /client`

*Update many existing clients*

> Body parameter

```json
{   
    "5655778694266880": {
        "client_type": "very new device name",
    },
    "5092828740845568": {
        "client_type": "very very new device name",
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880",
      "5092828740845568"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Client](#client-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>


# /event

`/event`

Events are the fragment of information that we store on the backend. It consist of a sensor value of a biometric measurement, along with a timestamp and information about the entities owning this event (sensor, beacon, beacon)

These events are aggregated and can be queried, however they are hardly ever created through the API: they are generally created when a sensor entity's `last_event` property is updated in our realtime database

<h2 id="event_create-one">Create One</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/event -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/event HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /event`

*Add a new event*

> Body parameter

```json
{
  "parent_org": 1234123412341234,
  "event_beacon": 5432543254325432,
  "event_sensor": 678967896789,
  "event_reading": 123,
  "event_type": "heart_rate",
  "event_timestamp": 1234567000,
}
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:28:34.716Z",
  "timestamp": 1517326114716,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
  "ids": [
    "4873960965537792"
  ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Event](#event-schema)|Event object that needs to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Event](#event-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_create-many">Create Many</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/event -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/event HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /event`

*Add many events*

> Body parameter

```json
[{
  "parent_org": 1234123412341234,
  "event_beacon": 5432543254325432,
  "event_sensor": 678967896789,
  "event_reading": 123,
  "event_type": "heart_rate",
  "event_timestamp": 1234567000,
},{
  "parent_org": 1234123412341234,
  "event_beacon": 5432543254325432,
  "event_sensor": 678967896789,
  "event_reading": 123,
  "event_type": "heart_rate",
  "event_timestamp": 1234567000,
}]
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:30:38.308Z",
  "timestamp": 1517326238308,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Events](#event-schema)|Array of event objects that need to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Event](#event-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_retrieve-one-by-id">Retrieve One by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/event/{id} -H "{headers}"
```

```http
GET https://api.signal.io/event/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /event/{id}`

*Retrieve a event by id*

> Success response

```json
{
  "created_at": "2018-02-05T20:18:25.706Z",
  "parent_org": {
    "id": "1234123412341234",
    "kind": "Org",
    "path": [
      "Org",
      "1234123412341234"
    ]
  },
  "event_beacon": {
    "id": "5432543254325432",
    "kind": "Beacon",
    "path": [
      "Beacon",
      "5432543254325432"
    ]
  },
  "event_sensor": {
    "id": "678967896789",
    "kind": "Sensor",
    "path": [
      "Sensor",
      "678967896789"
    ]
  },
  "event_timestamp": 1234567000,
  "event_reading": 123,
  "event_type": "heart_rate"
} 
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Event entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_retrieve-many-by-id">Retrieve Many by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/event/{id},{id},{id} -H "{headers}"
```

```http
GET https://api.signal.io/event/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /event/{id},{id},{id}`

*Retrieve many events by id*

> Success response

```json
[{
  "created_at": "2018-02-05T20:18:25.706Z",
  "parent_org": {
    "id": "1234123412341234",
    "kind": "Org",
    "path": [
      "Org",
      "1234123412341234"
    ]
  },
  "event_beacon": {
    "id": "5432543254325432",
    "kind": "Beacon",
    "path": [
      "Beacon",
      "5432543254325432"
    ]
  },
  "event_sensor": {
    "id": "678967896789",
    "kind": "Sensor",
    "path": [
      "Sensor",
      "678967896789"
    ]
  },
  "event_timestamp": 1234567000,
  "event_reading": 123,
  "event_type": "heart_rate"
},
{event},
{event}]
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Event entity ids separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_delete-one-by-id">Delete One by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/event/{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/event/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /event/{id}`

*Delete a event by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:55:13.871Z",
  "timestamp": 1517327713871,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Event entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_delete-many-id">Delete Many by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/event/{id},{id},{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/event/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /event/{id},{id},{id}`

*Delete many events by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:57:57.367Z",
  "timestamp": 1517327877367,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Event entity id separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_update-one-by-id">Update One by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/event -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/event HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /event`

*Update an existing event*

> Body parameter

```json
{ 
    "5655778694266880": {
	  "parent_org": 0987098709870987,
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Event](#event-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="event_update-many-by-id">Update Many by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/event -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/event HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /event`

*Update many existing events*

> Body parameter

```json
{   
    "5655778694266880": {
	  "parent_org": 8765876587658765
    },
    "5092828740845568": {
	  "parent_org": 3456345634563456
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880",
      "5092828740845568"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Event](#event-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

# /org

`/org`

Org are the top entities of our datastructure, and superseed or hold permission over all others. An organisation contains child beacons, users, clients, and sensors and it is itself owned by a user (or another org)  

Every child element can be enabled or disabled at the organisation level (see the [IdMap schema](#idmap-schema))


<h2 id="org_create-one">Create One</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/org -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/org HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /org`

*Add a new org*

> Body parameter

```json
{
 "org_owner": 5432543254325432,
 "org_active": true,
  "org_sensors": {
    "1234567890981111": 1515513411000
  },
  "org_beacons": {
    "1234567890987654": 1515513419829
  },
  "org_users": {
    "1234567890988888": 1515513419829
  },
  "org_alias": "FastCars Co.",
  "org_clients": {
    "1234567890987777": 1515513411000
  }
}
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-14T12:50:10.658Z",
  "timestamp": 1518612610658,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5713106038685696"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Org](#org-schema)|Org object that needs to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Org](#org-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_create-many">Create Many</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/org -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/org HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /org`

*Add many orgs*

> Body parameter

```json
[{
 "org_owner": 5432543254325432,
 "org_active": true,
  "org_sensors": {
    "1234567890981111": 1515513411000
  },
  "org_beacons": {
    "1234567890987654": 1515513419829
  },
  "org_users": {
    "1234567890988888": 1515513419829
  },
  "org_alias": "FastCars Co.",
  "org_clients": {
    "1234567890987777": 1515513411000
  }
},{
 "org_owner": 5432543254325643,
 "org_active": true,
  "org_sensors": {
    "1234567890983331": 1515513411333
  },
  "org_beacons": {
    "1234567890983211": 1515513419444
  },
  "org_users": {
    "1234567890983543": 1515513419666
  },
  "org_alias": "FastCars Co.",
  "org_clients": {
    "1234567890981234": 1515513417777
  }
}]
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:30:38.308Z",
  "timestamp": 1517326238308,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Orgs](#org-schema)|Array of org objects that need to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Org](#org-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|


<h2 id="org_retrieve-one-by-id">Retrieve One by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/org/{id} -H "{headers}"
```

```http
GET https://api.signal.io/org/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /org/{id}`

*Retrieve a org by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-14T17:06:43.393Z",
  "timestamp": 1518628003393,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "created_at": "2018-02-05T20:18:25.706Z",
    "parent_org": {
      "id": "1234123412341234",
      "kind": "Org",
      "path": [
        "Org",
        "1234123412341234"
      ]
    },
    "event_beacon": {
      "id": "5432543254325432",
      "kind": "Beacon",
      "path": [
        "Beacon",
        "5432543254325432"
      ]
    },
    "event_sensor": {
      "id": "678967896789",
      "kind": "Sensor",
      "path": [
        "Sensor",
      ]
        "678967896789"
    },
    "event_timestamp": 1234567000,
    "event_reading": 123,
    "event_type": "heart_rate"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Org entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_retrieve-many-by-id">Retrieve Many by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/org/{id},{id} -H "{headers}"
```

```http
GET https://api.signal.io/org/{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /org/{id},{id}`

*Retrieve many orgs by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-14T17:09:04.125Z",
  "timestamp": 1518628144125,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": [
    {
      "modified_at": "2018-01-09T16:11:43.148Z",
      "org_owner": {
        "id": "1234432112344321",
        "kind": "User",
        "path": [
          "User",
          "1234432112344321"
        ]
      },
      "org_active": true,
      "org_sensors": {
        "1234567890981111": 1515513411000,
        "1234567890987654": 1515513419829
      },
      "org_beacons": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "org_alias": "goo",
      "org_users": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "org_clients": {
        "1234567890981111": 1515513411000,
        "1234567890987654": 1515513419829
      },
      "created_at": "2018-01-09T16:11:43.148Z"
    },
    {
      "org_clients": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "created_at": "2018-01-09T16:07:31.373Z",
      "modified_at": "2018-01-09T16:07:31.374Z",
      "org_owner": {
        "id": "1234432112344321",
        "kind": "User",
        "path": [
          "User",
          "1234432112344321"
        ]
      },
      "org_active": true,
      "org_sensors": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "org_beacons": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "org_users": {
        "1234567890987654": 1515513419829,
        "1234567890981111": 1515513411000
      },
      "org_alias": "goo"
    }
  ]
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Org entity ids separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_delete-one-by-id">Delete One by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/org/{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/org/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /org/{id}`

*Delete a org by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:55:13.871Z",
  "timestamp": 1517327713871,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Org entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_delete-many-id">Delete Many by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/org/{id},{id},{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/org/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /org/{id},{id},{id}`

*Delete many orgs by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:57:57.367Z",
  "timestamp": 1517327877367,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Org entity id separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_update-one-by-id">Update One by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/org -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/org HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /org`

*Update an existing org*

> Body parameter

```json
{ 
    "5655778694266880": {
        "org_active": false
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Org](#org-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="org_update-many-by-id">Update Many by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/org -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/org HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /org`

*Update many existing orgs*

> Body parameter

```json
{   
    "5655778694266880": {
        "org_active": false 
    },
    "5092828740845568": {
        "org_alias": "New Company Name Inc."
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880",
      "5092828740845568"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Org](#org-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>


# /sensor

`/sensor`

Sensors are devices that pick up the events and broadcast them to beacons. A sensor could be an assembly doing both the acquisition (i.e. sensing device) and the broadcasting (i.e. the enclosure)  

Sensors can be provisioned (and re-reprovisioned) for different organisations, and we keep track of them as part of our event flow to add one more layer of data accountabilty/ownership

<h2 id="sensor_create-one">Create One</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/sensor -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/sensor HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /sensor`

*Add a new sensor*

> Body parameter

```json
{
  "parent_org": 5678567856785678,
  "sensor_version": "7",
  "sensor_type": "glove/palm",
  "sensor_id": "12345"
}
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-15T12:56:48.446Z",
  "timestamp": 1518699408446,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5759902660165632"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Sensor](#sensor-schema)|Sensor object that needs to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Sensor](#sensor-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_create-many">Create Many</h2>

> Code samples

```shell
curl -X POST https://api.signal.bio/sensor -H "{headers}" -d "{payload}"
```

```http
POST https://api.signal.io/sensor HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`POST /sensor`

*Add many sensors*

> Body parameter

```json
[{
  "parent_org": 5678567856785678,
  "sensor_version": "7",
  "sensor_type": "glove/palm",
  "sensor_id": "12345"
},{
  "parent_org": 5678567856785678,
  "sensor_version": "2",
  "sensor_type": "chest/mid",
  "sensor_id": "432"
}]
```

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:30:38.308Z",
  "timestamp": 1517326238308,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|[Sensor](#sensor-schema)|Array of sensor objects that need to be added|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Sensor](#sensor-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|


<h2 id="sensor_retrieve-one-by-id">Retrieve One by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/sensor/{id} -H "{headers}"
```

```http
GET https://api.signal.io/sensor/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /sensor/{id}`

*Retrieve a sensor by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-15T13:01:55.847Z",
  "timestamp": 1518699715847,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "sensor_version": "7",
    "created_at": "2018-02-15T12:56:48.338Z",
    "parent_org": {
      "id": "5678567856785678",
      "kind": "Org",
      "path": [
        "Org",
        "5678567856785678"
      ]
    },
    "sensor_type": "glove/palm",
    "modified_at": "2018-02-15T12:56:48.339Z",
    "sensor_id": "12345"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Sensor entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_retrieve-many-by-id">Retrieve Many by Id</h2>

> Code samples

```shell
curl https://api.signal.bio/sensor/{id},{id} -H "{headers}"
```

```http
GET https://api.signal.io/sensor/{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`GET /sensor/{id},{id}`

*Retrieve many sensors by id*

> Success response

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-02-15T13:08:14.228Z",
  "timestamp": 1518700094228,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": [
    {
      "sensor_version": "7",
      "created_at": "2018-02-15T13:06:03.016Z",
      "parent_org": {
        "id": "5678567856785678",
        "kind": "Org",
        "path": [
          "Org",
          "5678567856785678"
        ]
      },
      "sensor_type": "glove/palm",
      "modified_at": "2018-02-15T13:06:03.017Z",
      "sensor_id": "12345"
    },
    {
      "created_at": "2018-02-15T13:06:03.016Z",
      "parent_org": {
        "id": "5678567856785678",
        "kind": "Org",
        "path": [
          "Org",
          "5678567856785678"
        ]
      },
      "sensor_type": "glove/palm",
      "modified_at": "2018-02-15T13:06:03.016Z",
      "sensor_id": "12345",
      "sensor_version": "7"
    }
  ]
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Sensor entity ids separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_delete-one-by-id">Delete One by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/sensor/{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/sensor/{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /sensor/{id}`

*Delete a sensor by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:55:13.871Z",
  "timestamp": 1517327713871,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|id|url|Integer|Sensor entity id|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_delete-many-id">Delete Many by Id</h2>

> Code samples

```shell
curl -X DELETE https://api.signal.bio/sensor/{id},{id},{id} -H "{headers}"
```

```http
DELETE https://api.signal.io/sensor/{id},{id},{id} HTTPS/1.1
Host: api.signal.bio
Authorization: bearer {jwt}
```

`DELETE /sensor/{id},{id},{id}`

*Delete many sensors by id*

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T15:57:57.367Z",
  "timestamp": 1517327877367,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "6487942298075136",
      "4658354949455872"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|ids|url|Integer|Sensor entity id separated by commas|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_update-one-by-id">Update One by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/sensor -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/sensor HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /sensor`

*Update an existing sensor*

> Body parameter

```json
{ 
    "5655778694266880": {
        "parent_org": 1111222233334444
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Sensor](#sensor-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>

<h2 id="sensor_update-many-by-id">Update Many by Id</h2>

> Code samples

```shell
curl -X PUT https://api.signal.bio/sensor -H "{headers}" -d "{payload}"
```

```http
PUT https://api.signal.io/sensor HTTPS/1.1
Host: api.signal.bio
Content-Type: application/json
Authorization: bearer {jwt}
```

`PUT /sensor`

*Update many existing sensors*

> Body parameter

```json
{   
    "5655778694266880": {
        "parent_org": false 
    },
    "5092828740845568": {
        "sensor_id": "yul-773"
    }
}
```

> Success payload

```json
{
  "program": "ox",
  "version": "0.0.5",
  "datetime": "2018-01-30T16:13:33.062Z",
  "timestamp": 1517328813062,
  "code": 200,
  "status": "success",
  "message": "Call successful",
  "data": {
    "ids": [
      "5655778694266880",
      "5092828740845568"
    ],
    "info": "done"
  }
}
```

### Parameters

|Parameter|In|Type|Description|
|---|---|---|---|---|
|body|body|Object|Key/value pairs of entities and updates|

### Responses

|Status|Meaning|Description|Schema|
|---|---|---|---|
|200|OK|Request was successful|None|
|400|[Bad Request](#errors)|Invalid payload|[Sensor](#sensor-schema)|
|401|[Unauthorized](#errors)|Invalid token|None|

<aside class="warning">You must be authenticated to access this endpoint</aside>
