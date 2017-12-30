# Linux Server Configuration 
This project explains how to setup a linux server with one of the web application running live; Item Catalog Application is hosted for this project.

Vist the below link to access the application: 
http://34.215.214.53

Below are the steps followed for the entire setup. 

## Get your server

1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance. 

2. Follow the instructions provided to SSH into your server.
Download the PEM file (LightsailDefaultPrivateKey-us-west-2.pem) from Account Page and execute the below commands.
* `chmod 600 ~/Downloads/LightsailDefaultPrivateKey-us-west-2.pem`
* `ssh -i ~/Downloads/LightsailDefaultPrivateKey-us-west-2.pem -p 2200 ubuntu@34.215.214.53`

## Secure your server

3. Update all currently installed packages.
* `sudo apt-get update`
* `sudo apt-get upgrade`

4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.<br/>
In the Amazon Lightsail instance online (Webpage) go to Networking tab and under Firewall click on Add Another. Add Custom TCP port 2200 and save changes.

Update the port number from 22 to 2200 and save the /etc/ssh/sshd_config file
* `sudo nano /etc/ssh/sshd_config`

Restart ssh service.
* `sudo service ssh restart`

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
* `sudo ufw status`
* `sudo ufw allow 2200/tcp`
* `sudo ufw allow 80/tcp`
* `sudo ufw allow 123/tcp`
* `sudo ufw enable` 
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
* `sudo ufw status`

## Give grader access.

6. Create a new user account named grader.
* `sudo adduser grader`
* `sudo apt-get install finger`
* `finger grader`

7. Give grader the permission to sudo.
* `sudo touch /etc/sudoers.d/grader`
* `sudo nano /etc/sudoers.d/grader`<br/> 
Update the grader file in shudders.d folder with below content<br/> 
grader ALL=(ALL:ALL) ALL

8. Create an SSH key pair for grader using the ssh-keygen tool.<br/> 
Create SSH key pair on local machine for grader<br/> 
Private Key : grader_rsa<br/> 
Public Key : grader_rsa.pub<br/> 
* `ssh-keygen`

On virtual machine login as grader.
* `su - grader`

Check for .ssh file which wont be there in grader folder. Create .ssh file under grader folder.
* `mkdir .ssh`

Create authorized_keys file with in .ssh folder which will store all the public keys that this account is allowed to use for authentication<br/> 
touch .ssh/authorized_keys<br/> 

Copy the public key which was created locally.
* ` nano .ssh/authorized_keys`<br/> 

Edit the .ssh/authorized_keys to copy the key from /grader_rsa.pub which was created locally. Save and Exit.<br/> 

Set the permissions using below commands.<br/> 
* `chmod 700 .ssh`
* `chmod 644 .ssh/authorized_keys`

Test this by attempting to Public IP with User ID grader using private key which was generated locally .ssh/grader_rsa.

Log into 34.215.214.53 with user id grader.
* `ssh -i .ssh/grader_rsa -p 2200 grader@34.215.214.53`

## Prepare to deploy your project

9. Configure the local timezone to UTC.
* `sudo dpkg-reconfigure tzdata`

10. Install and configure Apache to serve a Python mod_wsgi application. (If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server) 
* `sudo apt-get install libapache2-mod-wsgi-py3`

Apache Installation:
* `sudo apt-get install apache2`

Start the apache service with below command
* `sudo service apache2 start`

Go to [http://34.215.214.53] and Verify with Apache2 Ubuntu Default page

Install WSGI with below command.
* `sudo apt-get install libapache2-mod-wsgi python-dev`

Configure Apache to use WSGI module.
* `sudo a2enmod wsgi`

* `sudo nano /etc/apache2/sites-enabled/000-default.conf`

Add below line before </VirtualHost> line.
WSGIScriptAlias / /var/www/catalog/catalog.wsgi  

Install virtual environment
* `sudo apt-get install python-pip`
* `sudo pip install virtualenv`
* `sudo pip install virtualenv`
* `sudo virtualenv venv`
* `sudo chmod -R 777 venv`
* `source venv/bin/activate`
* `sudo chmod -R 777 venv`
* `source venv/bin/activate`
* `sudo pip install Flask`
* `udo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`
* `sudo apt-get install libpq-dev python-dev`
* `sudo pip install requests`


11.  Install and configure PostgreSQL:
Create a new database user named catalog that has limited permissions to your catalog application database.
* `sudo apt-get install postgresql postgresql-contrib`
* `sudo adduser catalog`
* `psql` 

Run the below commands in psql terminal
* `CREATE USER catalog WITH PASSWORD 'catalog';`
* `ALTER USER catalog CREATEDB;`

For checking list of users 
* `\du`

Create Database
* `CREATE DATABASE itemcatalog WITH OWNER catalog;`

Connect to database
* `\c itemcatalog`

* `REVOKE ALL ON SCHEMA public FROM public;`
* `GRANT ALL ON SCHEMA public TO catalog;`

12. Install git
* `sudo apt-get install git`
* `cd /var/www`
* `sudo mkdir catalog`
* `sudo chown -R grader:grader catalog`

## Deploy the Item Catalog project.

13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.

Change to the www directory cd /var/www
Make a new directory called catalog 
* `sudo mkdir catalog`
Change the owner to grader 
* `sudo chown -R grader:grader catalog`<br/>
Change to the new directory cd catalog

Clone the github repo
* `git clone https://github.com/tejasrig/Item-Catalog-App`

Create a new catalog.wsgi file in the /var/www/catalog/ directory and open it in nano sudo nano catalog.wsgi

Add the following code: Save the changes and exit<br/>
#!usr/bin/python<br/>
import sys<br/>
import logging<br/>
logging.basicConfig(stream=sys.stderr)<br/>
sys.path.insert(0,"/var/www/catalog")<br/>
from item_catalog import app as application<br/>
application.secret_key = 'super_secret_key'<br/>

Copy your main project file (application.py) into the __init__.py file 
* `mv application.py __init__.py`

change the sqlite engine to postgres engine in all the necessary files wherever referred
change `sqlite:///itemcatalog.db` to `postgresql://catalog:catalog@localhost/itemcatalog`

Create tables and populate with initial data by executing files database_setup.py and lotsofitems.py

Change the path of client_secrets.json in the __init__.py 

Configure and enable virtual host sudo nano /etc/apache2/sites-available/caalog.conf and add this code:<br/>
<VirtualHost *:80><br/>
   ServerName YOUR PUBLIC_IP_ADDRESS <br/>
   ServerAdmin admin@YOUR_PUBLIC_IP_ADDRESS <br/>
   ServerAlias YOUR_HOST_NAME<br/>
   WSGIScriptAlias / /var/www/catalog/catalog.wsgi<br/>
   <Directory /var/www/catalog/item_catalog/>
       Order allow,deny
       Allow from all
   </Directory>
   Alias /static /var/www/catalog/item_catalog/static
   <Directory /var/www/catalog/item_catalog/static/>
       Order allow,deny
       Allow from all
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log<br/>
   LogLevel warn<br/>
   CustomLog ${APACHE_LOG_DIR}/access.log combined<br/>
</VirtualHost><br/>

Enable the virtual host 
* `sudo a2ensite catalog`

Make sure that your .git directory is not publicly accessible via a browser.
* `cd var/www/catalog/`
* `sudo nano .htaccess`<br/>
Add the below, Save file and exit
RedirectMatch 404 /\.git

Now change the localhost URL in your Google API project. Ensure that origins doesnt mismatch 

Restart apache server to see the web app: 
* `sudo apache2ctl restart`

This entire process might take sometime to launch the application. It requires lot of reading and documentation on Google.


















