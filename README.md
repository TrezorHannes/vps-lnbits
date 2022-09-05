# VPS-LNbits
_Documentation to setup LNbits on a VPS, connected to your Lightning Network Node through a secured tunnel_
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/Brenner_Base_Tunnel_Aicha-Mauls.jpg/640px-Brenner_Base_Tunnel_Aicha-Mauls.jpg" alt="Brennerbasistunnel – Wikipedia"/>

Here's my current setup shared with you, and your intend can be manyfold, you may
- have a dynamic IP from your Internet Service Provider
- want to hide your home IP from the world, for whatever reason
- desire to decrease your Lightning Node HTLC Routing times, so instead of running Tor only, you want Clearnet availability, too
- want others to leverage the LN Services you want to offer, via LNBits, BTCPay or others
- get a domain-name or use a free-domain host such as [DuckDNS](https://www.duckdns.org/) to point to your LNBits instance
- are just curious and want to tinker around a bit, because it's good to have those skills when demand for experience continues to rise

## Table of Content

- [Pre-Amble](#pre-amble)
  - [Objective](#objective)
  - [Challenge](#challenge)
  - [Proposed Solution](#proposed-solution)
- [Pre-Reads](#pre-reads)
- [Pre-Requisites](#pre-requisites)
- [Preperations](#preperations)
  - [Make notes](#make-notes)
  - [Visualize](#visualize)
  - [Secure](#secure)
- [Let's get started (LFG!)](#lets-get-started-lfg)
  - [1) Lightning Node](#1-lightning-node)
  - [2) VPS: Setup](#2-vps-setup)
  - [3) VPS: Connect to your VPS and tighten it up](#3-vps-connect-to-your-vps-and-tighten-it-up)
  - [4) VPS: Install OpenVPN Server](#4-vps-install-openvpn-server)
  - [5) VPS: Install LNBits](#5-vps-install-lnbits)
  - [6) VPS: Retrieve the OpenVPN config & certificate](#6-vps-retrieve-the-openvpn-config--certificate)
- [Into the Tunnel](#into-the-tunnel)
  - [7) LND Node: Install and test the VPN Tunnel](#7-lnd-node-install-and-test-the-vpn-tunnel)
  - [8) VPS: Add routing tables configuration into your droplet docker](#8-vps-add-routing-tables-configuration-into-your-droplet-docker)
  - [9) LND Node: LND adjustments to listen and channel via VPS VPN Tunnel](#9-lnd-node-lnd-adjustments-to-listen-and-channel-via-vps-vpn-tunnel)
- [Connect VPS LNBits to your LND Node](#connect-vps-lnbits-to-your-lnd-node)
  - [10) LND Node: provide your VPS LNBits instance read / write access to your LND Wallet](#10-lnd-node-provide-your-vps-lnbits-instance-read--write-access-to-your-lnd-wallet)
  - [11) VPS: Customize and configure LNBits to connect to your LNDRestWallet](#11-vps-customize-and-configure-lnbits-to-connect-to-your-lndrestwallet)
  - [12) VPS: Start LNBits and test the LND Node wallet connection](#12-vps-start-lnbits-and-test-the-lnd-node-wallet-connection)
  - [13) Your domain, Webserver and SSL setup](#13-your-domain-webserver-and-ssl-setup)
    - [Domain](#domain)
    - [VPS: SSL certificate](#vps-ssl-certificate)
    - [VPS: Webserver NGINX](#vps-webserver-nginx)
- [Appendix & FAQ](#appendix--faq)


## Pre-Amble

### Objective
Your [LNbits](https://github.com/lnbits/lnbits-legend) instance installed on a cheap, but anonymous [Virtual Private Server (VPS)](https://www.webcentral.com.au/blog/what-does-vps-stand-for), connected to your own, non-custodial [Lightning-Network](https://github.com/lightningnetwork/lnd) Node running on both Tor and Clearnet [(Hybrid-Mode)](https://github.com/blckbx/lnd-hybrid-mode).

### Challenge
We want payment options with ₿itcoin to be fast, reliable, non-custodial - but the service should ideally not be easily to be identifiable. LNBits provides a quick and simple setup today, for instance on your Raspiblitz oder Umbrel, however, if you want to build the setup from scratch on your own, you have to bypass a number of technical discovery and hurdles. 

### Proposed Solution
There are plenty of ways how to solve for this. This creates hesitance to implement, especially when you're not very technical. This guide is supposed to provide _one approach_, whilst there remain many other ways to Rome. 
Take your time following this through. It might take you 1-2hrs, depending highly on your skill. So don't go in here in a rush.


## Pre-Reads
This guide heavily relies on the intelligence and documentation of others 🙏, but putting those together to one picture creates the last 10% hurdle which is sometimes the highest. Have a careful read through the following articles, to get a deeper understanding on some of the lighter references we'll be using further below
- [Hybrid-Mode for LND](https://github.com/blckbx/lnd-hybrid-mode)
- [TURN YOUR SELF HOSTED LIGHTNING NETWORK NODE TO PUBLIC IN 10 MINUTES](https://www.mobycrypt.com/turn-your-self-hosted-lightning-network-node-to-public-in-10-minutes/)
- [OpenVPN for Docker](https://github.com/kylemanna/docker-openvpn)
- [How to configure Umbrel LNbits app without Tor](https://community.getumbrel.com/t/how-to-configure-umbrel-lnbits-app-without-tor/604)


## Pre-Requisites
- running `lnd-0.14.2-beta` or later. This can either be [Umbrel](https://getumbrel.com), [Raspiblitz](https://github.com/rootzoll/raspiblitz), [MyNode](https://mynodebtc.com/) or even a bare [RaspiBolt](https://raspibolt.org/)
- Technical curiosity and not too shy to use the command-line
- A domain name or a subdomain registered at [DuckDNS](duckdns.org)
- An SSH connection to your node, and to the VPS as well. On Windows, use something like [putty](https://www.putty.org/) and get [putty-gen](https://www.ssh.com/academy/ssh/putty/windows/puttygen), too
- VPS Account at DigitalOcean or any alternative VPS Solution out there offering similar capabilities (it's critical they offer a public IP for you)

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=5742b053ef6d&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

_Disclaimer: this is a ref link, gets you $100 in credit over 60 days, while the cheapest option we use here comes at a cost of $5/month._


## Preperations
The better we prepare, the more we can deal with blindspots and the unexpected.

### Make notes
It's generally advised to document your own steps. Make a bucket-list of things you've done, and a ToDo to go through in case your environment changes. Imagine yourself 18 months from now, you want to setup this new hardware-node: Will you remember all the steps or extra corners you've taken?
Suggested Laundry-List, you can tick them off while you go through this guide
- [ ] IP-Adresses of VPS external, VPS Tunnel, Node Tunnel
- [ ] Ports which needs forwarding
- [ ] ToDos
- [ ] Questions / open items

### Visualize
Some of us are visual people. Draw your diagram to get an idea how you want things to flow
![Hight-lvl-Flowchart](https://github.com/TrezorHannes/vps-lnbits/blob/main/VPN_LNBits.png?raw=true)

### Secure
It goes without saying, but this guide doesn't go into the necessary security steps in detail, and can't take on liability for any things breaking or losing funds. Ensure you don't get reckless, start with small funds you're ok to lose. Keep an eye on developments or in touch with the active Telegram Groups, to get news and updates with low delays. Also, would recommend to do those steps with a peer, so you follow a second pair of eye review. Lastly, 2fa / yubikeys are your friends!


## Let's get started (LFG!)
Well, let's get into it, shall we?!

### 1) Lightning Node
We will consider you have your **Lightning Node up and running**, connected via Tor and some funds on it. You also have SSH access to it and administrative privilidges

### 2) VPS: Setup 
In case you don't have a **VPS provider** already, sign-up with [my referal](https://m.do.co/c/5742b053ef6d) or [pick another](https://www.vpsbenchmarks.com/best_vps/2022) which provides you with a static IP and cheap costs. Maybe you even prefer one payable with Lightning ⚡. In case you go for DigitalOcean, here are the steps to create a Droplet, shouldn't take longer than a few minutes:
   - [ ] add a new Droplet on the left hand navigation
   - [ ] chose an OS of your preference, I have Ubuntu 20.04 (LTS) x64
   - [ ] take the Basic Plan with a shared CPU, that's enough power. You can upgrade anytime if necessary
   - [ ] Switch the CPU option to "Regular Intel with SSD", which should get you down to $5/month
   - [ ] You don't need an extra volume, but pick a datacenter region of your liking
   - [ ] Authentication: Chose the SSH keys option and follow the next steps to add your public keys in here for secure access. For Windows, with putty and putty-gen referenced above, you should be relatively quick to use those keys instead of a password. [For Linux users](https://serverpilot.io/docs/how-to-use-ssh-public-key-authentication/), you probably know your ways already.
   - [ ] Add backups (costs), Monitoring or IPv6 if you wish to, however this guide won't use any of those items
   - [ ] Lastly, chose a tacky hostname, something which resonates with you, eg myLNBits-VPS

After a few magic cloud things happening, you have your Droplet initiated and it provides you with a public IPv4 Adress. Add it to your notes! In this guide, I'll refer to it as `VPS Public IP: 207.154.241.101`


### 3) VPS: Connect to your VPS and tighten it up
Connect to your VPS via `SSH root@207.154.241.101` and you will be welcomed on your new, remote server. Next steps are critical to do right away, harden your setup:
   - [ ] Update your packages: `apt-get update` and `apt-get upgrade`
   - [ ] Install Docker: `apt-get install docker.io tmux`
   - [ ] Enable Docker automated start: `systemctl start docker.service`
   - [ ] Enable Uncomplicated Firewall (UFW) and add ports to be allowed to connected to: 
```
$ apt install ufw
$ ufw default deny incoming
$ ufw default allow outgoing
$ ufw allow OpenSSH
$ ufw allow 80 comment 'Standard Webserver'
$ ufw allow 443 comment 'SSL Webserver'
$ ufw allow 9735 comment 'LND Main Node 1'
$ ufw enable
```
   - [ ] Follow [further hardening steps](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04), eg setting up non-root users for additional security enhancements.
   - [ ] Install fail2ban to protect your SSH user, it runs automatically on it's own `sudo apt install fail2ban`

### 4) VPS: Install OpenVPN Server
Now we will get OpenVPN installed, but using a Docker Setup like [Krypto4narchista](https://twitter.com/_pthomann) suggests [here](https://www.mobycrypt.com/turn-your-self-hosted-lightning-network-node-to-public-in-10-minutes/). It's easier to setup, but needs some tinkering with port forwarding, which we will go into in a bit.
   - [ ] `export OVPN_DATA="ovpn-data"` which sets a global-name placeholder for your VPN to be used for all the following commands. You can make this permanent by adding this to survive any reboot via `nano .bashrc`, add it to the very bottom => CTRL-X => Yes. 
   - [ ] `docker volume create --name $OVPN_DATA` notice how the $ indicates picking up the placeholder you have defined above
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://207.154.241.101`, whereby you need to adjust the 207.154.241.101 with your own **VPS Public IP**.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki` this generates the necessary VPN certificate password. Take your password manager and create a secure pwd, which you will store safely. It will be needed once we create client-configuration files for your node to connect later.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp -p 9735:9735 -p 8080:8080 --cap-add=NET_ADMIN kylemanna/openvpn` this works under two assumptions. If any of those aren't true, you need to adjust your settings, either on your node, or by starting the docker container with different ports: 
     1) your current LND Node configuration is listening on port 9735, which you can verify by looking into your `cat ~/.lnd/lnd.conf` => `[Application Options]` => `listen=0.0.0.0:9735`
     2) your LND RestLNDWallet is listening on port 8080, same location under `[Application Options]` => `restlisten=0.0.0.0:8080`

Your OpenVPN Server is now running, which means the Internet can now connect to your VPS via ports 80, 443, 9735 (and 22 SSH), and it has a closed tunnel established on port 1194. You need to complement your notes with the IP-Adresses which are essentially added with the running server.

   - [ ] CONTAINER-ID: `docker ps` to list your docker container. In the first column, you will find the `CONTAINER-ID`, usually a cryptic 12-digit number/character combination. Copy into the clipboard and make a note of it. 
   - [ ] Docker Shell: Get into the container, with `docker exec -it <CONTAINER-ID> sh`. 
   - [ ] VPS Docker IP: Run `ifconfig` and you typically find 3 devices listed with IPs assigned. Make a note of the one with eth0, which is your own `VPS Docker IP: 172.17.0.2`. Type `exit` to get out of the docker-shell.

### 5) VPS: Install LNBits
Next we will install [LNBits](https://lnbits.com/) on this server, since it'll allow to keep your node independent and light-weight. It also allows to change nodes swiftly in-case you need to move things. We won't install it via Docker (like Umbrel does), but do the implementation based slightly on their [Github Installation Guide](https://github.com/lnbits/lnbits-legend/blob/main/docs/devs/installation.md). You can also follow their [own, excellent video walkthrough](https://youtu.be/WJRxJtYZAn4?t=49) here. Just don't use Ben's commands, since these are a little dated.
Since we assume you have followed the hardening guide above to add additional users, we will now have to use `sudo` in our commands.
```
$ sudo apt-get install git
$ git clone https://github.com/lnbits/lnbits-legend
$ sudo apt update
# ensure you have virtualenv installed, on debian/ubuntu 'apt install python3-venv' should work
$ python3 -m venv venv
$ ./venv/bin/pip install -r requirements.txt
$ ./venv/bin/uvicorn lnbits.__main__:app --port 5000
```
Now when this is successfully starting, you can abort with CTRL-C. We will come back to this for further configuration editing LNBits' config-file to our desired setup.


### 6) VPS: Retrieve the OpenVPN config & certificate
In this section we'll switch our work from setting up the server towards getting your LND node ready to connect to the tunnel. For this, we will retrieve and transfer the configuration file from your VPS to your node.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full NODE-NAME nopass` whereby `NODE-NAME` should be changed to a unique identifier you chose. For example, if your LND Node is called "BringMeSomeSats", I suggest to use that - with all lowercase.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient NODE-NAME > NODE-NAME.ovpn` which will prompt you to provide the secure password you have generated earlier. Afterwards, it'll store `bringmesomesats.ovpn` in the directory you currently are.


## Into the Tunnel
We have installed the tunnel through the mountain, but need to get our LND Node to use it.

### 7) LND Node: Install and test the VPN Tunnel
Now switch to another terminal window, and SSH into your **Lightning Node**. We want to connect to the VPS and retrieve the VPN-Config file, to be able to establish the tunnel
```
$ cd ~
$ mkdir VPNcert
$ scp user@207.154.241.101:/home/user/bringmesomesats.ovpn /home/admin/VPNcert/
$ chmod 600 /home/admin/VPNcert/bringmesomesats.ovpn
```
_Note: You need to adjust `user`, the **VPS Public IP** and the absolute directory where the ovpn file is stored. We keep a copy of the cert file in the home directory for backup, but the actual file we use is `CERT.conf`._

Now we need to install OpenVPN, start it up to see if it works.

**Important Warning**: Depending on your network-setup, there is a slight chance your LND Node Service gets interrupted. Be aware there might be small down-times of your lightning node, as we will reconfigure things. Be patient!

```
$ sudo apt-get install openvpn
$ sudo cp -p /home/admin/VPNcert/bringmesomesats.ovpn /etc/openvpn/CERT.conf
$ sudo systemctl enable openvpn@CERT
$ sudo systemctl start openvpn@CERT
```
You should see something similiar to the following output. Note this one line indicating the next important IP Adress `VPN Client IP: 192.168.255.6`. Make a note of it, we need it for port-configuration at the server, soon.
```
* openvpn@CERT.service - OpenVPN connection to CERT
     Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-04-06 13:11:13 CEST; 4s ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 1514818 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 18702)
     Memory: 1.0M
        CPU: 49ms
     CGroup: /system.slice/system-openvpn.slice/openvpn@CERT.service
             `-1514818 /usr/sbin/openvpn --daemon ovpn-CERT --status /run/openvpn/CERT.status 10 --cd /etc/openvpn --config /etc/openvpn/CERT.conf --writepid /run/openvpn/CERT.pid

Apr 06 13:11:13 debian-nuc ovpn-CERT[1514818]: WARNING: 'link-mtu' is used inconsistently, local='link-mtu 1541', remote='link-mtu 1542'
Apr 06 13:11:13 debian-nuc ovpn-CERT[1514818]: WARNING: 'comp-lzo' is present in remote config but missing in local config, remote='comp-lzo'
Apr 06 13:11:13 debian-nuc ovpn-CERT[1514818]: [207.154.241.101] Peer Connection Initiated with [AF_INET]207.154.241.101:1194
Apr 06 13:11:14 debian-nuc ovpn-CERT[1514818]: TUN/TAP device tun0 opened
Apr 06 13:11:14 debian-nuc ovpn-CERT[1514818]: net_iface_mtu_set: mtu 1500 for tun0
Apr 06 13:11:14 debian-nuc ovpn-CERT[1514818]: net_iface_up: set tun0 up
Apr 06 13:11:14 debian-nuc ovpn-CERT[1514818]: net_addr_ptp_v4_add: 192.168.255.6 peer 192.168.255.5 dev tun0
```
The tunnel between your LND Node and your VPS VPN is established. If you need to troubleshoot, call the systemctl journal via 
`sudo journalctl -u openvpn@CERT -f --since "1 hour ago"`


### 8) VPS: Add routing tables configuration into your droplet docker
Back to your terminal window connected to your VPS. We have the `VPN Client IP: 192.168.255.6` now, which we need to tell our VPS where it should route those packets to. To achieve that, we'll get back into the docker container and add IPTables rules.

   - [ ] [Remember](#4-vps-install-openvpn-server) how to get into the container? Arrow-up on your keyboard, or do `docker ps` and `docker exec -it <CONTAINER-ID> sh`
   - [ ] Doublecheck your VPN Client IP, and adjust it in the following IPtables commands you enter into the container and confirm with Enter

```
$ iptables -A PREROUTING -t nat -i eth0 -p tcp -m tcp --dport 9735 -j DNAT --to 192.168.255.6:9735
$ iptables -A PREROUTING -t nat -i eth0 -p tcp -m tcp --dport 8080 -j DNAT --to 192.168.255.6:8080
$ iptables -t nat -A POSTROUTING -d 192.168.255.0/24 -o tun0 -j MASQUERADE
$ exit
```
What we basically do here, is assign a ruleset to say: As soon a packet arrives at device `eth0` on `port 9735/tcp`, forward it to the `VPN client` at `192.168.255.6:9735`, and vice versa everything at device `tun0`. If you have different ports or IPs, please make adjustments accordingly. What you also see, is a port `8080` preperation for LNBits packets, we'll get to this later.

#### Permanent saving of IPtable rules
Your VPS needs operational maintenance, and a reboot sometimes isn't avoidable. If you want to make the above iptable entries permanent, follow these steps:

   - [Follow this step again](#4-vps-install-openvpn-server) to get into the container,  arrow-up on your keyboard, or do `docker ps` and `docker exec -it <CONTAINER-ID> sh`
   - within the container, change the directory `cd /etc/openvpn` and open the environment-settings file for OpenVPN via `vi ovpn_env.sh`. Nano isn't installed in the docker container,  follow the [vi-cheatsheet PDF](https://www.atmos.albany.edu/daes/atmclasses/atm350/vi_cheat_sheet.pdf) if you get confused (and you will). Add the following lines to the file

```
$ iptables -A PREROUTING -t nat -i eth0 -p tcp -m tcp --dport 9735 -j DNAT --to 192.168.255.6:9735
$ iptables -A PREROUTING -t nat -i eth0 -p tcp -m tcp --dport 8080 -j DNAT --to 192.168.255.6:8080
$ iptables -t nat -A POSTROUTING -d 192.168.255.0/24 -o tun0 -j MASQUERADE
$ exit
```
save with `:wq` and now your VPS adheres to those rules after a reboot, too.

### 9) LND Node: LND adjustments to listen and channel via VPS VPN Tunnel
We switch Terminal windows again, going back to your LND Node. A quick disclaimer again, since we are fortunate enough to have plenty of good LND node solutions out there, we cannot cater for every configuration out there. Feel free to leave comments or log issues if you get stuck for your node, we'll be looking at the two most different setups here. But this should work very similar on _MyNode_, _Raspibolt_ or _Citadel_.

Be very cautious with your `lnd.conf`. Make a backup before with `cp /mnt/hdd/lnd/lnd.conf /mnt/hdd/lnd/lnd.bak` so you can revert back when things don't work out. 
The brackets below indicate the section where each line needs to be added to. Don't place anything anywhere else, as it will cause your LND constrain from starting properly.

_Adjust ports and IPs accordingly!_

<details><summary>Click here to expand Raspiblitz / Raspibolt settings</summary>
<p>

   LND.conf adjustments, open with `sudo nano /mnt/hdd/lnd/lnd.conf`

[**Application Options**]
   | Command | Description |
   | --- | --- |
   | `externalip=207.154.241.101:9735`           | # to add your VPS Public-IP |
   | `nat=false`                                 | # deactivate NAT |
   | `tlsextraip=172.17.0.2`                     | # allow later LNbits-access to your rest-wallet API |

[**tor**]
   | Command | Description |
   | --- | --- |
   | `tor.active=true`                           | # ensure Tor is active |
   | `tor.v3=true`                               | # with the latest version. v2 is going to be deprecated this summer |
   | `tor.streamisolation=false`                 | # this needs to be false, otherwise hybrid mode doesn't work |
   | `tor.skip-proxy-for-clearnet-targets=true`  | # activate hybrid mode |

`CTRL-X` => `Yes` => `Enter` to save

RASPIBLITZ CONFIG FILE
`sudo nano /mnt/hdd/raspiblitz.conf` since Raspiblitz has some LND pre-check scripts which otherwise overwrite your settings.
   | Command | Description |
   | --- | --- |
   | `publicIP='207.154.241.101'`                | # add your VPS Public-IP |
   | `lndPort='9735'`                            | # define the LND port |
   | `lndAddress='207.154.241.101'`              | # define your LND public IP address |

`CTRL-X` => `Yes` => `Enter` to save

LND Systemd Startup adjustment

   | Command | Description |
   | --- | --- |
   | `sudo nano /etc/systemd/system/lnd.service` | edit the line 15 where it starts your LND binary, and add the following parameter: `ExecStart=/usr/local/bin/lnd ${lndExtraParameter}` |
   | `sudo systemctl restart lnd.service` | apply changes and restart your lnd.service. It will ask you to reload the systemd services, copy the command, and run it with sudo. This can take a while, depends how long your last restart was. Be patient. | 
   | `sudo tail -n 30 -f /mnt/hdd/lnd/logs/bitcoin/mainnet/lnd.log` | to check whether LND is restarting properly |
   | `lncli getinfo` | to validate that your node is now online with two uris, your pub-id@VPS-IP and pub-id@Tor-onion |

   ```
"03502e39bb6ebfacf4457da9ef84cf727fbfa37efc7cd255b088de426aa7ccb004@207.154.241.101:9736",
        "03502e39bb6ebfacf4457da9ef84cf727fbfa37efc7cd255b088de426aa7ccb004@vsryyejeizfx4vylexg3qvbtwlecbbtdgh6cka72gnzv5tnvshypyvqd.onion:9735"
```        
 - [ ] Restart your LND Node with `sudo reboot`
</p>
</details>

<details><summary>Click here to expand Umbrel / Citadel settings</summary>
<p>
<!-- Add further comments for Umbrel and validate how to adjust starting LND docker with those changes, and making them persistent -->

   LND.conf adjustments, open with `sudo nano /home/umbrel/umbrel/lnd/lnd.conf`

   
[**Application Options**]
   | Command | Description |
   | --- | --- |
   | `externalip=207.154.241.101:9735` | # to add your VPS Public-IP | 
   | `nat=false`                       | # deactivate NAT | 
   | `tlsextraip=172.17.0.2`           | # allow later LNbits-access to your rest-wallet API | 

[**tor**]
   | Command | Description |
   | --- | --- |
   | `tor.active=true`                          | # ensure Tor is active | 
   | `tor.v3=true`                              | # with the latest version. v2 is going to be deprecated this summer | 
   | `tor.streamisolation=false`                | # this needs to be false, otherwise hybrid mode doesn't work | 
   | `tor.skip-proxy-for-clearnet-targets=true` | # activate hybrid mode | 

`CTRL-X` => `Yes` => `Enter` to save

LND Restart to incorporate changes to `lnd.conf`
   | Command | Description |
   | --- | --- | 
   | `cd umbrel && docker-compose restart lnd` | this can take a while, depends how long your last restart was. Be patient. | 
   | `tail -n 30 -f ~/umbrel/lnd/logs/bitcoin/mainnet/lnd.log` | check whether LND is restarting properly | 
   | `~/umbrel/bin/lncli getinfo` | validate that your node is now online with two uris, your pub-id@VPS-IP and pub-id@Tor-onion | 
```
"03502e39bb6ebfacf4457da9ef84cf727fbfa37efc7cd255b088de426aa7ccb004@207.154.241.101:9736",
        "03502e39bb6ebfacf4457da9ef84cf727fbfa37efc7cd255b088de426aa7ccb004@vsryyejeizfx4vylexg3qvbtwlecbbtdgh6cka72gnzv5tnvshypyvqd.onion:9735"
```
 - [ ] Restart your LND Node with `sudo reboot`

**Warning**: This guide did not verify yet, if and how the docker LND service on Umbrel & Citadel needs to be adjusted to channel clearnet packets via tunnel. We will add peer-reviewed adjustments here from Umbrel / Citadel devs. Until then, consider this highly experimental, it might fail.
</p>
</details>


## Connect VPS LNBits to your LND Node
The traffic line between the two connection points is established. Worth noting that this can be extended: In case you run more than one node, just repeat the steps above for additional clients. Now, let's get LNBits talk to your node.


### 10) LND Node: provide your VPS LNBits instance read / write access to your LND Wallet
Assuming LND restarted well on your LND Node, your LND is now listening and connectable via VPS Clearnet IP and Tor. That's quite an achievement already. But we want to setup LNBits as well, right? So go grab another beverage, now we'll get LNBits running.
For that, let's climb another tricky obstacle; to respect the excellent security feats the LND engineering team has implemented. Since we don't want to rely on a custodial wallet provider, which would be super easy to add into LNBits, we have some more tinkering to do. Follow along to basically provide two things to your VPS from your LND Node. 

**Note of warning again**: Both of those files are highly sensitive. Don't show them to anyone, don't transfer them via Email, just follow the secure channel below and you should be fine, as long you keep the security barriers installed in [Section "Secure"](#secure) intact.

1) your tls.cert. Only with access to this file, your VPS is going to be allowed to leverage your LND Wallet via Rest-API
`scp ~/.lnd/tls.cert root@207.154.241.101:/root/` sends your LND Node tls.cert to your VPS, where we will use it in the next section.

2) your admin.macaroon. Only with that, your VPS can send and receive payments
`xxd -ps -u -c ~/.lnd/data/chain/bitcoin/mainnet/admin.macaroon` will provide you with a long, hex-encoded string. Keep that terminal window open, since we need to copy that code and use it in our next step on the VPS.


### 11) VPS: Customize and configure LNBits to connect to your LNDRestWallet
 Now since we're back in the VPS terminal, keep your LND Node Terminal open. We'll adjust the LNBits environment settings, and we'll distinguish between _necessary_ and _optional_ adjustments. First, send the following commands:
```
$ cd lnbits-legend
$ mkdir data
$ pwd
$ cp .env.example .env
$ sudo nano .env
```
Worth noting, that the directory `data` will hold all your database SQLite3 files. So in case you consider proper backup or migration procedures, this directory is the key to be kept.

#### Necessary adjustments
 | Variable | Description |
 | --- | --- |
 | `LNBITS_DATA_FOLDER="/user/lnbits-legend/data"` | enter the absolute path to the data folder you created above | 
 | `LNBITS_BACKEND_WALLET_CLASS=LndRestWallet` | Specify that we want to use our LND Node Wallet Rest-API
 | `LND_REST_ENDPOINT="https://172.17.0.1:8080"` | Add your `VPS Docker IP: 172.17.0.1` on port 8080 | 
 | `LND_REST_CERT="/root/tls.cert"` | Add the link to the tls.cert file copied over earlier | 
 | `LND_REST_MACAROON="HEXSTRING"` | Copy the hex-encoded snippet from your LND Node Terminal output from Section 11.2 in here | 
 
 #### Optional adjustments
 | Variable | Description |
 | --- | --- |
 | `LNBITS_SITE_TITLE="HODLmeTight LNbits"` | Give your Website a tacky title |
 | `LNBITS_SITE_TAGLINE="free and open-source lightning wallet"` | Define the sub-title in the body |
 | `LNBITS_SITE_DESCRIPTION="Offering free and easy Lightning Bitcoin Payment options for Friends & Family"` | Outline your offering |
 | `LNBITS_THEME_OPTIONS="classic, bitcoin, flamingo, mint, autumn, monochrome, salvador"` | Provide different color themes, or keep it simple |
 `CTRL-X` => `Yes` => `Enter` to save
 
 
 ### 12) VPS: Start LNBits and test the LND Node wallet connection
 As soon you got here, we got the most complex things done 💪. The next few steps will be a walk in the park. Get another beverage, and then start LNBits again in your tmux-environment
```
$ tmux new -s lnbits
$ cd ~/lnbits-legend
$ ./venv/bin/uvicorn lnbits.__main__:app --port 5000
```
When this is successful, it'll report your wallet balance of your node, and you can move on. If not, a good debugging approach is to connect from the VPS to your node via `curl https://172.17.0.1:8080 -v --cacert /root/tls.cert`. 

CTRL-C to cancel if successful, and follow the guide here to add lnbits-startup to systemd. This will allow an automated restart [LNBits further documentation](https://github.com/lnbits/lnbits-legend/blob/main/docs/guide/installation.md#additional-guides), but with our slightly adjusted default settings, LNBits should now be running and listening on all incoming requests on port 8000. If you're impatient, add a temporary[^1] ufw exception to test it: `sudo ufw allow 8000/tcp comment 'temporary lnbits check'` and open the corresponding `VPS Public IP: 207.154.241.101:8000`. 

If you see your own LNBits instance, with all your _Optional Adjustments_ added, we'll go to the last, final endboss. 

[^1]: To remove the ufw setting - we don't want to expose any unnecessary ports - call `sudo ufw status numbered`, followed by `sudo ufw delete #number` of the two port 8000 entries.


### 13) Your domain, Webserver and SSL setup
We don't want to share our IP-Adress for others to pay us, a domain name is a much better brand. And we want to keep it secure, so we need to get us an SSL certificate. Good for you, both options are available for free, just needs some further work.

#### Domain
While there are plenty of domain-name providers out there, we are going to use a free, easy and secure provider: [duckdns.org](https://www.duckdns.org/). They do their own elevator pitch why to use them on their site. Feel free to pick another, such as [Ahnames](https://ahnames.com/en), but this guide will use the former for simplicity
   - [ ] make an account on DuckDNS with GH or Email
   - [ ] add 1 of 5 free subdomains, eg. paymeinsats
   - [ ] point this domain to your `VPS Public IP: 207.154.241.101`
   - [ ] Make a note of your Token

Keep the site open, we'll need it soon

#### VPS: SSL certificate
You want your secure https:// site to confirm to your visitor's browser that you're legit. For this, we will use Certbot to manage our SSL certificate management, even though LNBits recommends [caddy](https://caddyserver.com/docs/install#debian-ubuntu-raspbian). Use your own preference, we'll walk through certbot here:
```
$ sudo apt update
$ sudo apt install nginx certbot
$ sudo certbot certonly --manual --preferred-challenges dns
```
Next to a few other things, Certbot will ask you for your domain, so add your `paymeinsats.duckdns.org`. Then it'll prompt you to place a TXT record for \_acme-challenge.paymeinsats.duckdns.org, which is basically their way to verify whether you really own this domain. 
To achieve this, leave the certbot alone without touching anything, and follow those steps in parallel:
   - [ ] Open a text editor, and add this URL: `https://www.duckdns.org/update?domains={YOURVALUE}&token={YOURVALUE}&txt={YOURVALUE}[&verbose=true]`
   - [ ] replace each variable[^2]
     - `domains={YOURVALUE}` with your subdomain only, in our case `domains=paymeinsats`
     - `token={YOURVALUE}` with your token from your duckdns.org overview
     - `txt={YOURVALUE}` with the random text-snippet certbot provided you to fill in
     - optional: set `verbose=true` if you want 2 lines more info as a response
   - [ ] Copy that whole string into a new Webbrowser window, and if verbose isn't set as true, it'll be as crisp as `OK`
   - [ ] In a new Terminal window, install dig `sudo apt-get install dnsutils` to check if the world knows about you solved the challenge: `dig -t txt _acme-challenge.paymeinsats.duckdns.org`. Compare the TXT record entry with what Certbot provided you. If both are similar, confirm with `Enter` in the Certbot Terminal, so it can do it's own verification
   - [ ] Once successful, you got your SSL certificates. Make a note in your calendar when the validation time is over, so you renew early enough. Also take note of the absolute paths of those two certificates you received.

[^2]: [Visit Specpage of duckdns.org for further details here](https://www.duckdns.org/spec.jsp)


#### VPS: Webserver NGINX
Uvicorn is working fine, but we'll add a more robust solution, to be able to do some caching and better log-management: nginx (engine-x). We'll add a new configuration file for your website.

_Please don't forget to adjust domain names and paths below accordingly_

   - [ ] `sudo nano /etc/nginx/sites-available/paymeinsats.conf` to create and edit your new configuration file nginx will use

Add the following entries
```
server {
        # Binds the TCP port 80
        listen 80;
        # Defines the domain or subdomain name
        server_name paymeinsats.duckdns.org;
        # Redirect the traffic to the corresponding 
        # HTTPS server block with status code 301
        return 301 https://$host$request_uri;
       }

server {
        listen 443 ssl; # tell nginx to listen on port 443 for SSL connections
        server_name paymeinsats.duckdns.org; # tell nginx the expected domain for requests

        access_log /var/log/nginx/paymeinsats-access.log; # Your first go-to for troubleshooting
        error_log /var/log/nginx/paymeinsats-error.log; # Same as above

        location / {
                proxy_pass http://127.0.0.1:8000; # This is your uvicorn LNbits local host IP and port
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header Host $host;
                proxy_http_version 1.1; # headers to ensure replies are coming back and forth through your domain
       }

        ssl_certificate /etc/letsencrypt/live/paymeinsats.duckdns.org/fullchain.pem; # Point to the fullchain.pem from Certbot 
        ssl_certificate_key /etc/letsencrypt/live/paymeinsats.duckdns.org/privkey.pem; # Point to the private key from Certbot
}
```
`CTRL-X` => `Yes` => `Enter` to save
Next we'll test the configuration and enable it by creating a symlink from sites-available to sites-enabled.
```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo ln -s /etc/nginx/sites-available/paymeinsats.conf /etc/nginx/sites-enabled/
$ sudo systemctl restart nginx
```

Now the moment of truth: Go to your Website [https://paymeinsats.duckdns.org](https://paymeinsats.duckdns.org) and either celebrate 🍻 
or troubleshoot where things could have gone wrong. If the former: Congratulations - you made it!

Hope you enjoyed this article. Please do share feedback and suggestions for improvement.
If this guide was of any help, I'd appreciate if you share the article with others, give me a follow on Twitter [![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/HandsdownI.svg?style=social&label=Follow%20%40HandsdownI)](https://twitter.com/HandsdownI)
or even donating some sats below

[<img src=https://user-images.githubusercontent.com/35168804/156722937-59239884-5398-48ae-a4f2-1d6c53dfd0fb.png width="100" height="100">](https://pwbtc.duckdns.org/lnurlp/2)

I'm also always grateful for incoming channels to my node: [HODLmeTight](https://amboss.space/node/037f66e84e38fc2787d578599dfe1fcb7b71f9de4fb1e453c5ab85c05f5ce8c2e3)


## Appendix & FAQ

#### I see anyone can create a wallet on my LNBits service, but I don't want that. How do I change that?
Once you have created your first user wallet, and you want only this to be accessible, go to the user-section in LNBits and notice the user-ID in the URL: `/usermanager/?usr=[32-digit-user-ID]`. Copy the user-id and add it to your `.env` file: `nano ~/lnbits-legend/.env` and add this to the variable `LNBITS_ALLOWED_USERS=""`. You can comma-seperate a list of user-ids.

#### I'm stuck and have no idea why it's not working. Who can help?
Please add an issue on Github with your question and provide as much detail as possible. Keep it safe though, no macaroon or user-ids!

#### So I have LNBits running, now what?
Head over to [LNBits Website](https://lnbits.com/) and check out the plethora of options you could do. For instance, I've built a donation wallet, which is shared 50:50 between the main author and my own wallet. All automated.

#### Why DigitalOcean - can't we pick a VPS where we can pay with Lightning, and anonymously
Consider this guide a work-in-progress. I've picked DigitalOcean since I know what I'm doing there. Heard good things about [Luna Node](https://www.lunanode.com/), it's cheaper and you can pay with sats, so will test this out next. Also happy to add further alternatives, leave comments if you think these can accomplish the same results. Fee free to provide suggestions here.

#### Can I add more nodes connecting to the tunnel? If so, how?
In fact, I have more than one node connected to the tunnel. You need to handle your port-forwarding appropriately, since every node needs their unique LND listen port. Eg Node 1 has 9735, Node 2 9736 and so on. Docker runs need to be called with further `-p for publish-options`, IPtable rules and UFW needs to be adjusted. But once you got this guide internalised, the principle should be clear. Otherwise, let me know.
