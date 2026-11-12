# Source Lineage

## Upstream source

- Repository: [nplongx/runalith](https://github.com/nplongx/runalith)
- Original crate: `rust/crates/runalith-linux-quota`
- Source revision: [`29c24cfa2b6ead5704d26cf2746bc384f2938b80`](https://github.com/nplongx/runalith/tree/29c24cfa2b6ead5704d26cf2746bc384f2938b80/rust/crates/runalith-linux-quota)
- Upstream license: MIT

The original crate directory was clean at the extraction revision.

## Included

- `src/lib.rs`: project quota controller, Unix socket client, privileged helper server, helper request validation, SQLite project registry, Linux ABI code, and inline unit tests.
- `src/bin/project-quota-helper.rs`: renamed standalone helper executable; environment settings use `PROJECT_QUOTA_HELPER_*` names.

The manifest replaces workspace-inherited settings with a standalone package named `linux-project-quota` and Rust import name `linux_project_quota`. Relevant direct dependency versions/features were retained.

## Excluded

- `src/bin/runalith-quota-integration.rs`: destructive live-mount integration program that creates files until byte/inode quota exhaustion, removes test workspaces, and permanently reserves project IDs. It requires a dedicated disposable test mount and explicit operator control; it is not shipped or run.
- Runalith worker workspace manager, service unit deployment files, host mount configuration, and production quota settings.

## Genericization

The helper's private workspace parent convention is now `.project-quota-workspaces`; helper environment names and executable name are generic. Kernel project quota semantics, ext4/XFS restrictions, project ID range, SQLite registry invariants, and Linux ABI operations remain unchanged.

## Independence statement

The new repository has no internal workspace path dependency, Git submodule, or Runalith runtime dependency. MIT attribution is retained. This extraction does not modify the original source crate or any live filesystem quota state.
