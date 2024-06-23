# Setup Ansible Environment

## Create Environment and Install Packages
Create python virtual environent for Ansible, activate, and install Ansible,
Podman, Molecule pip packages:
``` bash
python -m virtualenv ~/.config/venv/ansible
source ~/.config/venv/ansible/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install ansible
pip install argcomplete
pip install ansible-lint
pip install molecule
pip install molecule-plugins[podman]
pacman -Syu podman  # fuse-overlayfs is NOT needed anymore in latest releases
```
[Reference](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html)

## Setup Rootless Podman for Testing

### Verify Rootless Support
``` bash
podman info | grep -i overlay

  107:  graphDriverName: overlay
  114:    Native Overlay Diff: "true"
```
`overlay` and `Diff: "true"` means supported.

### Verify Unprivileged User Namespace Enabled
``` bash
sysctl kernel.unprivileged_userns_clone

kernel.unprivileged_userns_clone = 1
```
`1` means enabled.

### Create Subordinate UID/GID Mappings
Configuration entry must exist for each user that wants to use it. New users
created using useradd have these entries by default. If not add user defaults:

``` bash
cat /etc/subuid | grep -ri {USER}
cat /etc/subgid | grep -ri {GROUP}
```

``` bash
usermod --add-subuids 100000-165535 --add-subgids 100000-165535 {USER}
```

[Reference](https://github.com/systemd/systemd/issues/21952)

### Set Default Container Registry for Podman
/etc/containers/registries.conf
``` ini
[registries.search]
registries = ['ghcr.io']
unqualified-search-registries=['ghcr.io']
```
[Reference](https://halukkarakaya.medium.com/how-to-configure-default-search-registries-in-podman-ea930289692)

### Enable Rootless Use of Ping
Some services require ping.

/etc/containers/containers.conf
``` ini
default_sysctls = [
  "net.ipv4.ping_group_range=0 2147483647",
]
```

### Migrate Podman Installation
Applies configuration changes to Podman. Critical for unprivileged podman to
execute properly. Run as the current user:

``` bash
podman system migrate
```
[Reference](https://github.com/containers/podman/issues/12715)

## References

[Testing with Molecule and Podman](https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1/)

[Podman](https://wiki.archlinux.org/title/Podman)

## One-time Collection Initialization
Only for new repositories without existing collections.

``` bash
git clone https://github.com/{USER}/{REPO} ~/git/{COLLECTION}
cd ~/git/{COLLECTION}
ansible-galaxy collection init --init-path . r_pufky.srv
mv -vn r_pufky/srv/* .
rm -rfv r_pufky
mkdir -p plugins/modules  # for code like python
touch plugins/modules/demo_hello.py  (paste from site)  # demo
```
[Collection Structure](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collections-doc-dir)

[Collection Template](https://github.com/ansible-collections/collection_template/tree/main)

[Creating Collections](https://goetzrieger.github.io/ansible-collections/5-creating-collections/)

## Enable Submodule Summaries and Out of Date
Check out of date/uncommitted submodules on push and provide a better summary
of submodule status (stored in `.git/config`).

Enable submodule summary on main repo commits/status
``` bash
git config status.submodulesummary 1
```

Check for submodules out of date/uncommitted when pushing main repo
``` bash
git config push.recurseSubmodules check
```

## Setup VScode for Ansible
Install the [Ansible extension](https://marketplace.visualstudio.com/items?itemName=redhat.ansible)

ansible ➤ settings ➤ workspace
* ansible path: `~/.config/venv/ansible/bin/ansible`
* always use FQCN: ☑
* reuse terminal: ☑
* python interpreter path: `~/.config/venv/ansible/bin/python`
* python activatioon script: `~/.config/venv/ansible/bin/activate`
* ansible navigator path: `ansible-navigator`
* completion provide redirect modules: ☑
* completino provide module option aliases: ☑
* validation: ☑
* validation lint: ☑
* validation lint path: `ansible-lint`
* redhat telemetry: ☐
