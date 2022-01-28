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

## Checks

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

### Get Checks
- Method: `GET`
- Endpoint: `/api/v1/checks`

**Parameter**

| Parameter |           Type           | Optional | Description                                         |
|:----------|:------------------------:|:--------:|:----------------------------------------------------|
| filter    |   [List](#list) of int   |   Yes    | List of ids of checks you want to retrieve          |
| values    | [List](#list) of Strings |   Yes    | List of attributes of a check you want to retrieve. |

Checks can be gathered using this endpoint. You can use the `filter` parameter to limit the results to a list of given ids.
The `values` parameter can be used to retrieve only the given attributes (`id` is always returned, you don't need to specify it).

### Get single Check
- Method: `GET`
- Endpoint: `/api/v1/checks/<id>`

**Parameter**

| Parameter |           Type           | Optional | Description                                         |
|:----------|:------------------------:|:--------:|:----------------------------------------------------|
| values    | [List](#list) of Strings |   Yes    | List of attributes of a check you want to retrieve. |

You can retrieve a single check using this endpoint. The `values` parameter can be used to retrieve only
the given attributes (`id` is always returned, you don't need to specify it).

### Create Check
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

| Parameter |  Type  | Optional | Description                       |
|:----------|:------:|:--------:|:----------------------------------|
| name      | String |    No    | Name of the Check. Must be unique |
| cmd       | String |   Yes    | Commandline to execute            |
| comment   | String |   Yes    | Comment                           |

If the check was created successfully, the `id` is returned in the `data` field:
```json
{
  "success": true,
  "message": "Object was created",
  "data": 22
}
```

### Modify Check
- Method: `PUT`
- Endpoint: `/api/v1/checks/<id>`

**Example body**
```json
{
  "name": "Linux uptime check"
}
```

**Parameter**

| Parameter |  Type  | Description            |
|:----------|:------:|:-----------------------|
| name      | String | Name of the check      |
| cmd       | String | Commandline to execute |
| comment   | String | Comment                |

You can modify each of the above parameters. Include it in the body with its new value.

### Delete Check
- Method: `DELETE`
- Endpoint: `/api/v1/checks/<id>`




## Parameter

### List
Sometimes it is necessary to provide lists as URL encoded parameter. There are two ways:

```
/api/v1/endpoint?value=1&value=2&value=3
```

or 

```
/api/v1/endpoint?value=1,2,3
```


