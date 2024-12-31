# Molecule: Manual VM
Prerequisite:
* [ansible environment](../../environment/ansible.md)
* [virtualbox](../../environment/virtualbox.md)
* [vagrant](../../environment/vagrant.md)

**Extreme** circumstances only; such as testing UEFI configuration. Where full
hardware virtualization and manual input are needed to test (e.g. secure boot
requiring manual steps, etc). These cannot be automated and should always be
run as a **non-default** test; with a simple (even no runtime errors) test run
as the `default`.

Explicit reasons for using this test framework must be clearly documented in
`molecule.yml`.

Create in current working directory.

## System Images Used

* [debian netinstall iso](https://www.debian.org/CD/netinst/)

## Create VM
Use vagrant-like configuration to maintain similarities with other VM tests.

Always use absolute paths as some options will **not** translate '~', etc.

Create and list VM
``` bash
mkdir /home/${USER}/VirtualBox VMs/debian-12-efi-template
vboxmanage list ostypes
vboxmanage createvm --basefolder '/home/${USER}/VirtualBox VMs/debian-12-efi-template' --ostype debian12_64 --default --register --name 'debian-12-efi-template'
vboxmanage list vms
export UUID={UUID}  # always reference via the UUID
vboxmanage showvminfo ${UUID}
```

Use bridged networking (if needed)
``` bash
vboxmanage modifyvm ${UUID} --nic1 bridged
vboxmanage modifyvm ${UUID} --bridgeadapter1 {DEVICE}  # host linux device name
```
[Reference](https://forums.virtualbox.org/viewtopic.php?t=78292)

Enable UEFI and load certificates
``` bash
vboxmanage modifyvm ${UUID} --cpus 2 --memory 4096 --firmware efi64  # enable EFI. alternatives: efi, efi32
vboxmanage modifynvram $UUID inituefivarstore  # resets NVRAM state
vboxmanage modifynvram $UUID enrollmssignatures  # load default MS KEK/DB signatures for secure boot
vboxmanage modifynvram $UUID enrollorclpk  # enable default platform keys for secure boot
```
[Reference](https://www.virtualbox.org/manual/UserManual.html#vboxmanage-modifynvram)

Secure Boot Toggle
``` bash
vboxmanage modifynvram $UUID secureboot --disable
```
Secure boot state cannot be changed with `mokutil` in VM; explicitly set state
to disabled for base testing and enable when needing to test final state.

Attach Storage
``` bash
vboxmanage createmedium disk --size 6000 --format vdi --filename '/home/${USER}/VirtualBox VMs/debian-12-efi-template/disk.vdi'
vboxmanage storageattach ${UUID} --storagectl 'SATA' --device 0 --port 1 --type hdd --medium '/home/${USER}/VirtualBox VMs/debian-12-efi-template/disk.vdi'
vboxmanage storageattach ${UUID} --storagectl 'SATA' --port 2 --type dvddrive --medium '/home/${USER}/debian-12.8.0-amd64-netinst.iso'
```
Start VM
``` bash
vboxmanage startvm ${UUID}
```

[Reference](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/efi.html)

## Base OS Install
* Hostname: `debian`
* Domain: {EMPTY}
* users:
  * `root`:`vagrant`
  * `vagrant`:`vagrant`
* guided partition: `whole disk`, `one partition`, `overwrite`
* packages: `ssh`, `standard system utilities`

reboot

### Install vagrant SSH keys
``` bash
wget https://raw.githubusercontent.com/hashicorp/vagrant/refs/heads/main/keys/vagrant.pub -O /root/.ssh/authorized_keys
```
[Reference](https://github.com/hashicorp/vagrant/tree/main/keys)

### Zero disk, clear history
``` bash
apt get clean
dd if=/dev/zero of=/EMPTY bs=1M
rm -f /EMPTY
cat /dev/null > ~/.bash_history && history -c && exit
```
Enables high compression when image and a clean system state.

[Reference](https://gist.github.com/srijanshetty/86aa6540d27736c1c11b)

### Shutdown and Image VM
``` bash
VBoxManage controlvm ${UUID} acpipowerbutton
vboxmanage export ${UUID} --output exported-debian-12-efi.ovf --ovf20  # backup virtualbox image pristine w/ disk.
```

[Reference](https://www.engineyard.com/blog/building-a-vagrant-box-from-start-to-finish/)

[Reference](https://andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/)

## Molecule Setup
Standard molecule setup for manual VM.

### Copy vagrant SSH keys
Vagrant SSH keys [are published here](https://github.com/hashicorp/vagrant/tree/main/keys).

Copy to `molecule/id.{pub,key}` for use in all molecule tests. Copy to
`molecule/{TEST}/id.{pub,key}` for specific molecule tests (additional testing
frameworks in use).

Set restricted permissions
``` bash
chmod 0400 id.*
```
Molecule will fail SSH connections and not mention permissions issues.

### `0644 {USER}:{USER}` molecule.yml
``` yaml
---
dependency:
  name: 'galaxy'
driver:
  name: default
provisioner:
  name: 'ansible'
  config_options:
    defaults:
      interpreter_python: 'auto_silent'  # suppress warnings
      callback_whitelist: 'profile_tasks, timer, yaml'  # output profiling info
  inventory:
    group_vars:
      all:
        ansible_ssh_user: 'root'  # molecule uses logged in user by default
        ansible_ssh_private_key_file: '{{ lookup("env", "MOLECULE_PROJECT_DIRECTORY") }}/molecule/id.key'
        # ansible_ssh_private_key_file: '{{ lookup("env", "MOLECULE_EPHEMERAL_DIRECTORY") }}/id.key'  # key only in test location
platforms:
  - name: '10.2.2.39'  # IP of the VM
verifier:
  name: 'ansible'
lint: |
  set -e
  yamllint .
  ansible-lint .
scenario:
  test_sequence:
    # - 'dependency'
    # - 'cleanup'
    - 'destroy'
    - 'syntax'
    - 'create'
    # - 'prepare'
    - 'converge'
    - 'idempotence'
    # - 'side_effect'
    - 'verify'
    # - 'cleanup'
    - 'destroy'
```

### `0644 {USER}:{USER}` create.yml
``` yaml
---
- name: Create
  hosts: localhost
  connection: local
  gather_facts: false
  no_log: "{{ molecule_no_log }}"
  tasks:
    - name: 'Create | molecule inventory'
      ansible.builtin.debug:
        msg: skipping provisioning

    - name: 'Create | populate instance config'
      ansible.builtin.set_fact:
        instance_conf_dict: {'instance': '{{ item.name }}'}
      loop: '{{ molecule_yml.platforms }}'
      register: instance_config_dict
```
An exact copy of the default create with provisioning disabled.

[Reference](https://medium.com/@fabio.marinetti81/validate-ansible-roles-through-molecule-delegated-driver-a2ea2ab395b5)

[Reference](https://cloudautomation.pharriso.co.uk/post/molecule-vm-create/#:~:text=Molecule%20Configuration,yml)

# Troubleshooting

## Permission denied (publickey,password).
Verify permissions are correct and keys are loaded from the correct location.

``` bash
TASK [Install packages] ********************************************************
included: {ROLE} for 10.2.2.39
fatal: [10.2.2.39]: UNREACHABLE! => {
    "changed": false,
    "unreachable": true
}
MSG:
root@10.2.2.39: Permission denied (publickey,password).
```

Key are set correctly:

`0400 {USER}:{USER}` id.*

Add a debug line in converge to confirm key is located correctly.

`converge.yml`
``` yaml
- ansible.builtin.debug:
    msg: '{{ ansible_ssh_private_key_file }}'
```

## VM moved or not registered
VM's must be registerd to be started in VirtualBox. Register the VM.

``` bash
vboxmanage registervm /home/{USER}/VirtualBox\ VMs/debian-12-efi-template/debian-12-efi-template.vbox
```

## Manually export image and import into vagrant for use.
Exports a machine for vagrant use. This does not enable automatic loading of
the VM unless the image is also published.

``` bash
vagrant package --base ${UUID} --output ucs.box  # UUID of the virtualbox template vm
vagrant box add ucs.box --name vagrant-debian  # use vagrant-debian in molecule testing.
```
