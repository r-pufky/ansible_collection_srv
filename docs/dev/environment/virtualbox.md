# Setup VirtualBox Environment
Virtual machines are **ONLY** used to test cases which cannot be tested in
containers (kernel, firmware, advanced networking, etc); these should **never**
be `default` test cases.

## Install Packages
Prerequisite:
* [ansible environment](ansible.md)

``` bash
source {VENV}/bin/activate
pamac install virtualbox  # no additional dependencies
pamac install linux$(mhwd-kernel -l)-virtualbox-host-modules  # current kernel
pip install molecule-plugins[virtualbox]
gpasswd -a ${USER} vboxusers  # add user to virtualbox users
```
[Reference](https://wiki.manjaro.org/index.php/VirtualBox)

If testing `USB`, `webcam`, `RDP`, `PXE`, `disk encryption`, or `NVMe`:
``` bash
pamac build virtualbox-ext-oracle
```

**Reboot**
``` bash
modprobe vboxdrv vboxnetadp xvboxnetflt  # if no reboot wanted
```

After reboot confirm that each command works:
``` bash
vboxmanage --version
> 7.1.4r165100
```

## Set alternative VM storage location (optional)
Developing on VMs will thrash disk especially when running molecule. Relocate
high-use directories to a disk that can handle high wear. Prefer to config
change as this enables quick use without configuration changes.

Move and link user directories to an alternative location:
``` bash
# delete or move existing cache data.
ls -s /mnt/cache/config/VirtualBox ${HOME}/.config/VirtualBox  # config
ln -s '/mnt/cache/VirtualBox VMs' '${HOME}/VirtualBox VMs'  # VMs
```
[Reference](
https://docs.oracle.com/en/virtualization/virtualbox/6.0/admin/vboxconfigdata.html)

## VM repositories
* [hashicorp.com](https://portal.cloud.hashicorp.com/vagrant/discover/debian/bookworm64)
* [osboxes.org](https://www.osboxes.org/debian/)

## Troubleshooting

### vboxdrv kernel module is not loaded.
Install kernel modules and reboot / modprobe.

``` bash
WARNING: The vboxdrv kernel module is not loaded. Either there is no module
         available for the current kernel (4.4.0-22-generic) or it failed to
         load. Please recompile the kernel module and install it by

           sudo /sbin/rcvboxdrv setup

         You will not be able to start VMs until this problem is fixed.
```

``` bash
pamac install linux$(mhwd-kernel -l)-virtualbox-host-modules  # for current kernel
reboot
```

After reboot confirm that each command works:
``` bash
vboxmanage --version
> 7.1.4r165100
```
