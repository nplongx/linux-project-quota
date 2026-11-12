# Linux Project Quota

[![CI](https://github.com/nplongx/linux-project-quota/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/nplongx/linux-project-quota/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A narrow Rust interface for Linux filesystem project quotas on ext4 and XFS. Includes direct quota-controller APIs and a separately privileged, systemd socket-activated helper with a bounded Unix-socket protocol and SQLite project registry.

> **Status:** `0.1.0`, initial standalone extraction. Linux-only. This crate changes kernel-enforced filesystem quota state when mutating methods are called. It is not a generic storage manager, cgroup controller, sandbox, or production security certification.

## Capabilities

- Detect an ext4/XFS mount with active `prjquota` or `pquota` options.
- Bind a controller to one canonical mounted filesystem root.
- Assign a project ID and inheritance flag to a new, empty workspace directory.
- Set and read back byte/inode hard limits using kernel project-quota APIs.
- Report kernel-observed charged byte and inode usage.
- Remove limits only when charged usage is zero.
- Use exact device/inode identity when assigning, setting, and verifying through the privileged helper.
- Provide a bounded, length-prefixed JSON protocol over a systemd-activated Unix socket.
- Restrict helper requests to one configured worker UID, one configured root, a private workspace parent, a durable append-only project-ID registry, and operator-configured maximum limits.

## Platform and prerequisites

- Linux with project quota support.
- ext4 or XFS mounted with `prjquota` or `pquota` enabled.
- The filesystem mount source must be a block-device path under `/dev/`.
- The process performing kernel mutations needs the privileges required by the selected filesystem/kernel configuration.
- The systemd helper requires exactly one socket-activated descriptor and uses `SO_PEERCRED` to check the connecting UID.

Other filesystems and non-Linux targets are unsupported. There is no userspace accounting or unlimited fallback.

## Library API

```toml
[dependencies]
linux-project-quota = { git = "https://github.com/nplongx/linux-project-quota" }
```

```rust
use linux_project_quota::{ProjectQuotaController, ProjectQuotaLimits};

fn inspect(root: &str) -> Result<(), Box<dyn std::error::Error>> {
    let controller = ProjectQuotaController::open(root)?;
    let usage = controller.status(10_000)?;
    println!("{usage:?}");
    Ok(())
}
```

The example is read-only after controller initialization. `open` probes the quota subsystem. The following operations mutate filesystem quota state and require a deliberately configured quota mount and workspace:

- `assign_project_id` / `assign_project_id_exact`
- `set_hard_limits` / `set_hard_limits_exact`
- `clear_empty_project`

Project IDs below `10_000` are reserved. The helper further validates IDs against its configured worker policy. Byte hard limits are rounded up to the kernel's 1 KiB quota block unit.

## Privileged helper

The `project-quota-helper` binary accepts these required environment settings:

| Variable | Meaning |
|---|---|
| `PROJECT_QUOTA_HELPER_ROOT` | Canonical root on the dedicated project-quota mount. |
| `PROJECT_QUOTA_HELPER_REGISTRY` | SQLite registry path below a root-owned, mode-`0700` parent directory. |
| `PROJECT_QUOTA_HELPER_WORKER_UID` | Only UID allowed to call the helper. |
| `PROJECT_QUOTA_HELPER_MAX_BYTES` | Maximum byte limit allowed for one project. |
| `PROJECT_QUOTA_HELPER_MAX_INODES` | Maximum inode limit allowed for one project. |

The helper expects one systemd socket-activated Unix listener at file descriptor 3 (`LISTEN_PID`, `LISTEN_FDS=1`). Configure socket ownership/group/mode in the host's systemd socket unit. The helper independently validates peer UID. Do not expose the socket to general local users.

Expected worker workspace layout:

```text
<quota-root>/
└── .project-quota-workspaces/       # worker-owned, mode 0700
    └── execution-<project-id>/       # empty at initial project-ID assignment
```

The root and workspace parent must satisfy helper ownership/permission checks. Each project ID is permanently bound to the exact workspace path, device, and inode in a root-owned append-only registry. A released project ID is never reused. Limits become immutable after first reservation.

This repository intentionally does not install systemd units, mount filesystems, change `/etc/fstab`, enable quotas, allocate project IDs in a live filesystem, or start the privileged helper. Operators must review and configure those host-specific steps separately.

## Safety boundaries

- The helper is privileged code. Its socket ACL, allowed UID, root path, registry directory, and maximum byte/inode limits are security-critical configuration.
- The helper API is not a general-purpose `quotactl` proxy: it accepts a small closed set of operations and binds each workspace to an immutable registry identity.
- Filesystem path validation and device/inode checks reduce path substitution risk. They are not a formal guarantee against every hostile same-UID concurrent filesystem race; keep the workspace parent private and do not share it with untrusted processes.
- The library does not create users, groups, mounts, quota enablement, systemd units, or a worker sandbox.
- The helper trusts the administrator's configured worker UID. Compromise of that UID remains inside the helper's permitted workspace/quota boundary.
- The included integration binary from upstream is deliberately not shipped: it creates files until quota exhaustion and deletes test workspaces. No destructive live-mount test is part of the default test suite.

## Development

Requirements: Rust 1.85 or newer; Ubuntu/Linux CI.

```bash
cargo fmt --all -- --check
cargo test --all-targets --locked
cargo clippy --all-targets --locked -- -D warnings
cargo doc --no-deps --locked
```

The tests exercise parsers, helper request framing, SQLite registry invariants, inode binding, and recovery using temporary directories/connections. They do not prove kernel quota enforcement on a real ext4/XFS mount. No live quota operation was executed during this extraction.

## Documentation

- [Documentation index](docs/README.md)
- [Domain glossary](CONTEXT.md)
- [Architecture and security boundaries](docs/architecture.md)
- [Source lineage](docs/source-lineage.md)
- [Architecture decisions](docs/adr/)
- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [Changelog](CHANGELOG.md)

## License

MIT. See [LICENSE](LICENSE).
