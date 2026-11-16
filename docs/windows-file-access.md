# Windows file writes and deletions

English · [简体中文](windows-file-access.zh.md)

## Identify the failing operation

DeepShian delegates file operations to DeepSeek Harness. On Windows, a native file edit and a PowerShell command use different enforcement paths:

```text
Native file tool -> Harness filesystem policy -> host filesystem operation
PowerShell      -> Harness process sandbox   -> restricted Windows child
```

An edit succeeding does not prove that a shell command can write or delete the same file. A permission error from the child process also does not, by itself, prove that the signed-in Windows account lacks access.

The current plugin offers **Read Only** and **Workspace Write**. A model's suggestion to choose a session-wide “Full access” option does not describe this plugin's current UI. A separately approved tool escalation is a different operation.

## Check before changing permissions

1. Record the exact target path, failing tool, error, and versions. Use the latest failure rather than assuming an old message still reflects the current state.
2. Confirm **Workspace Write** and that the target is inside the session's vault workspace. Read Only intentionally denies mutations.
3. Distinguish errors. `Access denied` or `Permission denied` warrants an access investigation. `MethodInvocationNotSupportedInConstrainedLanguage` means PowerShell rejected a language/API operation; it is not evidence that a file cannot be deleted.
4. Have the diagnosis compare ordinary access with access under Harness's actual restricted token. A writable host process is not a substitute for this check.
5. Validate with a unique disposable file in the affected directory. Check creation, rewriting, deletion, and cleanup separately. A file-access handle check is useful evidence, but is not an executed deletion test.

For a PowerShell diagnostic, stop on errors and inspect the result instead of reporting success after an earlier command failed. Do not use `git clean` as a permission test: it selects untracked files rather than one explicitly scoped test object.

## Observed Harness 0.2.2-rc.1 case

In one Windows installation, the workspace root retained Harness's workspace capability grant, while two descendant note directories lacked the inherited grant. The installed provisioner reused an exact grant at the root instead of repairing those descendants. Native edits worked, but a restricted child returned Win32 error 5 for write/delete access.

A machine-specific repair of the two explicitly approved directories restored restricted-token file creation, rewriting, and deletion. File contents and existing access entries were checked against backups. That repair was local environment maintenance; this fork does not ship an automatic permission repair or turn this observation into a diagnosis for every Windows denial.

Permission changes should follow a verified cause and a concrete approved scope, with a backup of access rules and integrity labels. Broad ownership changes, recursive permission resets, and globally disabling confinement are not part of this plugin's troubleshooting workflow.

See the upstream [Windows sandbox implementation guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/sandbox/sandbox-windows-acl/README.md) for its capability grants, integrity labels, and recovery boundaries. When reporting a problem, share a minimal redacted error and version details rather than a full private conversation or vault.
