# Websocket protocol

All messages are encoded as a space separated list. The first part of a
message is always a numeric message type followed by message specific fields.

## Message id

Most messages require a `message_id` to be sent. Both server and client
must implement it as a counter without gaps starting at 1. It is used to
match match request and response messages together and provide
reliable messaging even in case of a unexpected connection timeout.

## Message types

The following message types are supported:

| Code | Message type   |
| ---- | -------------- |
|    0 | Heartbeat      |
|    1 | Notification   |
|    2 | Request        |
|    3 | Response       |
|    4 | Error response |
|   -1 | Disconnect     |

### Heartbeat

The heartbeat is used by the client and server to acknowledge messages
and keep the connection alive if there has been no traffic for a given
amount of time. For transports that do not keep the connection open for
an unlimited amount of time this is used for polling.

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `0` |
| `last_message_id` | Integer | The last message id of the remote side that has been received. |

Example:

    0 42

### Notification

Send a notification to the remote side and do not expect a response. This
message type is especially useful to implement broadcasts from the server
to the client where no response is expected. Implementations of webwire
MUST NOT expect a response to a notification.

Fields:

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `1` |
| `message_id` | Integer | The `id` of the message. |
| `method` | FQMN | The fully qualified method name of the method to be called. |
| `data` | Binary | The data field captures the rest of the frame and is treated as binary. |

Example:

    1 43 example.hello peter

### Request

Send a request to the remote side and expect a response.

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `2` |
| `message_id` | Integer | The `id` of the message. |
| `method` | FQMN | The fully qualified method name of the method to be called. |
| `data` | Binary | The data field captures the rest of the frame and is treated as binary. |

Example:

    2 44 example.get_version

### Response

This message is sent in response to a request if the remote method could be called
successfully.

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `3` |
| `message_id` | Integer | The `id` of the message. |
| `request_message_id` | Integer | The `id` of the message that started the request. |
| `data` | Binary | The data field captures the rest of the frame and is treated as binary. |

Example:

    3 7 44 "1.4.9"

### Error response

This message is sent in response to a request if the remote side encountered
an error while processing the request. Please note that this message type
MUST NOT be used to encode application level errors. This is only meant to be
used for errors which are outside of the application scope. e.g. parser errors,
data validation, internal server errors, etc.

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `4` |
| `message_id` | Integer | The `id` of the message. |
| `request_message_id` | Integer | The `id` of the message that started the request. |
| `error_code` | Binary | The data field captures the rest of the frame and is treated as binary. |
| `error_message` | String | Optional error message. |

Example:

    4 7 44 MethodNotFound

### Disconnect

Terminate the current connection. The remote side should respond with a `-1`
and close the connection.

| Field | Type | Description |
| --- | --- | --- |
| `message_type` | Integer | Constant `-1` |

Example:

    -1

## Example communication

Assuming the connection has been up for a while and the server has now
reached message id 117. The client has sent 5 messages so far and the
next message id is 5.

1. Heartbeat (client to server):
    ```
    0 117
    ```

2. Heartbeat (server to client):

    ```
    0 4
    ```

3. Notification (client to server):

    ```
    1 5 Player.ready true
    ```

4. Request (client to server):

    ```
    2 6 get_time
    ```

5. Response (server to client):

    ```
    3 118 6 1342106240
    ```
