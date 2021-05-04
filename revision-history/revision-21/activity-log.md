# Rev. 20 Activity Log

## infra-server upgrade

```
apt update
apt upgrade
reboot
apt remove kubectl
apt install kubectl
do-release-upgreade

    Third party sources disabled

    Some third party entries in your sources.list were disabled. You can
    re-enable them after the upgrade with the 'software-properties' tool
    or your package manager.

ubuntu@infra-server:~$ uname -a
Linux infra-server 5.4.0-1041-aws #43-Ubuntu SMP Fri Mar 19 22:06:16 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

ubuntu@infra-server:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.2 LTS
Release:	20.04
Codename:	focal

```

Replaced /etc/rc.local file with:
```
#!/bin/bash

ip address add 10.1.20.20/24 dev ens4
ip address add 10.1.20.30/24 dev ens4
ip link set dev ens4 up

## ..:: Max Map Count for VM when runnign ELK in a Docker container ::..
sysctl -w vm.max_map_count=262144

exit 0
```

Replaced /etc/netplan/50-cloud-init.yaml file with
```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens3:
            dhcp4: true
            dhcp4-overrides:
                route-metric: 100
            dhcp6: false
            match:
                macaddress: 06:0c:d4:8e:07:d3
            set-name: ens3
    version: 2
```

Added the following line to the /etc/dnsmasq.conf file 

    listen-address=10.1.1.4,10.1.20.20

and restart dnsmasq server

    sudo rm /etc/dnsmasq.d/lxd
    sudo systemctl restart dnsmasq

```
vim /lib/systemd/system/docker.service
systemctl daemon-reload
systemctl restart docker

openssl req -addext "subjectAltName = DNS:registry.f5-udf.com" -newkey rsa:4096 -nodes -sha256 -keyout registry-san.key -x509 -days 3650 -out registry-san.crt

sudo cp /home/ubuntu/registry-san.crt  /home/ubuntu/dockerhost-storage/registry//ca-certificates/.

cp registry-san.* f5-udf-infra-server/ansible/playbooks/files/docker-files/.
sudo cp /home/ubuntu/registry-san.crt  /usr/local/share/ca-certificates/.
sudo update-ca-certificates

sudo cp /home/ubuntu/registry-san.*  /etc/rancher/k3s/.
vim /etc/rancher/k3s/registries.yaml
===

mirrors:
  registry.f5-udf.com:
    endpoint:
      - "https://registry.f5-udf.com:5000"
configs:
  "registry.f5-udf.com:5000":
    tls:
      cert_file: /etc/rancher/k3s/registry-san.crt
      key_file:  /etc/rancher/k3s/registry-san.key
      insecure_skip_verify: true

===

cd ~/
docker stop f5Registry
docker rm f5Registry
./startInfraServerDockers.sh

sudo systemctl restart k3s
reboot
```

git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/
git checkout v1.11.1
vim nginx-repo.crt
vim nginx-repo.key
make debian-image-plus PREFIX=registry.f5-udf.com:5000/nginx-plus-ingress TARGET=container
make push PREFIX=registry.f5-udf.com:5000/nginx-plus-ingress
make debian-image-nap-plus PREFIX=registry.f5-udf.com:5000/nginx-plus-ingress-ap TARGET=container
make push PREFIX=registry.f5-udf.com:5000/nginx-plus-ingress-ap

**Copy new "-san" certificates and keys to each kubernetes node in the /etc/docker/certs.d/registry.f5-udf.com:5000/ and reboot**

## BIG-IP CIS

- BIG-IP was renamed to big-ip-cis.f5-udf.com
- Upgraded TMOS to 16.0.1.1

## Linux Jumphost

```
sudo apt update
sudo apt upgrade
sudo reboot
sudo su -
do-release-upgrade
```

## NGINX Controller host

Complete reinstall of 3.16.1 from scratch.

    apt update
    apt upgrade
    reboot

## NGINX Plus hosts

    apt update
    apt upgrade -y
    reboot
    do-release-upgrade