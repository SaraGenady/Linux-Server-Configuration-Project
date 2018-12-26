Linux Server Configuration Project

1- Project Overview

   a baseline installation of a Linux server and prepare it to host web applications.
  It will secure its server from a number of attack vectors, install and configure a database server, 
  and deploy one of existing web applications onto it.


2- Get your Server

  Start a new Ubuntu Linux server instance on Amazon Lightsail (https://lightsail.aws.amazon.com/).

3- INFO

  Server's Public IP: 3.120.134.210

  Server's Private IP:172.26.7.75

  SSH Port: 2200

  Grader User Name: CatalogItems


4- Connect to your Server using SSH

 - Create a directory.

   mkdir -p ~/.ssh

5- Move the downloaded .pem file to .ssh directory we just created.
  

   mv DefaultKey.pem ~/.ssh

   chmod +x ~/.ssh/DefaultKey.pem

6- Connect to instance.

   ssh -i ~/.ssh/DefaultKey.pem ubuntu@3.120.134.210

7- Update all currently installed packages

   sudo apt-get update

   sudo apt-get upgrade

8- Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it

 - Edit sshd_config file using GNU Nano.

   sudo nano /etc/ssh/sshd_config

   Change line #5 from 22 to 2200.

 - Restart SSH service.

   sudo service ssh restart

9- Configure the Uncomplicated Firewall (UFW)

 - Allow incoming connections for SSH (port 2200)

   sudo ufw allow 2200/tcp

 - Allow incoming connections for HTTP (port 80)

   sudo ufw allow 80/tcp

 - Allow incoming connections for NTP (port 123)

   sudo ufw allow 123/tcp

 - Enable UFW

   sudo ufw enable

10- Grader Access

 - Create a new user account named grader
  
   sudo adduser grader

 - Give grader the permission to sudo

  - Super user configuration

    sudo nano /etc/sudoers.d/grader

  - Add the following.

    grader ALL=(ALL) NOPASSWD:ALL 
    
11- Create an SSH key pair for grader using the ssh-keygen tool

   ssh-keygen -t rsa

 - Move generated key

   sudo su - grader

   mkdir .ssh
  
   touch .ssh/authorized_keys
  
   mv grader_key ~/.ssh/

 - Move content of grader_key.pub to that of authorized_keys

   mv grader_key.pub  ~/.ssh/authorized_keys

   sudo chmod 700 .ssh
  
   sudo chmod 644 .ssh/authorized_keys 

 - Connect to grader

   ssh -i ~/.ssh/grader_key grader@3.120.134.210 -p 2200

12- Prepare to deploy your project
 
 - Configure the local timezone to UTC.

   sudo dpkg-reconfigure tzdata

   - Choose None of the above.
 
   - Choose UTC

13- Install and configure Apache to serve a Python mod_wsgi application.

 - Install Apache2.

   sudo apt-get install apache2
  
 - Install mod_wsgi.

   sudo apt-get install libapache2-mod-wsgi-py3

 - Check whether mod_wsgi is enabled.

   sudo a2enmod wsgi

14- Install and configure PostgreSQL.

   sudo apt-get install postgresql

 - Switch to postgres, PostgreSQL User.

   sudo su - postgres
 
 - Connect to your own database

   psql

 - Create database user named Catalog.

   CREATE ROLE catalog WITH LOGIN;

 - Limit permissions of user.

   ALTER ROLE catalog CREATEDB;

 - Set user Catalog password

   \password Catalog

 - Exit psql by pressing Ctrl+D

 - Switch back to ubuntu user by running exit

15- Install Git

   sudo apt-get install git

16- Deploy the Catalog project

 - Change directory

   cd /var/www

 - Clone Item Catalog project repository

   sudo git clone https://github.com/SaraGenady/Catalog.git

 - Change ownership of Catalog directory to ubuntu user

   sudo chown -R ubuntu:ubuntu Catalog/

 - Change current directory to project directory

   cd Catalog

 - Create Catalog.wsgi file using GNU Nano

   sudo nano Catalog.wsgi

 - Add the following into the file created

  #!/usr/bin/python3
  import sys
  sys.stdout = sys.stderr

  activate_this = '/var/www/Catalog/env/bin/activate_this.py'
  with open(activate_this) as file_:
  exec(file_.read(), dict(__file__=activate_this))

  sys.path.insert(0,"/var/www/Catalog")

  from project import app as application


 - Install Python 3

   sudo apt-get install python3-pip

 - Install Virtual Enviroment

   sudo -H pip3 install virtualenv

 - Change current directory to catalog directory

   cd Catalog/

 - Create virtual enviroment called env

   virtualenv env

 - Activate env virtual enviroment

   source env/bin/activate

 - Install dependencies

   pip3 install httplib2
  
   pip3 install requests

   pip3 install --upgrade oauth2client

   pip3 install sqlalchemy

   pip3 install flask

   pip3 install psycopg2

17- Run project.py main project file

   python3 project.py

 - Check If you are getting this message when you're running the file

   Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

 - Configure Apache Server by editing 000-default.conf file

   sudo nano /etc/apache2/sites-enabled/000-default.conf

 - Add the following to 000-default.conf file

   <VirtualHost *:80>
            ServerName 3.120.134.210
            WSGIScriptAlias / /var/www/Catalog/Catalog.wsgi
            <Directory /var/www/Catalog/>
                  Order allow,deny
                  Allow from all
                  Options -Indexes
            </Directory>
            Alias /static /var/www/Catalog/static
            <Directory /var/www/Catalog/static/>
                  Order allow,deny
                  Allow from all
                  Options -Indexes
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>


 - Reload Apache Server

   sudo service apahce2 reload

 - Activate env virtual enviroment

   source env/bin/activate

 - Run database_setup.py

   python3 database_setup.py

 - Deactivate env virtual enviroment

   deactivate

 - Restart Apache Server

   sudo service apache2 restart
