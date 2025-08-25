# Hardware


# iGPU passthrough
https://pve.proxmox.com/wiki/PCI(e)_Passthrough
https://forum.proxmox.com/threads/proxmox-lxc-igpu-passthrough.141381/  # maybe not needed

/etc/default/grub
```diff
-GRUB_CMDLINE_LINUX_DEFAULT="quiet"
+GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
```
enables intel based iommu
enables iommu passthrough for SR-IOV

/etc/modules
```diff
+vfio
+vfio_iommu_type1
+vfio_pci
+vfio_virqfd #not needed if on kernel 6.2 or newer
```

```bash
update-grub
update-initramfs -u -k all
```



## Enable Mediated GPU access
/etc/default/grub
```diff
-GRUB_CMDLINE_LINUX_DEFAULT="quiet"
+GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt i915.enable_gvt=1"
```
enables intel based iommu
enables iommu passthrough for SR-IOV
* if mis-typed, kernel upgrades will fail with

``` bash
dpkg: error processing package proxmox-kernel-6.8.12-5-pve-signed (--configure):
 installed proxmox-kernel-6.8.12-5-pve-signed package post-installation script subprocess returned error exit status 2
dpkg: dependency problems prevent configuration of proxmox-kernel-6.8:
 proxmox-kernel-6.8 depends on proxmox-kernel-6.8.12-5-pve-signed | proxmox-kernel-6.8.12-5-pve; however:
  Package proxmox-kernel-6.8.12-5-pve-signed is not configured yet.
  Package proxmox-kernel-6.8.12-5-pve is not installed.
  Package proxmox-kernel-6.8.12-5-pve-signed which provides proxmox-kernel-6.8.12-5-pve is not configured yet.
```
* correct /etc/default/grub
* disable secure boot and boot into an older kernel
* manually install packages

https://forum.proxmox.com/threads/6-8-12-5-pve-install-problem.160573/


/etc/modules
```diff
+kvmgt
```
```bash
update-grub
update-initramfs -u -k all
```

## blacklist vidoe card modules (only if dedicating a GPU to a VM)
Only if dedicating a card to a VM, otherwise use mediated access or mounting
the device in the LXC container via cgroups and mount in config file.
```bash
lspci -k | grep -A 3 "VGA"
# use list of kernel module loaded
echo "blacklist {MODULE}" >> /etc/modprobe.d/blacklist.conf
```

# xplex 850 existing container modification
should be applied to all cluster nodes so LXC can bounce between them.
https://www.youtube.com/watch?v=0ZDr5h52OOE

cat /etc/group
video:44
render:104

cat lxc:/etc/group
video:44
render:106

shutdown LXC container

/etc/subgid
```diff
+root:44:1
+root:104:1
```

/etc/pve/lxc/850.conf
```diff
+lxc.cgroup2.devices.allow: c 226:0 rwm
+lxc.cgroup2.devices.allow: c 226:128 rwm
+lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
+lxc.idmap: u 0 100000 5555
+lxc.idmap: g 0 100000 44
+lxc.idmap: g 44 44 1
+lxc.idmap: g 45 100045 61
+lxc.idmap: g 106 104 1
+lxc.idmap: g 107 100107 5448
+lxc.idmap: u 5555 5555 1
+lxc.idmap: g 5555 5555 1
+lxc.idmap: u 5556 105556 59980
+lxc.idmap: g 5556 105556 59980
+#lxc.idmap: g 0 100000 5555
+# map UID 0-5555 to host 100000-105554
+# map GID 0-44 to host 100000-100043
+# map GID 44 to host 44
+# map GID 45-106 to host 100045-100105
+# map GID 106 to host 104
+# map GID 107-5554 to host 100107-105554
+# map UID 5555 to host 5555
+# map GID 5555 to host 5555
+# map UID 5556-65536 to host 165536
+# map GID 5556-65536 to host 165536
```
Use https://github.com/ddimick/proxmox-lxc-idmapper to generate mappings for
users (at least a broad template) `run.py 44=44 5555=5555 106=104`

add root to render,video groups for LXC passthrough
```bash
usermod -aG render,video root
```



reboot
```

