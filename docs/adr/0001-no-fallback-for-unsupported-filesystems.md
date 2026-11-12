# ADR-0001: Fail closed when project quota is unavailable

- **Status:** Accepted
- **Date:** 2026-11-12

## Context

Filesystem project quota is a kernel-enforced isolation boundary. Userspace estimates or a fallback to an unconstrained directory would falsely imply enforcement.

## Decision

Only ext4/XFS project-quota mounts with an active project-quota option and a block-device source are accepted. Unsupported mounts return an error. No userspace accounting or unlimited fallback is provided.

## Consequences

- Callers must configure a dedicated supported mount.
- Unsupported environments remain unavailable rather than silently unbounded.
- Tests that only exercise parsing/registry invariants do not prove kernel enforcement.
