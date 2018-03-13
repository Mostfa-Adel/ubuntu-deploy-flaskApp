# ubuntu-deploy-flaskApp

# ubuntu server configuration for flask app


## Project Description
it shows how to configure server to deploy item catalog application(flask and postgresql) on it protecting it from important attacks factors
it using third party oauth2 google+
informations about app 
- ip : 35.178.17.24
- accessible ssh port : 2200
- application github URL : https://github.com/Mostfa-Adel/catalog

## configure to serve catalog app
update and upgrade all packages
``` 
sudo apt-get update 
sudo apt-get upgrade 
```

make the local timezone to UTC
- ``` sudo dpkg-reconfigure tzdata ``` 
- and choose non of the above then UTC


install apache and wsgi mod
- ``` sudo apt-get install apache2 libapache2-mod-wsgi ```

edit 000-default.conf
- ``` sudo nano /etc/apache2/sites-enabled/000-default.conf ```
- and add the follwing line before ending block `</virtualHost>`
` WSGIScriptAlias / /var/www/html/catalog.wsgi `

create catalog.wsgi file
- ``` sudo nano /var/www/html/catalog.wsgi ```
- with content 
> import sys
> sys.path.insert(0,'/var/www/catalog')
> from ` __init__ ` import app as application
> application.secret_key = 'super_secret_key'

clone the app in dir /var/www 
- ``` sudo git clone https://github.com/Mostfa-Adel/catalog ```

install required installation for the app
```
sudo apt-get install python-pip
sudo pip install flask oauth2client sqlalchemy requests
pip install psycopg2
```

installating and configure postgresql database requirement
```
sudo apt-get install libpq-dev python-dev postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```

reatsrt apache
- ``` sudo service apache2 restart ```



required changes to __init__.py file
- replace
```CLIENT_ID = json.loads(open("client_secrets.json", 'r')```
by
```CLIENT_ID = json.loads(open("/var/www/catalog/client_secrets.json", 'r')```

to generate dummy categories to test run create_categories.py
you can edit the list in this file to customize your categories

errors found by
- ``` sudo cat /var/log/apache2/error.log ```

 now it should work 


## security configuration

create user grader and give him permission to sudo
- ``` sudo adduser grader```
- and give him password and other unnessecary informations
to give sudo permission make grader file
- ``` sudo nano /etc/sudoers.d/grader```
- and put the following
- ``` grader ALL=(ALL) NOPASSWD:ALL ```



Configure key-based authentication for grader
- ``` sudo mkdir /home/grader/.ssh ```
- ``` sudo nano /home/grader/.ssh/authorized_keys ``` 
- and paste inside it your public key which you can generate by ssh-keygen 
- and change file permissens for them
``` 
sudo chmod 700 /home/grader/.ssh 
sudo chmod 644 /home/grader/.ssh/authorized_keys
```


change ssh port to 2200
- ``` sudo nano /etc/ssh/sshd_config ```
- change port to 2200 (it often 22)

configure firewall
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```


Disable ssh login for root user
edit sshd_config file 
- ``` sudo nano /etc/ssh/sshd_config ```
- on line ``` PermitRootLogin without-password ```
- change without-password to no

