# Molecule: Podman Containers
Prerequisite:
* [ansible environment](../../environment/ansible.md)
* [podman](../../environment/podman.md)

Use podman as it is a full systemd container and does not many of the issues
(including network issues) that docker does.

## System Images Used

* ghcr.io/hifis-net/debian-systemd:12
* ghcr.io/hifis-net/debian-systemd:11
* ghcr.io/hifis-net/debian-systemd:10

[Reference](https://github.com/hifis-net/debian-systemd)

## Molecule Setup
Standard molecule setup for rootless podman debian container.

`0644 {USER}:{USER}` molecule.yml
``` yaml
---
dependency:
  name: 'galaxy'
driver:
  name: 'podman'
provisioner:
  name: 'ansible'
  config_options:
    defaults:
      interpreter_python: 'auto_silent'  # suppress warnings
      callback_whitelist: 'profile_tasks, timer, yaml'  # output profiling info
      # To cache facts between molecule steps:
      #   fact_caching: 'jsonfile'
      #   fact_caching_connection: '/tmp/facts_cache'
      #   fact_caching_timeout: 7200
      #
      # cache a fact between steps:
      #   ansible.builtin.set_fact:
      #     cacheable: yes
      #     my_fact: "howdy, world"
      #
      # always assert cache is not expired before using and document with args:
      #   - name: 'assert fact_caching not expired'
      #     ansible.builtin.assert:
      #       that:
      #         - '_test_cached_fact is defined'
      #       fail_msg: 'fact_caching has expired; re-run prepare.'
    ssh_connection:
      pipelining: false  # does not work with podman
  # inventory:  # Set all base testing configuration here.
  #   group_vars:
  #     all:
  #       setup_variables: true
  #   host_vars:
  #     {IMAGE}-{TEST}:
  #       setup_variables: true
platforms:
  - name: '{ROLE}-{IMAGE}-{MOLECULE}[-{TEST}]'  # debian-debian-12-default
    image: 'ghcr.io/hifis-net/debian-systemd:12'
    systemd: 'always'
    volumes:
      - '/sys/fs/cgroup:/sys/fs/cgroup:ro'
    # capabilities:  # always comment explicit need (file, command, etc)
    #   - 'SYS_ADMIN'  # required for systemd systems
    #   - 'NET_ADMIN'  # required for network-based testing
    command: '/lib/systemd/systemd'
    pre_build_image: true
    # privileged: true  # for some operations like sysctl, ulimit
verifier:
  name: 'ansible'
lint: |
  set -e
  yamllint .
  ansible-lint .
# Disable testing steps as needed with explicit reasons to minimize warnings.
scenario:
  test_sequence:
    - 'dependency'
    - 'cleanup'
    - 'destroy'
    - 'syntax'
    - 'create'
    - 'prepare'
    - 'converge'
    - 'idempotence'
    - 'side_effect'
    - 'verify'
    - 'cleanup'
    - 'destroy'
```
[Reference](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/)

[Reference](https://ansible.readthedocs.io/projects/molecule/examples/podman/)

[Reference](https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario)

[Reference](https://stackoverflow.com/questions/65350229/how-do-i-share-variables-facts-between-molecule-playbooks)

# Running Tests
GHCR.IO requires a github login to download images.

Podman driver does have a schema and will always generate a warning.
``` bash
WARNING Driver podman does not provide a schema.
```
[Reference](https://github.com/ansible/molecule/discussions/4108)

``` bash
podman login ghcr.io  # github account
cd roles/{ROLE}
molecule test
```

# Troubleshooting
See [molecule](molecule.md) for specific molecule troubleshooting.

## Podman required 'tmpfs' as dict, not list
Molecule documentation refers to using `tmpfs` as a list. Podman **only**
accepts dictionaries.

molecule.yml
``` yaml
platforms:
  - name: tmpfs as a dict
    image: ghcr.io/hifis-net/debian-systemd:12
    tmpfs:
      /tmp: 'rw,size=787448k,mode=1777'
```

Additionally `tmpfs` is **not** needed if `systemd` is set to `always`.
``` yaml
platforms:
  - name: 'tmpfs not needed with systemd always'
    image: 'ghcr.io/hifis-net/debian-systemd:12'
    systemd: 'always'
```

[Reference](https://docs.ansible.com/ansible/latest/collections/containers/podman/podman_container_module.html#parameter-tmpfs)

[Reference](https://github.com/ansible/molecule/issues/4140)

## Podman database static dir mis-match
Podman will not migrate things out of `/home` to prevent existing pod breakage.

Reset podman and migrate
```bash
podman system reset
podman system migrate
```

```bash
Error: database static dir ".../graph/libpod" does not match our static dir ".../graph/libpod": database configuration mismatch
```

[Reference](https://github.com/containers/podman/pull/20874)

### Cannot use `ping` from container
Some containers services require ping.

Set system-wide:

`0755 root:root` /etc/sysctl.d/net_ipv4_ping_group_range
``` bash
net.ipv4.ping_group_range=0 $MAX_UID
```

Alternatively set during container execution:

`0644 root:root` /etc/containers/containers.conf
``` ini
default_sysctls = [
  "net.ipv4.ping_group_range=0 2147483647",
]
```

[Reference](https://github.com/containers/podman/blob/main/troubleshooting.md#5-rootless-containers-cannot-ping-hosts)
