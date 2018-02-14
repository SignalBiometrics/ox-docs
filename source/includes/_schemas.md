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
  "client_secret": "$2a$10$L/GOsc5v4lw/iUzuYDOUH.zBEMb1fbYrFlJJXtLTYC78lG.1nl.Fa",
  "created_at": "2018-01-16T23:25:25.685Z",
  "modified_at": "2018-05-17T11:36:55.907Z"
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
|client_secret|Generated on creation|

### Modified Values

|Property|From|To|
|-|-|-|
|parent_org|Integer|Object(Key)|
|parent_user|Integer|Object(Key)|


<h2 id="event-schema">Event</h2>

> Request Payload

```json
{
  "collected_at": 1234567000,
  "parent_org": 1234123412341234,
  "parent_beacon": 5432543254325432,
  "parent_sensor": 678967896789,
  "event_reading": 123,
  "event_type": "heart_rate"
}
```

> Return Payload

```json
{
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
  "event_type": "heart_rate",
  "created_at": "2018-02-05T20:18:25.706Z",
  "modified_at": "2018-02-05T20:18:25.706Z",
  "parent_org": {
    "id": "1234123412341234",
    "kind": "Org",
    "path": [
      "Org",
      "1234123412341234"
    ]
  }
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
|created_at|When the entity was created|
|modified_at|When the entity was last modified|

### Modified Values

|Property|From|To|
|-|-|-|
|parent_org|Integer|Object(Key)|


<h2 id="org-schema">Org</h2>

> Request Payload

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

> Return Payload

```json
{
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
  },
  "created_at": "2018-02-14T12:50:10.283Z",
  "modified_at": "2018-02-14T12:50:10.285Z",
  "org_owner": {
    "id": "5432543254325432",
    "kind": "User",
    "path": [
      "User",
      "5432543254325432"
    ]
  },
  "org_active": true
}
```

### Properties

|Name|Type|Required|Description|
|-|-|-|-|
|org_owner|Integer|true|The id of this org's owner|

### Appended Values

|Property|Description|
|-|-|
|created_at|When the entity was created|
|modified_at|When the entity was last modified|

### Modified Values

|Property|From|To|
|-|-|-|


<h2 id="sensor-schema">Sensor</h2>
