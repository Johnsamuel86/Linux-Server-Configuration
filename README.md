# Linux server configuration project

## Project Overview

This project is designed to prepare Linux server to host any web applications and illustrate how to secure your server from a number of attack vectors, install and configure a database server, and deploy your web applications onto it.

## Hosted Web app info
- IP address: 52.50.155.88
- SSH port  : 2200
- Application URL: http://ec2-52-50-155-88.eu-west-1.compute.amazonaws.com

## Installation

### 1 - Create a new user named *grader* and grant this user sudo permissions.

1- Log into the remote VM as *root* user through ssh.
2- Add a new user called *grader*: 
```$ sudo adduser grader```
3- Create a new file named 'grader' under the suoders directory:
```$ sudo nano /etc/sudoers.d/grader```
then fill it with the following line and save it: 
```grader ALL=(ALL:ALL) ALL ```
Source: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### 2 - Update packages

```$ sudo apt-get update```
```$ sudo apt-get upgrade```
### 3 - Install finger
```$ apt-get install finger```

### 4 - Configure the local timezone to UTC and NTP

1- Open time configuration dialog and set it to UTC with: 
`$ sudo dpkg-reconfigure tzdata`.
2- select *None of the above* then select *UTC*
Source: [askubuntu](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)

3- Install *ntp daemon ntpd* for a better synchronization: 
`$ sudo apt-get install ntp`.
Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime).

### 5 - Configure the key-based authentication for *grader* user

1- Generate an encryption key **on your local machine** using:
`$ ssh-keygen `.
2- Log into the remote VM as *root* user through ssh and create the following directory: 
`$ mkdir /home/grader/.ssh`
3- then create the follwing file to host your public key.
`$ touch /home/grader/.ssh/authorized_keys`.
4- Copy the content of the *key.pub* file from your local machine to that file you just created on the remote VM. Then change some permissions:
`$ sudo chmod 700 /home/grader/.ssh`.
`$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
`$ sudo chown -R grader:grader /home/grader/.ssh`.

### 6 - Enforce key-based authentication
`$ sudo nano /etc/ssh/sshd_config`
Find the *PasswordAuthentication* line and edit it to *no*.
then restart the ssh service
`$ sudo service ssh restart`.

### 7 - Change the SSH port from 22 to 2200
1- Open below file and find the *Port* line and edit it to *2200*.
`$ sudo nano /etc/ssh/sshd_config`
2- Restart the ssh service
`$ sudo service ssh restart`.
Source: [askubuntu](https://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server)
### 8 - Disable ssh login for *root* user
1- Open below file and find *PermitRootLogin* line and edit it to *no*.
`$ sudo nano /etc/ssh/sshd_config`
2- Restart the ssh service
`$ sudo service ssh restart`.

### 9 - Configure the Uncomplicated Firewall (UFW)

Allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) using below commands.

`$ sudo ufw allow 2200/tcp`.
`$ sudo ufw allow 80/tcp`.
`$ sudo ufw allow 123/udp`.
`$ sudo ufw enable`.
you can check the firewall stautus using:
`$ sudo ufw status`

### 10 - Install Apache and mod_wsgi

1- Install Apache
`$ sudo apt-get install apache2`.
2- Install mod_wsgi to work with python3
`$ sudo apt-get install libapache2-mod-wsgi-py3 python3-dev`.
3- Enable *mod_wsgi*: 
`$ sudo a2enmod wsgi`.
4- Restart Apache Service
`$ sudo service apache2 start`.

### 11 - Install Git

1- Install Git
`$ sudo apt-get install git`.
2- You can configure your username and email using following commands: 
`$ git config --global user.name <username>`.
`$ git config --global user.email <email>`.

### 12 - Clone the Catalog app from Github

1- Create first directory inside *www* directory to host your web app.
`$ cd /var/www`
`$ sudo mkdir catalog`.
2- Change owner for the *catalog* folder: 
`$ sudo chown -R grader:grader catalog`.
3- Move inside that newly created folder and clone the catalog repository from Github: 
`$ cd /catalog`
`$ git clone <your project> catalog`.
4- Make the GitHub repository inaccessible:
`$ cd /var/www/catalog/`
`$ sudo vim .htaccess`
Paste the following:
`RedirectMatch 404 /\.git`
Source: [stackoverflow](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)

### 13- Install pip installer for python3

`sudo apt-get install python3-pip`
Source: [stackoverflow](https://stackoverflow.com/questions/6587507/how-to-install-pip-with-python-3)

### 14 - Install virtual environment, Flask and the project's dependencies

1- Make sure you are in your app directory
`$ cd /var/www/catalog/catalog`
2- Use *pip3* to install *virtualenv* using the following command: 
`$ sudo pip3 install virtualenv`.
3- Create new virtual environment with the following command: 
`$ sudo virtualenv venv`.
4- Change permissions to the virtual environment folder: 
`$ sudo chmod -R 777 venv`.
5- Activate the virtual environment: 
`$ source venv/bin/activate`.
6- Install Flask: 
`$ pip3 install Flask`.
7- Install all the other project's dependencies: 
`$ pip3 install <package name>`. 

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).

### 15 - Create new virtual host

1- Create a virtual host conifg file: 
`$ sudo nano /etc/apache2/sites-available/catalog.conf`
2- Paste in the following lines of code:
```
 <VirtualHost *:80>
      ServerName <Server_IP_ADD>
      ServerAdmin admin@<Server_IP_ADD>
      ServerAlias <Server_Resolved_Name>
      WSGIDaemonProcess catalog python-home=/var/www/catalog/catalog/venv/
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

3- Enable the new virtual host: 
`$ sudo a2ensite catalog`.

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps).


### 16 - Create the .wsgi File and Restart Apache
1- Create wsgi file:
`$ cd /var/www/catalog`
`$ sudo vim catalog.wsgi`
2- Paste in the following lines of code:

```
#/var/www/catalog/catalog/venv/bin/python3
import sys
import logging
import site

site.addsitedir('/var/www/catalog/catalog/venv/lib/python3.5/site-packages')
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'your_secret_key'

```
3- Restart Apache Service:
`$ sudo service apache2 restart`


### 17 - Install and configure PostgreSQL

1- Install PostgreSQL: 
`$ sudo apt-get install postgresql postgresql-contrib`
2- Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user then connect to the database: 
`$ sudo su - postgres`
`$ psql`
3- Create a new user called 'catalog' with his password: 
`# CREATE USER catalog WITH PASSWORD 'yourpassword';`
4- Give *catalog* user the CREATEDB capability: 
`# ALTER USER catalog CREATEDB;`
5- Create the 'catalog' database owned by *catalog* user: 
`# CREATE DATABASE catalog WITH OWNER catalog;`
6- Connect to the database: 
`# \c catalog`
7- Revoke all rights: 
`# REVOKE ALL ON SCHEMA public FROM public;`
8- Lock down the permissions to only let *catalog* role create tables: 
`# GRANT ALL ON SCHEMA public TO catalog;`
9- Log out from PostgreSQL and then return to the *grader* user: 
`# \q`
`$ exit`.
10- change Flask application and database file with: 
```python
engine = create_engine('postgresql://catalog:yourpassword@localhost/catalog')
```
11- Setup the database with: 
`$ python3 /var/www/catalog/catalog/setup_database.py`.
12- To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: 
`$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` 
and it shouls be like :
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).


### 18 - Launch App and Troubleshooting
1- Restart Apache Service: 
`$ sudo service apache2 restart`
2- For troubleshooting you can view the error logs from:
`$ sudo tail /var/log/apache2/error.log`
