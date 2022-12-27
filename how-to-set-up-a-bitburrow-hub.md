---
title: How to set up your own BitBurrow hub
---

## NOTE: THIS SOFTWARE DOES NOT EXIST YET

Everything below is proposed draft documentation. The software to do what is described is being developed and is not at all usable yet.

## Introduction

BitBurrow is a set of tools to help you set up and use a VPN server anywhere--at your parents' house, an office, or a friend's apartment.

BitBurrow has two major components--an app and a hub. ...

## What you will need

1. Some background in Linux system administration and working on the command line.
1. A Linux server with a fixed, public IPv4 address, e.g. from [DigitalOcean](https://www.digitalocean.com/), [Linode](https://www.linode.com/), or other VPS provider. BitBurrow can share the server with other services so long as TCP port 8443 is available.
1. A domain or subdomain name with the ability to host your own DNS (i.e. add an NS record for the subdomain).
1. Time to manage a public server long-term.

## Why would you want to do this

1. 

## Steps

### Create a domain

Purchase your new domain name from a [domain name registrar](https://en.wikipedia.org/wiki/Domain_name_registrar) such as [Hover](https://www.hover.com/) or [Google Domains](https://domains.google/), **or** create a new subdomain for an existing domain such as your company or organization.

When choosing a domain name or subdomain, it is recommended that it *not* contain `vpn`, `proxy`, `bitburrow`, `gateway`, or similar words. Public WiFi networks that restrict VPN usage may block domain names that include these. Also, choosing a name that is easy to type will make it easier for admins setting up VPN servers.

### Set up the Linux server

These instructions are written for Ubuntu. If you are setting up a new server, use the most recent Ubuntu LTS server image.

1. Sign in to the Linux server, preferably as a normal user with `sudo` privileges.
1. Configure automatic security updates ([more information](https://help.ubuntu.com/community/AutomaticSecurityUpdates)):
    ```
    sudo apt update
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure --priority=low unattended-upgrades  # choose Yes when prompted
    ```

### Create a container

This section is recommended but not required. It helps with security by isolating the BitBurrow hub from other applications running on the same server, provides a firewall from the internet, and allows the hub to run processes as root without actually being root.

1. Install LXD: `sudo snap install lxd`
1. Initialize LXD: `lxd init` and answer the prompts (accept the default where you are unsure)
1. Create a new container, configure ports for VPN servers, and sign in:
    ```
    VMNAME=bitburrow  # 'bitburrow' is the name of the new container (change it if you like)
    lxc launch ubuntu: $VMNAME
    lxc exec $VMNAME -- sudo -u ubuntu -i  # now you are in the container
    ```
1. Configure automatic security updates for the container as well as the host (run inside container):
    ```
    sudo apt update
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure --priority=low unattended-upgrades  # choose Yes when prompted
    ```

### Install BitBurrow hub

1. Install dependencies (run inside container):
    ```
    sudo apt update
    sudo apt install -y python3-pip ssh wireguard-tools sqlite3
    sudo adduser --disabled-password --gecos "" --quiet bitburrow
    printf "bitburrow  ALL = NOPASSWD: /usr/bin/wg *\nbitburrow  ALL = NOPASSWD: /bin/ip *\nbitburrow  ALL = NOPASSWD: /usr/sbin/sysctl *\nbitburrow  ALL = NOPASSWD: /usr/sbin/iptables *\n" |sudo tee /etc/sudoers.d/bitburrow >/dev/null
    ```
1. Install BitBurrow hub (run inside container):
    ```
    sudo -u bitburrow python3 -m pip install --no-warn-script-location git+https://github.com/BitBurrow/BitBurrow.git@main
    ```
### Run automatically at startup

1. Configure (run inside container):
    ```
    cd /tmp
    printf "[Unit]\nDescription=BitBurrow\n" >bitburrow.service
    printf "Documentation=https://bitburrow.com\n" >>bitburrow.service
    printf "After=network.target network-online.target\n\n" >>bitburrow.service
    printf "[Service]\nType=exec\nRestartSec=2s\nUser=bitburrow\n" >>bitburrow.service
    printf "Group=bitburrow\nWorkingDirectory=/home/bitburrow\n" >>bitburrow.service
    printf "ExecStart=/home/bitburrow/.local/bin/bbhub --api\n" >>bitburrow.service
    printf "Restart=always\n" >>bitburrow.service
    printf "PrivateTmp=true\nPrivateDevices=false\nNoNewPrivileges=false\n\n" >>bitburrow.service
    printf "[Install]\nWantedBy=multi-user.target\n" >>bitburrow.service
    sudo mv bitburrow.service /lib/systemd/system/bitburrow.service
    sudo systemctl enable /lib/systemd/system/bitburrow.service
    sudo systemctl daemon-reload
    sudo systemctl enable bitburrow
    sudo systemctl start bitburrow
    ```
1. Verify (run inside container):
    * Reboot (normally `sudo reboot`), wait for system to come back up, and sign in again.
    * Check that the service is running: `sudo systemctl status bitburrow`
    * If there are errors (e.g. `code=exited, status=3`), use `journalctl -u bitburrow` to help diagnose.


### Configure your domain and ports

1. Configure your domain (run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-domain 〚your BitBurrow domain〛
    ```
1. To view the configured domain, replace `set` with `get` in the above line.
1. Customize ports (optional; run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-ssh-port 〚your ssh port〛
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-wg-port 〚your WireGuard port〛
    ```
1. Forward ports from host to container (skip if not using a container; run on host):
    ```
    VMNAME=bitburrow  # use the same value used above
    bbhub() { lxc exec $VMNAME -- sudo -u bitburrow /home/bitburrow/.local/bin/bbhub "$1" }
    APIPORT=8443  # hard-coded in app
    SSHPORT=$(bbhub --get-ssh-port)
    WGPORT=$(bbhub --get-wg-port)
    lxc config device add $VMNAME apiport proxy listen=tcp:0.0.0.0:$APIPORT connect=tcp:127.0.0.1:$APIPORT
    lxc config device add $VMNAME sshport proxy listen=tcp:0.0.0.0:$SSHPORT connect=tcp:127.0.0.1:$SSHPORT
    lxc config device add $VMNAME wgport proxy listen=udp:0.0.0.0:$WGPORT connect=udp:127.0.0.1:$WGPORT
    ```
