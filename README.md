# catalog

## Project Description

This project is about baseline installation of a Linux server and prepare it to host your web applications. This includes installing updates, securing the server from attacks, and installing / configuring web and database servers.

## Getting Started

### Server Info 
Public IP: 18.221.53.246
Port: 2200
url: [http://ec2-18-221-53-246.us-east-2.compute.amazonaws.com/](http://ec2-18-221-53-246.us-east-2.compute.amazonaws.com/)

### Configure Server

#### Create newuser grader and give sudo access
- sudo adduser grader
- ```sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader```, save and exit

#### enable key based authentication
1. Generate new SSH key pair from aws lightsail account, name it grader and download it.
2. In your terminal and type in
- cp ~/Downloads/grader.pem ~/.ssh/grader.pem
- chmod 400 ~/.ssh/grader.pem
- ssh-keygen -y
- provide the path as:  path/.ssh/grader.pem
- copy generated key to clipboard
3. connect to your aws lightsail instance and type in
- sudo su grader
- cd
- mkdir .ssh
- chmod 700 .ssh
- touch .ssh/authorized_keys
- chmod 644 .ssh/authorized_keys
- sudo nano .ssh/authorized_keys
- paste the key from clipboard save it and exit
4. In your terminal
- ssh -i ~/.ssh/grader.pem grader@18.221.53.246

You should get connected as grader from your terminal now

#### Update currently installed packages
- sudo apt-get update
- sudo apt-get upgrade

#### Configure firewall
- Add port to your amazon lightsail instance networking tab: Custom TCP 2200 and Custom UDP 123
- In your terminal type:
- sudo nano /etc/ssh/sshd_config and add the Port 2200, save and exit
- sudo ufw allow ssh
- sudo ufw allow www
- sudo ufw allow ntp
- sudo ufw allow 2200/tcp
- sudo ufw allow 80/tcp
- sudo ufw allow 123/udp
- sudo ufw enable 
- sudo ufw status

#### Configure timezone to UTC
- sudo dpkg-reconfigure tzdata
- select none of the above, then UTC

#### Install and configure Apache to serve a Python mod_wsgi application
- Install Apache sudo apt-get install apache2
- Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
- Restart Apache sudo service apache2 restart

#### Install and configure PostgreSQL
- Install PostgreSQL sudo apt-get install postgresql
- Check if no remote connections are allowed sudo vi /etc/postgresql/9.3/main/pg_hba.conf
- Login as user "postgres" sudo su - postgres
- Get into postgreSQL shell psql
- Create a new database named catalog and create a new user named catalog in postgreSQL shell
- postgres=# CREATE DATABASE catalog;
- postgres=# CREATE USER catalog;
- Set a password for user catalog
- postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
- Give user "catalog" permission to "catalog" application database
- postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
- Quit postgreSQL postgres=# \q
- exit

#### Install git, clone and setup your Catalog App project.
- Install Git using sudo apt-get install git
- Use cd /var/www to move to the /var/www directory
- Create the application directory sudo mkdir catalog
- Move inside this directory using cd catalog
- Clone the Catalog App to the virtual machine sudo git clone https://github.com/bbhalani/catalog.git
- Rename project.py to __init__.py using sudo cp catalog/project.py catalog/__init__.py
- Edit database_setup.py and and __init__.py to change engine = create_engine('sqlite:///catalogitemwithuser.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog'), if not already done.
- Install pip sudo apt-get install python-pip
- Use pip to install dependencies -
- sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
- sudo pip install flask packaging oauth2client redis passlib flask-httpauth
- Install psycopg2 sudo apt-get -qqy install postgresql python-psycopg2
- Create database schema sudo python database_setup.py

#### Configure and Enable a New Virtual Host
- sudo nano /etc/apache2/sites-available/catalog.conf
- Add the following lines of code to the file to configure the virtual host.
<VirtualHost *:80>
	      ServerName catalog
        ServerAdmin 18.221.53.246
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/>
                Require all granted
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
                Require all granted
        </Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
- Enable the virtual host with the following command: sudo a2ensite catalog

#### Create the .wsgi File
- sudo nano /var/www/catalog/catalog.wsgi
- add following to the file
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from catalog import app as application
application.secret_key = 'super_secret_key'
 - save and exit
 - restart apache with sudo service apache2 restart
 
 ## References
 - Udacity's FSND Forum
 - (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)[https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps]
 - (https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)[https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps]
 - (https://www.systutorials.com/docs/linux/man/8-a2ensite/)[https://www.systutorials.com/docs/linux/man/8-a2ensite/]
 - (https://help.ubuntu.com/community/PostgreSQL)[https://help.ubuntu.com/community/PostgreSQL]
