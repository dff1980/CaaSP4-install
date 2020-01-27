# CaaSP4-install


#### 5. Configure RMT.
```bash
sudo zypper in rmt-server
```
Execute RMT configuration wizard. During the server certificate setup, all possible DNS for this server has been added (RMT FQDN, etc).
Add repositories to replication.

```bash

rmt-cli sync

repos=$(rmt-cli repos list --all); for REPO in SLE-Product-SLES15-SP1-{Pool,Updates} SLE-Module-Server-Applications15-SP1-{Pool,Updates} SLE-Module-Basesystem15-SP1-{Pool,Updates} SLE-Module-Containers15-SP1-{Pool,Updates} SUSE-CAASP-4.0-{Pool,Updates}; do  rmt-cli repos enable $(echo "$repos" | grep "$REPO for sle-15-x86_64" | sed "s/^|\s\+\([0-9]*\)\s\+|.*/\1/"); done


rmt-cli mirror 
```
Download next distro:
- SLE-15-SP1-Installer-DVD-x86_64-GM-DVD1.iso

Create install repositories:

```bash
mkdir -p /usr/share/rmt/public/repo/SUSE/Install/SLE-SERVER/15-SP1/

mkdir -p /srv/tftpboot/sle15sp1

mount SLE-15-SP1-Installer-DVD-x86_64-GM-DVD1.iso /mnt
rsync -avP /mnt/ /usr/share/rmt/public/repo/SUSE/Install/SLE-SERVER/15-SP1/
cp /mnt/boot/x86_64/loader/{linux,initrd} /srv/tftpboot/sle15sp1/
umount /mnt

```

#Get autoyast
```bash
sudo SUSEConnect -p sle-module-containers/15.1/x86_64
sudo SUSEConnect -p caasp/4.0/x86_64 -r {Registarion Key}
sudo zypper in -t pattern SUSE-CaaSP-Management
```
/usr/share/caasp/autoyast/bare-metal/autoyast.xml
