# Linux Server Configuration

Taking a baseline installation of a Linux distribution on a virtual machine in Amazon Web Services and hosting my web applications from the third project Item Catalog.

To visit the website vist [here](http://18.216.75.253/)

IP Address: 18.216.75.253

SSH port: 2200

URL: http://ec2-18-216-75-253.us-east-2.compute.amazonaws.com/

## Tasks

#### Getting server.

1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. SSH into the server.
#### Securing server.

3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Making sure to configure the Lightsail firewall to allow.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
#### Give grader access.

6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.
#### Prepare to deploy your project.

9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.

11. Install and configure PostgreSQL:

    Not allowing remote connections and
    create a new database user named catalog that has limited permissions to your catalog application database.

12. Install git.
#### Deploy the Item Catalog project.

13. Clone and setup Item Catalog project from the Github repository created from the third project of Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!


## Launching Virtual Machine
1. Sign up for [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Create an instance image: Ubuntu
3. Choose a plan and hostname

## Updating the server
* Clicked the Connect using SSH and updated using: ``sudo apt-get update`` and ``sudo apt-get upgrade``


## SSH port from 22 to 2200

In the Firewall in the Amazon Lightsail tab Networking added Custom for port 2200. In the terminal ``sudo ufw allow 2200/tcp`` and edit file ssh_config `` sudo nano /etc/ssh/sshd_config`` change the line "Port 22" to "Port 2200" next ``sudo service ssh restart`` to restart the service for SSH.


## Configure the Uncomplicated Firewall
Before editing the ssh_config file, ``sudo ufw allow www``, ``sudo allow 123/udp`` and ``sudo allow out 123/udp``

## Add user grader and give access
1. ``sudo adduser grader`` to create user grader
2. To give sudo access for grader login in as ubuntu in terminal ``sudo cp /etc/sudoers.d /etc/sudoers.d/grader ``
3. Next ``sudo nano /etc/sudoers.d/grader`` and repalce ubunut to grader

## Create SSH key for grader
After generated key pair from local machine,

1. login into the server as grader and create a directory .ssh using ``mkdir .ssh``
2. next create a new file authorized_keys within the directory .ssh using ``touch .ssh/authorized_keys``
3. In local machine of the pub file copy and paste in authorized_keys file using ``nano .ssh/authorized_keys``
4. Last is changing the permissions within the .ssh directory and the authorized_keys file using ``chmod 700 .ssh`` and ``chmod 644 .ssh/authorized_keys``

Now to login as grader use ``ssh grader@18.216.75.253 -p 2200 -i ~/.ssh/LinuxUser`` with password.

## Local timezone to UTC
Use ``sudo dpkg-reconfigure tzdata`` to open the timezone and change it to UTC

## Install Apache to serve a Python mod_wsgi application
1. ``sudo apt-get install apache2`` to install Apache
2. ``sudo apt-get install libapache2-mod-wsgi`` to install libapache2-mod-wsgi


## Install PostgreSQL and create user and database
1. ``sudo apt-get install postgresql`` to install PostgreSQL
2. ``sudo -u postgres createuser catalog`` to create user catalog
3. ``sudo -u postgres createdb -O catalog mydatabase`` to create database mydatabase

## Install Git
``sudo apt-get install git`` to install Git



## Deploy the Catalog project
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
1. login as ubuntu user terminal and make a new directory within /var/www/ ``mkdir Catalog`` then another directory with the same name ``mkdir Catalog``
2. next getting the project from github using ``git clone https://github.com/fonzi956/Catalog.git``
3. then ``cd ..`` to go back to the first Catalog directory to add file catalog.wsgi using touch catalog.wsgi and edit using ``sudo nano catalog.wsgi`` and copy and paste the following
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Catalog/")

from Catalog import app as application
application.secret_key = 'super_secret_key'

```
also edit __init__.py
line that has create_engine

to

``engine = create_engine('postgresql://user:password@localhost/database')``

and line flow_from_clientsecrets

to


``oauth_flow = flow_from_clientsecrets('/var/www/Catalog/Catalog/client_secrets.json', scope='')``

## Update Google OAuth and client secrets from the Catalog project

Within [Google Dashboard](https://console.developers.google.com/apis/) click the credentials tab and edit the OAuth 2.0 client ID of the Authorized JavaScript origins to

``http://ec2-18-216-75-253.us-east-2.compute.amazonaws.com`` and ``http://18.216.75.253``

and  Authorized redirect URIs to

``https://www.example.com/oauth2callback``

then download the json and upload to server or copy and past from the client_sercets.json

## Configure Apache2 to serve Catalog project
``cd /etc/apache2/sites-available/`` and ``touch Catalog.conf`` and paste the following:

```
<VirtualHost *:80>
                ServerName 18.216.75.253
                ServerAdmin fonzi850@gmail.com
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

next ``sudo a2dissite 000-default.conf`` to disable the default virtual host then ``sudo a2ensite Catalog.conf`` last is to restart the server using ``sudo service apache restart``

## Disable root login

``cd /etc/ssh/sshd_config`` and edit 

``PermitRootLogin yes``

to

``PermitRootLogin no`` 

then ``service ssh restart``

## Resources

*  https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
*  https://discussions.udacity.com/t/i-got-the-google-pop-up-login-to-somewhat-work-but-wont-render-to-login/499516
*  others in https://discussions.udacity.com
