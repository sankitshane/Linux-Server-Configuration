# Linux-Server-Configuration

## Launch Virtual Machine

## Instructions for SSH access to the instance
1. Download Private Key below
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/udacity_key.rsa```
4. In your terminal, type in
	```ssh -i ~/.ssh/udacity_key.rsa root@35.161.20.5```
5. Development Environment Information

	Public IP Address

	[35.161.20.5](35.161.20.5)

## Create a new user named grader
1. `sudo adduser grader`
2. `vim /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `vim /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```

3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@52.24.125.52`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade
	apt-get install unattended-upgrades
	dpkg-reconfigure -plow unattended-upgrades

## Change the SSH port from 22 to 2200
1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable

## Configure the local timezone to UTC
 Configure the time zone `sudo dpkg-reconfigure tzdata`


## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres"

	```
	exit
	```

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/kongling893/Item_Catalog_UDACITY.git`
6. Rename the project's name `sudo mv ./CatalogApp./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
9. Edit `database_setup.py`, `website.py` and `functions_helper.py` and change `engine = create_engine('sqlite:///catalogmenu.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo ./pg_config.sh`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
	<VirtualHost *:80>
		ServerName 35.161.20.5
		ServerAdmin your_email@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp:

	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi
	```
2. Add the following lines of code to the flaskapp.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## Trouble shouting

### Path of client_secrets.json and fb_client_secrets.json

1. import os module.
2. retrive the path of the directory.
3. add the path before opening the files.

  ```
  path = os.path.dirname(__file__)

  open(path+'/client_secrets.json', 'r').read())['web']['client_id']

  ```
### Check out the log files for any error

Path - /var/log/apache2/error.log

It logs all the errors.
