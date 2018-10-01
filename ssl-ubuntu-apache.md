

# Install SSL on Ubuntu(nginx)

### <a href="https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-16-04"> Link to full tutorial</a>
### <a href="#quick-setup"> Jump to quick-setup</a>


### Step 1 — Install the Let's Encrypt Client

Let's Encrypt certificates are fetched via client software running on your server. The official client is called Certbot, and its developers maintain their own Ubuntu software repository with up-to-date versions. Because Certbot is in such active development it's worth using this repository to install a newer version than Ubuntu provides by default.

First, add the repository:
```
sudo add-apt-repository ppa:certbot/certbot
```

You'll need to press ENTER to accept. Then, update the package list to pick up the new repository's package information.

```
sudo apt-get update
```

And finally, install Certbot from the new repository with apt-get:

```
sudo apt-get install python-certbot-apache
```

The certbot Let's Encrypt client is now ready to use.



### Step 2 — Set Up the SSL Certificate

Generating the SSL certificate for Apache using Certbot is quite straightforward. The client will automatically obtain and install a new SSL certificate that is valid for the domains provided as parameters.

To execute the interactive installation and obtain a certificate that covers only a single domain, run the certbot command like so, where example.com is your domain:


```
sudo certbot --apache -d example.com
```
If you want to install a single certificate that is valid for multiple domains or subdomains, you can pass them as additional parameters to the command. The first domain name in the list of parameters will be the base domain used by Let’s Encrypt to create the certificate, and for that reason we recommend that you pass the bare top-level domain name as first in the list, followed by any additional subdomains or aliases:


```
sudo certbot --apache -d example.com -d www.example.com

```

For this example, the base domain will be example.com.

If you have multiple virtual hosts, you should run certbot once for each to generate a new certificate for each. You can distribute multiple domains and subdomains across your virtual hosts in any way.

After the dependencies are installed, you will be presented with a step-by-step guide to customize your certificate options. You will be asked to provide an email address for lost key recovery and notices, and you will be able to choose between enabling both http and https access or forcing all requests to redirect to https. It is usually safest to require https, unless you have a specific need for unencrypted http traffic.

When the installation is finished, you should be able to find the generated certificate files at /etc/letsencrypt/live. You can verify the status of your SSL certificate with the following link (don’t forget to replace example.com with your base domain):

```
https://www.ssllabs.com/ssltest/analyze.html?d=example.com&latest
```
You should now be able to access your website using a https prefix.


### Step 3 — Verifying Certbot Auto-Renewal

Let's Encrypt's certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The certbot package we installed takes care of this for us by running ‘certbot renew’ twice a day via a systemd timer. On non-systemd distributions this functionality is provided by a script placed in /etc/cron.d. This task runs twice a day and will renew any certificate that's within thirty days of expiration.

To test the renewal process, you can do a dry run with certbot:

```
sudo certbot renew --dry-run
```

If you see no errors, you're all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.


# Quick Setup

```
sudo add-apt-repository ppa:certbot/certbot
```


```
sudo apt-get update
```


```
sudo apt-get install python-certbot-apache

```


```
sudo certbot --apache -d example.com

```

```
sudo certbot --apache -d example.com -d www.example.com
```

