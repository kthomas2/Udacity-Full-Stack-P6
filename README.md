## Linux Server Configuration Project

### Udacity - Full Stack Web Developer Nanodegree ###

Project 6: Configure an Amazon Lightsail Linux server instance to host web applications. Steps include securing the server from a number of attack vectors, installing and configuring a database server, and deployment of a web application.

### Amazon Lightsail Server Info

- **Public IP:** 18.222.167.4

- **SSH Port:** 2200

- **Project URL:** http://ec2-18-222-167-4.us-east-2.compute.amazonaws.com


### Configuration Summary

* Setup new Ubuntu Linux server instance on Amazon Lightsail per web site instructions.

* Download the private key from the Account page and copy to .ssh directory. SSH as ubuntu.
```
ssh -i c:/Users/xxxxxxx/.ssh/ls_key.rsa ubuntu@18.222.167.4
```

* Update all currently install packages
```
sudo apt-get update
sudo apt-get upgrade
```

* Configure Amazon Lightsail firewall on the Networking tab. Add two custom firewall rules.
```
Custom TCP 2200
Custom UDP 123
```

* Change SSH port from 22 to 2200, restart the service and log back in
```
sudo nano /etc/ssh/sshd_config
port 2200
sudo service sshd restart
ssh -i c:/users/xxxxxxx/.ssh/ls_key.rsa ubuntu@18.222.167.4 -p 2200
```

* Configure firewall
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/ntp
sudo ufw enable
```

* Add new user grader
```
sudo adduser grader
```

* Grant sudo permissions to grader. Copy 90-cloud-init-users as grader and edit file changing ubuntu to grader
```
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
grader ALL=(ALL) NOPASSWD:ALL
```

* Generate key on local machine. Make .ssh directory for grader. Create a new file call authorized-keys and copy key from grader_key.pub into .ssh/authorized-keys
```
ssh-keygen
sudo mkdir /home/grader/.ssh
sudo touch /home/grader/.ssh/authorized_keys
sudo nano /home/grader/.ssh/authorized_keys
```

* Change permissions
```
sudo chmod 700 /home/grader/.ssh
sudo chmod 644 /home/grader/.ssh/authorized_keys
sudo chown -R grader:grader /home/grader/.ssh
```

* Login with key pair
```
ssh -i c:/users/xxxx/.ssh/grader_key grader@18.222.167.4
```

* Set timezone
```
sudo timedatectl set-timezone UTC
```

* Install Apache and Python mod_wsgi and enable wsgi
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
```

* Install PostgreSQL and verify configurated to not allow remote connections (default)
```
sudo apt-get install postgresql
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

* Create Linux user - catalog
```
sudo adduser catalog
```

* Connect to psql and create PostgreSQL user - catalog. Restrict database permissions.
```
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'xxxxxx' CREATEDB; 'password';  
CREATE DATABASE catalog WITH OWNER catalog;  
\c catalog
REVOKE ALL ON SCHEMA public FROM public;  
GRANT ALL ON SCHEMA public TO catalog;
```

* Install GIT
```
sudo apt-get install git
```

* Create a directory for the catalog application, clone app from github and change owner to grader
```
sudo mkdir /var/www/catalog
sudo git clone https://github.com/kthomas2/Udacity-Full-Stack-P4.git catalog
sudo chown -R grader:grader /var/www/catalog
```

* Update database connection in database_setup.py, lotsofitems.py and application.py to connect to PostgreSQL instead of SQL
```
engine = create_engine('postgresql://catalog:XXXXX@localhost/catalog')
```

* Install software
```
sudo apt-get install python-pip
sudo pip install flask
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install sqlalchemy
sudo pip install psycopg2
sudo pip install psycopg2-binary
sudo pip install requests
sudo pip install virtualenv
sudo virtualenv venv
```

* Configure apache to handle requests
```
sudo nano /etc/apache2/sites-available/catalog.conf

<VirtualHost *:80>
		ServerName 18.222.167.4
		ServerAdmin kthomas@goodyear.com
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

* Create wsgi
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

* Restart apache
```
sudo service apache2 restart
```

* Active virtual environment
```
source vev/bin/activate
```

* Setup and populate database
```
python database_setup.py
python lotsofitems.py
```

* Rename application.py to __init__.py
```
sudo mv /var/www/catalog/catalog/application.py /var/www/catalog/catalog/__init__.py
```

* Disable default Apache site
```
sudo a2dissite 000-default.conf
sudo service apache2 reload
```

* Update __init__.py file to use full file path for client_secrets.json file.
```
sudo nano /var/www/catalog/catalog/__init__.py
/var/www/catalog/catalog/client_secrets.json
```

* Update Google OAuth authorized JavaScript origins and redirects
```
Origin - http://ec2-18-222-167-4.us-east-2.compute.amazonaws.com
Redirect - http://ec2-18-222-167-4.us-east-2.compute.amazonaws.com/gconnect
Redirect - http://ec2-18-222-167-4.us-east-2.compute.amazonaws.com/login
```

* Disable SSH for root 
```
sudo nano /etc/ssh/sshd_config
PermitRootLogin no
```

### Resources
Listed below are some of the resources I referenced. This project was difficult and required much googling. I captured a number of the significant sites that I utilized to complete this project, although there were many others that I reviewed.

- Udacity courses and quizzes -  Linux, Databases, Python
- https://www.postgresql.org/
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html#TroubleshootingInstancesConnectingMindTerm
- https://stackoverflow.com/questions/47342988/aws-ssh-port-timeout-after-changing-port-number
- https://forums.aws.amazon.com/thread.jspa?threadID=160352
- https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04
- https://forums.aws.amazon.com/thread.jspa?threadID=244253
- http://docs.sqlalchemy.org/en/latest/core/engines.html
- https://www.postgresql.org/docs/8.0/static/sql-createuser.html
- https://www.postgresql.org/docs/8.0/static/sql-createdatabase.html
- https://www.postgresql.org/docs/8.0/static/sql-revoke.html
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://github.com/bencam/linux-server-configuration
- http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
- http://www.codeasite.com/index.php/linux-a-apache/94-how-do-i-find-apache-http-server-log
- https://askubuntu.com/questions/603451/why-am-i-getting-the-apache2-ubuntu-default-page-instead-of-my-own-index-html-pa
- https://www.pythonanywhere.com/forums/topic/4200/
- http://swaroopsm.github.io/12-02-2012-Deploying-Python-Flask-on-Apache-using-mod_wsgi.html
- https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user
https://askubuntu.com/questions/449364/what-does-without-password-mean-in-sshd-config-file
