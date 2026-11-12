# Provenance and independence

This package was created from a source snapshot identified by revision `29c24cfa2b6ead5704d26cf2746bc384f2938b80`.

The snapshot supplied the ext4/XFS project-quota controller, bounded privileged helper protocol, SQLite project registry, and Linux ABI implementation. Destructive live-mount test programs and machine-specific service deployment files were excluded.

The package contains no Git submodule, sibling-package dependency, workspace path dependency, runtime workflow-host dependency, or production service configuration. It does not change host quota state unless a caller explicitly invokes a mutating API.
