# Changelog

## 0.1.0 — 2026-11-12

Initial standalone extraction from Runalith's `runalith-linux-quota` crate at source revision `29c24cfa2b6ead5704d26cf2746bc384f2938b80`.

- Linux ext4/XFS project quota controller.
- Exact directory device/inode fencing.
- Bounded systemd socket-activated Unix helper and worker UID check.
- Root-owned SQLite project identity/limit registry.
- Parser, IPC, registry, and concurrency/recovery unit tests.

The destructive live-mount quota integration binary and Runalith service deployment configuration were excluded.
