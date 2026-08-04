# AGENTS.md

This file defines how agents should work in this repository.

These instructions apply repo-wide unless the user gives a more specific request.

## Intent

This role should stay simple, explicit, and production-friendly.

The goal is to install and manage node-exporter, promtail, and cAdvisor as
systemd services without silently widening host permissions or privileges.

## Expected style

- Prefer clarity over cleverness.
- Prefer one clean public API over several overlapping inputs.
- Prefer removing dead code and dead variables over keeping backwards-compatible noise.
- Prefer explicit names that describe intent, not implementation detail.
- Keep tasks small and easy to scan.
- Do not leave debug tasks or temporary troubleshooting code in the final role.

## What good Ansible looks like here

- Use FQCNs for modules.
- Use explicit task names that describe the action.
- Use Ansible modules instead of `shell` or `command` whenever a proper module exists.
- Use YAML booleans as `true` and `false`, never `yes` or `no`.
- Fail fast on invalid states instead of silently working around them.
- Keep role inputs typed and structured as lists or dictionaries when the feature is repeatable.
- Keep defaults minimal and truthful: if a variable is exposed in `defaults/main.yml`, it must be supported end-to-end by the role.
- Avoid feature flags for half-supported behavior.

## Public API rules

- Public variable names must begin with `monitoring_agents_`.
- Public variables must reflect the real contract of the role.
- Keep agent-specific options under the corresponding agent prefix.
- Do not keep several competing ways to describe the same behavior unless there is a strong reason.
- Preserve explicit agent enablement through `monitoring_agents_<agent>_enabled`.

## Idempotence and safety

This repository values safe behavior on hosts that may already run monitoring
services and have protected system paths.

- Never silently run an agent as root.
- Create and manage the shared runtime user and group explicitly.
- Check the current state before creating or replacing filesystem paths.
- If a path exists and is not a directory, fail clearly.
- Avoid recursive permission changes on existing application, configuration, or state paths unless the user explicitly requests them.
- Do not make destructive behavior implicit in an idempotent task.
- Verify downloaded release assets with the upstream checksum before installation.

## Service expectations

- Treat each agent as a binary, configuration, and systemd unit managed together.
- Keep service names, binary names, configuration paths, and release metadata explicit.
- Enable and start a service only after its binary and unit file are in place.
- Use handlers to restart changed services and reload systemd deliberately.
- Document any additional log, cgroup, container-runtime, or group permissions that an agent needs; do not grant them implicitly.

## Task design

- Group tasks by purpose: validation, installation, configuration, and service management.
- Use `loop_control.label` when looping over agent definitions or paths.
- Register intermediate facts only when they are used by following tasks.
- Keep `when` conditions explicit and readable.
- Avoid deep Jinja expressions inline when a clearer approach exists.
- Assign role tags through `include_tasks` and `apply` so selecting a tag runs the complete task group.

## Testing and validation

Before considering a change done, run what is available locally:

- `yamllint`
- `ansible-lint`
- `molecule` when Docker access is available

Use Ansible `verify.yml` playbooks for scenario verification.

If a full runtime test cannot be executed because of sandbox or Docker access limitations,
say so clearly in the final report.

## Documentation expectations

- Code can move before docs while the API is being refactored.
- Once the API is stabilized, README examples must match the real role contract.
- Do not leave documentation for removed variables or removed workflows.
- Examples should be runnable, not aspirational.

## When to ask the user

Ask before implementing if the answer changes the public contract of the role, for example:

- supported operating systems or architectures
- ownership and permission handling
- runtime privileges or group membership
- backward compatibility vs cleanup

If the intent is already clear, implement directly.
