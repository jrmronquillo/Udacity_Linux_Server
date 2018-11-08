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

1. Login using providing information for default Amazon lightsail user 'ubuntu'

2. Enter the following commands:

    >$ sudo apt-get install updates

    >$ sudo apt-get install upgrade

    >$ reboot
    -----------
    Edit: The above commands worked on legacy Ubuntu. The do not work on >=16.04, instead use the below:
    >$ sudo apt update
    >$ sudo apt upgrade
    >$ reboot

### Create User 'grader' with sudo permissions
___
1. Enter the following to create user 'grader':

    > $ sudo adduser grader

    > $ sudo touch /etc/sudoers.d/grader

    >$ sudo nano /etc/sudoer.d/grader

2. Append the following line to the file:
    ```
    grader ALL=(ALL) NOPASSWD:ALL
    ```
3. Type 'ctrl-o' to save.
4. Type 'ctrl-x' to exit.

### Setup ssh-key based ssh login
___
1. Enter the following to Generate SSH Key on local Machine:
    >  $ ssh-keygen

- Note the filename and file location used (I used the default that was created at .ssh/id_rsa)
2. When prompted, create passphrase for ssh key (I created passphrase: 'grader' for this instance)

### Copy public key from local machine to virtual machine
___

1. On VPS, Create directory and file for authorized ssh keys

    > $ sudo mkdir .ssh

    > $ touch .ssh/authorized_keys

    > $ sudo nano .ssh/authorized_keys

2. Copy public key from local machine (this instance used: .ssh/id_rsa) and paste into .ssh/authorized_keys file on virtual machine

    > $ sudo chmod 700 .ssh

    > $ sudo chmod 622 .ssh/authorized_keys

    > $ sudo service ssh restart

- Exampe of login command from Local Machine:
> $ ssh -i .ssh/id_rsa grader@13.58.78.181

### Change SSH Port that Virtual Machine is listening for to 2200
___
1. Edit sshd config file:
    > $ sudo nano /etc/ssh/sshd_config

2.   Delete 'Port 22' line or make sure Port 22 line is commented out and Append 'Port 2200' to file like below:
    ```
    # What ports, IPs and protocols we listen for
    # Port 22
    Port 2200
    ```
3.  Type 'ctrl-o' to save.
4. Type 'ctrl-x' to exit.

### Configure Uncomplicated Firewall
___
1. Enter the following commands to configure defaults:

    > $ sudo ufw default deny incoming

    > $ sudo ufw default allow outgoing
2. Enter the following to allow only specified ports:

    > $ sudo ufw allow 2200/tcp

    > $ sudo ufw allow 80/tcp

    > $ sudo ufw allow 123/udp
3. Enable Firewall and make sure port 22 is disabled:

    > $ sudo ufw status

    > $ sudo ufw enable

    > $ sudo ufw deny 22

    > $ sudo service ufw restart
Note: If using Amazon Lightsail, Amazon also applies a firewall, need to make sure the same ports are enabled in the Amazon console as well.
### Configure Linux to use UTC timezone
___
1. Open linux time zone configuration:
> $ sudo dpkg-reconfigure tzdata

2. Navigate to and Select 'None of the Above'
3.  Navigate to and Select 'UTC'

### Apache and Python Mod-wsgi Configuration
___
1. Install Apache and mod-wsgi

> $ sudo apt-get install apache2

> $ sudo apt-get install libapache2-mod-wsgi

> $ sudo service apache2 restart

### Install and Configure PostgreSQL database
___

1. Enter the following to install PostgreSQL

    > $ sudo apt-get install postgresql

2. Create database user 'catalog'

    > $ sudo -u postgres psql postgres

    > postgres=# CREATE DATABASE catalog;

    > postgres=# CREATE USER catalog;

    > postgres=# ALTER ROLE catalog with PASSWORD 'catalog';

    > postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

    > postgres=# \q

3. Install pyscopg2 postgresql adapter
    > $ sudo pip install psycopg2

### Install git, clone catalog app respository and configure postgresql database
___
1. Install git

    >  $ sudo apt-git install git
2. Create main directory that server will use for serving application

    > $ sudo mkdir /var/www/catalog

3.  Clone files from git repository to server

    > $ git clone https://github.com/jrmronquillo/item_catalog

4. Move these files to main directory

    > $ sudo mv catalog /var/www/catalog

    > $ cd /var/www/catalog

5. Edit project.py, database_setup.py to use postgresql database instead of sqlite

- Note: This configuration change has already been made in the repository

- If app was cloned from https://github.com/jrmronquillo/item_catalog; the sqlite configuration is already commented out and replaced with postgresql as shown below:
    ```
    # engine = create_engine('sqlite:///catalogwithusers.db')
    engine = create_engine(
        'postgresql+psycopg2://catalog:catalog@localhost/catalog')
    ```

### Install Catalog App Dependencies
___
1. Install all frameworks that the application needs:

    > $ sudo pip install Flask

    > $ sudo pip install SQLAlchemy

    > $ sudo pip install requests

    > $ sudo pip install --upgrade oauth2client

### Configure Apache to work with mod_wsgi
___

1. Edit the Apache configuration file:
    > $ sudo nano /etc/apache2/sites-available/000-default.conf

2. Add the following lines:

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

3. Create wsgi file:
    > $ sudo touch /var/www/catalog/project.wsgi

    >  $ sudo nano /var/www/catalog/project.wsgi

4. Append the following lines to the wsgi file:

    ```
    import sys

    sys.path.append('/var/www/catalog')

    from project import app as application


    ```
5.  'ctrl-o' to save.
6.  'ctrl-x' to exit.

### Modify app.secret_key location
___
Move app.secret_key so that it becomes available to the app in the new wsgi configuration

1. Edit the project.py file and move the app.secret_key out of ...

    ```
    if __name__ == '__main__':
        app.secret_key = 'super_secret_key1'
        app.run()
    ```
- ...by moving it to the following line:
    ```
    app = Flask(__name__)

    app.secret_key = 'super_secret_key1'
    ```

### Deploy App
___
1. Restart apache to activate application configurations
> $ sudo service apache2 restart

## List of Third-Party Resources
---
- https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt

- https://stackoverflow.com/questions/26080872/secret-key-not-set-in-flask-session
