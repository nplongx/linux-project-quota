# ADR-0002: Keep privileged quota operations behind a narrow helper

- **Status:** Accepted
- **Date:** 2026-11-12

## Context

Project ID assignment and hard-limit updates require privileged filesystem operations. Giving the worker direct authority over arbitrary project IDs or mount paths would exceed its workspace scope.

## Decision

The optional helper accepts one configured worker UID, a fixed root, maximum limits, and a closed operation set. Every workspace operation validates the exact expected path, owner/mode, device, inode, and durable project registry binding. Project IDs are never recycled.

## Consequences

- The host administrator must configure the systemd socket ACL, helper environment, root, registry, and quota maxima.
- The worker cannot choose an arbitrary mount path or arbitrary project ID binding through the helper API.
- The helper remains privileged attack surface and requires independent review before production installation.
