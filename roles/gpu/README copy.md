* PVE nodes: /etc/subuid,subgid mappings
* Create LXC container - DO NOT START.
* Add LXC.idmaps -  /etc/pve/lxc/{ID}.conf
* Start container, login, enable root password logins /etc/ssh/sshd_config
  systemctl restart ssh
* init LXC container - tags root
* media LXC container - no options

/tmp - mounted in RAM in debian role.

TODO(plex): The PLEX Media cache is ~300GB, 1GB DB, and 30-50GB (of the 300GB)
            as cache files. The Media/ directory contains all the .bif files
            used for seeking media -- this is disposable but needs to be
            reindexed too.

TODO(xdl): map /d/srv - and setup job to automatically backup the databases to
    this location for all services. All services use LOCAL disk for standard
    execution with a backup to a mounted directory.

TODO(roles): LXC containers should set IMMUTABLE autofs, so writes fail if NFS
    isn't mount (these are replaced on start)

TODO(roles): LXC passthrough - set DRI permissions as well
https://bookstack.swigg.net/books/linux/page/lxc-gpu-access
https://forum.proxmox.com/threads/proxmox-lxc-igpu-passthrough.141381/
-- see hv.md for original config notes.

chmod -R 0666 /dev/dri/* -- promox SERVER
usermod --append --groups video,render root -- LXC container

main disk - standard debian install
plex disk - /var/lib/plexmediaserver/ - 350GB - plex install only.

----
GPU passthrough

https://forum.proxmox.com/threads/proxmox-lxc-igpu-passthrough.141381/

# Not used.
##apt install intel-opencl-icd  # GUC/GVT - mediated GPU passthrough https://wiki.archlinux.org/title/Intel_GVT-g https://wiki.archlinux.org/title/Intel_graphics

/etc/default/grub
  GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt i915.enable_gvt=1"

upgrade-grub
dmesg | grep -e DMAR -e IOMMU  # reboot then confirm IOMMU enabled.

/etc/modules
  # virtual IO for hardware access
  vfio
  vfio_iommu_type1
  vfio_pci
  # vfio_virqfd  # not required on 6.2+ kernel
  # passthrough for GUC/GVT
  kvmgt
  # xengt  # typo in the PVE listing
  # vfio-mdev  # requires mdevctl package

update-initramfs -u -k all
lspci -nnv | grep VGA  # should report card

EXISTING INSTRUCTIONS IN HV.MD work -- see notes above; condense and remove.

Verify PLEX working, backup Preferences, re-appply config ONLY; then remove
"old" -- the original install. Disable legacy.
----
# Plex Media Server container
#
# Nvidia GPU Passthru
#
# Map media%3Amedia (5555%3A5555) directly
# Map %3Avideo (44) directly
# Map %3Arender (106 -> 104) directly
#lxc.idmap%3A g 0 100000 5555
# map UID 0-5555 to host 100000-105554
# map GID 0-44 to host 100000-100043
# map GID 44 to host 44
# map GID 45-106 to host 100045-100105
# map GID 106 to host 104
# map GID 107-5554 to host 100107-105554
# map UID 5555 to host 5555
# map GID 5555 to host 5555
# map UID 5556-65536 to host 165536
# map GID 5556-65536 to host 165536
arch: amd64
cores: 4
features: nesting=1
hostname: xplex
memory: 5120
mp0: /autofs/media-ro,mp=/data/media,ro=1
mp1: /autofs/movies-ro,mp=/data/media/movies,ro=1
mp2: /autofs/tv-ro,mp=/data/media/tv,ro=1
mp3: /autofs/music-ro/final,mp=/data/final,ro=1
mp4: /autofs/srv/plex,mp=/data/plex
nameserver: 10.5.5.1
net0: name=eth0,bridge=vmbr0,firewall=1,gw=10.5.5.1,hwaddr=02:06:52:51:03:CF,ip=10.5.5.225/24,tag=5,type=veth
onboot: 0
ostype: debian
rootfs: local-lvm:vm-850-disk-0,size=4G
searchdomain: pufky.com
startup: order=850
swap: 512
unprivileged: 1
# lxc.cgroup2.devices.allow: c 226:0 rwm
# lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 226:* rwm
#lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file,mode=0666
lxc.mount.entry: /dev/dri/card1 dev/dri/card1 none bind,optional,create=file,mode=0666
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.idmap: u 0 100000 5555
lxc.idmap: g 0 100000 44
lxc.idmap: g 44 44 1
lxc.idmap: g 45 100045 61
lxc.idmap: g 106 104 1
lxc.idmap: g 107 100107 5448
lxc.idmap: u 5555 5555 1
lxc.idmap: g 5555 5555 1
lxc.idmap: u 5556 105556 59980
lxc.idmap: g 5556 105556 59980