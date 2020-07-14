## Example

The following example assumes a Rust server and a TypeScript client. Webwire
is by no means limited to those two but those languages show the potential of
webwire best.

Given the following IDL file:

```webwire
webwire 1.0;

struct HelloRequest {
    name: String,
}

struct HelloResponse {
    message: String,
}

service Hello {
    hello: HelloRequest -> HelloResponse
}
```

The server and client files can be generated using the code generator:

```shell
$ webwire generate rust server api/hello.ww server/src/api.rs
$ webwire generate ts client api/hello.ww client/src/api.ts
```

A Rust server implementation for the given code would look like this:

```rust
use std::net::SocketAddr;
use webwire::{Context, Request, Response}
use webwire::hyper::Server;

mod api;
use api::v1::{Hello, HelloRequest, HelloResponse}; // this is the generated code

struct HelloService {}

impl Hello for HelloService {
    fn hello(&self, ctx: &Context, request: &HelloRequest) -> HelloResponse {
        HelloResponse {
            message: format!("Hello {}!", request.name)
        }
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error> {
    let addr = SocketAddr::from(([127, 0, 0, 1], 8000));
    let service = HelloService {};
    let server = webwire::Server::bind(addr).serve(service);
    server.await
}
```

A TypeScript client using the generated code would look like that:

```js
import { Client } from 'api/v1' // this is the generated code

client = new Client('http://localhost:8000/')
const response = await client.hello({ name: 'World' })
assert(response.message === 'Hello World!')
```
