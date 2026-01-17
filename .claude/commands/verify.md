# Verify

Build the project and run tests compatible with local TinkerPop setup.

## Instructions

Run the following steps in order:

1. **Check for warnings** - Run `cargo check --all-targets` and report any warnings
2. **Build the project** - Run `cargo build`
3. **Run unit tests** - Run `cargo test --lib`
4. **Run integration tests** - Run the integration tests that work with TinkerPop on port 8182:

```bash
cargo test --test integration_client --test integration_traversal -- \
  --skip credentials \
  --skip test_vertex_query \
  --skip test_edge_query \
  --skip test_add_e
```

## Notes

Tests that are skipped:
- `credentials` - Needs SSL server on port 8183
- `test_vertex_query`, `test_edge_query`, `test_add_e` - Need "modern" graph data
- `custom_vertex_ids` test file - Needs JanusGraph on port 8184
- `*_v2` test files - GraphSON V2 serialization issues

Report a summary of results at the end.
