# Molecule: Vagrant with VirtualBox VMs
Prerequisite:
* [ansible environment](../../environment/ansible.md)
* [virtualbox](../../environment/virtualbox.md)
* [vagrant](../../environment/vagrant.md)

Vagrant VMs are **ONLY** used to test cases which cannot be tested in
containers (base OS, kernel, firmware, advanced networking, etc) these should
**never** be `default` test cases.

Created in current working directory.

## System Images Used

* debian/bookworm64

[Reference](https://portal.cloud.hashicorp.com/vagrant/discover/debian)

## Molecule Setup
Standard molecule setup for vagrant with virtualbox VM.

`0644 {USER}:{USER}` molecule.yml
``` yaml
---
dependency:
  name: 'galaxy'
driver:
  name: 'vagrant'
  provider:
    name: 'virtualbox'
    enable_efi: true
    config_options:
      ssh.keep_alive: true
      ssh.remote_user: 'root'  # default login user, different per VM image
    #provider_raw_config_args:
    #  - "customize [ 'modifyvm', :id, '--firmware', 'efi' ]"
    # enable_efi: true
    # both options do not seem to work? maybe an image thing.
    options:
      append_platform_to_hostname: false
provisioner:
  name: 'ansible'
  config_options:
    defaults:
      interpreter_python: 'auto_silent'  # suppress warnings
      callback_whitelist: 'profile_tasks, timer, yaml'  # output profiling info
      # To cache facts between molecule steps:
      # fact_caching: jsonfile
      # fact_caching_connection: /tmp/facts_cache
      # fact_caching_timeout: 7200
      #
      # and set a fact (converge.yml):
      # ansible.builtin.set_fact:
      #   cacheable: yes
      #   my_fact: "howdy, world"
  # inventory:  # Set all base testing configuration here.
  #   group_vars:
  #     all:
  #       setup_variables: true
  #   host_vars:
  #     {IMAGE}-{TEST}:
  #       setup_variables: true
platforms:
  - name: '{IMAGE}-{TEST}'  # debian-bookworm64-vm-test
    box: 'debian/bookworm64'
    memory: 4096
    cpus: 2
    interfaces:
      - network_name: private_network  # network_name required
        auto_config: true
        type: 'dhcp'
        # type: static
        # ip: 192.168.56.10  # default is 192.168.56.0/21
        # guest: 80
        # host: 8080
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
[Reference](https://ansible.readthedocs.io/projects/molecule/configuration/?h=test_sequence#scenario)

[Reference](https://floatingpoint.sorint.it/blog/post/setting-up-molecule-for-testing-ansible-roles-with-vagrant-and-testinfra)

# Running Tests
Always **become** the `ssh.remote_user` when creating Molecule tests or setup
an ansible user after VM turnup to apply ansible tasks.

``` yaml
- name: 'Molecule testing step'
  hosts: 'all'
  gather_facts: false
  become: true
...
```

[Reference](#failed-to-lock-apt-for-exclusive-operation)

Vagrant driver does have a schema and will always generate a warning.
``` bash
WARNING Driver vagrant does not provide a schema.
```
[Reference](https://github.com/ansible/molecule/discussions/4108)


# Troubleshooting

## Failed to lock apt for exclusive operation.
Debian virutalbox VM uses root to configure and test the instance. All Molecule
setup/teardown steps require root user.

``` yaml
- name: 'Molecule testing step'
  hosts: 'all'
  gather_facts: false
  become: true
...
```
[Reference](https://stackoverflow.com/questions/33563425/ansible-1-9-4-failed-to-lock-apt-for-exclusive-operation)

## Failed to start the VM(s).
Virtualbox may fail to bring up the VM properly due to system constraints.
Re-run the test.

``` bash
PLAY [Create] ******************************************************************
fatal: [localhost]: FAILED! => {
    "changed": false,
    "cmd": [
        "/usr/bin/vagrant",
        "up",
        "--no-provision"
    ],
    "rc": 1
}

STDERR:

### YYYY-MM-DD 15:10:23 ###
### YYYY-MM-DD 15:10:23 ###
### YYYY-MM-DD 15:10:23 ###
### YYYY-MM-DD 15:32:14 ###
### YYYY-MM-DD 15:32:15 ###
### YYYY-MM-DD 15:32:15 ###
The SSH connection was unexpectedly closed by the remote end. This
usually indicates that SSH within the guest machine was unable to
properly start up. Please boot the VM in GUI mode to check whether
it is booting properly.

MSG:

Failed to start the VM(s): See log file '.../molecule/debian/firmware/vagrant.err'
```

``` bash
molecule test --scenario-name={TEST}
```

## VM seems flaky
Vagrant uses base VM images; a previous molecule test was likely not cleaned up
properly.

Clean all VMs and re-run:
``` bash
vagrant global-status
vagrant destroy {ID}
vagrant box list
vagrant box destroy {ID}
```
* Additionally check GUI if needed
