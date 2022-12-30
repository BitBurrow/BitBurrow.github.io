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
1. A Linux server with a fixed, public IPv4 address, e.g. from [DigitalOcean](https://www.digitalocean.com/), [Linode](https://www.linode.com/), or other VPS provider. BitBurrow can share the server with other services so long as TCP ports 53 and 8443, and UDP port 53 are available.
1. A domain or subdomain name with the ability to host your own DNS (i.e. add an NS record for the subdomain).
1. Time to manage a public server long-term.

## Why would you want to do this

1. 

## Steps

### Acquire a domain name

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
1. Install (or update) BitBurrow hub (run inside container):
    ```
    sudo -u bitburrow python3 -m pip install --force-reinstall --no-warn-script-location git+https://github.com/BitBurrow/BitBurrow.git@main
    ```

### Configure your domain and ports

In the steps below, replace `rxb.example.org` with your BitBurrow hub domain.

1. Configure your domain (run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-domain rxb.example.org
    ```
1. To view the configured domain, replace `set` with `get` in the above line and omit the domain.
1. Customize ports (optional (the default is to use random available ports); run inside container):
    ```
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-ssh-port 〚your ssh port〛
    sudo -u bitburrow /home/bitburrow/.local/bin/bbhub --set-wg-port 〚your WireGuard port〛
    ```
1. Forward ports from host to container (skip if not using a container; run on host):
    ```
    VMNAME=bitburrow  # use the same value used above
    bbhub() { lxc exec $VMNAME -- sudo -u bitburrow /home/bitburrow/.local/bin/bbhub "$1"; }
    APIPORT=8443  # hard-coded in app
    SSHPORT=$(bbhub --get-ssh-port)
    WGPORT=$(bbhub --get-wg-port)
    lxc config device add $VMNAME apiport proxy listen=tcp:0.0.0.0:$APIPORT connect=tcp:127.0.0.1:$APIPORT
    lxc config device add $VMNAME sshport proxy listen=tcp:0.0.0.0:$SSHPORT connect=tcp:127.0.0.1:$SSHPORT
    lxc config device add $VMNAME wgport proxy listen=udp:0.0.0.0:$WGPORT connect=udp:127.0.0.1:$WGPORT
    ```

### Run BitBurrow hub automatically at startup

All of these should be run inside the container.

1. Configure:
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
1. Verify:
    * Reboot (normally `sudo reboot`), wait for system to come back up, and sign in again.
    * Check that the service is running: `sudo systemctl status bitburrow`
    * If there are errors (e.g. `code=exited, status=3`), use `journalctl -u bitburrow` to help diagnose.

### Set up BIND nameserver

In the steps below, replace `rxb.example.org` with your BitBurrow hub domain name and `example.org` with the parent domain. Also, replace `1.2.3.4` with the IP address of your BitBurrow hub machine.

All of these should be run inside the container.

1. Forward DNS from internet (on host):
    ```
    VMNAME=bitburrow
    lxc config device add $VMNAME udpdns proxy listen=udp:1.2.3.4:53 connect=udp:127.0.0.1:53
    lxc config device add $VMNAME tcpdns proxy listen=tcp:1.2.3.4:53 connect=tcp:127.0.0.1:53
    ```
1. Install: `sudo apt install -y bind9`
1. Test recursive DNS look-ups via `dig @127.0.0.1 google.com +short`--should give answers
1. Edit `/etc/bind/named.conf.options` and after `listen-on-v6` line add: `recursion no;`
1. Restart BIND (`sudo service bind9 restart`) and retest lookups (`dig @127.0.0.1 google.com +short`)--should give no answers
1. Edit `/etc/bind/named.conf.local` and add:
    ```
    zone "rxb.example.org" {
	    type master;
	    file "/var/cache/bind/db.bitburrow";
	    update-policy local;
    };
    ```
1. Begin with a template: `sudo cp -i /etc/bind/db.local /var/cache/bind/db.bitburrow`
1. Edit zone file `/var/cache/bind/db.bitburrow` (PROOF OF CONCEPT):
    ```
    $TTL	500
    @	IN	SOA	rxb.example.org. root.localhost. (
			          5		; Serial
			     604800		; Refresh
			      86400		; Retry
			    2419200		; Expire
			    500 )	; Negative Cache TTL
    ;
    @	IN	NS	localhost.
    @	IN	A	1.2.3.4
    test	IN	A	192.168.10.1
    test2	IN	CNAME	test
    ```
1. Fix permissions:
    ```
    sudo chmod 644 /var/cache/bind/db.bitburrow
    sudo chown bind:bind /var/cache/bind/db.bitburrow
    ```
1. Check config and restart BIND: `sudo named-checkconf && sudo service bind9 restart`
1. Test dynamic updates (see that no errors are displayed): `printf "zone rxb.example.org\nupdate delete testa.rxb.example.org.\nupdate add testa.rxb.example.org. 600 IN A 11.11.11.11\nsend\n" |sudo -u bind /usr/bin/nsupdate -l`

### Make BitBurrow hub the authoritative name server

1. Configure the nameserver for example.org (usually done through the website of your domain name registrar), add an NS record for rxb.example.org which points to rxb.example.org. (This tells the internet that rxb.example.org is the authoritative nameserver for *.rxb.example.org.) Then, add an A record for rxb.example.org which points to rxb.example.org (this is called a "glue record"; [more info](https://en.wikipedia.org/wiki/Domain_Name_System#Circular_dependencies_and_glue_records)). Here is a partial zone file for example.org which represents these two records:
	```
	rxb.example.org. 86400 IN NS rxb.example.org.
	rxb.example.org. 3600 IN A 1.2.3.4
	```
1. From a computer elsewhere on the internet, test your DNS server: `dig testa.rxb.example.org`--should display `11.11.11.11`

### Request Let's Encrypt wildcard TLS certificate:

All of these should be run inside the container.

1. Install Certbot: `sudo snap install --classic certbot`
1. Create `/opt/certbot_hook.sh`:
    ```
    #!/bin/bash
    DNS_ZONE=
    HOST='_acme-challenge'
    sudo -u bind /usr/bin/nsupdate -l << EOM
    zone ${DNS_ZONE}
    update delete ${HOST}.${CERTBOT_DOMAIN} A
    update add ${HOST}.${CERTBOT_DOMAIN} 300 TXT "${CERTBOT_VALIDATION}"
    send
    EOM
    sleep 5
    ```
1. Fix permissions:
    ```
    sudo chown bind:bind /opt/certbot_hook.sh
    sudo chmod 660 /opt/certbot_hook.sh
    ```
1. Run Certbot:
    ```
    DNS_ZONE=rxb.example.org
    sudo perl -p -i -e "s|^DNS_ZONE[ =].*|DNS_ZONE=$DNS_ZONE|;" /opt/certbot_hook.sh
    sudo certbot certonly --agree-tos --manual --preferred-challenge=dns --manual-auth-hook=/opt/certbot_hook.sh --register-unsafely-without-email --manual-public-ip-logging-ok -d '*.'$DNS_ZONE -d $DNS_ZONE --server https://acme-v02.api.letsencrypt.org/directory
    sudo systemctl list-timers snap.certbot.renew.timer  # FYI
    ```
1. Allow BitBurrow hub access to files for https:
    ```
    sudo apt install -y acl
    sudo setfacl -Rm d:user:bitburrow:rx,user:bitburrow:rx /etc/letsencrypt/
    ```
