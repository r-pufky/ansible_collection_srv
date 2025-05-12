# Role Testing (Molecule)
Molecule Testing framework for ansible roles.
Prerequisite:
* [ansible environment](../../environment/ansible.md)

## [Podman Configuration](podman.md)
Primary test framework. Use podman as it provides a full systemd container
without **many** of the *systemd*, *networking*, *daemon*, and space issues
that come along with using [docker](https://github.com/r-pufky/ansible_pihole/pull/21#issuecomment-1716822044).

## [Vagrant Configuration](vagrant.md)
Vagrant VMs are **ONLY** used to test cases which cannot be tested in
containers (kernel, firmware, advanced networking, etc); always use **podman**
first.

## [Manual VM Configuration](manual_vm.md)
**Extreme** circumstances only; such as testing UEFI configuration. Where full
hardware virtualization and manual input are needed to test (e.g. secure boot
requiring manual steps, etc). These cannot be automated and should always be
run as a **non-default** test; with a simple (even no runtime errors) test run
as the `default`.

Explicit reasons for using this test framework must be clearly documented in
`molecule.yml`.

## Molecule Testing
Created in current working directory.

``` bash
molecule init scenario --driver-name=podman  # 'default' scenario, podman
molecule init scenario test --driver-name=vagrant  # 'vm_test' scenario, vagrant
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

### Test Patterns
Each pattern requires a separate molecule test setup.

* Molecule test name reflects component tested.
* `molecule.yml`:
  * Describe test.
  * Define test variables for test setup.
* `converge.yml`:
  * `unsafe` variables **must** be defined in `converge.yml` until fixed. See
    reference.
  * Define test variable here for edge cases outside of the base setup (e.g.
    multiple convergence steps).
  * For large sets of testing variables, define test values here and use
    `molecule.yml` for test setup.
  * Inject mock scripts to replace binaries if that binary cannot be used in a
    container (see secure_boot); enables testing role logic without the need
    for manual VMs.
* `prepare.yml`:
  * For complex dynamic generation of valid data, e.g. certificates, network.
  * Prefer `/tmp`.
  * Specific dynamic target files (such as SSH testing)
  * load generated files in `converge.yml` and inject test variables there.
* Use `ansible.builtin.assert` to validate test results. Any tests missing this
  are not explicitly validating correctness.
  https://www.puppeteers.net/blog/ansible-quality-assurance-part-1-ansible-variable-validation-with-assert/
  srv.tests
* Use `{ROLE}_testing_enable` in `defaults/testing.yml`:
  * Disable live API usage (which cannot be mocked easily).
  * Enable use of role in other roles without the need for a VM.
  * Use of `{ROLE}_testing_enable` **anywhere** in the role should always
    default the value to `false`. Roles which consume this role will likely not
    import all defaults and therefore `{ROLE}_testing_enable` will be
    undefined.
  * Roles should be thoroughly tested with VMs for tasks not covered in
    containers.
* Test instances should always use least privilege; add comments when used:
  * `gather_facts: false`
  * Idempotent
  * No `root` or `SYS_` capabilities
* Do not re-test encapsulated roles and collections.
* Decisions **not** to test a task must contain a
  [`Decision`](../../docstrings/variables.md#docstring-decision).
* Test before changes and **disable** `test_sequence` when testing new
  changes.

[Reference](https://github.com/ansible/molecule/issues/4348)

#### Testing documentation.
Provide testing documentation in `molecule.yml` explicitly stating the expected
conditions to test. Use the following format:

``` yaml
###############################################################################
# [Molecule Test Name]
###############################################################################
# Description of the molecule test / setup.
#
# Tests:
# * Each condition to verify on completion.
```

#### Default
`default` is effectively an role integration test with default values; only
disabling external systems/API usage, or components which cannot be tested on
the testing platform.

``` yaml
- name: 'Default | converge | apply debian role all defaults'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.debian'
```

* Validates no runtime errors.
* Explicit testing only if no components are used.

#### Component
Isolate component-level testing to individually test components within a role;
focusing on testing **all** options and **code paths**. A component **may**
test many options if they can be individually explicitly verified.

``` yaml
- name: 'isolate specific tasks from role for testing'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.debian'
    tasks_from: 'apt/packages.yml'
```

Components are not limited to:
* services, options, components
* mutually-exclusive settings which cannot be tested at the same time
* Test name reflects component used.

#### Exceptions (Special Case)
Edge cases and known finicky configurations should be explicitly tested
individually.

#### Regression (Bug Fix)
All reported bugs must have a test case explicitly testing that the bug has
been fixed.

* Always note the bug reference in `verify.yml` with the description of test.
* Prefix test with `regression_`.

#### VM Tests
VM testing is heavy. Use containers unless kernel namespace and 'hardware'
level access is needed.

#### Name Directives
Use nested pipes to specify testing stage and description. Additional pipe for
task subdirectory or file may be used if needed for additional clarification.

`0644 {USER}:{USER}` converge.yml
``` yaml
- name: '{TEST} | Converge | run locales.yml'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.debian'
    tasks_from: 'locales.yml'
```

`0644 {USER}:{USER}` verify.yml
``` yaml
- name: '{TEST} | verify'
  hosts: 'all'
  gather_facts: false
  tasks:
    - name: '{TEST} | verify | assert locales set'
      ...

    - name: '{TEST} | verify | sources | stat debian.sources'
      ...
```

#### Debug Directives
Use verbosity to keep debug statements in production releases; and var to
cleanly print complex variables.

``` yaml
- name: '{TASK} | debug verbose'
  ansible.builtin.debug:
    var: role_var  # auto jinja escaped
    verbosity: 2  # corresponds to -v count; 0 always displays
```

## Execute Tests
Manually run through testing steps (allows for repeating of application and
verification):
``` bash
molecule create
molecule converge -- -v
molecule verify
```
* `scenario-name` may be used in each step to execute non-default scenario.

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

``` bash
molecule --debug ${COMMAND}  # will enabling verbose debugging.
```

## Troubleshooting
See [podman](podman.md), [vagrant](vagrant.md), and [manual vm](manual_vm.md)
for framework specific troubleshooting.

### ERROR! Could not find or access
Molecule uses galaxy roles instead of local roles as dependencies when testing;
however for roles with high inter-dependence, this will cause the following
error when testing:

``` bash
ERROR! Could not find or access '.../ansible_collection_srv/roles/{ROLE}/molecule/{TEST}/{TASK_FROM_FILE}.yml on the Ansible Controller.

fatal: [{INSTANCE}]: FAILED! => {
    "changed": false,
    "include": "{TASK_FROM_FILE}.yml",
    "reason": "Could not find or access '/var/git/ansible_collection_srv/roles/debian/molecule/network/global.yml' on the Ansible Controller."
}
```

Until bug is fixed do one of the following:

TODO(role): see if molecule_yml can be used or this
https://stackoverflow.com/questions/74736985/how-come-default-vars-are-not-visible-to-ansible-molecule-verify

with https://ansible.readthedocs.io/projects/molecule/configuration/#molecule.config.Config

* Publish required roles to galaxy.

* Include the **entire** role:
``` yaml
- name: 'include full role'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.debian'
```

* Only `include_role` with `tasks_from` when no other tasks are sourced within:
``` yaml
- name: 'include task from role if there are no additional include_roles'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.debian'
    tasks_from: 'optimizations/firmware.yml'
```

[Reference](https://github.com/ansible/molecule/issues/3857)

### CRITICAL Idempotence test failed because of the following tasks
Not all operations are idempotent. Explicitly disable for testing where
idempotence cannot be guaranteed.

`0644 {USER}:{USER}` converge.yml
``` yaml
# Idempotent testing disabled as specific commands will never be idempotent:
# * {EXPLICIT REASONS FOR DISABLING}.
#
# Reference:
# * https://github.com/ansible/molecule/issues/816

- name: '{TEST} | converge'
  ...
  tags: 'molecule-idempotence-notest'
```

### Molecule 'Gathering Facts' Failed
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

### Molecule removed 'lint' dict option
Linting move to external commands to maximize flexibility. Provide a string of
commands to run instead.

`0644 {USER}:{USER}` molecule.yml
``` yaml
lint: |
  set -e
  yamllint .  # YAML linting
  ansible-lint .  # Ansible linting
  flake8  # Pything linting
```
[Reference](https://github.com/ansible/molecule/pull/3802/files)

### Role changes are not being used during tests
Role likely depends on changes in the collection that are not released yet.

[build](../../collection/testing.md#build-and-install-local-collection).

Or

[link](../../collection/testing.md#development-testing).
