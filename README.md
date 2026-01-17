## A word of warning

This branch contains numerous changes which are exclusively geared towards making the library useful for
the gremlin UI [Peltzer](https://github.com/flq/peltzer).

* Removal of async-std support
* Update of dependencies
* Added connectionconfig extension to be able to replace the hardcode path
* Some adaptations to Graphson V2 deserialization to be able to read the Cosmos DB responses.

## gremlin-client

A Rust client for Apache TinkerPop™.

### Installation

Install from [crates.io](https://crates.io/)

```toml
[dependencies]
gremlin-client = "0.8"
```

with [tokio](https://tokio.rs/) async support

```toml
[dependencies]
gremlin-client = { git = "https://github.com/flq/gremlin-rs", branch = "peltzer-support", features = ["tokio-runtime"] }
```

### Examples

#### Basic usage

Execute a simple Gremlin query with an id and collect the results

**Synchronous**

```rust
use gremlin_client::{GremlinClient, Vertex};

fn main() -> Result<(), Box<std::error::Error>> {
    let client = GremlinClient::connect("localhost")?;

    let results = client
        .execute("g.V(param)", &[("param", &1)])?
        .filter_map(Result::ok)
        .map(|f| f.take::<Vertex>())
        .collect::<Result<Vec<Vertex>, _>>()?;

    println!("{:?}", results);

    Ok(())
}
```


**Asynchronous**

With [tokio](https://tokio.rs/)

activate the feature `tokio-runtime`

`gremlin-client = { version = "*", features = ["tokio-runtime"] }`

```rust

use gremlin_client::{aio::GremlinClient, Vertex};
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() -> Result<(), Box<std::error::Error>> {

    let client = GremlinClient::connect("localhost").await?;
    let results = client.execute("g.V(param)", &[("param", &1)]).await?
        .filter_map(Result::ok)
        .map(|f| f.take::<Vertex>())
        .collect::<Result<Vec<Vertex>, _>>().await?;
    println!("{:?}", results);
    Ok(())

}
```

#### Traversal example Rust GLV

Create a remote traversal with the provided `GremlinClient` and build a traversal
using Rust language.

**Synchronous**

```rust
 use gremlin_client::{GremlinClient, Vertex, process::traversal::traversal};

 fn main() -> Result<(), Box<std::error::Error>> {
    let client = GremlinClient::connect("localhost")?;

    let g = traversal().with_remote(client);

    let results = g.v(()).has_label("person").has(("name","Jon")).to_list()?;

    println!("{:?}", results);
    Ok(())
}
```


**Asynchronous**

With [tokio](https://tokio.rs/)

```rust
use gremlin_client::{aio::GremlinClient, Vertex, process::traversal::traversal};
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() -> Result<(), Box<std::error::Error>> {

    let client = GremlinClient::connect("localhost").await?;

    let g = traversal().with_remote_async(client);

    let results = g.v(()).has_label("person").has(("name","Jon")).to_list().await?;

    println!("{:?}", results);
    Ok(())
}
```


### Additional Features

#### `derive` feature

By including the `derive` feature in your Cargo.toml

```
[dependencies]
gremlin-client = { version = "*", features = ["derive"] }
```

two derive macros are available

- FromGMap
- FromGValue

which you can use to derive the mapping from GMap and GValue (only Map currently) into structs.


with `GValue`

```rust
use gremlin_client::derive::{FromGMap, FromGValue};
use gremlin_client::process::traversal::traversal;
use gremlin_client::GremlinClient;
use std::convert::TryFrom;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = GremlinClient::connect("localhost")?;

    #[derive(Debug, PartialEq, FromGValue, FromGMap)]
    struct Person {
        name: String,
    }

    let results = client
        .execute("g.V(param).valueMap()", &[("param", &1)])?
        .filter_map(Result::ok)
        .map(|f| Person::try_from(f))
        .collect::<Result<Vec<Person>, _>>()?;

    println!("Person {:?}", results[0]);
    Ok(())
}

```

with `GMap`

```rust
use gremlin_client::derive::FromGMap;
use gremlin_client::process::traversal::traversal;
use gremlin_client::GremlinClient;
use std::convert::TryFrom;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = GremlinClient::connect("localhost")?;

    #[derive(Debug, PartialEq, FromGMap)]
    struct Person {
        name: String,
    }

    let g = traversal().with_remote(client);

    let results = g
        .v(1)
        .value_map(())
        .iter()?
        .filter_map(Result::ok)
        .map(Person::try_from)
        .collect::<Result<Vec<Person>, _>>()?;

    println!("Person {:?}", results[0]);

    Ok(())
}
```


### Development


#### Compiling

```
git clone https://github.com/wolf4ood/gremlin-rs.git
cd gremlin-rs
cargo build
```


#### Running Tests

Some tests run against a running instance of Gremlin Server with a sample in-memory graph installed.

You can use docker-compose to start an instance for testing. Use the env variable `GREMLIN_SERVER`
in order to specify the version of the Gremlin Server

```
cd docker-compose
export GREMLIN_SERVER=3.4.4
docker-compose up -d
cd ..
cargo test --all-features
```
