# Project: gremlin-rs

A Rust client for Apache TinkerPop/Gremlin graph databases.

## Project Structure

```
gremlin-rs/
├── gremlin-client/     # Main client library
│   ├── src/
│   │   ├── aio/        # Async client implementation
│   │   ├── io/         # Serialization (GraphSON V2/V3)
│   │   ├── process/    # Traversal API
│   │   └── structure/  # Graph types (Vertex, Edge, etc.)
│   └── tests/          # Integration tests
│       └── common/     # Shared test utilities (mod.rs)
├── gremlin-derive/     # Derive macros
├── gremlin-cli/        # CLI tool
└── gremlin-tutorial/   # Examples
```

## Build Commands

```bash
cargo build                    # Build all
cargo check --all-targets      # Check for warnings
cargo test --lib               # Unit tests only (needs TinkerPop on 8182)
```

## Test Infrastructure

Tests expect multiple Gremlin servers:

| Port | Server | Required For |
|------|--------|--------------|
| 8182 | TinkerPop (no auth) | Most tests |
| 8183 | TinkerPop (SSL + auth) | `*credentials*` tests |
| 8184 | JanusGraph | `custom_vertex_ids` tests |

### Running Tests with TinkerPop Only (port 8182)

```bash
# Unit + integration tests (skipping what needs 8183/8184 or modern graph data)
cargo test --lib --test integration_client --test integration_traversal -- \
  --skip credentials \
  --skip test_vertex_query \
  --skip test_edge_query \
  --skip test_add_e
```

### Tests Requiring "Modern" Graph Data

These tests need the TinkerPop modern toy graph loaded (`TinkerFactory.createModern()`):
- `test_vertex_query` - looks for vertex with name "marko"
- `test_edge_query` - looks for edge labeled "knows"
- `test_add_e` - looks for vertices "marko", "jon" and "created" edges

### GraphSON V2 vs V3

- Default serializer is GraphSON V3
- V2 tests (`*_v2.rs` files) may have compatibility issues with some servers
- Serializer selection: `ConnectionOptions::builder().serializer(GraphSON::V2)`

## Key Types

- `GremlinClient` - Main client (sync in `src/client.rs`, async in `src/aio/client.rs`)
- `GValue` - Universal value type for Gremlin data
- `Vertex`, `Edge`, `VertexProperty`, `Property` - Graph elements
- `GraphTraversal` - Fluent traversal API

## Connection Pools

- Sync: uses `r2d2` (`src/pool.rs`)
- Async: uses `mobc` (`src/aio/pool.rs`)

## Feature Flags

```toml
async_gremlin          # Enable async support
tokio-runtime          # Use Tokio runtime
async-std-runtime      # Use async-std runtime
derive                 # Enable derive macros
```

## Common Patterns

### Sync Client
```rust
let client = GremlinClient::connect(("localhost", 8182))?;
let results = client.execute("g.V().limit(1)", &[])?;
```

### Async Client
```rust
let client = GremlinClient::connect(("localhost", 8182)).await?;
let results = client.execute("g.V().limit(1)", &[]).await?;
```

### Traversal API
```rust
let g = traversal().with_remote(client);
let vertices = g.v(()).has_label("person").to_list()?;
```

## Custom Slash Commands

- `/verify` - Build and run tests compatible with local TinkerPop setup
