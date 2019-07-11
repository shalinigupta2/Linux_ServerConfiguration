# Linux_ServerConfiguration
This is the final project for "Full Stack Web Developer Nanodegree" on Udacity. A Linux virtual machine (Amazon Lightsail / Ubuntu) is configured for Item Catalog application website.

You can visit the web site at : http://52.66.170.46.xip.io/
```
Public Static IP Address: 52.66.170.46
SSH Port: 2200
HTTP Port: 80
NTP Port: 123
```
# Environment Used
This page explains how to secure and set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application..

* The Linux distribution is Ubuntu 16.04 LTS.
* The virtual private server is [Amazon Lighsail](https://aws.amazon.com/lightsail/).
* The web application is my [Item Catalog](https://github.com/shalinigupta2/Catalog-App) project created earlier in this Nanodegree program.
* The database server is PostgreSQL.
* Local machine Windows 10

# Get a Server
## Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail

* Login into [Amazon Lightsail](https://aws.amazon.com/lightsail/) using an Amazon Web Services account.
* Once you are login into the site, click ```Create instance```.
* Choose ```Linux/Unix``` platform, ```OS Only``` and ```Ubuntu 16.04 LTS```.
* Choose a instance plan (I took the cheapest, $3/month).
* Keep the default name provided by AWS or rename your instance.
* Click the ```Create``` button to create the instance.
* Wait for the instance to start up.

## Create Static IP Address

* Inside the instance dashboard .
* On the Lightsail home page, choose Networking.
* Choose Create static IP.
* Select the AWS Region where you want to create your static IP.
* Choose the Lightsail resource (Catalog-Item)
* Name your static IP, and then choose Create.

## Step 2: SSH into the server

* From the ```Account``` menu on Amazon Lightsail, click on ```SSH keys``` tab and download the Default Private Key.
* Move this private key file named ```LightsailDefaultPrivateKey-*.pem``` into the local folder ```~/.ssh``` and rename it Catalog-Item.pem.
* In your terminal, type: ```chmod 644 ~/.ssh/Catalog-Item.pem```.
* To connect to the instance via the terminal: ```ssh ubuntu@52.66.170.46 -i ~/.ssh/Catalog-Item.pem```, where ```52.66.170.46``` is the public static IP address of the instance.
## Secure the server

## Step 3: Update and upgrade installed packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
```
## Step 4: Change the SSH port from 22 to 2200

* Edit the ```/etc/ssh/sshd_config``` file: ```sudo nano /etc/ssh/sshd_config```.
* Then change the following
* Add Port 2200 below Port 22 (don't delete port 22 until 2200 is working) write file and exit (ctrl o, enter, ctrl x)
* Find PermitRootLogin and edit to no.
* Find the PasswordAuthentication line and edit it to no
* Restart SSH: ```sudo service ssh restart```.

## Step 5: Configure the Uncomplicated Firewall (UFW)
* Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw status                  # The UFW should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
sudo ufw allow www               # Allow HTTP traffic in.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
```
* Turn UFW on: sudo ufw enable. The output should be like this:
```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
* Check the status of UFW to list current roles: sudo ufw status. The output should be like this:
```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere
  80/tcp                     ALLOW       Anywhere
  123/udp                    ALLOW       Anywhere
  80                         ALLOW       Anywhere
  22                         DENY        Anywhere
  2200/tcp (v6)              ALLOW       Anywhere (v6)
  80/tcp (v6)                ALLOW       Anywhere (v6)
  123/udp (v6)               ALLOW       Anywhere (v6)
  80 (v6)                    ALLOW       Anywhere (v6)
  22 (v6)                    DENY        Anywhere (v6)
```
* Exit the SSH connection: exit.

* Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above. 

* Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22. 

* From your local terminal, run: ssh ubuntu@52.66.170.46 -p 2200 -i ~/.ssh/Catalog-Item.pem, where 52.66.170.46 is the public IP address of the instance.

## Give ```grader``` access
## Step 6: Create a new user account named ```grader```

* While logged in as ubuntu, add user: ```sudo adduser grader```.
* Enter a password (twice) and fill out in ```sudo adduser grader``` formation for this new user.
* Confirm if the user is added by ```finger grader```.

## Step 7: Give ```grader``` the permission to sudo
* list all the users file: ```sudo ls /etc/sudoers.d``` .

* Read the content of the file and copy it to a new file grader: ```sudo cat /etc/sudoers.d/90-cloud-init-users sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader```

* Run ```sudo nano /etc/sudoers.d/grader``` Edit the line to give sudo privileges to grader user. ```grader ALL=(ALL:ALL) ALL```

* Save and exit using CTRL+X and confirm with Y.

* Verify that grader has sudo permissions. Run ```su - grader```, enter the password, run ```sudo -l``` and enter the password again.

## Step 8: Create an SSH key pair for grader using the ssh-keygen tool
* On the local machine:

- Run ```ssh-keygen```
- Enter file in which to save the key (I gave the name ```grader```) in the local directory ```~/.ssh```
- Enter in a passphrase twice. Two files will be generated ( ```~/.ssh/grader and ~/.ssh/grader.pub```)
- Run ```cat ~/.ssh/grader.pub``` and copy the contents of the file
- Log in to the grader's virtual machine ```sudo su - grader```
- On the grader's remote machine:

* Create a new directory called ```~/.ssh (mkdir .ssh)```
- Run ```sudo nano ~/.ssh/authorized_keys``` and paste the content into this file, save and exit
- Give the permissions: ```chmod 700 .ssh``` and ```chmod 644 .ssh/authorized_keys```
- Check in ```/etc/ssh/sshd_config``` file if PasswordAuthentication is set to no
- Restart SSH: ```sudo service ssh restart```
* On the local machine, run: ```ssh grader@52.66.170.46 -p 2200 -i ~/.ssh/grader```.

# Prepare to deploy the project
## Step 9: Configure the local timezone to UTC
* While logged in as grader, configure the time zone: ```sudo dpkg-reconfigure tzdata```. You should see something like that:
```
  Current default time zone: 'US/Central'
  Local time is now:      Mon Jun  3 04:43:32 CDT 2019.
  Universal Time is now:  Mon Jun  3 09:43:32 UTC 2019.
```
## Step 10: Install and configure Apache to serve a Python mod_wsgi application
* While logged in as ```grader```, Install Apache: ```sudo apt-get install apache2```.
* Install mod_wsgi ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
* Restart Apache ```sudo service apache2 restart```
## Step 11: Install and configure PostgreSQL
* Install PostgreSQL ```sudo apt-get install postgresql```

* Check if no remote connections are allowed ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf```

* Login- as user "postgres" ```sudo su - postgres```

* Get into postgreSQL shell ```psql```

* Create a new database named ```catalog``` and create a new user named catalog in postgreSQL shell
```
 postgres=# CREATE USER catalog WITH PASSWORD 'password'; 
 postgres=# CREATE DATABASE catalog WITH OWNER catalog; 
```
* Give user "catalog" permission to "catalog" application database
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Quit postgreSQL postgres=# \q
```
* Exit from user "postgres"
```
 exit
```
## Step 12: Install git
* While logged in as ```grader```, install ```git``` using: ```sudo apt-get install git```.
## Deploy the Item Catalog project
## Step 13:
1. Clone and setup the Item Catalog project from the GitHub repository

  * While logged in as ```grader```, Create application directory ```/var/www/catalog/```.

  * Change to that directory and clone the catalog project:
  ```sudo git clone https://github.com/shalingupta2/CatalogApp.git catalog```.

  * Move to the inner catalog directory using ```cd catalog```

  * Rename the ```itemCatalogProject.py``` file to ```__init__.py``` using: ```sudo mv application.py __init__.py```.

  * In ```__init__.py```, replace the following lines below line:
```
# engine = create_engine('sqlite:///catalogitemwithusers.db')
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
```
# app.run(host="0.0.0.0", port=8000, debug=True)
app.run()
```
  * In setup_database.py, replace the following line:
```
# engine = create_engine('sqlite:///catalogitemwithusers.db')
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
2.Authenticate login through Google

* Go to [Google Cloud Plateform].
* Click ```APIs & services``` on left menu.
* Add ```Authorised Domains``` and add ```52.66.170.46.xip.io``` .Click Save
* Click ```Credentials```.
* Create an OAuth Client ID (under the Credentials tab), and add http://52.66.170.46.xio.io as authorized JavaScript origins.
* Add http://http://52.66.170.46.xip.io as authorized redirect URI.
* Download the corresponding JSON file, open it et copy the contents.
* Open ```/var/www/catalog/catalog/client_secret.json``` and paste the previous contents into the this file.

## Step 14: Set up the server, so that it functions correctly when visiting your server’s IP address in a browser
1.Install the virtual environment and dependencies

  * While logged in as grader , Install pip ```sudo apt-get install python-pip```
  * Install the virtual environment: ```sudo pip install virtualenv```
  * Change to the ```/var/www/catalog/catalog/``` directory.
  * Create the virtual environment: ```sudo pip install virtualenv```.
  * Activate the new environment: cd .
  * Use pip to install dependencies -
  ```
  * sudo pip install sqlalchemy flask-sqlalchemy bleach requests
  * sudo pip install flask packaging oauth2client redis passlib flask-httpauth
  * sudo pip install flask github-flask httplib2
  * sudo apt-get -qqy install python-flask
  ```
  * Install psycopg2 ```sudo apt-get -qqy install postgresql python-psycopg2```
  * Create database schema ```sudo python setup_database.py```
  * Fill database sudo python ```catalogsAndItems.py```
  * Run python __init__.py and you should see: * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  * Deactivate the virtual environment: ```deactivate```.
2. Configure and Enable a New Virtual Host

  * Issue the following command in your terminal:
```
 sudo nano /etc/apache2/sites-available/catalog.conf
```
  * Add the following lines of code to the file to configure the virtual host.
```
 <VirtualHost *:80>
  	ServerName 52.66.170.56
 	ServerAlias http://52.66.170.56.xip.io
 	WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python2.7/sites-available
 	WSGIProcessGroup catalog
 	<Directory /var/www/catalog/catalog/>
 		Order allow,deny
 		Allow from all
 	</Directory>
 	Alias /static /var/www/catalog/catalog/static
 	<Directory /var/www/catalog/catalog/static/>
 		Order allow,deny
 		Allow from all
 	</Directory>
 	ErrorLog ${APACHE_LOG_DIR}/error.log
 	LogLevel warn
 	CustomLog ${APACHE_LOG_DIR}/access.log combined
 	WSGIScriptAlias / /var/www/catalog/catalog.wsgi

 </VirtualHost>
 ```
  * Enable the virtual host with the following command run:
```
 sudo a2ensite catalog
 sudo service apache2 reload
```
3. Create the .wsgi File

  * Create the .wsgi File under /var/www/FlaskApp:
```
 cd /var/www/catalog
 sudo nano catalog.wsgi
```
  * Add the following lines of code to the flaskapp.wsgi file:
```
 #! /usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/catalog/")

 from catalog import app as application
 application.secret_key = 'super_secret_key'
```
  * Restart Apache with the following command to apply the changes:
```
 sudo service apache2 restart
``` 
