# Udacity Linux Server Configuration

Final project for the Full Stack Nanodegree. The purpose was to create and secure a server, install and configure database, and deploy one of my existing apps from the course.

IP: `18.188.178.251`  
Port: 2200  
Site: http://ec2-18-188-178-251.us-east-2.compute.amazonaws.com/

## Included Packages
- Python 2.7
- PostgresSQL
- Apache2
- Git

## Configuration

### Get Server
- Start new Ubuntu Linux Server instance with Amazon Lightsail
- Follow instructions to SSH into the server.
- Log in with `ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@18.188.178.251` on local machine
- Enter `Yes` when prompted

### Update all currently installed packages
- `sudo apt-get update` to update packages  
- `sudo apt-get upgrade` to upgrade packages  
- select `Y` to continue when prompted  
- `sudo apt-get install finger` (optional)  

### Give grader Access
- `sudo adduser grader`  
- create password  
- confirm password  
- edit information  
- select `Y` to create and save the user  

### Grant grader User sudo Permissions
- `sudo touch /etc/sudoers.d/grader`  
- `sudo nano /etc/sudoers.d/grader`  
- add `grader ALL=(ALL) NOPASSWD:ALL` to the file  
- exit and save changes  

### Configure Key-Based Authentication for grader User
- `sudo mkdir /home/grader/.ssh`  
- `sudo chown grader:grader /home/grader/.ssh`  
- `sudo chmod 700 /home/grader/.ssh`  
- `sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/`  
- `sudo chown grader:grader /home/grader/.ssh/authorized_keys`  
- `sudo chmod 644 /home/grader/.ssh/authorized_keys`  
- `sudo nano /home/grader/.ssh/authorized_keys`  
- On your local maching `cat ~/.ssh/udacity_key.rsa.pub`
- Copy details of the *udacity_key.rsa.pub* file to the *authorized_keys* file in _grader_  
- Exit and save changes.  

### Change Timezone to UTC
- `sudo timedatectl set-timezone UTC`  

### Disable Login for root User
- `sudo nano /etc/ssh/sshd_config`  
- Change `PermitRootLogin prohibit-password` to `PermitRootLogin no`  
- `sudo service ssh restart`  

### Change Port to 2200
- `sudo nano /etc/ssh/sshd_config`  
- Change `Port 22` to `Port 2200` (4th line from the top)  
- Be sure to restart SSH otherwise the connection is refused
- `sudo service ssh restart`

### Configure UFW Firewall
- `sudo ufw default deny incoming`  
- `sudo ufw default allow outgoing`  
- `sudo ufw deny 22`  
- `sudo ufw allow 2200/tcp`  
- `sudo ufw allow www`  
- `sudo ufw allow ntp`  
- `sudo ufw show added`  

Should Read:

ufw deny 22
ufw allow 2200/tcp
ufw allow 80/tcp
ufw allow 123

- `sudo ufw enable`  
- select `Y` to enable the firewall  
- `sudo ufw status`  

Should Read:
```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere                  
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123                        ALLOW       Anywhere                  
22 (v6)                    DENY        Anywhere (v6)             
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123 (v6)                   ALLOW       Anywhere (v6)    
```

### Configure Lightsail Dashboard
- Login to your Lightsail Dashboard
- Click on your instance
- Then click the **Networking** Tab
- Remove **SHH Port 22**
- Add **Custom TCP Port 2200**
- Add **Custom TCP Port 123**
- Save Changes
- Be sure to complete these steps othwerise you'll lock yourself out of the server

```
Networking should look like:

Application	Protocol	Port range	
HTTP	        TCP	        80	
Custom	        TCP	        123	
Custom	        TCP	        2200
```

**Can now login with** `ssh -i ~/.ssh/udacity_key.rsa grader@18.188.178.251 -p 2200`   

### Install Apache
- `sudo apt-get install apache2`  
- select `Y` to continue  
- `sudo apt-get install libapache2-mod-wsgi`  
- select `Y` to continue  
- `sudo service apache2 restart`  

### Install & Configure PostgresSQL
- `sudo apt-get install postgresql`  
- select `Y` to continue  
- `sudo apt-get install postgresql-contrib`  
- `sudo -u postgres createuser -P catalog`  
- Enter a password and confirm password of your choice for the new role. 
- `sudo -u postgres createdb -O catalogUser catalogDb`  

### Install Git
- `sudo apt-get install git`  

### Clone Item Catalog Repository
- `cd /var/www`
- `sudo mkdir CatalogApp`
- `cd CatalogApp`
- `sudo git clone https://github.com/lomo009/item-catalog.git`
- `sudo mv ./item-catalog ./CatalogApp`
- `cd CatalogApp`
- `sudo mv app.py __init__py`
- `sudo nano app.py`
- Change: `engine = create_engine('sqlite:///itemCatalog.db')` to `engine = create_engine('postgresql://catalogUser:password@localhost/catalogDb')` 
- `sudo nano database_setup.py`  
- Change: `engine = create_engine('sqlite:///itemCatalog.db')` to `engine = create_engine('postgresql://catalogUser:password@localhost/catalogDb')` 
- `sudo nano lotsofitems.py`
- Change: `engine = create_engine('sqlite:///itemCatalog.db')` to `engine = create_engine('postgresql://catalogUser:password@localhost/catalogDb')` 

### Install App Dependencies
- `sudo apt-get install python-pip`
- select `Y` to continue  
- `sudo pip install flask`
- `sudo pip install sqlalchemy`
- `sudo pip install oauth2client`
- `sudo pip install psycopg2`
- `sudo pip install requests`

### Seed Database
- `python database_setup.py`
- `python lotsofitems.py`

### Update catalogapp.wsgi file
- Inside of /var/www/CatalogApp  
- `sudo nano catalogapp.wsgi`  
- Copy and paste the following into the file:  

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/CatalogApp/")

from CatalogApp import app as application
application.secret_key = "my_secret_key"  
```  
- Exit and Save  

### Configure Apache Server

- `sudo nano /etc/apache2/sites-available/CatalogApp.conf`  
- Copy and paste the following into the file:  

```
<VirtualHost *:80>
	ServerName 18.188.178.251
        ServerAdmin logan.morrow@me.com
	WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
	<Directory /var/www/CatalogApp/CatalogApp/>
	    Order allow,deny
            Allow from all
	</Directory>
	Alias /static /var/www/CatalogApp/CatalogApp/static
	<Directory /var/www/CatalogApp/CatalogApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```  
- Exit and Save

### Configure Google OAuth2 API

- Add http://ec2-18-188-178-251.us-east-2.compute.amazonaws.com/ to authorized javascript origin
- Add http://ec2-18-188-178-251.us-east-2.compute.amazonaws.com/login and http://ec2-18-188-178-251.us-east-2.compute.amazonaws.com//login to authorized redirect URL
- Be sure to add http://ec2-18-188-178-251.us-east-2.compute.amazonaws.com/ to the `Javascript Origins` part of client_secrets.json
- Altertnatively download a new clients_secrets.json file from Google and replace contents when you have added the new routes.

### Restart Apache
- `sudo a2dissite 000-default.conf`
- `sudo a2ensite CatalogApp.conf`
- `sudo service apache2 reload`

### Debugging:
- Initially there was an error loading my app, as it couldn't find my clients_secrets.json file. Inside of `__init__.py` I had to change the path from relative to absolute by chaging it to `/var/www/CatalogApp/CatalogApp/client_secrets.json` and it worked.

### References
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps  

https://help.pythonanywhere.com/pages/NoSuchFileOrDirectory/  

Udacity Full Stack Nanodegree Lessons