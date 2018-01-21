# UD falsk python apache2 server

This is project is my last project of udacity web program. Here my this project is based on the deploying my 'item catalog webapp' based on flask miniframework on digital ocean.

## Notice
The name of database is `catalog` and the project so cloned is stored in folder named `catalog`.
This project is for linux user.

## Contents
These are the things which is required to make this project. 

    (a). Make a new server on Digital Ocean.
    (b). You need to SSH into the server.
    (c). Update all currently installed packages.
    (d). We need to change the SSH port from 22 to 2200. Make sure to configure the firewall to allow it.
    (e). Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80),          and      NTP (port 123).
    (f). we would be creating new user account named `grader`.
    (g). Give `grader` the permission to sudo.
    (h). Create an SSH key pair for `grader` using the ssh-keygen tool.
    (e). Configure the local timezone to UTC.
    (g). Install and configure Apache to serve a Python mod_wsgi application.
    (h). Install and configure PostgreSQL.
    (i). Install git.
    (j). Clone and setup your git project (here Catalog).
    (k). Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make            sure      that your .git directory is not publicly accessible via a browser.
    (l). Use a custom domain with Digital Ocean.
    (m). Disable root remote login and enforce key-based authentication.

### Stepwise demonstration of the project.
<a name="step1"></a>
## Step 1:

To start a new server on `Digital Ocean`, create a new droplet.

- Choose any distribution - I have choosen Ubuntu 16.04 x64.
- Size - As you wish
- Datacenter region - I have choosen near to my location, you can choose yours.
- Create a new ssh-key called `catalog_root`.
	- Open your terminal and run `ssh-keygen`.
	- You will be asked for the file path to save your key. Enter `/home/USER_NAME/.ssh/catalog_root` where *USER_NAME* is your username.
	- Set up a passphrase, here we are setting it to "password".
	- Done, you should have your key pair now.
- In the DO interface, click on *New SSH key* and paste the contents of `/home/USER_NAME/.ssh/catalog_root.pub` in it. Name the key as `catalog_root`. Enter.
- Keep number of droplets to `1`.
- Finish. Click on Create button.

YAHOO WE ARE READY WITH OUR DROPLET.


## Step 2:

To ssh into the server, copy the IP address from Digital Ocean interface. Here it is `139.59.46.47`.

ssh into the server with the following command.

```sh
ssh -i /home/USER_NAME/.ssh/catalog_root root@139.59.46.47
# ssh -i /full/path/to/key root@IP
# Here USER_NAME is your username
```

You will have to enter the passphrase for the ssh key that you created. 

Run `pwd` to check if things are working. 
congrats, you officially now have access to your server.

## Step 3:

To update currently installed packages, run the following command-

```sh
sudo apt-get update
sudo apt-get upgrade
# security updates
sudo apt-get install unattended-upgrades
# install them manually
sudo unattended-upgrades
```


## Step 4:

To change ssh port to 2200, follow the given steps-

Open ssh config in `nano`.

```sh
sudo nano /etc/ssh/sshd_config
```

Locate "Port 22" in that file. Change it to "Port 2200".

Restart ssh service.

```sh
sudo service ssh restart
```


## Step 5:

To configure UFW to allow given connections, follow the steps:

```sh
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
sudo ufw allow 2200/tcp
sudo ufw allow ssh
sudo ufw allow www
```

Then enable ufw.

```sh
sudo ufw enable
```

Now ssh port has been changed to 2200. Try exiting the ssh connection and re-connecting with the following command.

```sh
ssh -p 2200 -i ~/.ssh/catalog_root root@139.59.46.47
# YOU NEED TO GIVE YOUR OWN I.P
```
Check if you are connected via port `2200`.


## Step 6:

To create a user called `grader`, run the following command.

```sh
sudo adduser grader
```

Fill in the details, use `password` as the password for this user.
In detail it would ask you about basic things that you can do easily.

To check if the new user has been created successfully or not, run `ls /home/grader`. If the command exits without any problems, then that means
the path is valid.


## Step 7:

To give `grader` sudo permission, we first create `grader` file inside `sudoers.d`.

```sh
touch /etc/sudoers.d/grader
```

Then we open the file in `nano` using command (`sudo nano /etc/sudoers.d/grader`) and add the following line-

```sh
grader ALL=(ALL) NOPASSWD:ALL
```


## Step 8:

We will now have to setup a ssh key-pair for grader.

Follow the steps in [Step 1](#step1) to create a new ssh key at `/home/USER_NAME/.ssh/catalog_grader`.
Don't give a password this time as ssh keys are themselves meant to be credentials so you can say that they are themselves passwords.

Copy the contents of `catalog_grader.pub` file.

On the server terminal, run -

```sh
su - grader
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
# paste the contents and save the file
chmod 644 .ssh/authorized_keys
```

Restart the ssh service.

```sh
sudo service ssh restart
```

Now you should be able to login as `grader`. Exit current connection and do the following.

```sh
ssh -p 2200 -i ~/.ssh/catalog_grader grader@139.59.46.47
```


## Step 9:

Now we can configure timezone, just run the following command.

```sh
sudo dpkg-reconfigure tzdata
```

Select "None of the above" and then select UTC.

## Step 10:

To serve Python using Apache and mod_wsgi, install the following components.

```sh
sudo apt-get install python3
sudo apt-get install python3-setuptools
sudo apt-get install apache2 libapache2-mod-wsgi-py3
# this is for Python 3
# for python2 it is python, python-setuptools and libapache2-mod-wsgi
```

Then start apache service.

```sh
sudo service apache2 restart
```


## Step 11:

Install postgresql as follows

```sh
sudo apt-get install postgresql
```

To disable remote connections, make sure you don't have any other IPs besides `127.0.0.1` in the following file.

```sh
sudo nano /etc/postgresql/VERSION/main/pg_hba.conf
# VERSION = your postgres version
# here I have 9.5.10. check with the command psql --version
# sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

Now to create `catalog` database, run the following to get into psql shell.

```sh
sudo -u postgres psql
```

Then when inside psql shell, run the following.

```sql
create user catalog with password 'password';
# Here we are creating user name catalog
create database catalog with owner catalog;
```

Then exit psql shell with the following command.

```sql
\q
```


## Step 12:

To install git, run-

```sh
sudo apt-get install git
```


## Step 13:

My project can be found at https://github.com/AnkurBegining/ud-item-catalog- . It is private for now.

First we need to clone it on server.

```sh
cd /var/www
sudo git clone https://github.com/AnkurBegining/ud-item-catalog- catalog
```

Then we need to setup the project. 
```sh
# install pip3
sudo easy_install3 pip
# install requirements
cd catalog
sudo pip3 install -r requirements.txt
```


## Step 14:

Now you need to run the project using Apache and mod-wsgi. So we will first create a configuration file for your project.
Where .conf thus formed has name catalog
```sh
sudo nano /etc/apache2/sites-available/catalog.conf
```

It should have the following components.

```xml
<VirtualHost *:80>
    ServerName 165.227.182.77

    WSGIScriptAlias / /var/www/catalog/wsgi.py

    <Directory /var/www/catalog>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

The wsgi file should look like this.
application keyword is must
```python
import sys

sys.path.insert(0, '/var/www/catalog')

from item_catalog import app as application

application.secret_key = 'New secret key. Change it on server'
```

Once this is done, enable the site and restart Apache.

```sh
sudo a2ensite catalog  # enable site
sudo service apache2 reload
```

The server should be live now. Visit the IP to check (http://139.59.46.47). If an error occurs, check the logs.

```sh
sudo cat /var/log/apache2/error.log
```


## Step 15:

Now we would like to use a domain name for the server. I am using CloudShare as the domain registar but the steps should be same everywhere.

But why are we doing so? Because Google OAuth doesn't work for bare IPs
Since i already have a Domain so...
Now go visit your sub-domain. It should work. (Here: http://ankur.singhpratyush.in)

**IMPORTANT** You will have to to disable the default nginx site here. `sudo a2dissite 000-default`


## Step 16:

To disable root login & password-based login through ssh, open the ssh config file.

```sh
sudo nano /etc/ssh/sshd_config
```

Make changes as shown below.

```sh
PermitRootLogin no
PasswordAuthentication no
```

Save the file and restart ssh server.

```sh
sudo service ssh restart
```


https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user
https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets 
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

------
