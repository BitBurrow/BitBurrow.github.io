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

### Make BitBurrow hub the authoritative name server

In the instructions below, `YOUR_HUB_DOMAIN` is the BitBurrow hub domain name, e.g. `vxm.example.org` or `example.org`, and `YOUR_HUB_IP` is the primary public IP address of your BitBurrow hub (if you don't know, try `wget -q -O- api.bigdatacloud.net/data/client-ip |grep ipString |grep -Eo [0-9\.]+`).

1. **Do this only if you have a new subdomain of an existing domain for your organization or company.** Configure the nameserver for `YOUR_PARENT_DOMAIN` (e.g. `example.org`) to have two additional records. This is usually done through the website of your domain name registrar. First, add an NS record (short for name server) for `YOUR_HUB_DOMAIN` which points to `YOUR_HUB_DOMAIN`. This tells the internet that `YOUR_HUB_DOMAIN` (e.g. `vxm.example.org`) is the authoritative nameserver for all of it's subdomains, e.g. `foxes.vxm.example.org`. Use the default TTL value. Second, add an A record for `YOUR_HUB_DOMAIN` which points to the IP address of your host. This is called a "glue record" ([more info](https://en.wikipedia.org/wiki/Domain_Name_System#Circular_dependencies_and_glue_records)). Here is a partial zone file for `example.org` which represents these two records:
	```
	vxm.example.org. 86400 IN NS vxm.example.org.
	vxm.example.org. 3600 IN A 1.2.3.4
	```
1. **Do this only if you have a domain exclusively for BitBurrow hub.** Configure `example.org` (usually done through the website of your domain name registrar) to have it's NS (name server) records point to the IP address of your BitBurrow hub. This tells the internet that `example.org` (your BitBurrow hub) is the authoritative nameserver for all of it's subdomains, e.g. `foxes.example.org`.

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

A container helps with security by isolating BitBurrow hub from other applications running on the same server, provides a firewall from the internet, and allows the hub to run processes as root without actually being root. Additionally, to simplify the installation process below, you can run Ubuntu inside the container even if the host uses another Linux flavor.

1. Install LXD:
    ```
    sudo snap install lxd
    ```
1. Initialize LXD--choose **one** of these lines; the first one will prompt you with options:
    ```
    lxd init
    lxd init --auto --storage-backend=btrfs --storage-create-loop=10
    ```
1. Create a new container and make aliases to run commands in it. To make future management easier, add these three 'alias' lines to your `.bashrc` or similar file. Replace `YOUR_LXC_NAME` with something to identify the container, e.g. `bitburrow` or your subdomain.
    ```
    lxc launch ubuntu: YOUR_LXC_NAME
    alias bbsetup="lxc exec YOUR_LXC_NAME -- sudo -u ubuntu -i"
    alias bbadmin="lxc exec YOUR_LXC_NAME -- sudo -u bitburrow -i"
    alias bbhub="lxc exec YOUR_LXC_NAME -- sudo -u bitburrow -i /home/bitburrow/bitburrow/.venv/bin/bbhub"
    ```

### Install BitBurrow hub

1. Download and run the installer. If the script fails, comments within the script may be helpful. It is safe to rerun `preinstall.sh`. If the script fails with "Missing sudo password", run `bbsetup nano preinstall.sh` (substitute `vi` for `nano` if you like), insert ` --ask-become-pass` after `ansible-playbook`, and run `preinstall.sh` again.
    ```
    bbsetup wget https://raw.githubusercontent.com/BitBurrow/BitBurrow/main/hub_installer/preinstall.sh
    bbsetup sudo bash preinstall.sh  YOUR_HUB_DOMAIN  YOUR_HUB_IP
    ```
1. Configure. When editing `config.yaml`, note that the `help` section at the top is documentation; edit the data near the bottom to match your set-up.
    ```
    bbadmin nano .config/bitburrow/config.yaml
    ```
1. Forward ports to the container. If the public-facing IP is not on the computer hosting the LXC, before running it, you will need to edit `/tmp/pfscript`, replace every occurrence of the public IP with the host's internet-facing IP, and forward those ports from the public-facing router.
    ```
    bbhub port-forward-script >/tmp/pfscript
    bash /tmp/pfscript
    ```
1. Test. Rerun this until all tests pass. The most common problem is the configuration at the domain name registrar. If possible, download a zone file and compare it to the 2-line example above.
    ```
    bbhub test all
    ```

### Create a BitBurrow hub administrator account and coupon code

1. Add an admin account:
    ```
    bbhub -q create-admin-account
    ```
1. Write down this login key in a safe place. It is effectively the master username and password for this BitBurrow hub.
1. Create a coupon code (run inside container):
    ```
    bbhub -q create-coupon-code
    ```
1. Write down this coupon. It is what you will distribute to others to set up their own VPN server.

### Restart the container and test

1. Reboot:
    ```
    bbsetup sudo reboot
    ```
1. You will probably need to configure and restart your reverse proxy service.
1. Test a connection from the client app (`https://YOUR_HUB_DOMAIN/welcome`)

### Upgrade BitBurrow

This will pull updates from GitHub, but it will also temporarily disrupt users who are using the server:
    ```
    bbsetup sudo systemctl stop bitburrow
    bbadmin git -C bitburrow pull
    bbadmin poetry --directory bitburrow install
    bbsetup sudo systemctl start bitburrow
    ```

## Notes for developers

The BitBurrow hub runs [NiceGUI](https://nicegui.io/), which uses FastAPI and Uvicorn.

### Set-up steps

* set up a test server via [How to set up a BitBurrow hub](https://bitburrow.com/hub/)
* (optional) fork the BitBurrow project on GitHub
* clone the repo to your local computer
    ```
    cd /your/code/repo
    mkdir bitburrow && cd $_
    git clone https://github.com/BitBurrow/BitBurrow.git .  # or your BitBurrow fork
    ```
* configure the pre-commit hooks:
    ```
    git config core.hooksPath $(git rev-parse --show-toplevel)/git_hooks
    ```
* so we can all agree on code formatting, set up [Black](https://pypi.org/project/black/) in your code editor with the options `--line-length 100 --skip-string-normalization` (a pre-commit hook will warn you if `black --check ...` fails)
* edit code

### Testing your changes

* stop the BitBurrow hub service on your_hub (as user ubuntu or root):
    ```
    sudo systemctl stop bitburrow
    ```
* copy your modified files to `~/bitburrow/` on your_hub via something similar to:
    ```
    rsync -av --delete --files-from <(git ls-files; git ls-files --others --exclude-standard) \
        ./ bitburrow@your_hub:bitburrow/
    ```
* reinstall:
    ```
    cd ~/bitburrow/
    poetry install
    ```
* run BitBurrow hub in debug mode:
    ```
    bitburrow/.venv/bin/bbhub -vv serve
    ```

### Berror codes

A Berror code, e.g. 'B46373', is a 'B' followed by a 5-digit unique number that the user can give the developer to quickly find a line of code. Every log entry and `raise` should have one. The pre-commit hook `git_hooks/pre-commit` verifies that Berror codes are unique. Generate a new one (which is *probably* unique) via:

```
echo -n B; export LC_ALL=C; tr -dc 0-9 </dev/urandom |head -c 5; echo
```

You can also just make them up, but the above method is much less likely to generate duplicates.

### Coding conventions

* Black as mentioned above
* Berror codes as mentioned above
* Use double quotes for strings that should (at some point) be translated in the UI for i18n, single quotes otherwise.
* Put methods above where they are called in the same file (where reasonable).
* Where there is embedded JavaScript, use two-space indent.
* Don't include blank lines within methods except those already mandated by Black.

