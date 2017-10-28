Udacity Linux Server Configuration Project - Jerome Ronquillo
=======
This is a submission for the project, where a baseline installation of a Linux distrution was prepared on a virtual machine and is configured to host a web app. It is configured with the most recent linux updates, postgres database and a firewall.

## Server Information
___
> - IP address - 13.58.78.181, Port - 2200
> - Web Application URL - http://13.58.78.181/

## Software and Configuration
___
### Update Linux Distrubution with latest updates
___

- Login using providing information for default Amazon lightsail user 'ubuntu'

- Enter the following commands:

>$ sudo apt-get install updates

>$ sudo apt-get install upgrade

>$ reboot

### Create User 'grader' with sudo permissions
___

> $ sudo adduser grader

> $ sudo touch /etc/sudoers.d/grader

>$ sudo nano /etc/sudoer.d/grader

- Append the following line:
```
grader ALL=(ALL) NOPASSWD:ALL
```
- Type 'ctrl-o' to save.
- Type 'ctrl-x' to exit.

### Setup ssh-key based ssh login
___
- Generate SSH Key on local Machine:
>  $ ssh-keygen

- Note the filename and location (I used the default that was created at .ssh/id_rsa)
- Create passphrase for ssh key, (I created Passphrase: 'grader' for this instance)

### Copy public key from local machine to virtual machine
___

- Create directory for authorized ssh keys

> $ sudo mkdir .ssh

> $ touch .ssh/authorized_keys

> $ sudo nano .ssh/authorized_keys

- Copy public key from local machine (this instance used: .ssh/id_rsa) and paste into .ssh/authorized_keys file on virtual machine

> $ sudo chmod 700 .ssh

> $ sudo chmod 622 .ssh/authorized_keys

> $ sudo service ssh restart

- Login Command Example from Local Machine:
> $ ssh -i .ssh/id_rsa grader@13.58.78.181

### Change SSH Port that Virtual Machine is listening for to 2200
___
> $ sudo nano /etc/ssh/sshd_config

-   Delete Port 22 line or make sure Port 22 line is commented out: '# Port 22'
-  Append 'Port 2200' to file
```
# What ports, IPs and protocols we listen for
# Port 22
Port 2200
```
-  Type 'ctrl-o' to save.
- Type 'ctrl-x' to exit.

### Configure Uncomplicated Firewall
___
> $ sudo ufw default deny incoming

> $ sudo ufw default allow outgoing

> $ sudo ufw allow 2200/tcp

> $ sudo ufw allow 80/tcp

> $ sudo ufw allow 123/udp

> $ sudo ufw status

> $ sudo ufw enable

> $ sudo ufw deny 22

> $ sudo service ufw restart

### Configure Linux to use UTC timezone
___
> $ sudo dpkg-reconfigure tzdata

- Navigate to and Select 'None of the Above'
-  Navigate to and Select 'UTC'

### Apache and Python Mod-wsgi Configuration
___

> $ sudo apt-get install apache2

> $ sudo apt-get install libapache2-mod-wsgi

> $ sudo service apache2 restart

### Install and Configure PostgreSQL database
___
> $ sudo apt-get install postgresql

> $ sudo -u postgres psql postgres

> postgres=# CREATE DATABASE catalog;

> postgres=# CREATE USER catalog;

> postgres=# ALTER ROLE catalog with PASSWORD 'catalog';

> postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

> postgres=# \q

> $ sudo pip install psycopg2

### Install git, clone catalog app respository and configure postgresql database
___

>  $ sudo apt-git install git

> $ sudo mkdir /var/www/catalog

> $ git clone https://github.com/jrmronquillo/item_catalog

> $ sudo mv catalog /var/www/catalog

> $ cd /var/www/catalog

- Edit project.py, database_setup.py to use postgresql database instead of sqlite

- Note: This configuration change has already been made in the repository

- If app was cloned from https://github.com/jrmronquillo/item_catalog; the sqlite configuration is already commented out and replaced with postgresql as shown below:
```
# engine = create_engine('sqlite:///catalogwithusers.db')
engine = create_engine(
    'postgresql+psycopg2://catalog:catalog@localhost/catalog')
```

### Install Catalog App Dependencies
___

> $ sudo pip install Flask

> $ sudo pip install SQLAlchemy

> $ sudo pip install requests

> $ sudo pip install --upgrade oauth2client

### Configure Apache to work with mod_wsgi
___

> $ sudo nano /etc/apache2/sites-available/000-default.conf

Edit the file with the following lines:

```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/catalog/project.wsgi

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        WSGIDaemonProcess project user=ubuntu group=ubuntu threads=5 home=/var/www/
        WSGIScriptAlias / /var/www/catalog/project.wsgi

        <directory /var/www/catalog>
        WSGIProcessGroup project
        WSGIApplicationGroup %{GLOBAL}
        WSGIScriptReloading On
        Order deny,allow
        Allow from all
        </directory>

</VirtualHost>

```
Create wsgi file:
> 2. $ sudo touch /var/www/catalog/project.wsgi
> 3. $ sudo nano /var/www/catalog/project.wsgi

Append the following lines to the wsgi file:

```
import sys

sys.path.append('/var/www/catalog')

from project import app as application


```
-  'ctrl-o' to save.
-  'ctrl-x' to exit.

### Modify app.secret_key location
___
Move app.secret_key so that it becomes available to the app in the new wsgi configuration

- Edit the project.py file and move the app.secret_key out of :

```
if __name__ == '__main__':
    app.secret_key = 'super_secret_key1'
    app.run()
```
Move it to the following line:
```
app = Flask(__name__)

app.secret_key = 'super_secret_key1'
```

### Deploy App
___
> $ sudo service apache2 restart

## List of Third-Party Resources
---
- https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt

- https://stackoverflow.com/questions/26080872/secret-key-not-set-in-flask-session