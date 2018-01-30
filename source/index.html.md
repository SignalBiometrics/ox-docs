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
    "1234123412341234": {
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
    "1234123412341234": {
        "beacon_env": "iOS 10.0",
        "beacon_type": "iphone",
        "beacon_version": "2.2.0/5"
    },
    "4321432143214321": {
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

# /event
# /org
# /sensor
