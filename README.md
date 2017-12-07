# Linux Server Configuration Project

> This project is meant to deploy the [Catalog APP](https://github.com/esraaMohamed/catalog_app)
on Amazon Lightsail Ubuntu to using a wsgi application that handles the python flask application with python and postgresql database

> The app is also deployed on heroku on [Catalog App](https://sports-catalog-app.herokuapp.com/)

## Server Information:
	Public IP: 34.207.103.98
	SSH Port: 2200
	HTTP Port: 80
	NTP Port: 123
	Website URL: [Catalog APP](http://34.207.103.98/)

## Steps:
#### 1] Create a Lightsail Instance using Ubuntu from [Amazon Lightsail Start Page](https://amazonlightsail.com)

#### 2] Update Ubuntu packages using : 'sudo apt-get update' and 'sudo apt-get upgrade'

#### 3] Create New User Grader
		- Generate a key for the default user ubuntu on your local machine
		using `ssh-keygen` in this case I named the file grader then add it to the authorized_keys for that user
		on the server using the command `sudo nano .ssh/authorized_keys`

		- Log into the remote VM as ubuntu user through ssh:  `ssh ubuntu@34.207.103.98 -p 22 -i ~/.ssh/grader`

		- Add a new user called grader with password 'grader': `$ sudo adduser grader`

		- Create a new file in the suoders directory: `$ sudo nano /etc/sudoers.d/grader`

		- Edit the file using sudo and add this line to give grader sudo abilities:
		   `grader ALL=(ALL:ALL) ALL`


#### 4] Generate authentication keys for new user grader
		- Generate a key pair on your local machine with: `ssh-keygen` and name the file graderKey

		- Using the ubuntu user enable the PasswordAuthentication to yes so that I can login to the new user grader using its password

		- Restart the ssh service using the command : `sudo service ssh restart`

		- Log in remotely as grader using username and password  

		- Create an .ssh directory for the user grader using command : `mkdir .ssh`

		- Create an authorized_keys file for grader:
		   `touch /home/grader/.ssh/authorized_keys`

		- Copy the contents of graderKey.pub from your local machine to the `/home/grader/.ssh/authorized_keys` file

		- Change the permissions on the file and the .ssh directory:
		   `sudo chmod 700 /home/grader/.ssh` and `sudo chmod 644 /home/grader/.ssh/authorized_keys`

    - User grader can ssh with the following command: `$ ssh grader@34.207.103.98 -p 22 -i ~/.ssh/graderKey`

		- Edit the sshd_config file to enforce key pair authentication only by disabling password authentication
		  using the command `sudo nano /etc/ssh/sshd_config` and going through the file once again changing back
		  the PasswordAuthentication to no instead of yes, restart the service once again for the changes to take
			effect using the command : `$ sudo service ssh restart`

#### 5] Change ssh default Port to 2200 instead of 22
			- Open the sshd_config file `sudo nano /etc/ssh/sshd_config` and look for the line that contains the Ports and change it to 2200, save the change and then restart the ssh service using command `sudo service ssh restart`

    	# Now to connect using ssh we need to use port 2200 instead of 22: `ssh grader@34.207.103.98 -p 2200 -i ~/.ssh/graderKey`

#### 6] - Disable SSH for Root User
    	- By opening the sshd_config file using the command : `sudo nano /etc/ssh/sshd_config` look forthe PermitRootLogin line
			  and change it to no instead of prohibit-password save the file then restart the ssh service using the command :
    	   `sudo service ssh restart`

#### 7] - Configure the FireWall
   		- Change the firewall configurations based on project requirements by using the ufw command as follows:
			 ` sudo ufw default deny incoming `
			 ` sudo ufw default allow outgoing `
		   	 ` sudo ufw allow 2200/tcp `
		  	 ` sudo ufw allow 80/tcp `
		  	 ` sudo ufw allow 123/udp `
		  	 ` sudo ufw enable `

		 - Change the firewall settings on the Lightsail instance by:
		 	 - add a new rule Custom with port number 2200 on tcp
			 - add a new rule Custom with port number 123 on udp
			 - remove the ssh rule for port 22
			 - save your changes

#### 8] - Install Apache2 and WSGI
		- Install apache using the command : ` sudo apt-get install apache2` then check that the apache server is running by
		  visiting the public ip `34.207.103.98`

		- Install mod_wsgi using the command: `sudo apt-get install libapache2-mod-wsgi`

		- Restart apache2 to enable wsgi: `sudo /etc/init.d/apache2 restart`

#### 9] - Install Git to clone Catalog App repository
		- Install Git using the command: `sudo apt-get install git`

		- Set username using the command: `git config --global user.name <your git username>`

		- Set email using the command: `git config --global user.email <your git email>`

		- Create a directory for the repo called catalog-app using the command: `$ sudo mkdir /var/www/catalog-app`

		- Navigate to the catalog folder to clone the repo using the command: `cd /var/www/catalog-app`

		- Clone the catalog repo to the folder using the command: `sudo git clone https://github.com/esraaMohamed/catalog_app.git`

#### 10] - Create the wsgi file catalog-app.wsgi
			- Navigate to the html folder: ` cd /var/www/html`

			- Create the wsgi file using the command: `sudo touch catalog-app.wsgi`

			- Edit the wsgi file using the command: `sudo nano catalog-app.wsgi`

			- Insert these lines:
		    	```
			import sys
			import logging
			logging.basicConfig(stream=sys.stderr)
			sys.path.insert(0, "/var/www/catalog-app/catalog_app/catalog/")

			from catalog import app as application
			application.secret_key = 'super_secret_key'
			```

#### 11] - Installing Python Dependencies
		- First we need to install pip in order to install Python packages, to install pip we use the command: `sudo apt-get install python-pip`

		- Then we python using the command : `sudo apt-get install python-psycopg2`

	  - Install the python dependencies using the pip command :
		   `pip install Flask bleach httplib2 requests oauth2client SQLAlchemy flask_httpauth google_api_python_client passlib`

		- Install a virtual environment using the command: `sudo pip install virtualenv`

		- Move into the catalog directory using the command: `cd /var/www/catalog`

		- Create a virtual environment using the command: `sudo virtualenv venv`

		- Start the virtual environment using the command: `$ source venv/bin/activate`

#### 12] - Configure apache Virtual Host
		- Create the config file for the virtual host using the command : `sudo nano /etc/apache2/sites-available/catalog-app.conf`

		- Copy and paste the following to the file:
		   ```
			  <VirtualHost *:80>
				ServerName 34.207.103.98
				ServerAlias http://ec2-34-207-103-98.us-east-1.compute.amazonaws.com/
				ServerAdmin admin@34.207.103.98
				WSGIDaemonProcess catalog python-path=/var/www/catalog-app/catalog_app/catalog:/var/www/catalog-app/venv/lib/python2.7/site-packages
				WSGIProcessGroup catalog
				WSGIScriptAlias / /var/www/html/catalog-app.wsgi
				<Directory /var/www/catalog-app/>
				Order allow,deny
				Allow from all
				</Directory>
				Alias /static /var/www/catalog-app/catalog_app/catalog/static
				<Directory /var/www/catalog-app/catalog_app/catalog/static/>
				Order allow,deny
				Allow from all
				</Directory>
				ErrorLog ${APACHE_LOG_DIR}/error.log
				LogLevel warn
				CustomLog ${APACHE_LOG_DIR}/access.log combined
			</VirtualHost>
				```
		- Enable the virtual host using the command: `sudo a2ensite catalog-app`

		- Restart apache2 using the command: `sudo service apache2 restart`

#### 13] - Installing and Configure Postgresql and changing the python code to use Postgresql
		- Install python packages to work with psql using the command: `sudo apt-get install libpq-dev python-dev`

		- Install psql using the command: `sudo apt-get install postgresql postgresql-contrib`

		- Change to the super user postgres using the command: `sudo su - postgres`

		- Connect to psql using the command: `psql`

		- Create a new user named catalog and with password catalog_pass using the command:
		`CREATE USER catalog WITH PASSWORD 'catalog_pass';`

		- Give user catalog the CREATEDB ability using the command: `ALTER USER catalog CREATEDB;`

		- Create 'catalog' database for the catalog user using the command: `CREATE DATABASE catalog WITH OWNER catalog;`

		- Connect to db using the command: `\c catalog;`

		- Revoke all other rights using the command: `REVOKE ALL ON SCHEMA public FROM public;`

		- Only let the user catalog create tables using the command: `GRANT ALL ON SCHEMA public TO catalog;`

		- Log out of psql using the command: `\q`

		- Exit the psql shell and back to user grader using the command: `exit`

		- Inside the python file database_setup.py and catalog.py,
			change the database connection from `engine = create_engine('sqlite:///catalog.db')` to
			`engine = create_engine('postgresql://catalog:catalog_pass@localhost/catalog')`
			by using the commands : `sudo nano /var/www/catalog-app/catalog_app/catalog/database_setup.py` and
			`sudo nano /var/www/catalog-app/catalog_app/catalog/catalog.py` in lines 76 and 25 respectively

		- Setup the Database using the command: `$ python /var/www/catalog-app/catalog_app/catalog/database_setup.py`

		- Prevent remote access by opening the pg_hba.conf using the command: `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
			and checking to make sure it looks like :
			```
			# Database administrative login by Unix domain socket
			local   all             postgres                                peer

			# TYPE  DATABASE        USER            ADDRESS                 METHOD

			# "local" is for Unix domain socket connections only
			local   all             all                                     peer
			# IPv4 local connections:
			host    all             all             127.0.0.1/32            md5
			# IPv6 local connections:
			host    all             all             ::1/128                 md5
			```

#### 13] - Finally Launch the Catalog App
		- First restart the apache service using the command : `sudo service apache2 restart`

		- Then visit the app page at the public IP [The Catalog APP](http://34.207.103.98)


## Helpful resources
- [Linux Server Configuration](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175461/lessons/4331066009/concepts/48010894470923)

- [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

- [Installing Pip](https://docs.python.org/3/installing/index.html)

- [Installing Python Virtual Environments](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/)

- [mod_wsgi and Apache with a virtualenv Python environment](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

- [Install Postgresql](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
(https://wixelhq.com/blog/how-to-install-postgresql-on-ubuntu-remote-access)

- [Secure Postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

- [Udacity Forums](https://discussions.udacity.com/search?q=%20unable%20to%20open%20database%20file)
