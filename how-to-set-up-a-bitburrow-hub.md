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
1. A domain or subdomain name with the ability to change the DNS name servers to point to the BitBurrow hub.
1. Time to manage a public server long-term.

## Why would you want to do this

1. 

## Steps

### Set up the Linux server

These instructions are written for Ubuntu. If you are setting up a new server, use the most recent Ubuntu LTS server image.

1. Sign in to the Linux server, preferably as a normal user with `sudo` privileges.
1. Configure automatic security updates ([more information](https://help.ubuntu.com/community/AutomaticSecurityUpdates)):
    ```
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure --priority=low unattended-upgrades
    ```

### Create a container

This section is recommended but not required. It helps with security by isolating the BitBurrow hub from other applications running on the same server, provides a firewall from the internet, and allows the hub to run processes as root without actually being root.

1. Install LXD: `sudo snap install lxd`
1. Initialize LXD: `lxd init` and answer the prompts (accept the default where you are unsure)
1. Create a new container, configure ports for VPN servers, and sign in:
    ```
    VMNAME=bitburrow  # 'bitburrow' is the name of the new container (change it if you like)
    lxc launch ubuntu: $VMANME
    APIPORT=8443  # hard-coded in app
    SSHPORT=$(shuf --input-range=8444-65535 -n 1)  # random port for ssh (change it if you like)
    WGPORT=$(shuf --input-range=2000-65535 -n 1)  # random port for WireGuard (change it if you like)
    lxc config device add $VMNAME apiport proxy listen=tcp:0.0.0.0:$APIPORT connect=tcp:127.0.0.1:$APIPORT
    lxc config device add $VMNAME sshport proxy listen=tcp:0.0.0.0:$SSHPORT connect=tcp:127.0.0.1:$SSHPORT
    lxc config device add $VMNAME wgport proxy listen=udp:0.0.0.0:$WGPORT connect=udp:127.0.0.1:$WGPORT
    lxc exec $VMNAME -- sudo -u ubuntu -i
    ```

### Install BitBurrow hub

1. Install required software:
    ```
    sudo apt update
    sudo apt install -y python3-pip ssh wireguard-tools sqlite3
    sudo adduser --disabled-password --gecos "" --quiet bitburrow
    printf "bitburrow  ALL = NOPASSWD: /usr/bin/wg *\nbitburrow  ALL = NOPASSWD: /bin/ip *\nbitburrow  ALL = NOPASSWD: /usr/sbin/sysctl *\nbitburrow  ALL = NOPASSWD: /usr/sbin/iptables *\n" |sudo tee /etc/sudoers.d/bitburrow >/dev/null
    sudo -u bitburrow python3 -m pip install --no-warn-script-location git+https://github.com/BitBurrow/BitBurrow.git@main
    ```
1. Run automatically at startup:
    ```
    cd /tmp
    printf "[Unit]\nDescription=BitBurrow\n" >bitburrow.service
    printf "Documentation=https://bitburrow.com\nAfter=network.target network-online.target\n\n" >>bitburrow.service
    printf "[Service]\nType=exec\nRestartSec=2s\nUser=bitburrow\nGroup=bitburrow\nWorkingDirectory=/home/bitburrow\n" >>bitburrow.service
    printf "ExecStart=/home/bitburrow/.local/bin/bbhub\nRestart=always\n" >>bitburrow.service
    printf "PrivateTmp=true\nPrivateDevices=false\nNoNewPrivileges=false\n\n" >>bitburrow.service
    printf "[Install]\nWantedBy=multi-user.target\n" >>bitburrow.service
    sudo mv bitburrow.service /lib/systemd/system/bitburrow.service
    sudo systemctl enable /lib/systemd/system/bitburrow.service
    sudo systemctl daemon-reload
    sudo systemctl enable bitburrow
    sudo systemctl start bitburrow
    ```
1. Verify:
    * Reboot (normally `sudo reboot`), wait for system to come back up, and sign in again.
    * Check that the service is running: `sudo systemctl status bitburrow`
    * If there are errors (e.g. `code=exited, status=3`), use `journalctl -u bitburrow` to help diagnose.




