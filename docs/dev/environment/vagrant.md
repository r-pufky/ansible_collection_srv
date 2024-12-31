# Setup Vagrant Environment
Vagrant is only used to manage [VirtualBox](virtualbox.md) VMs.

## Install Packages
Prerequisite:
* [ansible environment](ansible.md)
* [virtualbox](virtualbox.md)

``` bash
source {VENV}/bin/activate
pamac install vagrant
pip install molecule-plugins[vagrant]
```
[Reference](https://wiki.archlinux.org/title/Vagrant)

## Set alternative storage location (optional)
Developing on vagrant will thrash disk especially when running molecule with
large (multi-GB) reads and writes. Relocate high-use directories to a disk that
can handle high wear. Prefer to config change as this enables quick use without configuration changes.

Move and link user directories to an alternative location:
``` bash
# delete or move existing cache data.
ls -s /mnt/cache/local/share/containers ${HOME}/.local/share/containers  # graph
```
* Consider moving and linking entire `.cache` directory.

## Troubleshooting

### A Vagrant environment or target machine is required to run this command.
Virtualbox kernel modules are likely not loaded.

``` bash
A Vagrant environment or target machine is required to run this
command. Run `vagrant init` to create a new Vagrant environment. Or,
get an ID of a target machine from `vagrant global-status` to run
this command on. A final option is to change to a directory with a
Vagrantfile and to try again.
```

``` bash
pamac install linux$(mhwd-kernel -l)-virtualbox-host-modules  # for current kernel
reboot
```

After reboot confirm that each command works:
``` bash
vboxmanage --version  # should be executable, with modules loaded
> 7.1.4r165100
vagrant global-status
vagrant up --provider virtualbox  # will detail why it is not running.
```
[Reference](virtualbox.md)
