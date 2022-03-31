# REST API
This API was designed to provide access via REST to all configurations that are accessible by the Web UI.
In order to access the API you'll need a user with the corresponding permissions for the endpoints you want to use.

The UI also utilizes this API, plus an additional websocket for live updates.

## General information
The API accepts only `application/json`.

Responses are in the following format:
```json
{
  "success": true,
  "message": "Some message",
  "data": [
    1, 2, 3
  ]
}
```

In case of an error, there will be the field `message` instead of `data` to provide additional information:
```json
{
  "success": false,
  "message": "Parameter username is required, but missing"
}
```

## Authentication
- Method: `POST`
- Endpoint: `/api/v1/authenticate`

**Example body**
```json
{
  "username": "user01",
  "password": "super-secure-password"
}
```

If the request was successful, you'll be provided with a cookie named `sessionid`. 
The cookie will be valid 24 hours. After that period, you have to reauthenticate again.

Every other endpoint requires a valid cookie to be sent to.

## Logout
- Method: `GET`, `POST`
- Endpoint: `/api/v1/logout`

This endpoint is used to invalidate the current session for a logged-in user.

## Test
- Method: `GET`
- Endpoint: `/api/v1/test`

This endpoint is used to check if the user is authenticated or not. Mainly intended for use in the frontend.

**Response:**
```json
{
  "success": true
}
```

## Routine API
This API provides access to actions instead of objects. Therefore, no RESTful API is used.

### Update declaration of proxies
- Method: `POST`
- Endpoint: `/api/v1/updateDeclaration`

**Parameter**

| Parameter |           Type           | Optional | Description                                              |
|:----------|:------------------------:|:--------:|:---------------------------------------------------------|
| proxies   |   [List](#list) of int   |   Yes    | List of ids of proxies the declaration should be updated |

With this endpoint, you can update the declaration of proxies to start scheduling new / updated hosts or metrics.
If `proxies` is not specified, the declaration of **all** proxies is updated, 
so be careful, when using in deployments with many proxies.

### Generate configuration for proxy
- Method: `POST`
- Endpoint: `/api/v1/generateConfiguration`

**Parameter**

| Parameter | Type | Optional | Description                                       |
|:----------|:----:|:--------:|:--------------------------------------------------|
| proxy     | int  |    No    | ID of the proxy to generate the configuration for |

With this method, you can generate the configuration for a proxy. In the `data` field of the response,
you'll find the command to execute on the proxy to apply the new configuration:
```json
{
  "success": true,
  "message": "Request was successful",
  "data": "/usr/sbin/q-proxy/venv/bin/python3 /usr/sbin/q-proxy/manage.py init --b64 'eyJpZCI6IDEsICJuYW1lIjogImxvY2FsaG9zdCIsICJhZGRyZXNzIjogImh0dHA6Ly9leGFtcGxlLmNvbSIsICJwb3J0IjogODQ0MywgInNlY3JldCI6ICJzdXBlci1zZWNyZXQiLCAid2ViX3NlY3JldCI6ICJzdXBlci1zZWNyZXQiLCAid2ViX2FkZHJlc3MiOiAiaHR0cDovL2V4YW1wbGUuY29tIiwgIndlYl9wb3J0IjogNDQ0MywgImRpc2FibGVkIjogZmFsc2UsICJjb21tZW50IjogIiJ9'"
}
```

## Model API
This API provides access to all model based objects like hosts, metrics, ... 

### Checks

Checks are the heart of the monitoring engine. They define how a specific metric should be gathered. 
The parameter `cmd` is executed by the `q-scheduler`. 
```json
{
  "id": 1,
  "name": "Linux uptime",
  "cmd": "$q-plugins$ --plugin linux.uptime",
  "comment": "A check to retrieve the current uptime of a linux box."
}
```

#### Get Checks
- Method: `GET`
- Endpoint: `/api/v1/checks`

**Parameter**

| Parameter |         Type         | Optional | Description                                         |
|:----------|:--------------------:|:--------:|:----------------------------------------------------|
| filter    | [List](#list) of int |   Yes    | List of ids of checks you want to retrieve          |
| values    | [List](#list) of str |   Yes    | List of attributes of a check you want to retrieve. |

Checks can be gathered using this endpoint. You can use the `filter` parameter to limit the results to a list of given ids.
The `values` parameter can be used to retrieve only the given attributes (`id` is always returned, you don't need to specify it).

#### Get single Check
- Method: `GET`
- Endpoint: `/api/v1/checks/<id>`

**Parameter**

| Parameter |         Type         | Optional | Description                                         |
|:----------|:--------------------:|:--------:|:----------------------------------------------------|
| values    | [List](#list) of str |   Yes    | List of attributes of a check you want to retrieve. |

You can retrieve a single check using this endpoint. The `values` parameter can be used to retrieve only
the given attributes (`id` is always returned, you don't need to specify it).

#### Create Check
- Method: `POST`
- Endpoint: `/api/v1/checks`

**Example body**
```json
{
  "name": "Linux uptime",
  "cmd": "$q-plugins$ --plugin linux.uptime",
  "comment": "A check to retrieve the current uptime of a linux box."
}
```

**Parameter**

| Parameter | Type | Optional | Description                       |
|:----------|:----:|:--------:|:----------------------------------|
| name      | str  |    No    | Name of the Check. Must be unique |
| cmd       | str  |   Yes    | Commandline to execute            |
| comment   | str  |   Yes    | Comment                           |

If the check was created successfully, the `id` is returned in the `data` field:
```json
{
  "success": true,
  "message": "Object was created",
  "data": 22
}
```

#### Modify Check
- Method: `PUT`
- Endpoint: `/api/v1/checks/<id>`

**Example body**
```json
{
  "name": "Linux uptime check"
}
```

**Parameter**

| Parameter | Type | Description                       |
|:----------|:----:|:----------------------------------|
| name      | str  | Name of the check. Must be unique |
| cmd       | str  | Commandline to execute            |
| comment   | str  | Comment                           |

You can modify each of the above parameters. Include it in the body with its new value.

#### Delete Check
- Method: `DELETE`
- Endpoint: `/api/v1/checks/<id>`

### Contacts
Contacts represent the targets of notifications that should be sent. 
```json
{
  "id": 1,
  "name": "Max Mustermann",
  "mail": "max@example.com",
  "linked_host_notifications": [
    1, 2
  ],
  "linked_host_notification_period": 1,
  "linked_metric_notifications": [
    3, 4
  ],
  "linked_metric_notification_period": 1,
  "variables": {
    "matrix": "@max:matrix.example.com"
  },
  "comment": ""
}
```

#### Get Contacts
- Method: `GET`
- Endpoint: `/api/v1/contacts`

**Parameter**

| Parameter |         Type         | Optional | Description                                           |
|:----------|:--------------------:|:--------:|:------------------------------------------------------|
| filter    | [List](#list) of int |   Yes    | List of ids of contacts you want to retrieve          |
| values    | [List](#list) of str |   Yes    | List of attributes of a contact you want to retrieve. |

Contacts can be gathered using this endpoint. You can use the `filter` parameter to limit the results to a list of given ids.
The `values` parameter can be used to retrieve only the given attributes (`id` is always returned, you don't need to specify it).

#### Get single Contact
- Method: `GET`
- Endpoint: `/api/v1/contacts/<id>`

**Parameter**

| Parameter |         Type         | Optional | Description                                           |
|:----------|:--------------------:|:--------:|:------------------------------------------------------|
| values    | [List](#list) of str |   Yes    | List of attributes of a contact you want to retrieve. |

You can retrieve a single contact using this endpoint. The `values` parameter can be used to retrieve only
the given attributes (`id` is always returned, you don't need to specify it).

#### Create Contact
- Method: `POST`
- Endpoint: `/api/v1/contacts`

**Example body**
```json
{
  "name": "Max Mustermann",
  "mail": "max@example.com",
  "linked_host_notifications": [
    1, 2
  ],
  "linked_host_notification_period": 1,
  "comment": "Max Mustermann from example corp."
}
```

**Parameter**

| Parameter                         |                     Type                      | Optional | Description                                                                 |
|:----------------------------------|:---------------------------------------------:|:--------:|-----------------------------------------------------------------------------|
| name                              |                      str                      |    No    | Name of the contact. Must be unique                                         |
| mail                              |                      str                      |   Yes    | Mail address                                                                |
| linked_host_notifications         |             [List](#list) of int              |   Yes    | List of ids of commands you want to execute if a host state change occurs   |
| linked_host_notification_period   |                      int                      |   Yes    | TimePeriod in which host notifications should be sent                       |
| linked_metric_notifications       |             [List](#list) of int              |   Yes    | List of ids of commands you want to execute if a metric state change occurs |
| linked_metric_notification_period |                      int                      |   Yes    | TimePeriod in which metric notifications should be sent                     |
| variables                         | [Dictionary](#dictionary) of<br/> `str : str` |   Yes    | Additional variables, they can be accessed by the notification commands     |
| comment                           |                      str                      |   Yes    | Comment                                                                     |

If the contact was created successfully, the `id` is returned in the `data` field:
```json
{
  "success": true,
  "message": "Object was created",
  "data": 22
}
```

#### Modify Contact
- Method: `PUT`
- Endpoint: `/api/v1/contacts/<id>`

**Example body**
```json
{
  "name": "Maria Mustermann",
  "mail": "maria@example.com",
  "variables": {
    "matrix": "@maria:matrix.example.com",
    "mail2": "maria2@example.com"
  }
}
```

**Parameter**

| Parameter                         |                     Type                      | Description                                                                 |
|:----------------------------------|:---------------------------------------------:|:----------------------------------------------------------------------------|
| name                              |                      str                      | Name of the contact. Must be unique                                         |
| mail                              |                      str                      | Mail address                                                                |
| linked_host_notifications         |             [List](#list) of int              | List of ids of commands you want to execute if a host state change occurs   |
| linked_host_notification_period   |                      int                      | TimePeriod in which host notifications should be sent                       |
| linked_metric_notifications       |             [List](#list) of int              | List of ids of commands you want to execute if a metric state change occurs |
| linked_metric_notification_period |                      int                      | TimePeriod in which metric notifications should be sent                     |
| variables                         | [Dictionary](#dictionary) of<br/> `str : str` | Additional variables, they can be accessed by the notification commands     |
| comment                           |                      str                      | Comment                                                                     |

You can modify each of the above parameters. Include it in the body with its new value.

#### Delete Contact
- Method: `DELETE`
- Endpoint: `/api/v1/contacts/<id>`

### ContactGroups
Contact groups represent a list of contacts.

```json
{
  "id": 3,
  "name": "Mustermann notifications",
  "comment": "Just a notification group",
  "linked_contacts": [
    1, 3, 5, 7
  ]
}
```

#### Get ContactGroups
- Method: `GET`
- Endpoint: `/api/v1/contactgroups`

**Parameter**

| Parameter |         Type         | Optional | Description                                                  |
|:----------|:--------------------:|:--------:|:-------------------------------------------------------------|
| filter    | [List](#list) of int |   Yes    | List of ids of contact groups you want to retrieve           |
| values    | [List](#list) of str |   Yes    | List of attributes of a contact groups you want to retrieve. |

Contact groups can be gathered using this endpoint. You can use the `filter` parameter to limit the results to a list of given ids.
The `values` parameter can be used to retrieve only the given attributes (`id` is always returned, you don't need to specify it).

#### Get single ContactGroup
- Method: `GET`
- Endpoint: `/api/v1/contactgroups/<id>`

**Parameter**

| Parameter |         Type         | Optional | Description                                                  |
|:----------|:--------------------:|:--------:|:-------------------------------------------------------------|
| values    | [List](#list) of str |   Yes    | List of attributes of a contact groups you want to retrieve. |

You can retrieve a single contact group using this endpoint. The `values` parameter can be used to retrieve only
the given attributes (`id` is always returned, you don't need to specify it).

#### Create ContactGroup
- Method: `POST`
- Endpoint: `/api/v1/contactgroups`

**Example body**
```json
{
  "name": "Admin notification group",
  "comment": "Admins: Max, Maria",
  "linked_contacts": [
    1, 3
  ]
}
```

**Parameter**

| Parameter                         |         Type         | Optional | Description                               |
|:----------------------------------|:--------------------:|:--------:|-------------------------------------------|
| name                              |         str          |    No    | Name of the contact group. Must be unique |
| linked_contacts                   | [List](#list) of int |   Yes    | Linked contacts of the group              |
| comment                           |         str          |   Yes    | Comment                                   |

If the contact group was created successfully, the `id` is returned in the `data` field:
```json
{
  "success": true,
  "message": "Object was created",
  "data": 22
}
```

#### Modify ContactGroup
- Method: `PUT`
- Endpoint: `/api/v1/contactgroups/<id>`

**Example body**
```json
{
  "comment": "",
  "linked_contacts": [
    1, 3, 7
  ]
}
```

**Parameter**

| Parameter        |         Type         | Description                              |
|:-----------------|:--------------------:|:-----------------------------------------|
| name             |         str          | Name of the contactgroup. Must be unique |
| linked_contacts  | [List](#list) of int | List of contacts                         |
| comment          |         str          | Comment                                  |

You can modify each of the above parameters. Include it in the body with its new value.

#### Delete ContactGroup
- Method: `DELETE`
- Endpoint: `/api/v1/contactgroups/<id>`

### GlobalVariables
A global variable is used as a variable in commands. You may save things like your default ssh port or ssh user
in a variable, so you don't have to apply them on each host or metric individually.

```json
{
  "id": 62,
  "key": "ssh_user",
  "value": "q",
  "comment": "default ssh user"
}
```

#### Get GlobalVariables
- Method: `GET`
- Endpoint: `/api/v1/globalvariables`

**Parameter**

| Parameter |         Type         | Optional | Description                                                   |
|:----------|:--------------------:|:--------:|:--------------------------------------------------------------|
| filter    | [List](#list) of int |   Yes    | List of ids of global variables you want to retrieve          |
| values    | [List](#list) of str |   Yes    | List of attributes of a global variable you want to retrieve. |

Global variables can be gathered using this endpoint. You can use the `filter` parameter to limit the results to a list of given ids.
The `values` parameter can be used to retrieve only the given attributes (`id` is always returned, you don't need to specify it).

#### Get single GlobalVariable
- Method: `GET`
- Endpoint: `/api/v1/globalvariables/<id>`

**Parameter**

| Parameter |         Type         | Optional | Description                                                   |
|:----------|:--------------------:|:--------:|:--------------------------------------------------------------|
| values    | [List](#list) of str |   Yes    | List of attributes of a global variable you want to retrieve. |

You can retrieve a single global variable using this endpoint. The `values` parameter can be used to retrieve only
the given attributes (`id` is always returned, you don't need to specify it).


#### Create GlobalVariable
- Method: `POST`
- Endpoint: `/api/v1/globalvariables`

**Example body**
```json
{
  "key": "ssh_port",
  "value": "2222",
  "comment": "default ssh port"
}
```

**Parameter**

| Parameter | Type | Optional | Description |
|:----------|:----:|:--------:|-------------|
| key       | str  |    No    | Key         |
| value     | str  |    No    | Value       |
| comment   | str  |   Yes    | Comment     |

If the global variable was created successfully, the `id` is returned in the `data` field:
```json
{
  "success": true,
  "message": "Object was created",
  "data": 22
}
```

#### Modify GlobalVariable
- Method: `PUT`
- Endpoint: `/api/v1/globalvariables/<id>`

**Example body**
```json
{
  "value": "q-user"
}
```

**Parameter**

| Parameter | Type | Description |
|:----------|:----:|:------------|
| key       | str  | Key         |
| value     | str  | Value       |
| comment   | str  | Comment     |

You can modify each of the above parameters. Include it in the body with its new value.

#### Delete GlobalVariable
- Method: `DELETE`
- Endpoint: `/api/v1/globalvariables/<id>`

### Hosts
Hosts can represent actual hosts, but they don't have to. Think of them as a logical unit to aggregate multiple metrics.
```json
{
  "id": 2
  
}
```

#### Get Hosts
- Method: `GET`
- Endpoint: `/api/v1/hosts`


#### Delete Host
- Method: `DELETE`
- Endpoint: `/api/v1/hosts/<id>`

### HostTemplates

#### Delete HostTemplate
- Method: `DELETE`
- Endpoint: `/api/v1/hosttemplates/<id>`
### Metrics

#### Delete Metric
- Method: `DELETE`
- Endpoint: `/api/v1/metrics/<id>`
### MetricTemplates

#### Delete MetricTemplate
- Method: `DELETE`
- Endpoint: `/api/v1/metrictemplates/<id>`
### Proxies

#### Delete Proxy
- Method: `DELETE`
- Endpoint: `/api/v1/proxies/<id>`
### TimePeriods

#### Delete TimePeriod
- Method: `DELETE`
- Endpoint: `/api/v1/timeperiods/<id>`

## Parameter definition

### List
Sometimes it is necessary to provide lists as URL encoded parameter. 
This is only necessary when a `GET` Request is made. There are two ways:

```
/api/v1/endpoint?value=1&value=2&value=3
```

or 

```
/api/v1/endpoint?value=1,2,3
```

### Dictionary
Dictionary are key-value-pairs. The keys must be unique. Mostly used for variables.

```json
{
  "key": "value"
}
```
