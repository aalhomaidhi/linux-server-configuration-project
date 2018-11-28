# Udacity linux server configuration project


this project lets you take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications (Game store application) onto it.



## Quick information need it to start

1. Public IP address : ````3.121.185.102````
2. SSH Port : ````2200````
2. username : grader
4. URL of Application	````http://ec2-3-121-185-102.eu-central-1.compute.amazonaws.com````
6. to login with ````sudo ssh -p 2200 grader@3.121.185.102 -i ~/.ssh/grader_key```` 


To connect to EC2 instance you need the grader_key file (supplied separately in the submit process):





## Create a new user named grader
1. Run ````$ sudo adduser grader```` to create a new user named grader
2. Create a new file in the sudoers directory with ````sudo nano /etc/sudoers.d/grader````
3. Add the following text  ````grader ALL=(ALL:ALL) ALL ````


## Set ssh login using keys
1. generate keys on local machine using ````ssh-keygen```` then save the private key in```` ~/.ssh```` on local machine
2. deploy public key on developement enviroment
     ###### On you virtual machine:
````
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys 
````
Copy the public key generated on your local machine to this file and save 
````
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys 
 ````

3. reload SSH using ````sudo service ssh restart````
4. now you can use ssh to login with the new user you created

     ````ssh -i [privateKeyFilename] grader@3.121.185.102````

## Update all currently installed packages
````
sudo apt-get update
sudo apt-get upgrade
````


## Change the SSH port from 22 to 2200
1. Use ````sudo vim /etc/ssh/sshd_config```` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using ````sudo service ssh restart````






## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Just run the following commands to configure: 
````
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
````


## Configure the local timezone to UTC
hanged EC2 instance time zone to UTC:

````sudo dpkg-reconfigure tzdata````

## Server needs setup

1. Installed Apache HTTP Server: ````sudo apt-get install apache2````
2. install mod_wsgi ````sudo apt-get install libapache2-mod-wsgi````
3. And configured a new Virtual Host by sudo vim /etc/apache2/sites-available/catalogApp.conf with the following content:
````
<VirtualHost *:80>
                ServerName 3.121.185.102
                ServerAdmin grader@3.121.185.102
                ServerAlias ec2-3-121-185-102.eu-central-1.compute.amazonaws.com
                WSGIScriptAlias / /var/www/catalogApp/catalogApp.wsgi
                WSGIApplicationGroup %{GLOBAL}
                <Directory /var/www/catalogApp/catalogApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalogApp/catalogApp/static
                <Directory /var/www/catalogApp/catalogApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
````



And enabled the new Virtual Host:
````sudo a2ensite catalogApp````

4. After that create the .wsgi file by

   ````sudo vim /var/www/catalogApp/catalogApp.wsgi ```` with the following content:
````
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalogApp/")

from catalogApp import app as application
application.secret_key = 'super_secret_key'
````
And restarted Apache: ````sudo service apache restart````

## Install and configure PostgreSQL
1. Installed PostgreSQL by : ````sudo apt-get install postgresql postgresql-contrib````
2. Login as user "postgres" ````sudo su - postgres````
3. Get into postgreSQL shell ````psql````
4. Create a new database named catalogApp and create a new user named catalogApp in postgreSQL shell
````
CREATE USER catalogapp WITH PASSWORD 'catalogapp';
ALTER USER catalogapp CREATEDB;
CREATE DATABASE catalogapp WITH OWNER catalogapp;

````
5. Connect to the database ```` \c catalogapp  ```` 
6. Revoke all the rights: ```` REVOKE ALL ON SCHEMA public FROM public;```` 
7. Lock down the permissions to only let catalogapp role create tables: ```` GRANT ALL ON SCHEMA public TO catalogapp;````
8. Log out from PostgreSQL: ````\q````

## Install git, clone and setup your Catalog App project
1. Install Git using ````sudo apt-get install git````
2. Use ````cd /var/www```` to move to the /var/www directory
3. Create the application directory ````sudo mkdir catalogApp````
4. Move inside this directory using  ````cd FlaskApp ````
5. Clone the Catalog App to the virtual machine ````https://github.com/aalhomaidhi/Item-Catalog.git catalogApp ````
6. Move to the inner catalogApp directory using ````cd catalogApp````
7. Rename application.py to __init__.py using  ````sudo mv application.py __init__.py ````
8. Edit database_setup.py, __init__.py and intDB.py
change engine = create_engine('sqlite:///...') to ````engine = create_engine('postgresql://catalogapp:password@localhost/catalogapp')````
9. Update path of client_secrets.json file
````sudo nano __init__.py````
Change client_secrets.json path to ````/var/www/catalog/catalog/client_secrets.json````


## Install Flask and other dependencies
1. Install pip with ````sudo apt-get install python-pip````
2. Install Flask ````pip install Flask````
3. Install other project dependencies ````sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils````
4. restart apache ````sudo service apache2 restart````





#### Setup the database and the pre data:
1. run ```` pytohn3 database_setup.py```` to create the database.
2. then run ````python3 intDB.py```` to fill the database with initial data.
4. Access the application using ````3.121.185.102````



## reset Google Login
To get the Google login working there are a few additional steps:

1. Go to Google Dev Console and Sign up or Login.
3. Go to Credentials > Create Crendentials > OAuth Client ID
5. Select Web application
7. Authorized JavaScript origins = 
````
http://ec2-3-121-185-102.eu-central-1.compute.amazonaws.com	
http://3.121.185.102	
````
8. Authorized redirect URIs =
```` 
http://ec2-3-121-185-102.eu-central-1.compute.amazonaws.com/oauth2callback	
http://ec2-3-121-185-102.eu-central-1.compute.amazonaws.com/gconnect	
http://ec2-3-121-185-102.eu-central-1.compute.amazonaws.com/login	

````
9. Select save

## Resources 
thanks to digital ocean for this source , that helps me alot :
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps




