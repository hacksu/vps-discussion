# Virtual Private Servers
What are they? How do I get started? What they do? What to use? Time to take a deep dive into the amazing world of the internet!

Virtual Private Servers are the easiest way to host websites and programs on the internet!

You've probably heard of a "server" before, so what's "virtual" and "private" about this? Well, these servers are hosted in the cloud by hosting providers through the use of Virtual Machines. You don't have any physical hardware, thus the "virtual" part. Then, its "private" because your resources that are allocated to your Virtual Machine are yours; you don't share them with anyone else. There are other, cheaper forms of hosting out there that share your resources, leading to lessened reliability and the possibility of some other user hogging all your machine's resources because you both are using the same CPU.

For as little as $5 a month, you can own a Virtual Private Server from [DigitalOcean](https://www.digitalocean.com/products/droplets/)!

A VPS can do the same things as a normal server can; it's just virtualized and hosted in the cloud! You can host your website right inside a datacenter and get lightning fast speeds and reliability for an extremely low cost!

# Signing up for DigitalOcean
You can use my [Referral Code](https://m.do.co/c/100bab0e5e4e) to get started. You will get a $100 credit that lasts 60 days so you can play around with everything DigitalOcean has to offer! However, be sure to have your credit or debit card ready as you will be required to enter payment info so they can verify you are a real person (don't worry, you won't be charged unless you exceed your available credit).

## Destroying Droplets - VERY IMPORTANT
From now on, you will get a bill at the end of each month based on what resources you use.

Any and all things you do on DigitalOcean can be turned off and destroyed.

**If you do not intend on continuing use with DigitalOcean**, you __must destroy your resources so you don't get charged for them__.

# Creating a Virtual Private Server
You can head [here](https://cloud.digitalocean.com/droplets/new?distro=ubuntu-20-04-x64&size=s-1vcpu-1gb) to set up a new Ubuntu droplet, or you can open https://cloud.digitalocean.com, go to the top right, and click `CREATE`, then `Droplet`. My personal recommendation is to use Ubuntu 20.04 and the $5/month tier.

**WARNING**: The $40/month tier is selected by default! Don't end up with a $40 bill if you don't want to!

If you've got that $100 free-trial credit, you can do any server you want. 

If you want to YOLO your free-trial credits (**not recommended**), you can buy a General-Purpose Droplet with **160GB Memory**, **40 Virtual CPUs**, and a **500GB SSD**; __but only for 2 days and 7 hours__. Be sure to destroy that droplet after 2 days and 7 hours or else you'll be charged $1.7/hour and possibly end up with a **$1200 bill at the end of the month**!!!


<img src="https://i.imgur.com/rEPIqMu.jpg" height="200px">

## SSH Key Authentication / Password Authentication
https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04

SSH Keys will not be covered in-depth here. Follow DigitalOcean's tutorial if you don't already have one set up.

Password-based Authentication is available, but **is not recommended**.

Once you've selected your server and added your SSH key, we're good to create our VPS!

# SSHing into our VPS
`ssh root@ip.of.our.server`

# Setting up our Firewall
```bash
ufw allow 22 # Allow SSH Connections through the firewall. VERY IMPORTANT
ufw allow 8080 # We'll use this port to demonstrate how ports work.
ufw enable # Enable the firewall
```

# Run a simple NodeJS Website
```bash
apt install -y nodejs
```

## Install NPM
We'll use this a little later, but it takes a moment to install so we're gonna do it in the background and continue with other topics.

```bash
screen -S npm
apt install -y npm
# Press CTRL + A + D to detach the screen session
```
Now NPM is installing behind-the-scenes for us!

## Make a simple no-dependency NodeJS Website
```bash
mkdir simple-server
cd simple-server
nano index.js
```

```js
require('http').createServer(function(request, response) {
  console.log('Someone visited our http server!');
  response.writeHead(200); // 200 OK HTTP response code. Tells browser "we're all good, everything worked!"
  response.end('Hi there!'); // Send some content to the browser
}).listen(8080);
```

To edit and save a file with NANO, press `Ctrl + X`, then press `Y` so that your changes save.

Once we've done that, we can run it with `node index.js` and then visit `http://your.server.ip.address:8080` and see `Hi there!` pop up!

You can do `Ctrl + C` to stop the server so we can proceed with other topics.

# Install Routing Software
Now if you may be confused with how ports work, or how networking works, or how you would be able to possibly run multiple websites on a single server, or even just get `8080` out of the URL; well that's what Routing Software is for.

There are many different ones out there, but these are the 2 majorly popular ones:
- [Apache](https://httpd.apache.org/)
- [NGINX](https://www.nginx.com/)

I'll be showing off NGINX today because its my personal favorite and has similar syntax to JavaScript. You could use Apache if you want, but the configuration I showcase won't correlate directly to Apache.

```bash
apt install -y nginx
```

Routing Software does all the work for you; all you have to do is just configure it so it knows what to do. NGINX specializes in "reverse proxies", which is a super cool and simple way to forward a port to be accessible on your server's IP without even needing to put the port in the URL! Perfect for people like me who run almost a dozen different websites on their Virtual Private Server!

```bash
ufw app list # Show what configurations are available
ufw allow 'Nginx Full' # Trust NGINX. Any port it needs, it'll have access through the firewall for.
# You can use 'Nginx HTTP' to do only port 80, or 'Nginx HTTPS' to do only 443
```

Run `service nginx status` to make sure its up and running!

## Serving Static Files
```bash
cd /etc/nginx/sites-enabled
unlink default
nano default
```

default
```nginx
server {
  listen 80 default_server;
  root /var/www;
}
```
`Ctrl + X`, `Y` to exit and save.

Run `nginx -t` to test your new configuration and make sure it doesn't have any syntax errors.

`service nginx reload` to update NGINX with your new configuration.

```bash
echo "eeeeeeey! it works!" > /var/www/index.html
```

Go and visit your server's IP now!

## Now that we can serve some files, lets serve something bigger!

```bash
cd /var/www
git clone https://github.com/hacksu/hacksu-2021.git # Clone Hacksu 2021 Website
cd hacksu-2021
npm install && npm run build # Install dependencies, Build static files for the website
```

Update our configuration so we can serve the Hacksu 2021 website.
```bash
nano /etc/nginx/sites-enabled/default
```

default
```nginx
server {
  listen 80 default_server;
  root /var/www/hacksu-2021/dist;
  try_files $uri /index.html; # Fallback to index.html, since Hacksu-2021 is a Single-Page Application
}
```
`Ctrl + X`, `Y`

```bash
nginx -t
service nginx reload
```

# Domains & DNS
You can get a domain name for usually less than $20 a year from various different providers. I like [NameCheap](https://www.namecheap.com/) because they're really nicely priced and NameCheap gives you a bunch of other minor benefits that other providers would usually nickle-and-dime you for!

NameCheap, as with any domain provider, provides their own Domain Name System to make the domain function; but you don't have to use the one they provide! I like to use [CloudFlare](https://www.cloudflare.com/) to keep my domains up and running with maximum performance!

I already have a domain name, so I'm going to attach my server's IP to a subdomain there. Since I'm using CloudFlare for me, it'll also handle SSL encryption for me! Now I can visit my server using HTTPS on its domain!



