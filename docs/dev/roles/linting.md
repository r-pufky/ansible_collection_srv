# Role Linting
Prerequisite:
* [ansible environment](../environment/ansible.md)

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
  when: _ufw_config.changed
  community.general.ufw:
    state: 'reloaded'
```

Also for display debug messages if specific states have occurred.
``` yaml
- name: 'HOST CHANGE'  # noqa no-handler execute immediately warning message
  when: _ssh_rsa_host_key_changed.changed
  ansible.builtin.debug:
    msg: |
      Target RSA host keys have changed. WILL produce host warning.
  changed_when: false
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

# Default random data.
{ROLE}_role_generated_api_key: '{{ lookup("ansible.builtin.password", "/dev/null", chars=["ascii_letters", "digits"], length=32) }}'
```

## Name Directives

### Tasks
Use bare descriptions.
``` yaml
- name: 'Config'
  ansible.builtin.include_tasks: 'config.yml'

- name: 'Disable firewall'
  when: not ufw_enable
  community.general.ufw:
    state: 'disabled'
```

### tasks/[sub_section/]file.yml
Use nested pipes to locate file then descriptions.
``` yaml
tasks/file.yml
- name: 'File | disable firewall'
  when: not ufw_enable
  community.general.ufw:
    state: 'disabled'

tasks/sub_section/file.yml
- name: 'Sub section | file | disable firewall'
  when: not ufw_enable
  community.general.ufw:
    state: 'disabled'
```

### When Clause
Immediately following `name:` clause which follows `ansible-lint` block
specification; allowing immediate determination of whether a task should be
run.

``` yaml
- name: 'task with a clause'
  when: some_var_to_test
  community.general.ufw:
    state: 'disabled'
```

### Notify Clause
Immediately following `when:` clause which follows `ansible-lint` block
specification; allowing immediate determination of whether a task should be
run.

``` yaml
- name: 'task with a clause'
  when: some_var_to_test
  notify: 'handler'
  community.general.ufw:
    state: 'disabled'
```

### Become Clause
Immediately following `task` as primary additional context.

``` yaml
- name: 'task with a clause'
  community.general.ufw:
    state: 'disabled'
  become: true
  register: _some_var
```

### Comment Headers
One vertical space to allow for individual task and variable comments.

``` yaml
###############################################################################
# Capped Header Text (Clarifications)
###############################################################################
#
#

- name: 'task with a clause'
  when: some_var_to_test
  community.general.ufw:
    state: 'disabled'
```
* `defaults`: No space if header is **only** describing a single variable.
* Extended comment section is optional.
