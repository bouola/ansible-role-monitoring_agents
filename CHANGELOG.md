## [1.2.1](https://github.com/bouola/ansible-role-monitoring_agents/compare/v1.2.0...v1.2.1) (2026-09-30)


### Bug Fixes

* add systemd unit support for managing ACLs on runtime sockets ([#5](https://github.com/bouola/ansible-role-monitoring_agents/issues/5)) ([97c99e2](https://github.com/bouola/ansible-role-monitoring_agents/commit/97c99e24ff3e52a073143caeeac24434364f68aa))

# [1.2.0](https://github.com/bouola/ansible-role-monitoring_agents/compare/v1.1.1...v1.2.0) (2026-09-30)


### Features

* add configurable listen address for node-exporter ([57258d5](https://github.com/bouola/ansible-role-monitoring_agents/commit/57258d596c09eed5cc13b2dd66ae1281a5ae3c27))

## [1.1.1](https://github.com/bouola/ansible-role-monitoring_agents/compare/v1.1.0...v1.1.1) (2026-08-05)


### Bug Fixes

* remove redundant handlers and enhance ACL management ([a443d99](https://github.com/bouola/ansible-role-monitoring_agents/commit/a443d99eea8be3b069fabc99dbee4e16fd8801af))

# [1.1.0](https://github.com/bouola/ansible-role-monitoring_agents/compare/v1.0.0...v1.1.0) (2026-08-04)


### Features

* enable systemd journal access for Promtail ([25166cf](https://github.com/bouola/ansible-role-monitoring_agents/commit/25166cf6d94531cce43e333c7d818a39c7203f52))

# 1.0.0 (2026-08-04)


### Features

* add ACL management for cAdvisor container runtime sockets ([9076a7c](https://github.com/bouola/ansible-role-monitoring_agents/commit/9076a7c9c92f7419c7c5e9263f088999f8972d7e))
* add Ansible role for managing monitoring agents ([c496ce4](https://github.com/bouola/ansible-role-monitoring_agents/commit/c496ce42813983d33f25b23ebd57125cd14759d3))
* add Promtail Docker log and path access configuration. ([3181f24](https://github.com/bouola/ansible-role-monitoring_agents/commit/3181f24aa72bd3943704922cfc8ee87879130266))
* add support for configurable Promtail log labels and paths ([dd0773f](https://github.com/bouola/ansible-role-monitoring_agents/commit/dd0773f9615119d53a6274dccbb1b6ee41aa32cc))
