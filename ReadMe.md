>   **Linux server configuration**

>   This project is about configuring Linux server on cloud, and I am using AWS
>   services to run my server

>   Also, I used Google login as credential control, so no need to handle the
>   login credentials and user date locally.

>   **Prerequisites Software:**

-   Instance IP: **3.121.219.47**

-   Instance URL: ec2-3-121-219-47.eu-central-1.compute.amazonaws.com

-   User terminal like Git Bash or another terminal

-   Python 3.7 or above

-   Any python IDE, as Atom or VScode

-   Flask

-   The Linux distribution
    is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.

-   The virtual private server is [Amazon
    Lighsail](https://lightsail.aws.amazon.com/).

-   The web app is item catalog project on this
    [link](https://github.com/Zakria-kh/Item.git)

-   The database server is [PostgreSQL](https://www.postgresql.org/).

>   **Lets get Started:**

1.  **Setup AWS instance as per the tutorial below**

-   Login into [Amazon
    Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) .

-   click Create instance.

-   Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.

-   Choose a instance plan

-   Rename your instance.

-   Click the Create button to create the instance.

-   Wait for the instance to start up.

1.  Access the instance locally using SSH

-   From account download the ssh key

-   Create .ssh directry in your home directory using this command mkdir .ssh

-   Copy the key into your ssh directory

-   Rename it to lightsail_key.rsa

-   In your terminal type: chmod 600 \~/.ssh/lightsail_key.rsa

-   To connect to the instance via the terminal: ssh -i
    \~/.ssh/lightsail_key.rsa ubuntu\@(your instance ip address)

1.  **Update the packages**

-   sudo apt-get update

    sudo apt-get upgrade

1.  **Enable automatic security updates**

>   sudo apt-get install unattended-upgrades

>   sudo dpkg-reconfigure --priority=low unattended-upgrades

1.  **Create a new user grader and Give him sudo access**

>   sudo adduser grader

>   sudo visudo

add this line ((grader ALL=(ALL:ALL) ALL))

after root ALL=(ALL:ALL) ALL

1.  **Change timezone to UTC and Fix language issues**

>   sudo timedatectl set-timezone UTC

>   sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8

1.  **Setup SSH keys for grader**

-   On local machine ssh-keygen Then choose the path for storing public and
    private keys

-   On remote machine home as user grader

>   sudo su - grader

>   mkdir .ssh

>   touch .ssh/authorized_keys

>   sudo chmod 700 .ssh

>   sudo chmod 600 .ssh/authorized_keys

>   nano .ssh/authorized_keys

1.  **Change ssh port**

-   To access your instance and deploy your app, you need to change the ssh port
    from 22 to 2200

-   Edit the /etc/ssh/sshd_config file by writing sudo nano
    /etc/ssh/sshd_config.

-   Change the port number on line 5 from 22 to 2200.

-   Find the PasswordAuthentication line and edit it to no.

-   Find the PermitRootLogin line and edit it to no.

-   Save and exit using CTRL+X and confirm with Y.

-   Restart SSH: sudo service ssh restart

-   Go to your instance setting in you aws account and change the networking
    rules, by deleting 22 and adding 2200 as tcp and 123 as udp

1.  **Access the instance through ssh**

-   ssh -i \~/.ssh/lightsail_key.rsa -p 2200
    [ubuntu\@3.121.219.47](mailto:ubuntu@54.93.51.7)

1.  **Configure the Uncomplicated Firewall**

>   sudo ufw default deny incoming

>   sudo ufw default allow outgoing

>   sudo ufw allow 2200/tcp

>   sudo ufw allow www

>   sudo ufw allow ntp

>   sudo ufw allow 8000/tcp

>   sudo ufw enable

1.  **Install Apache2 and mod-wsgi for python3 and Git**

-   sudo apt-get install apache2 libapache2-mod-wsgi-py3 git

1.  **configure PostegreSQL**

>   sudo apt-get install libpq-dev python3-dev

>   sudo apt-get install postgresql postgresql-contrib

>   sudo su - postgres

>   psql

1.  **create the DB and user**

>   CREATE USER catalog WITH PASSWORD 'password';

>   CREATE DATABASE catalog WITH OWNER catalog;

>   \\c catalog

>   REVOKE ALL ON SCHEMA public FROM public;

>   GRANT ALL ON SCHEMA public TO catalog;

>   \\q

>   Exit

1.  **Set up and enable a virtual host**

>   sudo apt-get install python3-pip

>   sudo -H pip3 install virtualenv

>   virtualenv env

>   source env/bin/activate

>   pip3 install -r requirements.txt

1.  **Change config/database.py content to :**

>   import os

>   db = {

>   'default': 'sqlite',

>   'sqlite': {

>   'driver': 'sqlite',

>   'database': os.path.join(os.path.dirname(file), '../database.db')

>   },

>   'mysql': {

>   'driver': 'mysql',

>   'host': 'localhost',

>   'database': 'catalog',

>   'user': 'catalog',

>   'password': 'password',

>   },

>   'postgres': {

>   'driver': 'postgres',

>   'host': 'localhost',

>   'database': 'catalog',

>   'user': 'catalog',

>   'password': 'password',

>   }

>   }

>   ORATOR_DATABASES = {

>   'development': db[db['default']]

>   }

1.  **cloning the web app, in my case I cloned my item catalog project**

-   In your terminal navigate to cd /var/www/

    >   New directory called catalog

>   sudo mkdir catalog

>   sudo chown grader:grader catalog

>   git clone https://github.com/Zakria-kh/Item.git catalog

>   cd catalog

>   git checkout production

>   nano catalog.wsgi

1.  **add the below code to your catalog.wsgi file**

>   \#!/usr/bin/python3

>   import sys

>   sys.stdout = sys.stderr

>   \# Add this if you'll create a virtual environment, So you need to activate
>   it

>   \# -------

>   activate_this = '/var/www/catalog/env/bin/activate_this.py'

>   with open(activate_this) as file_:

>   exec(file_.read(), dict(file=activate_this))

>   \# -------

>   sys.path.insert(0,"/var/www/catalog")

>   from app import app as application

>   application.secret_key = 'SUP3R_SEKR3T'

1.  **configure apache server**

>   sudo nano /etc/apache2/sites-enabled/000-default.conf

>   **copy the below line to the prombet will show after deleting what is
>   there**

>   \<VirtualHost \*:80\>

>   \# The ServerName directive sets the request scheme, hostname and port that

>   \# the server uses to identify itself. This is used when creating

>   \# redirection URLs. In the context of virtual hosts, the ServerName

>   \# specifies what hostname must appear in the request's Host: header to

>   \# match this virtual host. For the default virtual host (this file) this

>   \# value is not decisive as it is used as a last resort host regardless.

>   \# However, you must set it for any further virtual host explicitly.

>   ServerName 3.121.219.47

>   ServerAdmin webmaster\@localhost

>   DocumentRoot /var/www/catalog

>   WSGIDaemonProcess catalog user=grader group=grader

>   WSGIScriptAlias / /var/www/catalog/catalog.wsgi

>   WSGIProcessGroup catalog

>   WSGIApplicationGroup %{GLOBAL}

>   Require all granted

>   \# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,

>   \# error, crit, alert, emerg.

>   \# It is also possible to configure the loglevel for particular

>   \# modules, e.g.

>   \#LogLevel info ssl:warn

>   ErrorLog \${APACHE_LOG_DIR}/error.log

>   CustomLog \${APACHE_LOG_DIR}/access.log combined

1.  **Restart Appachi server**

-   sudo service apache2 restart

1.  **run the app through the ip : 3.121.219.47**

**References:**

-   DigitalOcean [How To Deploy a Flask Application on an Ubuntu
    VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

-   [Fix locale
    issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue).

-   [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/).

-   [Automatic Security
    Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package).

>   **This project made with Love 😊**
