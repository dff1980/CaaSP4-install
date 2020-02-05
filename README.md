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

#### 6 Get autoyast
```bash
sudo SUSEConnect -p sle-module-containers/15.1/x86_64
sudo SUSEConnect -p caasp/4.0/x86_64 -r {Registarion Key}
sudo zypper in -t pattern SUSE-CaaSP-Management
```

```bash
mkdir /usr/share/rmt/public/autoyast
cp /usr/share/caasp/autoyast/bare-metal/autoyast.xml /usr/share/rmt/public/autoyast/autoinst_caasp.xml
```

```bash
cd /usr/share/rmt/public/
chown -R _rmt:nginx autoyast
```
get AutoYast Fingerprint

```bash
openssl x509 -noout -fingerprint -sha256 -inform pem -in /etc/rmt/ssl/rmt-ca.crt
```
Change /usr/share/rmt/public/autoyast/autoinst_caasp.xml <suse_register> (<reg_server>, <reg_server_cert_fingerprint>)
```
  <!-- register -->
  <suse_register>
    <do_registration config:type="boolean">true</do_registration>
    <install_updates config:type="boolean">true</install_updates>
    <slp_discovery config:type="boolean">false</slp_discovery>
      <reg_server>https://YOU FQDN</reg_server>
      <reg_server_cert_fingerprint_type>SHA256</reg_server_cert_fingerprint_type>
      <reg_server_cert_fingerprint>YOUR SMT FINGERPRINT</reg_server_cert_fingerprint>
    <addons config:type="list">
      <addon>
        <name>sle-module-containers</name>
        <version>15.1</version>
        <arch>x86_64</arch>
      </addon>
      <addon>
        <name>caasp</name>
        <version>4.0</version>
        <arch>x86_64</arch>
      </addon>
    </addons>
  </suse_register>
```  
Add to /etc/nginx/vhosts.d/rmt-server-http.conf and rmt-server-https.conf
```
    location /autoyast {
        autoindex on;
    }
```
```bash
systemctl restart nginx
```

Change /usr/share/rmt/public/autoyast/autoinst_caasp.xml `<ntp-client><ntp_servers><ntp_server><address>`

use `ssh-keygen` for generate ssh key pair
```
cat /root/.ssh/id_rsa.pub
```
Change /usr/share/rmt/public/autoyast/autoinst_caasp.xml `<users><user><username>sles</username><authorized_keys config:type="list"> <authorized_key>`

### Deploy SUSE CaaS Platform
add 127.0.0.1 to /etc/resolve.conf
#### configure NAT
```
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --permanent --zone=external --add-interface=eth0
firewall-cmd --permanent --zone=internal --add-interface=eth1
firewall-cmd --permanent --zone=internal --set-target=ACCEPT
firewall-cmd --reload
```
```
eval "$(ssh-agent)"
ssh-add ~/.ssh/id_rsa
```
```
skuba cluster init --control-plane 192.168.17.10 my-cluster
cd my-cluster
skuba node bootstrap --user sles --sudo --target master.caasp.local master
skuba node join --role worker --user sles --sudo --target worker-01.caasp.local worker-01
skuba node join --role worker --user sles --sudo --target worker-02.caasp.local worker-02
skuba node join --role worker --user sles --sudo --target worker-03.caasp.local worker-03
skuba node join --role worker --user sles --sudo --target worker-04.caasp.local worker-04
skuba cluster status
```
```
sudo zypper in kubernetes-client
mkdir -p ~/.kube
cp admin.conf ~/.kube/config
```
