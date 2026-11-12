# Security Policy

## Scope

This package performs privileged Linux filesystem project-quota operations. Its helper may run with elevated privileges and changes kernel-enforced quota state. It is security-sensitive infrastructure, not a sandbox certification.

## Deployment requirements

- Use a dedicated ext4/XFS project-quota mount; do not point the helper at `/` or a general-purpose shared mount.
- Protect the systemd socket with restrictive owner/group/mode and allow only the configured worker identity.
- Set a dedicated worker UID, a canonical quota root, a root-owned mode-`0700` registry directory, and strict maximum byte/inode limits.
- Keep the quota root and `.project-quota-workspaces` parent private to the worker/helper boundary.
- Protect and back up the registry; reconcile it with kernel state before any restore.
- Do not reuse released project IDs.
- Do not run destructive quota integration on production/shared filesystems.

## Known limits

Path and device/inode checks reduce substitution risk but are not a formal proof against every hostile same-UID concurrent filesystem race. The configured worker UID is trusted within the helper's permitted path/limit policy. Compromise of the helper, its service manager, the worker UID, or registry owner can affect quota integrity.

No systemd unit, mount setup, quota enablement, or privileged installation is shipped. No production deployment or live quota enforcement test is claimed by this repository.

## Reporting

Use GitHub private vulnerability reporting when available. Do not publish socket details, privileged service configuration, production registry contents, or filesystem metadata in a public issue. Include affected version and a reproducible test case.
