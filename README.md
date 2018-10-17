# Udacity-FullStack-Linux-server-configration
Set-up information for Udacity Full Stack Nanodegree project on how to configure a Linux Server.

## Project Description

> Taking a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application deployed here is the **Catalog Items Web App**, previously developed for [Item-Catalog Project](https://github.com/Amr905/Udacity-Catalog-Items).

## Server/App Info

IP address: 35.229.60.28

SSH port: 2200.

Application URL: [http://35.229.60.28](http://35.229.60.28).

Username and password for Udacity reviewer: `grader`, `123456`

## Configurations in Steps

### 1 - Launching an AWS Lightsail Instance and connect to it via SSH

1. Launching an google cloud VPS instance
2. The instance's security group provides a SSH port 22 by default
3. The public IP is 35.229.60.28

### 2 - User, SSH and Security Configurations

1. Create a new user *grader*:  `$ sudo adduser grader`.
2. Grant udacity the permission to sudo, by adding a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. In the file put in: `grader ALL=(ALL:ALL) ALL`, then save and quit.
3. Generate a new key pair by entering the following command at the terminal of your *local machine*.
    1. `$ ssh-keygen` with `grader_key`
    2. Print the public key `$ cat ~/.ssh/grader_key.pub`.
    3. Select the public key and copy it.
    4. Create a new directory called .ssh `$ mkdir .ssh` on your *virtual machine*.
4. Paste the public key `grader_key.pub` to `authorized_keys`, and change the permissions:
	1. `$ sudo chmod 700 /home/grader/.ssh`.
	2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	3. Change the owner from `ubuntu` to `grader`: `$ sudo chown -R grader:grader /home/grader/.ssh`
5. Enforce key-based authentication, change SSH port to `2200` and disable remote login of *root* user:
   1. `$ sudo nano /etc/ssh/sshd_config`  
   2. Change `PasswordAuthentication` to `no`.
   3. Change `Port` to `2200`.
   4. Change `PermitRootLogin` to `no`
   5. `$ sudo service ssh restart`.
   6. In AWS Lightsail Security Group,  add `2200` as the inbound custom TCP Rule port.



### 3 - Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.


### 4 - Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw default deny incoming`.
2. `$ sudo ufw default allow outgoing`.
3. `$ sudo ufw allow 2200/tcp`.
4. `$ sudo ufw allow 80/tcp`.
5. `$ sudo ufw allow 123/udp`.
6. `$ sudo ufw enable`.


### 5 - Install Apache, mod_wsgi and Git

1. `$ sudo apt-get install apache2`.
2. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
4. `$ sudo service apache2 start`.
5. `$ sudo apt-get install git`.

### 6 - Configure Apache to serve a Python mod_wsgi application

1. Clone the catalog-item app from Github
   ```
   $ cd /var/www
   $ sudo mkdir catalog
   $ sudo chown -R grader:grader catalog
   $ cd catalog
   $ git clone https://github.com/Amr905/Udacity-Catalog-Items catalog
   ```
2. To make .git directory is not publicly accessible via a browser, create a .htaccess file in the .git folder and put the following in this file:  `RedirectMatch 404 /\.git`
3. Install pip , virtualenv (in /var/www/catalog)
   ```
   $ sudo apt-get install python-pip
   $ sudo pip install virtualenv
   $ sudo virtualenv venv
   $ source venv/bin/activate
   $ sudo chmod -R 777 venv
   ```
4. Install Flask and other dependencies:
   ```
    $ sudo pip install httplib2
    $ sudo pip install flask
    $ sudo pip install sqlalchemy
    $ sudo pip install flask_sqlalchemy
    $ sudo pip install psycopg2
    $ sudo pip install requests
    $ sudo pip install oauth2client
   ```

5. Configure and Enable a New Virtual Host
   ```
   $ sudo nano /etc/apache2/sites-available/catalog.conf
   ```
   Add the following content:
   
   ```
   <VirtualHost *:80>
   ServerName 35.229.60.28

   ServerAdmin grader@35.229.60.28
   WSGIProcessGroup catalog
   WSGIDaemonProcess catalog python-path=/var/www/catalog/venv/lib/python3.5/site-packages
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
   Enable the new virtual host:
   ```
   $ sudo a2ensite catalog
   ```
7. Create and configure the .wsgi File
   ```
   $ cd /var/www/catalog/
   $ sudo nano catalog.wsgi
   ```
   Add the following content:
   ```python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'secret'
   ```
the server now is running with the project 
# Resources 
https://libraries.io/github/golgtwins/Udacity-P7-Linux-Server-Configuration
https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps

