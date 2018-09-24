# DJANGO SERVER TUTO
Buenos Dias.
Det er her er Williams ligetil (og lidt dumme) fortolkning af Johans Django Server Tutorial for Ubuntu 18 (nok også 16).
Følg disse steps nøje og de vil blive belønnet.

### as Root user:
```
apt-get update    
apt-get install mercurial python-pip python3-venv nginx supervisor
adduser django
su - django
```

### as Django user
```
ssh-keygen
cat .ssh/id_rsa.pub
```
Add ssh keys to repo

```
python3 -m venv venv
source venv/bin/activate
clone repo
cd your_project
pip install -r requirements.txt
touch settings/production_server.py
```
Add the following to production_server.py

```
from .settings import *

DEBUG = False

ALLOWED_HOSTS = [your_ip, your_domain]

BASE_URL = your_domain

STATIC_ROOT="/home/django/static"
MEDIA_ROOT="/home/django/media"
```
Remember to collectstatic or migrate or other django-stuff

## Gunicorn 
```
cd /home/django
source venv/bin/activate (if not already sourced)
pip install gunicorn
mkdir /home/django/logs
touch gunicorn_config.py
```
Add the following to gunicorn_config.py

```
bind = '127.0.0.1:8000'
workers = 2
group = 'django'
user = 'django'
daemon = False
accesslog = "/home/django/logs/access-gunicorn.log"
errorlog = "/home/django/logs/error-gunicorn.log"
loglevel = "error"
debug = False
pythonpath = '/home/django/your_project'
raw_env = ['DJANGO_SETTINGS_MODULE=your_project.settings.production_server']
```

### as Root:
```
chown root:django /home/django/gunicorn_config.py
chmod 640 /home/django/gunicorn_config.py
```

## Nginx
```
touch /etc/nginx/sites-available/your_project
ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/your_project
rm /etc/nginx/sites-enabled/default
```

Add the following to /etc/nginx/sites-available/your_project

```
server {
   listen 80 default_server;
   server_name your_ip your_domain ;

   error_log /home/django/logs/nginx_error.log warn;
   access_log /home/django/logs/nginx_access.log;

   client_max_body_size 100M;

   location /static/ {
      alias /home/django/static/;
   }

   location /media/ {
      alias /home/django/media/;
   }

   location / {
      include proxy_params;
      proxy_redirect off;
      proxy_buffering off;

      proxy_pass http://127.0.0.1:8000;
   }
   location = /favicon.ico {
      access_log off;
      log_not_found off;
   }
}
```

```
systemctl restart nginx
```

## Supervisor
Create a custom config file for our gunicorn process:

```
touch /etc/supervisor/conf.d/your_project.conf
```
This will be loaded automatically from /etc/supervisor/supervisor.conf.

Add the following to /etc/supervisor/conf.d/your_project.conf

```
[program:gunicorn]
command=/home/django/venv/bin/gunicorn your_project.wsgi -c /home/django/gunicorn_config.py
directory=/home/django/your_project
user=django
group=django
autorestart=true
autostart=true
redirect_stderr=false
```
Now you can run the supervisorctl command and interact with the processes. You might need to restart the supervisor service.

```
sudo systemctl enable supervisor
sudo systemctl start supervisor
supervisorctl restart all
```


### GO MAKE A CUP OF COFFEE, YOUR SITE IS READY TO GO


# RabbitMQ
Install RabbitMQ with this:
```
apt-get install -y erlang
apt-get install rabbitmq-server
```

Then start it by running:
```systemctl enable rabbitmq-server```
```systemctl start rabbitmq-server```
To check the status of the rabbitmq service
```systemctl status rabbitmq-server```


# Celery
If you haven’t already installed Celery through the requirements file, install Celery as `su - django` and activate the virtual environment `source venv/bin/active`. 
Then run `pip install Celery`

Add this to your settings file.
```
CELERY_BROKER_URL = 'amqp://localhost'
```

Now create a `celery.py` file alongside you main application. In this example in `/home/django/your_project/your_app`.
Here is a standard settings file:
```
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project.settings.production_server')

app = Celery('your_project')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

If you want to make sure its imported every time you start django, add this to you `your_project/your_app/__init__.py`
```
from .celery import app as celery_app

__all__ = ['celery_app']
```

Add this  config to the supervisor config file in `/etc/supervisor/conf.d/akvariet.conf`
```
...

[program:celery]
command=/home/django/venv/bin/celery worker -A your_project --loglevel=INFO
directory=/home/django/your_project
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