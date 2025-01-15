# Setup Podman Environment
Rootless podman environment is used to test **all** cases unless there are
required bare-metal cases (kernel, firmware, advanced networking, etc).

## Install Packages
Prerequisite:
* [ansible environment](ansible.md)

``` bash
source {VENV}/bin/activate
pip install molecule-plugins[podman]
pacman -Syu crun  # OCI implementation (faster, less memory than runc)
pacman -Syu podman  # service testing (non kernel, sysctl, networking, etc)
```
[Reference](https://github.com/ansible-community/molecule-podman)

## Setup Rootless Podman

### Verify Rootless Support
``` bash
podman info | grep -i overlay

> 107:  graphDriverName: overlay
> 114:    Native Overlay Diff: "true"
```
`overlay` and `Diff: "true"` means supported.

### Verify Unprivileged User Namespace Enabled
``` bash
sysctl kernel.unprivileged_userns_clone

> kernel.unprivileged_userns_clone = 1
```
`1` means enabled.

### Create Subordinate UID/GID Mappings
Configuration entry must exist for each user that wants to use it. New users
created using useradd have these entries by default. If not add user defaults:

``` bash
cat /etc/subuid | grep -ri {USER}
> {USER}:100000:65536
cat /etc/subgid | grep -ri {GROUP}
> {GROUP}:100000:65536
```

``` bash
usermod --add-subuids 100000-165535 --add-subgids 100000-165535 {USER}
```

[Reference](https://github.com/systemd/systemd/issues/21952)

### Set Default Container Registry for Podman
`0644 root:root` /etc/containers/registries.conf
``` ini
[registries.search]
registries = ['ghcr.io']
unqualified-search-registries=['ghcr.io']
```

The github container repository requires logging in with your github account.
``` bash
podman login
```

[Reference](https://halukkarakaya.medium.com/how-to-configure-default-search-registries-in-podman-ea930289692)

## Set alternative container storage location (optional)
Developing on containers will thrash disk especially when running molecule.
Relocate high-use directories to a disk that can handle high wear. Prefer to
config change as this enables quick use without configuration changes.

`graphroot` and `runroot` are **ignored** in rootless containers and use
following defaults if not defined:
``` bash
XDG_CONFIG_HOME=${HOME}/.config
XDG_DATA_DIR=${HOME}/.local/share
XDG_RUNTIME_DIR=/run/user/${UID}
```

Move and link user directories to an alternative location:
``` bash
# delete or move existing cache data.
# rootless relocation
ls -s /mnt/cache/local/share/containers ${HOME}/.local/share/containers  # graph
ln -s /mnt/cache/cache/containers ${HOME}/.cache/containers  # config
# podman relocation
ln -s /hdd/cache/storage /var/lib/containers/storage  # graph
```
* Consider moving and linking entire `.cache` directory.

[Reference](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md#user-configuration-files)

## Migrate Podman Installation
Applies configuration changes to Podman. Critical for unprivileged podman to
execute properly. Run as the **current** user:

``` bash
podman system reset
podman system migrate
```
[Reference](https://github.com/containers/podman/issues/12715)

## References

[Testing with Molecule and Podman](https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1/)

[Podman](https://wiki.archlinux.org/title/Podman)
