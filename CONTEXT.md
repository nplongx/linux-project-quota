# Linux Project Quota Language

**Project quota**
A filesystem-level quota assigned to an inode tree/project ID. It limits charged file data and inode count. It is distinct from cgroup CPU/memory controls.

**Project ID**
A filesystem project identifier. In this crate IDs below `10_000` are reserved. The helper records permanent path/device/inode bindings and does not reuse released IDs.

**Project inheritance**
The filesystem `FS_XFLAG_PROJINHERIT` flag. Descendants created beneath an assigned directory inherit its project ID according to filesystem semantics.

**Hard byte limit**
The project quota's hard limit on charged file data. Linux quota ABI expresses byte blocks in 1 KiB units; requests are rounded up to that unit.

**Hard inode limit**
The project quota's hard limit on charged inodes.

**Quota readback**
A fresh kernel query after a mutation. A successful syscall alone is not treated as proof that requested limits were retained.

**Exact workspace identity**
The tuple of canonical path, device number, and inode number. The privileged helper binds each project ID to this tuple and rejects identity drift.

**Quota registry**
A root-owned SQLite ledger for project ID, workspace identity, reserved limits, and monotonic lifecycle state. It is append-only: records and IDs are not deleted/recycled.

**Socket-activated helper**
A privileged service receiving its Unix listener from systemd. It checks peer UID with `SO_PEERCRED`, parses a bounded request frame, applies a closed operation set, and returns a bounded response.
