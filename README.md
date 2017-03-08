# linux-server-configuration
Udacity Full Stack Nanodegree project to configure a linux web server. This readme file explains some of the settings and configuration options that were chosed when building the server.

### General Settings
Public IP: 52.91.39.237

Username: ubuntu

Port: 2200

URL: http://52.91.39.237/

### Run updates

```
$ sudo apt-get update
$ sudo apt-get upgrade
```
### Change ssh port to 2200
Create Custom rule in Lightsail firewall to allow TCP on port 2200.

Run following command to open sshd_config and look for line specifying which ports to list for. Change 22 to 2200
```
$ sudo nano /etc/ssh/sshd_config
$ sudo service ssh restart
```
### Configure Uncomplicated Firewall (ufw)
Run the following commands to allow correct access through the firewall
```
$ sudo ufw allow 2200/tcp
$ sudo ufw allow http
$ sudo ufw allow ntp
$ sudo ufw enable
```
### Install Finger (optional)
Install finger to see additional information for users
```
$ sudo apt-get install finger
```
### Add grader User
Create a new user called grader to allow Udacity reviewer to login
```
$ sudo adduser grader
```
Then grant sudo access  - create a grader file in /etc/sudoers.d with the same content as the default ubuntu one

### Generate key pair
On host machine, generate a new key pair and save file in /Users/*username*/.ssh/grader_key using
```
$ ssh-keygen
```
Copy content of ```.pub``` file to ```/home/grader/.ssh/authorized_keys``` and then log on as grader user
```
$ ssh -i ~/.ssh/grader_key grader@52.91.39.237
```
When you can log on, modify the permissions 
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
### Force Key Based Authentication
To ensure users can only log in with a key, run the following command (already configured on Lightsail)
```
$ sudo nano /etc/ssh/sshd_config
```
And change ```PasswordAuthentication``` to ```no```. Once changed, run the following command so the server restarts for it to take effect
```
$ sudo service ssh restart
```
### Install and configure PostgreSQL
```
$ sudo apt-get install postgresql
```
Switch to postgres user ``` $ sudo -i -u postgres ``` and then create new user ``` $ createuser --interactive ```

Name of user should be ```catalog``` and all permissions set to No.

Move to postgres shell, create new database and assign privileges to catalog user
``` 
$ psql
postgres=# CREATE DATABASE catalog;
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog to catalog;
```
### Clone item-catalogue Repository
To install the catalog application, you need to install Git, which is already installed on the Lightsail instance - if not then ``` $ sudo apt-get install git ```.
``` 
$ cd /var/www
$ sudo mkdir Catalog
$ cd Catalog
$ sudo git clone http://github.com/cubittz/item-catalogue.git Catalog
```
### Install Dependencies
The application imports from the following libraries, which need to be installed (as well as pip for 3rd party libraries)
```
$ sudo apt-get install python-pip
$ sudo apt-get install requests
$ sudo apt-get -qqy python-psycopg2
$ sudo pip install flask
$ sudo pip install sqlalchemy
$ sudo pip install oauth2client
```
### Change sqlite Connection to PostgreSQL
From /var/www/Catalog/Catalog, run ``` $ sudo nano database_setup.py ``` and ``` $ sudo nano app.py ```. Change the following lines from
```python
engine = create_engine('sqlite:///itemcatalogwithusers.db')
```
to
```python
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
Also add/edit following lines to correctly read from client_secrets.json
```python
import os
path = os.path.dirname(__file__)
CLIENT_ID = json.loads(open(path+'/client_secrets.json', 'r').read())['web']['client_id']
```
Once this has been done, the database can be initialized
```
$ sudo python database_setup.py
```
### Rename app.py to __init__.py
From /var/www/Catalog/Catalog
```
$ sudo mv app.py __init__.py
```
### Configure and Enable Virtual Host
Create anew Catalog.conf file
```
sudo touch /etc/apache2/sites-available/Catalog.conf
sudo nano /etc/apache2/sites-available/Catalog.conf
```
Add the following code to configure the virtual host.
```xml
<VirtualHost *:80>
    ServerName 52.91.39.237
    ServerAdmin paul.cubitt@gmail.com
    WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
    <Directory /var/www/Catalog/Catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/Catalog/Catalog/static
    <Directory /var/www/Catalog/Catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Run ``` $ sudo a2ensite Catalog ``` to enable the virtual host
### Create the .wsgi File
Create the .wsgi file in /var/www/Catalog
```
cd /var/www/Catalog
sudo touch catalog.wsgi 
sudo nano catalog.wsgi 
```
And then add the following lines of code
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Catalog/")

from Catalog import app as application
application.secret_key = 'Add your secret key'
```
### Restart Apache
```
$ sudo service apache2 restart
```
### With Thanks to 
Digital Ocean blog post which covered 90% of the required steps to get this working
<https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>
