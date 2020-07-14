# Transport Protocol

The webwire transport protocol exists in two variants.

1. Stateless unidirectional (e.g. HTTP)
2. Stateful bidirectional (e.g. WebSocket connections)

Both variants share some common terms and definitions which are
explained on this page.

## Framing

It is assumed that all protocols being used to transfer webwire messages
already implement framing. Thus the webwire protocol does not encode its
own frame length but leaves that to the underlying protocol.

If the underlying protocol does not implement framing a framing layer
must be implemented first. This is to be defined in the transport layer
specific documentation.

## Fully qualified method names (FQMN)

Fully qualified method names are dot (`.`) separated identifiers. All
identifiers must be ASCII only, start with a letter followed by any number
of alphanumeric characters. The last two parts are called the service name
and the method name. Any leading part is called the namespace.

Examples:

| FQMN | Namespace | Service | Method |
| --- | --- | --- | --- |
| `Example.hello` | - | Example | hello |
| `foo.Example.hello` | foo | Example | hello |
| `foo.bar.Example.hello` | foo.bar | Example | hello |

Invalid examples:

| FQMN | Why is it invalid? |
| --- | --- |
| `hello` | Only one part. Service and method name are mandatory |
| `hey.123test` | The method name must not start with a number |
| `123hey.test` | The service name must not start with a number |
| `123ns.hey.test` | The namespace must not start with a number |
| `Über.awesome` | Non ASCII character |

## Error codes

| Code | Description |
| --- | --- |
| ServiceNotFound | The requested service does not exist. |
| MethodNotFound | The requested method does not exist. |
| ValidationError | The data could not be deserialized and validated. |
| InternalError | Something bad happened while processing the request. |
