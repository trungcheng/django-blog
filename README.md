# Django Blog
A tutorial about Python Django for starter.

## Table of contents

- Application & Routes
- Templates
- Admin Page
- Database & Migrations
- User Registration
- Login & Logout System
- User Profile & Picture
- Update User Profile
- Create, Update, Delete Posts
- Pagination
- Email & Password Reset
- AWS Upload Files

## Installation Steps:

1. pip install virtualenv 
2. cd project_folder && virtualenv venv
3. ./venv/Scripts/activate
4. pip install -r requirements.txt
5. cp .env.example .env
6. cd app && python manage.py makemigrations
7. python manage.py createsuperuser
8. python manage.py migrate
9. python manage.py runserver

## Deployment to Linux server

1. Add user to ssh with sudo:
   - apt-get update && apt-get upgrade (software update)
   - hostnamectl set-hostname django-server (set host name)
   - hostname (check name)
   - nano /etc/hosts (add IP server address. Ex: 198.58.119.183 django-server)
   - adduser <username>
   - adduser <username> sudo (add user to sudo group)

2. Generate ssh key on local machine to login to server without password
   - mkdir -p ~/.ssh
   - ssh-keygen -b 4096 (on local machine)
   - scp ~/.ssh/id_rsa.pub <username>@<ip_server>:~/.ssh/authorized_keys (on local machine)
   - sudo chmod 700 ~/.ssh/
   - sudo chmod 600 ~/.ssh/*
   - sudo nano /etc/ssh/sshd_config (change PermitRootLogin -> no, PasswordAuthentication -> no)
   - sudo systemctl restart sshd

3. Setup firewall
   - sudo apt-get install ufw 
   - sudo ufw default allow outgoing
   - sudo ufw default deny incoming
   - sudo ufw allow ssh
   - sudo ufw allow 8000
   - sudo ufw enable
   - sudo ufw status

4. Package libs on local machine and copy to server
   - pip freeze > requirements.txt 
   - scp -r django-blog <username>@<ip_address>:~/

5. Library install necessary:
   - sudo apt-get install python3-pip 
   - sudo apt-get install python3-venv
   - python3 -m venv django-blog/venv
   - cd django-blog
   - source venv/bin/activate
   - pip install -r requirements.txt
   - nano django-blog/app/app/settings.py (add ip_address to ALLOWED_HOSTS, add STATIC_ROOT = os.path.join(BASE_DIR, 'static'))
   - python manage.py collectstatic
   - python manage.py runserver 0.0.0.0:8000

6. Config apache webserver
   - cd /etc/apache2/sites-available
   - sudo cp 000-default.conf django-blog.conf
   - sudo nano django-blog.conf (in source code)
   - sudo a2ensite django-blog
   - sudo a2dissite 000-default.conf
   - sudo chown :www-data django-blog/db.sqlite3
   - sudo chmod 664 django-blog/db.sqlite3
   - sudo chown :www-data django-blog/
   - sudo chown -R :www-data django-blog/media/
   - sudo chmod -R 775 django-blog/media/

7. Config secret key
   - sudo touch /etc/config.json
   - sudo nano django-blog/app/settings.py
   - sudo nano /etc/config.json (add all secret keys to here)
   - sudo nano django-blog/app/settings.py (import json and open file + load)
   - sudo ufw delete allow 8000
   - sudo ufw allow http/tcp
   - sudo service apache2 restart

## Config https/ssl (Let's Encrypt)

- sudo apt-get update && apt-get install software-properties-common
- sudo add-apt-repository universe
- sudo add-apt-repository ppa:certbot/certbot
- sudo apt-get update
- sudo apt-get install python-certbot-apache
- sudo nano /etc/apache2/sites-available/django-blog.conf (change ServerName, comment WSGI)
- sudo certbot --apache
- sudo nano /etc/apache2/sites-available/django-blog.conf (comment static + WSGI sections)
- sudo nano /etc/apache2/sites-available/django-blog-le-ssl.conf (remove comment static + WSGI section)
- sudo apachectl configtest
- sudo ufw allow https
- sudo service apache2 restart
- sudo certbot renew --dry-run
- sudo crontab -e (30 4 1 * * sudo certbot renew --quite)
