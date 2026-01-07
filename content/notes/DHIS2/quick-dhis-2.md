+++
title = "Quick DHIS 2" 
+++

Needed a quick DHIS 2 for testing something - here is an aide-mémoire of doing it with [dhis2/dhis2-server-tools: Tools to support installation and management of DHIS2](https://github.com/dhis2/dhis2-server-tools) on a new Ubuntu 22.04 VM (4GB RAM, 2 vCPU, 20GB storage, static IP address 192.168.1.104) on Proxmox.

As root in the new VM:

```bash
apt-get update && apt-get -y upgrade
apt-get install -y qemu-guest-agent  # only if you use this
sed -i -e 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' -e 's/^PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd
useradd -m -s /usr/bin/bash dhis
passwd dhis
usermod -a -G sudo dhis
```

Log out and log in as `dhis` user:

```bash
sudo ufw limit 22
sudo ufw enable
git clone https://github.com/dhis2/dhis2-server-tools
cp dhis2-server-tools/deploy/inventory/{hosts.template,hosts}
vim dhis2-server-tools/deploy/inventory/hosts  # just set email and timezone
cd dhis2-server-tools/deploy/
sudo ./deploy.sh
```

It then runs a long Ansible playbook to install DHIS 2, the database, a reverse proxy and some monitoring.

You can see the containers with:

```bash
sudo lxc list
```
```
+----------+---------+--------------------+------+-----------+-----------+
|   NAME   |  STATE  |        IPV4        | IPV6 |   TYPE    | SNAPSHOTS |
+----------+---------+--------------------+------+-----------+-----------+
| dhis     | RUNNING | 172.19.2.11 (eth0) |      | CONTAINER | 0         |
+----------+---------+--------------------+------+-----------+-----------+
| monitor  | RUNNING | 172.19.2.30 (eth0) |      | CONTAINER | 0         |
+----------+---------+--------------------+------+-----------+-----------+
| postgres | RUNNING | 172.19.2.20 (eth0) |      | CONTAINER | 0         |
+----------+---------+--------------------+------+-----------+-----------+
| proxy    | RUNNING | 172.19.2.2 (eth0)  |      | CONTAINER | 0         |
+----------+---------+--------------------+------+-----------+-----------+
```

On my local network I can access:

- https://192.168.1.104/dhis
- https://192.168.1.104/munin
- https://192.168.1.104/dhis-glowroot

User name is "admin" and password is "district" for all.

I then exported some organisation unit metadata from our production instance and imported into my local test instance. 

You can log into a container with e.g.:

```bash
sudo lxc exec proxy -- bash
```

Still haven't found a way to use a Cloudflare Tunnel with this (in case I wanted to do some testing from the office) but good to have it locally anyway.

