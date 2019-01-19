# Linux Server Configuration
Linux Server Configuration is a project for Udacity Full stack Nanodegree course.   
It is about install a Linux server and prepare it to host web applications, secure the server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## IP Adress
http://52.57.98.57.xip.io/

## Technologies used
* Amazon Lightsail
* Apache
* ssh-keygen tool
* PostgreSQL
* Python
* HTML
* CSS
* OAuth
* Flask Framework


## Amazon Lightsail Setup
1. Log in to [Lightsail](https://aws.amazon.com/lightsail/). If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Create an instance Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
3. Choose an instance image: Ubuntu Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications
For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
4. Choose your instance plan : The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine.
5. Give your instance a hostname : Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it
6. After naming the instance click create and wait for it to start up
7. Once your instance has started up, you can log into it with SSH from your browser.
The public IP address `52.57.98.57`
8. Download the SSH key pairs from Account page at the bottom
9. Configure the ports By default the firewall is set to only allow connects from port 22 and port 80. We need to set up port 2200 and 123. click the networking tab,click add another under "Firewall" and choose Custom for application, TCP for protocol, and the port number under Port Range. Then click save.

## Linux Configuration
1. Move The downloaded key in .ssh folder you can find it in windows at this path `C:\Users\WinDows\.ssh`
2. To secure the key type into the terminal `# chmod 600 ~/.ssh/[keyname].pem`
3. Log into the server as ubuntu user with our key type into the terminal `$ ssh -i ~/.ssh/[keyname].pem ubuntu@[PUBLIC IP ADDRESS]``
4. Switch to root user type into the terminal `$ sudo su -`

#### Update all currently installed packages
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
```
#### Install finger
`$ sudo apt-get install finger`

#### Create New Users
###### 1. Create grader
 1.  Type into the terminal `$ sudo adduser grader`
 2. Enter password
 3. There are some field you can leave it blank

###### 2. Give grader the permission to sudo
1. Create a new file type into the terminal `$ sudo nano /etc/sudoers.d/grader`
2. Paste into the file `grader ALL=(ALL:ALL) ALL`
3. Close and save the file

###### 3. Create an SSH key pair
1. To generate the key type into the terminal `$ ssh-keygen -f ~/.ssh/linuxProject.rsa`
2. Enter passphrase to prevent anyone from accessing the file
3. Read and copy the public key type into the terminal `$ cat ~/.ssh/linuxProject.rsa.pub`
4. Type into the terminal  `$ cd /home/grader`
4. Create directore type into the terminal `$ mkdir .ssh`
5. Create file type into the terminal `$ touch .ssh/authorized_keys`
6. Edit the file to paste the puplic key in it  type into the terminal `$ nano .ssh/authorized_keys`
7. Set up file permissions type into the terminal
```
$ chmod 700 /home/grader/.ssh
$ chmod 644 /home/grader/.ssh/authorized_keys
```
8. Change the owner of the .ssh directory from root to grader type into the terminal `$ sudo chown -R grader:grader /home/grader/.ssh`
9. Disable password base logins type into the terminal `$ sudo nano /etc/ssh/sshd_config`
`
change PasswordAuthentication  yes to no  
change PermitRootLogin from prohibit-password to no

10. Restart the service type into the terminal `$ sudo service ssh restart`
11. login as grader user `$ ssh -i ~/.ssh/linuxProject.rsa grader@52.57.98.57`

#### Change the SSH port and configure the Uncomplicated Firewall
1. Change the SSH port you have to edit file type into the terminal `$ sudo nano /etc/ssh/sshd_config` change Port from 22 to 2200

2. Restart the service type into the terminal
`$ sudo service ssh restart`
3. Configure Uncomplicated Firewall
type into the terminal
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw deny 22
$ sudo ufw enable
```
4. After changing the port to log as grader user
`$ ssh -i ~/.ssh/linuxProject.rsa grader@52.57.98.57 -p 2200`

## Application Deployment
#### Configure the local timezone to UTC
`$ sudo timedatectl set-timezone UTC`

#### Install required packages
1. Install and configure Apache to serve a Python mod_wsgi application
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
```
2. Enable mod_wsgi
`$ sudo a2enmod wsgi`
3. restart Apache
`$ sudo service apache2 restart`
4. If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page

#### create a directory for our catalog application
1. Create a directory and make grader the owner
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R $ grader:grader catalog
cd catalog
```
2. Clone the project  [catalog project](https://github.com/Lo0ofah/Item_Catalog) `$ git clone [repository url] catalog`

3. Create .wsgi file
`sudo nano catalog.wsgi`
add the following inside it
```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'
```
4. Rename the main python file to `__init__.py` by the following  command
`mv project.py __init__.py`
5. Create our virtual environment, make sure you are in `/var/www/catalog`
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
6. Install Flask and othe packages
```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2
$ sudo pip install requests
$ sudo pip install --upgrade oauth2client
$ sudo apt-get install libpq-dev
$ sudo pip install sqlalchemy_utils
```
7. nano to `__init__.py` and change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
8. Configure and enable our virtual host to run the site
create file `$ sudo nano /etc/apache2/sites-available/catalog.conf`
add the following inside it
```
<VirtualHost *:80>
    ServerName [Public IP]
    ServerAlias [Hostname]
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
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
</VirtualHost>
```
To find the servers [Hostname]( https://whatismyipaddress.com/ip-hostname ) open the link and paste the IP address
9. Enable to virtual host `$ sudo a2ensite catalog.conf`
10. Disable the default host `$ a2dissite 000-default.conf`
11. Deactivate the virtual environment `$ deactivate`

## Setup the database
1. Install the following
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
```
2. Create a database user and password
```
# CREATE USER catalog WITH PASSWORD [password];
# ALTER USER catalog CREATEDB;
# CREATE DATABASE catalog with OWNER catalog;
# \c catalog
# REVOKE ALL ON SCHEMA public FROM public;
# GRANT ALL ON SCHEMA public TO catalog;
# \q
```
3. Edit these files  `__init__.py`, catalog_database_setup.py, and catalog_data.py and replace `sqlite:///databasename.db` to `postgresql://catalog:[password]@localhost/catalog` in create_engin(...)
4. Run these files catalog_database_setup.py, catalog_data.py
by `$ python filename.py`

## Setup Google OAuth
1. Go to console cloud google
2. From OAuth consent screen tab add to Authorized domains `xip.io`
3. Add to Authorized JavaScript origins `http://52.57.98.57.xip.io`
4. Add to Authorized redirect URIs `http://52.57.98.57.xip.io/login` and `http://52.57.98.57.xip.io/gconnect `
5. Download the updated JSON file
6. Edit the client_secrets files `$ sudo nano client_secrets.json`
paste into it the updated JSON file
7. Restart `$ sudo service apache2 restart`
8. Open into the browser your ip addres with extension xip.io
http://52.57.98.57.xip.io/

## References
- https://github.com/mulligan121/Udacity-Linux-Configuration
