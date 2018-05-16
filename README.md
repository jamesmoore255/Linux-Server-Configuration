# Udacity Full Stack Web Developer Nanodegree Project

## Linux Server Configuration

### *Project Details*
##### Preparing and configuring a Linux Server, and deploying a Flask based Python application using postgreSQL, Facebook SignIn, Ubuntu. Also configuring a new user named grader with sudo permission.
* IP address: 54.238.25.234
* SSH port: 2200
* Application URL: ec2-54-238-25-234.ap-northeast-1.compute.amazonaws.com

### *Guide*
1. ssh into the newly created amazon lightsail server using 

		"ssh ubuntu@IP-ADDRESS -p 22 -i path-to-file/LightsailDefaultPrivateKey.pem"
2. Update all available packages for the Linux Server
3. Configure the UFW to allow incoming connections for SSH(port 2200), HTTP(port 80) and NTP(port 123), allow SSH incoming connections, allow all outgoing requests, then enable the UFW.
4. Change the SSH port within the sshd_config file from 22 to 2200 to allow the ssh incoming connection from 2200:

		"sudo nano /etc/ssh/sshd_config"
	* You will now be able to ssh into the server using port 2200
5. Create user 'grader'

		"sudo adduser grader"
6. Give the new user permission to sudo
	* Create a new file in the sudoers directory with:

		"sudo nano /etc/sudoers.d/graders"
	* Add the following text:

		'grader ALL=(ALL) NOPASSWD:ALL'
7. Configure local timezone:

		"sudo dpkg-reconfigure tzdata"
	* Then choose UTC
8. Generate a private and public key using the ssh-keygen tool on your local machine
	* Make a directory within grader called .ssh

		"sudo mkdir .ssh"
	* Create a file called authorized_keys

		"sudo nano /.ssh/authorized_keys"
		* Copy and paste the public key for grader made by ssh_keygen into the authorized_keys file.
	* You will now be able to login using grader and the passphrase for grader:

		"ssh grader@IP-ADDRESS -p 2200 -i path-to-file/graderPrivateKey"
9. Install Apache

		"sudo apt-get install apache2"
10. Install mod_wsgi
		"sudo apt-get install libapache2-mod-wsgi python-dev"
	* Enable mod_wsgi with: 

		"sudo a2enmod wsgi"
	* Start the webserver with:

		"sudo service apache2 start"
11. Install git, then Clone Catalog app from Github
		"sudo apt-get install git"
	* cd into /var/www
	* mkdir catalog
	* Change the owner of newly created folder to grader:

		"sudo chown -R grader:grader catalog"
	* cd into catalog
	* Clone your project into another folder named catalog within the catalog folder

		"git clone https://github.com/username/catalog-app.git catalog"
	* Create catalog.wsgi file in the first catalog folder, then add:

		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, "/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'supersecretkey'
	* Rename project.py to init.py 

		"mv project.py __init__.py"
12. Install virtual environment
	* Install the virtual environment:

		"sudo pip install virtualenv"
	* Create a new virtual environment with:

		"sudo virtualenv venv"
	* Activate the virutal environment: 

		"source venv/bin/activate"
	* Change permissions 

		"sudo chmod -R 777 venv"
	* Install Flask and other dependencies:
		
		"sudo apt-get install python-pip"
		"pip install Flask"
		"sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils"
	* Update path of fb_ client_secrets.json file in __init__.py to:
		
		"/var/www/catalog/catalog/client_secrets.json"
13. Configure and enable a new virtual host
		
		"sudo nano /etc/apache2/sites-available/catalog.conf"
	* Paste this code:

	<VirtualHost *:80>
	    ServerName IPADDRESS
	    ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
	    ServerAdmin admin@IPADDRESS
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

	* Enable the virtual host:

		"sudo a2ensite catalog"
14. Install and configure PostgreSQL

		"sudo apt-get install libpq-dev python-dev"
		"sudo apt-get install postgresql postgresql-contrib"
		"sudo su - postgres"
	* Enter into the postgreSQL command-line using:

		"psql"
	* Then create a user and database within the psql command line: 
		
		CREATE USER usercatalog WITH PASSWORD 'password';
		ALTER USER catalog CREATEDB;
		CREATE DATABASE usercatalog WITH OWNER usercatalog;
		\c usercatalog
		REVOKE ALL ON SCHEMA public FROM public;
		GRANT ALL ON SCHEMA public TO catalog;
		\q
		exit
	* Alter the create engine line in your __init__.py and database_setup.py to: 

		"engine = create_engine('postgresql://usercatalog:password@localhost/usercatalog')"
	* Activate the database:
		
		"python /var/www/catalog/catalog/database_setup.py"

	* Check if the contents of the file pg_hba.conf looks as follows, to make sure no remote connections are allowed: 

		"sudo nano /etc/postgresql/9.3/main/pg_hba.conf"

		local   all             postgres                                peer
		local   all             all                                     peer
		host    all             all             127.0.0.1/32            md5
		host    all             all             ::1/128                 md5

15. Check for other errors:
	* There may be other erros which come up in the code, for example for the Facebook login, it is necessary to include the IPADDRESS and Application URL are included in the valid OAuth Redirect URIs.

16. Restart Apache
		
		"sudo service apache2 restart"

Visit site at http://54.238.25.234