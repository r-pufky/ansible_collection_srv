# Role Testing
Assumes ansible and collection [environment setup](../collection/setup.md).

Molecule Testing framework for ansible roles. Use podman as it is a full
systemd container and does not many of the issues (including network issues)
that docker does.

Created in current working directory.

``` bash
molecule init scenario --driver-name=podman  # 'default' scenario
molecule init scneario alt_test --driver-name=podman  # 'alt_test' scenario
```

Directory layout (as directories or `yml` files):
```
├── dependency
├── lint
├── cleanup
├── destroy
├── syntax
├── create
├── prepare  # bring host to testable status
├── converge  # (required) apply role
├── idempotence
├── side_effect
├── verify  # assert role application correct
├── cleanup
└── destroy
```
[Reference](https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml)

[Reference](https://sbarnea.com/slides/molecule/#/13)

## System Images Used
* ghcr.io/hifis-net/debian-systemd:12
* ghcr.io/hifis-net/debian-systemd:11
* ghcr.io/hifis-net/debian-systemd:10

[Reference](https://github.com/hifis-net/debian-systemd)

### molecule.yml
Standard molecule setup for rootless podman debian container.
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
      # To cache facts between molecule steps
      # fact_caching: jsonfile
      # fact_caching_connection: /tmp/facts_cache
      # fact_caching_timeout: 7200
      #
      # and set a fact in a molecule step
      # ansible.builtin.set_fact:
      #   cacheable: yes
      #   my_fact: "howdy, world"
    ssh_connection:
      pipelining: false  # does not work with podman
platforms:
  - name: '{INSTANCE NAME}'
    image: 'ghcr.io/hifis-net/debian-systemd:12'
    systemd: 'always'
    volumes:
      - '/sys/fs/cgroup:/sys/fs/cgroup:ro'
    capabilities:
      - 'SYS_ADMIN'  # required for systemd systems
      # - 'NET_ADMIN'  # required for network-based testing
    command: '/lib/systemd/systemd'
    pre_build_image: true
    # privileged: true  # for some operations like sysctl, ulimit
verifier:
  name: 'ansible'
lint: |
  set -e
  yamllint .
  ansible-lint .
```

[Reference](https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/)

[Reference](https://ansible.readthedocs.io/projects/molecule/examples/podman/)

[Reference](https://stackoverflow.com/questions/65350229/how-do-i-share-variables-facts-between-molecule-playbooks)

# Running Tests
GHCR.IO requires a github login to download images.

Podman driver does have a schema and will always generate a warning.
``` bash
WARNING Driver podman does not provide a schema.
```
[Reference](https://github.com/ansible/molecule/discussions/4108)

``` bash
podman login ghcr.io
cd roles/{ROLE}
```

`scenario-name` may be used in each step to execute non-default scenario.

Manually run through testing steps (allows for repeating of application and
verification):
``` bash
molecule create
molecule converge -- -v
molecule verify
```

Run through the default scenario or specified scenarios:
``` bash
molecule test
molecule test --scenario-name=alt_test
```

Run through all existing Molecule scenarios:
``` bash
molecule test
molecule test --all
molecule test --all -- -v
```

# Troubleshooting
## Molecule 'Gathering Facts' Failed
Molecule state has not been reset; or there is leftover state from individual
testing steps.

``` bash
TASK [Gathering Facts] *********************************************************
fatal: [{HOST}]: UNREACHABLE! => {"changed": false, "msg": "Failed to create temporary directory. In some cases, you may have been able to authenticate and did not have permissions on the target
```

Fix by resetting molecule:
``` bash
molecule reset
```

## Molecule removed 'lint' dict option
Linting move to external commands to maximize flexibility. Provide a string of
commands to run instead.

molecule.yml
``` yaml
lint: |
  set -e
  yamllint .  # YAML linting
  ansible-lint .  # Ansible linting
  flake8  # Pything linting
```
[Reference](https://github.com/ansible/molecule/pull/3802/files)

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
