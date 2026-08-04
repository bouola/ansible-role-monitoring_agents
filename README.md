# Ansible Role: monitoring_agents

Deploy and manage node-exporter, promtail, and cAdvisor as systemd services using
upstream release binaries.

Galaxy FQCN: `bouola.monitoring_agents`

## Requirements

- Ansible 2.15 or newer
- Debian 12, Debian 13, Ubuntu 22.04, or Ubuntu 24.04
- systemd on the managed host
- Network access from the managed host to GitHub releases
- `unzip` is installed automatically when a zip release asset must be extracted

## Role Variables

| Variable | Type | Default | Description |
| --- | --- | --- | --- |
| `monitoring_agents_binary_install_dir` | string | `/usr/local/bin` | Directory where monitoring agent binaries are installed. |
| `monitoring_agents_config_dir` | string | `/etc/monitoring-agents` | Directory where monitoring agent configuration files are installed. |
| `monitoring_agents_state_dir` | string | `/var/lib/monitoring-agent` | Directory where monitoring agent runtime state files are stored. |
| `monitoring_agents_systemd_dir` | string | `/etc/systemd/system` | Directory where systemd unit files are installed. |
| `monitoring_agents_user` | string | `monitoring-agent` | System user used to run all monitoring agents. |
| `monitoring_agents_group` | string | `monitoring-agent` | System group used to run all monitoring agents. |
| `monitoring_agents_user_shell` | string | `/usr/sbin/nologin` | Login shell assigned to the monitoring agent system user. |
| `monitoring_agents_acl_paths` | list | `[]` | Existing paths granted ACLs for `monitoring_agents_group`. Each entry requires `path`, `permissions`, `recursive`, and `default_acl`. |
| `monitoring_agents_node_exporter_enabled` | boolean | `true` | Whether node-exporter is installed and managed. |
| `monitoring_agents_node_exporter_version` | string | `1.12.1` | Version of node-exporter to install. |
| `monitoring_agents_node_exporter_port` | integer | `9100` | TCP port where node-exporter listens. |
| `monitoring_agents_node_exporter_extra_args` | list | `[]` | Additional CLI flags passed to node-exporter. |
| `monitoring_agents_promtail_enabled` | boolean | `false` | Whether promtail is installed and managed. |
| `monitoring_agents_promtail_docker_enabled` | boolean | `false` | Whether promtail's runtime user is added to the `docker` group. This grants Docker-equivalent root access. |
| `monitoring_agents_promtail_systemd_journal_access_enabled` | boolean | `false` | Whether promtail's runtime user is added to the `systemd-journal` group for explicitly configured journal scrape jobs. |
| `monitoring_agents_promtail_version` | string | `3.6.11` | Version of promtail to install. |
| `monitoring_agents_promtail_port` | integer | `9080` | TCP port where promtail exposes its HTTP endpoint. |
| `monitoring_agents_promtail_loki_url` | string | empty | Loki base URL used by promtail, for example `http://loki:3100`. Required when promtail is enabled. |
| `monitoring_agents_promtail_bearer_token_file` | string | empty | Bearer token file used by promtail when pushing to Loki. |
| `monitoring_agents_promtail_default_labels` | dict | `{}` | Labels added to every file scrape configuration. Per-path labels override matching defaults. |
| `monitoring_agents_promtail_log_paths` | list | `[]` | File scrape configurations. Each entry requires `path` and optionally accepts `labels`. |
| `monitoring_agents_promtail_extra_scrape_configs` | list | `[]` | Additional promtail `scrape_configs` entries. |
| `monitoring_agents_cadvisor_enabled` | boolean | `false` | Whether cAdvisor is installed and managed. |
| `monitoring_agents_cadvisor_version` | string | `0.60.5` | Version of cAdvisor to install. |
| `monitoring_agents_cadvisor_port` | integer | `8080` | TCP port where cAdvisor listens. |
| `monitoring_agents_cadvisor_extra_args` | list | `[]` | Additional CLI flags passed to cAdvisor. |
| `monitoring_agents_cadvisor_checksums` | dict | version map | SHA256 checksums for cAdvisor release binaries by version and architecture. Add the matching checksum when changing `monitoring_agents_cadvisor_version`. |

Promtail is pinned to the latest Loki release that still publishes Promtail
binary assets.

When `monitoring_agents_promtail_docker_enabled` is `true`, Docker must already
be installed and its `docker` group must exist. The role does not create it.

When `monitoring_agents_promtail_systemd_journal_access_enabled` is `true`, the
host must provide the `systemd-journal` group. This grants access only; configure
the journal scrape itself with `monitoring_agents_promtail_extra_scrape_configs`.

Use `monitoring_agents_acl_paths` to grant the shared monitoring group access
without changing existing ownership or group permissions. Configuring one or
more entries installs the `acl` package. Use `rX` for a readable directory tree:
it grants execute only to directories and executable files. Include parent
directories as separate entries when they are not already traversable by the
monitoring group.

Runtime socket ACLs are ephemeral: Docker, containerd, or the host can recreate
a socket without its ACL. Rerun this role or manage persistence with a
runtime-specific systemd drop-in.

```yaml
monitoring_agents_acl_paths:
  - path: "/var/log/my-application"
    permissions: "rX"
    recursive: true
    default_acl: true
  - path: "/var/run/docker.sock"
    permissions: "rw"
    recursive: false
    default_acl: false
  - path: "/run/containerd/containerd.sock"
    permissions: "rw"
    recursive: false
    default_acl: false
```

## Dependencies

None.

## Example Playbook

Minimal node-exporter deployment:

```yaml
---
- name: "Install node-exporter"
  hosts: monitoring_targets
  become: true
  roles:
    - role: "bouola.monitoring_agents"
```

Full deployment with all agents enabled:

```yaml
---
- name: "Install monitoring agents"
  hosts: monitoring_targets
  become: true
  vars:
    monitoring_agents_promtail_enabled: true
    monitoring_agents_promtail_loki_url: "http://loki:3100"
    monitoring_agents_promtail_bearer_token_file: "/etc/monitoring-agents/loki.token"
    monitoring_agents_promtail_docker_enabled: true
    monitoring_agents_promtail_systemd_journal_access_enabled: true
    monitoring_agents_promtail_default_labels:
      environment: "production"
      team: "platform"
    monitoring_agents_promtail_log_paths:
      - path: "/var/log/my-application/*.log"
        labels:
          application: "my-application"
    monitoring_agents_acl_paths:
      - path: "/var/log/my-application"
        permissions: "rX"
        recursive: true
        default_acl: true
      - path: "/var/run/docker.sock"
        permissions: "rw"
        recursive: false
        default_acl: false
      - path: "/run/containerd/containerd.sock"
        permissions: "rw"
        recursive: false
        default_acl: false
    monitoring_agents_cadvisor_enabled: true
    monitoring_agents_node_exporter_extra_args:
      - "--collector.systemd"
    monitoring_agents_cadvisor_extra_args:
      - "--docker_only=false"
  roles:
    - role: "bouola.monitoring_agents"
```

## Molecule Scenarios

- `default`: installs node-exporter, promtail, and cAdvisor together with minimal non-conflicting configuration.
- `node_exporter`: installs node-exporter only with custom port and extra arguments.
- `promtail`: installs promtail only with log paths, bearer token file, and extra scrape configs.
- `cadvisor`: installs cAdvisor only with custom port and extra arguments.

## License

MIT
