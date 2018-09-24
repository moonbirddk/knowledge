# Install SSL on Ubuntu(nginx)

### <a href="https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04"> Link to full tutorial</a>
### <a href="#quick-setup"> Jump to quick-setup</a>


### Step 1 — Installing Certbot
The first step to using Let's Encrypt to obtain an SSL certificate is to install the Certbot software on your server.
Certbot is in very active development, so the Certbot packages provided by Ubuntu tend to be outdated. 
However, the Certbot developers maintain a Ubuntu software repository with up-to-date versions, so we'll use that repository instead.

First, add the repository.

```
sudo add-apt-repository ppa:certbot/certbot
```

You'll need to press ENTER to accept. Then, update the package list to pick up the new repository's package information.

```
sudo apt-get update
```

And finally, install Certbot's Nginx package with apt-get.

```
sudo apt-get install python-certbot-nginx
```

Certbot is now ready to use, but in order for it to configure SSL for Nginx, we need to verify some of Nginx's configuration.


### Step 2 — Setting up Nginx
Certbot can automatically configure SSL for Nginx, but it needs to be able to find the correct server block in your config. 
It does this by looking for a server_name directive that matches the domain you're requesting a certificate for.
If you're starting out with a fresh Nginx install, you can update the default config file. Open it with nano or your favorite text editor.

```
sudo nano /etc/nginx/sites-available/default
```

Find the existing server_name line and replace the underscore, _, with your domain name:

```
/etc/nginx/sites-available/default
```
```
. . .
server_name example.com www.example.com;
. . .
```

Save the file and quit your editor.
Then, verify the syntax of your configuration edits.

```
sudo nginx -t
```

If you get any errors, reopen the file and check for typos, then test it again.

Once your configuration's syntax is correct, reload Nginx to load the new configuration.

```
sudo systemctl reload nginx
```

Certbot will now be able to find the correct server block and update it. Next, we'll update our firewall to allow HTTPS traffic.

### Step 3 — Allowing HTTPS Through the Firewall
If you have the ufw firewall enabled, as recommended by the prerequisite guides, you'll need to adjust the settings to allow for HTTPS traffic. Luckily, Nginx registers a few profiles with ufw upon installation.

You can see the current setting by typing:

```
sudo ufw status
```

It will probably look like this, meaning that only HTTP traffic is allowed to the web server:

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

To additionally let in HTTPS traffic, we can allow the Nginx Full profile and then delete the redundant Nginx HTTP profile allowance:

```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```
Your status should look like this now:

```
sudo ufw status
```

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

We're now ready to run Certbot and fetch our certificates.

### Step 4 — Obtaining an SSL Certificate
Certbot provides a variety of ways to obtain SSL certificates, through various plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary:

```
sudo certbot --nginx -d example.com -d www.example.com
```

This runs certbot with the --nginx plugin, using -d to specify the names we'd like the certificate to be valid for.

If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let's Encrypt server, then run a challenge to verify that you control the domain you're requesting a certificate for.

If that's successful, certbot will ask how you'd like to configure your HTTPS settings.

```
Output
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
```

Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
Select your choice then hit ENTER. The configuration will be updated, and Nginx will reload to pick up the new settings. certbot will wrap up with a message telling you the process was successful and where your certificates are stored:

```
Output
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your cert will
   expire on 2017-10-23. To obtain a new or tweaked version of this
   certificate in the future, simply run certbot again with the
   "certonly" option. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Your certificates are downloaded, installed, and loaded. Try reloading your website using https:// and notice your browser's security indicator. It should indicate that the site is properly secured, usually with a green lock icon. If you test your server using the SSL Labs Server Test, it will get an A grade.

Let's finish by testing the renewal process.

### Step 5 — Verifying Certbot Auto-Renewal
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
sudo apt-get install python-certbot-nginx
```


```
sudo nano /etc/nginx/sites-available/default
```

```
. . .
server_name example.com www.example.com;
. . .
```


```
sudo nginx -t
```


```
sudo systemctl reload nginx
```



```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

```
sudo certbot --nginx -d example.com -d www.example.com
```


 RabbitMQ
Install RabbitMQ with this:
`apt-get install -y erlang`
`apt-get install rabbitmq-server`

Then start it by running:
`systemctl enable rabbitmq-server`
`systemctl start rabbitmq-server`
To check the status of the rabbitmq service
`systemctl status rabbitmq-server`


# Celery
If you haven’t already installed Celery through the requirements file, install Celery as `su - django` and activate the virtual environment `source venv/bin/active`. 
Then run `pip install Celery`

Add this to your settings file.
```
CELERY_BROKER_URL = 'amqp://localhost'
```

Now create a `celery.py` file alongside you mainapplication. In this example in `/home/django/jyllandsakvariet_django/jyllandsakviaret_django`.
Here is a standard settings file:
```
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'jyllandsakvariet_django.settings.production_server')

app = Celery('jyllandsakvariet_django')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

If you want to make sure its imported every time you start django, add this to you `jyllandsakvariet_django/jyllandsakvariet_django/__init__.py`
```
from .celery import app as celery_app

__all__ = ['celery_app']
```

Add this  config to the supervisor config file in `/etc/supervisor/conf.d/akvariet.conf`
```
...

[program:celery]
command=/home/django/venv/bin/celery worker -A jyllandsakvariet_django --loglevel=INFO
directory=/home/django/jyllandsakvariet_django
user=django
group=django
numprocs=1
stdout_logfile=/home/django/logs/celery.log
stderr_logfile=/home/django/logs/celery.log
autostart=true
autorestart=true
startsecs=10

; Need to wait for currently executing tasks to finish at shutdown.
; Increase this if you have very long running tasks.
stopwaitsecs = 600

stopasgroup=true

; Set Celery priority higher than default (999)
; so, if rabbitmq is supervised, it will start first.
priority=1000
```

```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart celery
```