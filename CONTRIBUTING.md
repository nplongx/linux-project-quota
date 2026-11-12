# Contributing

## Kernel-facing code

Changes to `quotactl`, `FS_IOC_FSGETXATTR`, `FS_IOC_FSSETXATTR`, `repr(C)` ABI structs, path validation, helper IPC, peer credentials, registry transitions, or project-ID lifecycle require focused negative tests and independent review.

Never run the upstream destructive integration test against a production or shared mount. A real enforcement test requires a dedicated disposable ext4/XFS project-quota mount, a separate registry, reserved test project-ID range, and explicit operator approval. Do not add it to default CI.

## Compatibility

Treat the helper's closed JSON operations and environment configuration as service contracts. State any incompatibility explicitly. Project IDs remain permanently reserved after release; never delete/recycle registry rows to recover space.

## Required checks

Requirements: Linux and Rust 1.85 or newer.

```bash
cargo fmt --all -- --check
cargo test --all-targets --locked
cargo clippy --all-targets --locked -- -D warnings
cargo doc --no-deps --locked
```

CI compiles/tests Ubuntu 24.04 with Rust 1.85.0 and stable. This does not validate real kernel quota enforcement.
