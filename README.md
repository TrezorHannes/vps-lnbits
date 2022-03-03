# VPS-LNbits
_Documentation how to setup LNbits on a VPS, connected to your Lightning Network Node_

Here's my current setup shared with you, and your intend can be manyfold, you may
- have a dynamic IP from your Internet Service Provider
- want to hide your home IP from the world, for whatever reason
- want others to leverage the LN Services you want to offer, via LNBits, BTCPay or others
- get a domain-name or use a free-domain host such as [DuckDNS](duckdns.org) to point to your LNBits instance
- are just curious and want to tinker around a bit, because it's good to have those skills when demand for experience continues to rise


## Pre-Amble

### Objective
Your [LNbits](https://github.com/lnbits/lnbits-legend) instance installed on a cheap, but anonymous [Virtual Private Server (VPS)](https://www.webcentral.com.au/blog/what-does-vps-stand-for), connected to your own, non-custodial [Lightning-Network](https://github.com/lightningnetwork/lnd) Node running on both Tor and Clearnet [(Hybrid-Mode)](https://github.com/blckbx/lnd-hybrid-mode).

### Challenge
We want payment options with ₿itcoin to be fast, reliable, non-custodial - but the service should ideally not be easily to be identifiable. LNBits provides a quick and simple setup today, for instance on your Raspiblitz oder Umbrel, however, if you want to build the setup from scratch on your own, you have to bypass a number of technical discovery and hurdles. 

### Proposed Solution
There are plenty of ways how to solve for this. This creates hesitance to implement, especially when you're not very technical. This guide is supposed to provide _one approach_, whils there remain many other ways to Rome. 


## Pre-Reads
This guide heavily relies on the intelligence and documentation of others 🙏, but putting those together to one picture creates the last 10% hurdle which is sometimes the highest. Have a careful read through the following articles, to get a deeper understanding on some of the lighter references we'll be using further below
- [Hybrid-Mode for LND](https://github.com/blckbx/lnd-hybrid-mode)
- [TURN YOUR SELF HOSTED LIGHTNING NETWORK NODE TO PUBLIC IN 10 MINUTES](https://www.mobycrypt.com/turn-your-self-hosted-lightning-network-node-to-public-in-10-minutes/)
- [OpenVPN for Docker](https://github.com/kylemanna/docker-openvpn)
- [How to configure Umbrel LNbits app without Tor](https://community.getumbrel.com/t/how-to-configure-umbrel-lnbits-app-without-tor/604)


## Pre-Requisites
- running `lnd-0.14.1-beta` or later. This can either be [Umbrel](https://getumbrel.com), [Raspiblitz](https://github.com/rootzoll/raspiblitz), [MyNode](https://mynodebtc.com/) or even a bare [RaspiBolt](https://raspibolt.org/)
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
Suggested Placeholders
- [ ] IP-Adresses of VPS external, VPS Tunnel, Node Tunnel
- [ ] Ports which needs forwarding
- [ ] ToDos
- [ ] Questions / open items

### Visualize
Some of us are visual people. Draw your diagram to get an idea how you want things to flow
[![Hight-lvl-Flowchart](https://github.com/TrezorHannes/vps-lnbits/blob/main/High_levl-diagram.png)

### Secure
It goes without saying, but this guide doesn't go into the necessary security steps in detail, and can't take on liability for any things breaking or losing funds. Ensure you don't get reckless, start with small funds you're ok to lose. Keep an eye on developments or in touch with the active Telegram Groups, to get news and updates with low delays. Also, would recommend to do those steps with a peer, so you follow a second pair of eye review. Lastly, 2fa / yubikeys are your friends!


## Let's get started (LFG!)
Well, let's get into it, shall we?!
### 1) Lightning Node
We will consider you have your **Lightning Node up and running**, connected via Tor and some funds on it. You also have SSH access to it and administrative privilidges

### 2) VPS Setup 
In case you don't have a **VPS provider** already, sign-up with [my referal](https://m.do.co/c/5742b053ef6d) or [pick another](https://www.vpsbenchmarks.com/best_vps/2022) which provides you with a static IP and cheap costs. Maybe you even prefer one payable with Lightning ⚡. In case you go for DigitalOcean, here are the steps to create a Droplet, shouldn't take longer than a few minutes:
   - [ ] add a new Droplet on the left hand navigation
   - [ ] chose an OS of your preference, I have Ubuntu 20.04 (LTS) x64
   - [ ] take the Basic Plan with a shared CPU, that's enough power. You can upgrade anytime if necessary
   - [ ] Switch the CPU option to "Regular Intel with SSD", which should get you down to $5/month
   - [ ] You don't need an extra volume, but pick a datacenter region of your liking
   - [ ] Authentication: Chose the SSH keys option and follow the next steps to add your public keys in here for secure access. For Windows, with putty and putty-gen referenced above, you should be relatively quick to use those keys instead of a password. [For Linux users](https://serverpilot.io/docs/how-to-use-ssh-public-key-authentication/), you probably know your ways already.
   - [ ] Add backups (costs), Monitoring or IPv6 if you wish to, however this guide won't use any of those items
   - [ ] Lastly, chose a tacky hostname, something which resonates with you, eg myLNBits-VPS

After a few magic cloud things happening, you have your Droplet initiated and it provides you with a public IPv4 Adress. Add it to your notes! In this guide, I'll refer to it as `VPS Public IP: 207.154.241.207`

### 3) Connect to your VPS and tighten it up
Connect to your VPS via `SSH root@207.154.241.207` and you will be welcomed on your new, remote server. Next steps are critical to do right away, harden your setup:
   - [ ] Update your packages: `apt-get update` and `sudo apt-get upgrade`
   - [ ] Install Docker & Tmux: `apt-get install docker.io tmux`
   - [ ] Enable Docker automated start: `systemctl start docker.service`
   - [ ] Enable Uncomplicated Firewall (UFW) and add ports to be allowed to connected to: 
```
apt install ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow OpenSSH
ufw allow 80 comment 'Standard Webserver'
ufw allow 443 comment 'SSL Webserver'
ufw allow 9735 comment 'LND Main Node 1'
ufw enable
```
   - [ ] Follow [further hardening steps](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04), eg setting up non-root users for additional security enhancements.

### 4) Install OpenVPN Server on your VPS
Now we will get OpenVPN installed, but using a Docker Setup like [Krypto4narchista](https://twitter.com/_pthomann) suggests [here](https://www.mobycrypt.com/turn-your-self-hosted-lightning-network-node-to-public-in-10-minutes/). It's easier to setup, but needs some tinkering with port forwarding, which we will go into in a bit.
   - [ ] `export OVPN_DATA="ovpn-data"` which sets a global-name placeholder for your VPN to be used for all the following commands. You can make this permanent by adding this to survive any reboot via `nano .bashrc`, add it to the very bottom => CTRL-X => Yes. 
   - [ ] `docker volume create --name $OVPN_DATA` notice how the $ indicates picking up the placeholder you have defined above
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://207.154.241.207`, whereby you need to adjust the 207.154.241.207 with your own **VPS Public IP**.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki` this generates the necessary VPN certificate password. Take your password manager and create a secure pwd, which you will store safely. It will be needed once we create client-configuration files for your node to connect later.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp -p 9735:9735 -p 9735:9735/udp -p 8080:8080 -p 8080:8080/udp --cap-add=NET_ADMIN kylemanna/openvpn` this works under two assumptions. If any of those aren't true, you need to adjust your settings, either on your node, or by starting the docker container with different ports: 
     - First, your current LND Node configuration is listening on port 9735, which you can verify by looking into your `cat ~/.lnd/lnd.conf` => `[Application Options]` => `listen=0.0.0.0:9735`
     - Second, your LND RestLNDWallet is listening on port 8080, same location under `[Application Options]` => `restlisten=0.0.0.0:8080`

Your OpenVPN Server is now running, which means the Internet can now connect to your VPS via ports 80, 443, 9735 (and 22 SSH), and it has a closed tunnel established on port 1194. You need to complement your notes with the IP-Adresses which are essentially added with the running server.

   - [ ] Collect the Docker Tunnel IP adress: `docker ps` to list your docker container. In the first column, you will find the `CONTAINER-ID`, usually a cryptic 12-digit number/character combination. Copy into the clipboard and make a note of it. Then go into the container shell, with `docker exec -it <CONTAINER-ID> sh`. Run `ifconfig` and you typically find 3 devices listed with IPs assigned. Make a note of the one with eth0, which is your own **VPS Docker IP**: 172.0.0.2

### 4) Install LNBits on your VPSInstall LNBits on your VPS
Next we will install [LNBits](https://lnbits.com/) on this server, since it'll allow to keep your node independent and light-weight. It also allows to change nodes swiftly in-case you need to move things. We won't install it via Docker (like Umbrel does), but do the implementation based slightly on their [Github Installation Guide](https://github.com/lnbits/lnbits-legend/blob/main/docs/devs/installation.md). You can also follow their [own, excellent video walkthrough](https://youtu.be/WJRxJtYZAn4?t=49) here. Just don't use Ben's commands, since these are a little dated.
Since we assume you have followed the hardening guide above to add additional users, we will now have to use `sudo` in our commands.
```
sudo apt-get install git
git clone https://github.com/lnbits/lnbits-legend
sudo apt update
sudo apt install pipenv
cd lnbits-legend
pipenv --python 3.9 shell
pipenv run pip install -r requirements.txt
pipenv run python -m uvicorn lnbits.__main__:app
```
Now when this is successfully starting, you can abort with CTRL-C. We will come back to this for further configuration editing LNBits' config-file to our desired setup.

### 5) Retrieve and transfer the OpenVPN config & certificate to your Node
In this section we'll switch our work from setting up the server towards getting your LND node ready to connect to the tunnel. For this, we will retrieve and transfer the configuration file from your VPS to your node.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full NODE-NAME nopass` whereby `NODE-NAME` should be changed to a unique identifier you chose. For example, if your LND Node is called "BringMeSomeSats", I suggest to use that - with all lowercase.
   - [ ] `docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient NODE-NAME > NODE-NAME.ovpn` which will prompt you to provide the secure password you have generated earlier. Afterwards, it'll store `bringmesomesats.ovpn` in the directory you currently are.





### ) Customize and configure LNBits to connect to your LNDRestWallet
```
cd lnbits-legend
mkdir data
cp .env.example .env
sudo nano .env
```
#### Necessary adjustments





