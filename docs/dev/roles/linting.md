# Role Linting
Assumes ansible and collection [environment setup](../collection/setup.md).

# File Formatting
**Always** follow `yamllint` and `ansible-lint` linting with **no** global
execeptions.

## Accepted Disables

### yamllint: line-length (file/line)
Must appear immediately after YAML specifier if file disable.
``` yaml
---
# yamlint disable rule:line-length

some_long_yaml_line: 0  # yamllint disable-line rule:line-length
```

### ansible-lint: name[template] (line)
For multiple variables in a `name` directive, typically file paths.
``` yaml
- name: 'file: {{ item.path }}/.hidden_file'  # noqa name[template] file path
```

### ansible-lint: no-handler (line)
For targeted handler execution where `ansible.builtin.flush_handlers` will
trigger handlers that affect additional unconfigured services.
``` yaml
- name: 'reload for support changes'  # noqa no-handler execute immediately
  community.general.ufw:
    state: 'reloaded'
  when: _ufw_config.changed
```

Also for display debug messages if specific states have occurred.
``` yaml
- name: 'HOST CHANGE'  # noqa no-handler execute immediately warning message
  ansible.builtin.debug:
    msg: |
      Target RSA host keys have changed. WILL produce host warning.
  changed_when: false
  when: _ssh_rsa_host_key_changed.changed
```

## role/vars
Vars listed below are required if used. All role vars `{ROLE}_role_` prefixed.
``` yaml
# Last time {ROLE} options were validated against a default configuration.
{ROLE}_role_validate_date: '2024-06-14'
{ROLE}_role_validate_release: 'bookworm'

# Default packages for {ROLE}.
{ROLE}_role_packages:
  - 'ssh'  # meta package provides both ssh and sshd.

# Default state for apt package installation. See ansible.builtin.apt.
{ROLE}_role_package_state: 'latest'
```

## Name Directives

### tasks
Use bare descriptions.
``` yaml
- name: 'Config'
  ansible.builtin.include_tasks: 'config.yml'

- name: 'Disable firewall'
  community.general.ufw:
    state: 'disabled'
  when: not ufw_enable
```

### tasks/[sub_section/]file.yml
Use nested pipes to locate file then descriptions.
``` yaml
tasks/file.yml
- name: 'File | disable firewall'
  community.general.ufw:
    state: 'disabled'
  when: not ufw_enable

tasks/sub_section/file.yml
- name: 'Sub section | file | disable firewall'
  community.general.ufw:
    state: 'disabled'
  when: not ufw_enable
```
