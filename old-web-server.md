### Your domain, Webserver and SSL setup (old version with Certbot and Nginx)
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
                proxy_pass http://127.0.0.1:5000; # This is your uvicorn LNbits local host IP and port
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
