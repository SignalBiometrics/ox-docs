# Schemas

Schemas **might not** be identical our internal representation (i.e. models) of the entities or even how entities are returned when queried. To illustrate this, we are including examples payload that should be passed with POST requests (_Request Payload_), and what is the expected results from GET requests (_Response Payload_)

There is also table with _Appended Values_ and _Modified Values_. _Appended Values_ are added during the creation or modification of entities, while _Modified Values_ are changed on the fly, for instance when casting a type into another

<aside class="notice">Remember that the <i>Response Payload</i> will be in <code>response.data</code></aside>

<h2 id="beacon-schema">Beacon</h2>

> Request Payload

```json
{
  "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA",
  "beacon_env": "iOS 9.0",
  "beacon_type": "iphone",
  "beacon_version": "1.1.1/34",
  "beacon_last_event": {
    "foo": "bar"
  },
  "parent_org": 1234123412341234,
}
```

> Return Payload

```json
{
  "beacon_id": "014DB0FE-06D4-4FE3-A81F-14037E8701AA",
  "beacon_env": "iOS 9.0",
  "beacon_type": "iphone",
  "beacon_version": "1.1.1/34",
  "beacon_last_event": {
    "foo": "bar"
  },
  "parent_org": {
    "name": "1234123412341234",
    "kind": "Org",
    "path": [
      "Org",
      "1234123412341234"
    ]
  },
  "created_at": "2018-01-16T23:25:25.685Z",
  "modified_at": "2018-05-17T11:36:55.907Z",
}
```

### Properties

|Name|Type|Required|Description|
|-|-|-|-|
|parent_org|Integer|true|The id of this beacon's parent org|
|beacon_id|String|true|Custom local id of this device|
|beacon_env|String|true|Device OS or model (including version)|
|beacon_type|String|true|Device type| 
|beacon_version|String|true|Version number of the Signal app|
|beacon_last_event|Object([Event](#event-schema))|false|The last event captured by this beacon|

### Appended Values

|Property|Description|
|-|-|
|id|Id of the entity|
|created_at|When the entity was created|
|modified_at|When the entity was last modified|

### Modified Values

|Property|From|To|
|-|-|-|
|parent_org|Integer|Object(Key)|


<h2 id="client-schema">Client</h2>

> Request Payload

```json
{
  "client_id": "ipad/123",
  "client_name": "My first client",
  "parent_org": 1234123412341234,
  "parent_user": 432143214321432
}
```

> Return Payload

```json
{
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
  "created_at": "2018-01-16T23:25:25.685Z",
  "modified_at": "2018-05-17T11:36:55.907Z",
}
```

### Properties

|Name|Type|Required|Description|
|-|-|-|-|
|client_id|String|true|User defined id for the client|
|client_name|String|true|User defined common name for the client|
|parent_org|Integer|_true_|The id of this beacon's parent org|
|parent_user|Number|_true_|The id of this client's parent org\*|

\* *DEPRECATED* do not use

### Appended Values

|Property|Description|
|-|-|
|created_at|When the entity was created|
|modified_at|When the entity was last modified|

### Modified Values

|Property|From|To|
|-|-|-|
|parent_org|Integer|Object(Key)|
|parent_user|Integer|Object(Key)|
