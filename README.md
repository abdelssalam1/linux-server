Linux server configuration

host      : Amazon lightsail
public ip : 18.130.38.226
ssh port  : 2200

this configuration is used to deploy catalog item application
sign up and login are not functioning due to google and facebook strict policies
I recommend testing the app through seeders there is a python file called lotsofcats.py
that i have used to insert some data to make it visible

instructions to setup the server :

- create aws account
- create ubuntu instacne
- download the private key
- make a file named udacity_key.rsa under ~/.ssh directory
- copy the private key and paste it in udacity_key.rsa
- set the permission as owner only $ chmod 600 ~/.ssh/udacity_key.rsa
- login into your instance $ ssh -i ~/.ssh/udacity_key.rsa ubuntu@18.130.38.226

update all installed packages

- run the following copmmands
	$ sudo apt-get update
	$ sudo apt-get upgrade

change the ssh port to 2200

- run $ sudo nano /etc/ssh/sshd_config to edit the file
- change the port number from 22 to 2200 then save and exit
- run $ sudo nano /etc/ssh/sshd_config

the firewall

- deny all incomings $ sudo ufw default deny incoming
- allow all outgoings $ sudo ufw default allow outgoing
- allow incoming tcp packets on port 2200 to allow ssh $ sudo ufw allow 2200/tcp
- allow incoming tcp packets on port 80 to allow www $ sudo ufw allow www
- allow incoming udp packets on port 123 to allow ntp $ sudo ufw allow 123/udp
- close port 22 $ sudo ufw deny 22
- enable firewall $ sudo ufw enable
- check the status $sudo ufw status

- now you can ssh through port 2200 $ ssh -i ~/.ssh/udacity_key.rsa ubuntu@18.130.38.226 -p 2200

create user grader

- create new user $ sudo adduser grader
- $ sudo nano /etc/sudoers.d/grader
- copy and paste the following :
	grader ALL=(ALL) NOPASSWD:ALL

- create ssh key pair on your local machine in ~/.ssh/udacity_key.rsa
- then on the server : $ mkdir .ssh
		       $ touch .ssh/authorized_keys
		       $ nano .ssh/authorized_keys copy the key.pub from your local machine and paste it here
- change the permissions of .ssh and .ssh/authorized_keys
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys

change the timezone
- $ sudo dpkg-reconfigure tzdata
- select none of the above to set the time zone to utc

install apache

- $ sudo apt-get install apache2

install python mod_wsgi
- $ sudo apt-get install libapache2-mod-wsgi python-dev
- $ sudo a2enmod wsgi
- restart apache $ sudo service apache2 restart

install postgresql

- $ sudo apt-get install postgresql

create postgresql user catalog

- $ sudo su - postgres
- $ psql
- # CREATE ROLE catalog WITH PASSWORD 'password';
- # ALTER USER catalog CREATEDB;
- # CREATE DATABASE catalog WITH OWNER catalog;
- # \c catalog
- # REVOKE ALL ON SCHEMA public FROM public;
- # GRANT ALL ON SCHEMA public TO catalog;
- \q
- exit

create new linux user called catalog

- $ sudo adduser catalog
- $ sudo visudoAdd
- $ catalog ALL=(ALL:ALL) ALL under line $ root ALL=(ALL:ALL) ALL
- $ sudo su - catalog
- createdb catalog

install git
- $ sudo apt-get install git
- $ mkdir /var/www/catalog
- $ cd /var/www/catalog
- $ sudo git clone your-project-url-on-github catalog
- $ sudo chown -R ubuntu:ubuntu catalog/
- $ cd /var/www/catalog/catalog
- change the main project file to __init__.py : $ mv project.py __init__.py
- change app.run(host='0.0.0.0', port=8000) to app.run() in init.py file

install pip and packages
- $ sudo apt-get install python-pip
- $ sudo pip install httplib2
- $ sudo pip install requests
- $ sudo pip install --upgrade oauth2client
- $ sudo pip install sqlalchemy
- $ sudo pip install flask
- $ sudo apt-get install libpq-dev
- $ sudo pip install psycopg2

the virtual host

- $ sudo nano /etc/apache2/sites-available/catalog.conf
- Add the following :
	<VirtualHost *:80>
                ServerName 18.130.38.226
                ServerAdmin mohamedabdelsalam9499@gmail.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

- $ sudo a2ensite catalog to enable the virtual host
- $ sudo service apache2 reload

.wsgi file

$ sudo nano /var/www/catalog/catalog.wsgi
- add the following :
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")
	from catalog import app as application
	application.secret_key = 'super_secret_key'

- $ sudo service apache2 reload

edit the database path
- eplace lines in __init__.py, database_setup.py, and lotsofcats.py with
  engine = create_engine('postgresql://catalog:password@localhost/catalog')

disable apache default page
- $ sudo a2dissite 000-defualt.conf
- $ sudo service apache2 reload

run database
- $ sudo python database_setup.py
- $ sudo python lotsofcats.py
- $ sudo service apache2 reload

now you can visit http:// 18.130.38.226 to check the website
