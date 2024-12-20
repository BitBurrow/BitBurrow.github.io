---
title: How to set up a BitBurrow hub
---

## NOTE: THIS SOFTWARE DOES NOT EXIST YET

Everything below is proposed draft documentation. The software to do what is described is being developed and is not at all usable yet.

## Introduction

This page will help you set up a BitBurrow hub. For an introduction, see [BitBurrow overview](index.md).

## What you will need

1. Some background in Linux system administration and working on the command line.
1. A Linux server with a fixed, public IPv4 address, e.g. from [DigitalOcean](https://www.digitalocean.com/), [Linode](https://www.linode.com/), or other VPS provider. BitBurrow can share the server with other services so long as TCP ports 53 and 8443, and UDP port 53 are available.
1. A domain or subdomain name with the ability to host your own DNS (i.e. add an NS record for the subdomain). The nameserver service provided by some domain name registrars does not allow adding NS records for subdomains. This was true of Hover as of March, 2023. To work around this, yuu will need to use a different nameserver service, such as DigitalOcean, for the entire domain.
1. Time to manage a public server long-term.

## Steps

### Acquire a domain name

Purchase your new domain name from a [domain name registrar](https://en.wikipedia.org/wiki/Domain_name_registrar) such as [Hover](https://www.hover.com/) or [Google Domains](https://domains.google/). Alternatively, create a new subdomain for an existing domain such as your company or organization. For example, if your organization owns `example.org`, you could set up `vxm.example.org` to use for BitBurrow, where `vxm` is a subdomain you choose.

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

This section is recommended but not required. A container helps with security by isolating BitBurrow hub from other applications running on the same server, provides a firewall from the internet, and allows the hub to run processes as root without actually being root. Additionally, to simplify the installation process below, you can run Ubuntu inside the container even if the host uses another Linux flavor.

After these steps, you should have a command prompt *in* the container.

1. Install LXD:
    ```
    sudo snap install lxd
    ```
1. Initialize LXD--choose **one** of these lines; the first one will prompt you with options:
    ```
    lxd init
    lxd init --auto --storage-backend=btrfs --storage-create-loop=10
    ```
1. Create a new container and connect to it:
    ```
    VMNAME=bitburrow  # 'bitburrow' is the name of the new container (change it if you like)
    lxc launch ubuntu: $VMNAME
    lxc exec $VMNAME -- sudo -u ubuntu -i  # if this fails, wait 20 seconds and try again
    ```

### Install BitBurrow hub

In the commands below, replace `vxm.example.org` with your BitBurrow hub domain name and replace `1.2.3.4` with the public IP address of your BitBurrow hub host machine. At the `BECOME password` prompt (after the `ansible-playbook` command), enter your `sudo` password, or just press enter if there isn't one, which is the default for `lxc`. In most cases, the `ansible-playbook` command will fail at `Test BIND and DNS config` the first time it is run. This is normal.

1. Download the installer (run inside container):
    ```
    rm -f preinstall.sh
    wget https://raw.githubusercontent.com/BitBurrow/BitBurrow/main/hub_installer/preinstall.sh
    sudo bash preinstall.sh
    ```
1. Run Ansible (run inside container; remember above-mentioned replacements):
    ```
    cd ~/hub
    ansible-playbook -i localhost, install.yaml --extra-vars "domain=vxm.example.org ip=1.2.3.4"
    ```
1. If the script fails with "Missing sudo password", run it again with `--ask-become-pass ` before `install.yaml`.
1. If the script fails for any other reason *prior* to the `Test BIND and DNS config` step, resolve whatever caused the failure (the `debugging` tips in `install.yaml` may be useful) and rerun the above `ansible-playbook` line

### Forward ports to the container

Skip this section if you are not using a container.

1. Type `exit` and enter to exit the container.
1. Download and run the script generated above (run on host):
    ```
    VMNAME=bitburrow  # use the same name you used for this above
    lxc file pull $VMNAME/home/bitburrow/set_port_forwarding.sh /tmp/set_port_forwarding.sh
    /tmp/set_port_forwarding.sh
    ```
1. The above should display several lines beginning `Device`
1. Connect to the container again (run on host):
    ```
    lxc exec $VMNAME -- sudo -u ubuntu -i
    ```

### Make BitBurrow hub the authoritative name server

1. **Do this only if you have a new subdomain of an existing domain for your organization or company.** Configure the nameserver for `example.org` (usually done through the website of your domain name registrar) to have two additional records. Substitute `vxm.example.org` with your domain. First, add an NS record (short for name server) for `vxm.example.org` which points to `vxm.example.org`. This tells the internet that `vxm.example.org` (your BitBurrow hub) is the authoritative nameserver for all of it's subdomains, e.g. `foxes.vxm.example.org`. Use the default TTL value. Second, add an A record for `vxm.example.org` which points to the IP address of your host. This is called a "glue record" ([more info](https://en.wikipedia.org/wiki/Domain_Name_System#Circular_dependencies_and_glue_records)). Here is a partial zone file for `example.org` which represents these two records:
	```
	vxm.example.org. 86400 IN NS vxm.example.org.
	vxm.example.org. 3600 IN A 1.2.3.4
	```
1. **Do this only if you have a domain exclusively for BitBurrow hub.** Configure `example.org` (usually done through the website of your domain name registrar) to have it's NS (name server) records point to the IP address of your BitBurrow hub. This tells the internet that `example.org` (your BitBurrow hub) is the authoritative nameserver for all of it's subdomains, e.g. `foxes.example.org`.

### Finish installing BitBurrow hub

1. Run Ansible again (run inside container):
    ```
    cd ~/hub
    ansible-playbook -i localhost, install.yaml
    ```
1. If the script fails with "Missing sudo password", run it again with `--ask-become-pass ` before `install.yaml`.
1. If the script fails for another reason, resolve whatever caused the failure (the `debugging` tips in `install.yaml` may be useful) and rerun the above `ansible-playbook` line

### Configure ssh directly to container (optional)

In the steps below, replace `vxm.example.org` with your BitBurrow hub domain and `18962` with a port number of your choosing.

This assumes `authorized_keys` on the host is already configured for public-key ssh authentication.

1. Set up ssh to container (run on host):
    ```
    VMNAME=bitburrow
    lxc exec $VMNAME -- apt install -y ssh
    cat ~/.ssh/authorized_keys |lxc exec $VMNAME -- sudo -u ubuntu bash -c 'cd; mkdir -p .ssh; cat - >> ~/.ssh/authorized_keys; chmod go-w . .ssh; chmod ugo-x,go-w .ssh/authorized_keys'
    lxc config device add $VMNAME ssh proxy listen=tcp:0.0.0.0:18962 connect=tcp:127.0.0.1:22
    ```
1. On your personal computer, edit `~/.ssh/config` and add:
    ```
    Host bitburrow
        HostName vxm.example.org
        Port 18962
        User ubuntu
    ```

### Create a BitBurrow hub administrator account and coupon code

1. Add the admin account (run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/bitburrow/.venv/bin/bbhub --create-admin-account
    ```
1. Write down this login key in a safe place. It is effectively the master username and password for this BitBurrow hub.
1. Create a coupon code (run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/bitburrow/.venv/bin/bbhub --create-coupon-code
    ```
1. Write down this coupon. It is what you will distribute to others to set up their own VPN server.

### Restart the container and test

1. Reboot (run inside the container):
    ```
    sudo reboot
    ```
1. Test a connection from the client app
