# HTTP

## Path

Requests and notifications are sent relative to a given `base_url` with
the fully qualified method name appended:

Example:

    base_url: /ww
    fqmn: Example.hello

    path: /ww/Example.hello

## Headers

The following headers are respected by webwire:

| Header | Description |
| --- | --- |
| X-Webwire | This must be either `Notification` or `Request` |

## Notification

The server is expected to answer with `204 OK` or `400 Bad Request`.
In case of an internal server error the status code `500 Internal
Server Error` is identical to a 400 Request with the `InternalError`
payload.

HTTP request:

    POST /ww/example HTTP/1.1
    Host: example-api.webwire.dev
    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
    X-Webwire: Notification

    "world"

HTTP response - Ok:

    HTTP/1.1 204

HTTP response - Error:

    HTTP/1.1 400

    "MethodNotFound"

## Request

The server is expected to answer with `200 OK` or `400 Bad Request`.


HTTP request:

    POST /ww/Example.get_version? HTTP/1.1
    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
    X-Webwire: Request

HTTP response - Ok:

    HTTP/1.1 200 OK

    "1.12.3"

HTTP response - Error:

    HTTP/1.1 400

    "ValidationError"
