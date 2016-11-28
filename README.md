# Udacity's Linux Server Configuration Project

We were given a baseline installation of a Ubuntu Linux distribution on a virtual machine and asked to configure it in order to host a Flask web application. Some of the things that needed to be done include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Server Information
1. IP Address:
``` 
35.161.52.116
```
2. SSH Port:
``` 
2200 
```

## Web Application URL
```
http://35.161.52.116 
```

## Create Development Environment
(Resource: [Udacity](https://www.udacity.com/account#!/development_environment))

1. Create a new development environment based on the instructions Udacity provided in the project prep section.

2. Download the private key Udacity provided.

3. Move the private key to the folder ` ~/.ssh/ `:<br> 
` $ mv ~/Downloads/udacity_key.rsa ~/.ssh/ `

4. Change the modification permissions:<br>
` $ chmod 600 ~/.ssh/udacity_key.rsa ` 

5. Type in ` $ ssh -i ~/.ssh/udacity_key.rsa root@35.161.52.116 ` to access the terminal of the vm.

## Configuration Details
(Resource: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)) <br>

1. Create new user named grader: <br>
` $ sudo adduser grader `

2. Grant **grader** user sudo permissions:<br> 
Open configuration file:<br>
` $ sudo visudo `<br>
Add  the following code below ` root  ALL=(ALL:ALL) ALL `:<br>
` grader ALL=(ALL:ALL) ALL `<br>

3. Update and Upgrade packages:<br>
Update package list<br>
` $ sudo apt-get update `<br>
Upgrade packages<br>
` $ sudo apt-get upgrade `<br>

4. Configure the local time to UTC:<br>
(Resource: [Ask Ubuntu](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt))<br>
` $ sudo dpkg-reconfigure tzdata `<br>
Select **Other** from the menu then select **UTC** and press enter.

5. Change SHH port to 2200:<br>
(Resource: [Ask Ubuntu](http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server))<br>
  - Open the SSH configuration file<br>
  ` $ sudo nano /etc/ssh/sshd_config `<br>
  - In the config file change port 20 to ` 2200 `<br>
  - Change the ` PermitRootLogin ` to ` no `<br>
  - Add ` AllowUsers grader ` to the file to allow **grader** user to login through SSH<br> 
  - Restart SSH<br>
  ` $ sudo service ssh restart `<br>

6. Create SSH Keys:<br>
(Resource: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server))<br>
On local machine generate a SSH key pair by typing:<br>
` $ ssh-keygen `<br>
Open and copy the public key on local machine:<br>
` $ cat ~/.ssh/id_rsa.pub `<br>
Login to SSH<br>
` $ ssh grader@35.161.52.116 `<br>
Create a `.ssh` directory<br>
` $ mkdir .ssh `<br>
Create an `authorized_keys` file to store the public key<br>
` $ touch .ssh/authorized_keys `<br>
Open the `authorized_keys` file and paste the public key<br>
` $ nano .ssh/authorized_keys `<br>
Change the permissions on the `.ssh` folder and `authorized_keys` file<br>
` $ chmod 700 .ssh `<br>
` $ chmod 644 .ssh/authorized_keys `<br>
Disable Password Authorization in SSH config file<br>
` $ sudo nano /etc/ssh/sshd_config `<br>
Restart SSH service<br>
` $ sudo service ssh restart `<br>

7. Configure Uncomplicated Firewall(UFW):<br>
(Resource: [Udacity]())<br>
Deny all incoming connections<br>
` $ sudo ufw default deny incoming `<br>
Allow all outgoing connections<br>
` $ sudo ufw default allow outgoing `<br>
Allow SSH on port 2200<br>
` $ sudo ufw allow 2200/tcp `
Allow HTTP<br>
` $ sudo ufw allow 80/tcp `
Allow NTP<br>
` $ sudo ufw allow 123/udp `<br>
Enable UFW<br>
` $ sudo ufw enable `<br>

8. Install Apache and mod_wsgi:<br>
(Resource: [Udacity](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4340119836/concepts/48189486140923#))<br>
Install Apache and mod_wsgi<br>
` $ sudo apt-get install apache2 libapache2-mod-wsgi python-setuptools `<br>
Enable mod_wsgi<br>
` $ sudo a2enmod wsgi `
Restart Apache Service<br>
` $ sudo service apache2 restart `<br>

9. Install and setup PostgreSQL<br>
(Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps))<br>
Install PostgreSQL<br>
` $ sudo apt-get install postgresql postgresql-contrib `<br>
**Confirm that Remote Connections are denied**<br>
Open PostgreSQL config file<br>
` $ sudo nano /etc/postgresql/9.1/main/pg_hba.conf `<br>
Make sure that the seetings are the same as below<br>
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Connect to database system<br>
` $ psgl `<br>
Create user **catalog** and give it the ability to create databases<br>
(Reference: [PostgreSQL](https://www.postgresql.org/docs/8.0/static/sql-createuser.html))<br>
` CREATE USER catalog WITH PASSWORD '<password>'; `<br>
` ALTER USER catalog CREATEDB; `<br>
Create a database named **catalog**<br>
(Reference: [PostgreSQL](https://www.postgresql.org/docs/9.0/static/sql-createdatabase.html))<br>
` CREATE DATABASE catalog WITH OWNER catalog; `<br>
Lock down database so only **catalog** user can create tables<br>
` REVOKE ALL ON SCHEMA public FROM public; `<br>
` GRANT ALL ON SCHEMA public TO catalog; `<br>
Exit database system<br>
` \q `<br>

10. Setup Server for Flask App<br>
(Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps))<br>
Create initial project directory in `/var/www/` directory<br>
` $ sudo mkdir catalog `<br>
Move to new directory<br>
` $ sudo cd catalog `<br>
Create another directory<br>
` $ sudo mkdir catalog `<br>
Move to new directory<br>
` $ sudo cd catalog `<br>
Create a directory **static** and **templates**<br>
` $ sudo mkdir static templates `<br>
Install PIP<br>
` $ sudo apt-get install python-pip `<br>
If _virtualenv_ is not installed use PIP to install<br>
` $ sudo pip install virtualenv `<br>
Rename the temporary environment<br>
` $ sudo virtualenv venv `<br>
Activate virtual environment to install Flask and other project dependencies<br>
` $ source venv/bin/activate `<br>
Install Flask and other project dependencies<br>
` $ sudo pip install Flask oauth2client httplib2 requests bleach `<br>
Deactivate virtual environment<br>
` $ sudo deactivate `<br>
**Configure virtual host**<br>
Open config file<br>
` $ sudo nano /etc/apache2/sites-available/catalog.conf `<br>
Add the following code<br>
```
<VirtualHost *:80>
		ServerName 35.161.52.116.com
		ServerAdmin admin@35.161.52.116.com
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
Save file and exit<br>
Enable virtual host<br>
` $ sudo a2ensite FlaskApp `<br>

**Create .wsgi file**<br>
Move to project folder<br>
` $ cd /var/www/catalog `<br>
Create file<br>
` $ sudo nano catalog.wsgi `<br>
Add the following code and save<br>
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```
Restart Apache<br>
` $ sudo service apache2 restart `<br>

11. Add project 3 files to linux server<br>
(Reference: [StackExchange](http://unix.stackexchange.com/questions/115560/use-scp-to-transfer-a-file-from-local-directory-x-to-remote-directory-y))<br>
Securly copy files from local machine to remote machine<br>
` $ scp -i ~/.ssh/id_rsa -P 2200 -r /path/to/project/folder grader@35.161.52.116:/var/www/catalog/catalog `<br>
Reorganize folder and files as needed<br>

12. Adjust project file code to allow application to run on new server<br>
Rename ` project.py ` file to ` __init__py `<br>
Correct the path for ` client_secrets.json ` and ` fb_client_secrets.json ` in ` __init__.py ` file<br>
Change the ` create_engine ` code to reflect the use of PostgreSQL instead of SQLite in the ` database_setup.py `, ` initialRecipes.py `, ` __init__.py ` files<br>
` engine = create_engine("postgresql://catalog:<db password>@localhost/catalog")`<br>

13. Adjust Google and Facebook OAuth logins working<br>
**Add server alias to apache config file**<br>
(Reference: [Apache](https://httpd.apache.org/docs/2.2/vhosts/name-based.html))<br>
Open config file<br>
` $ sudo nano /etc/apache2/sites-available/catalog.conf `<br>
Add the following code<br>
` ServerAlias http://ec2-35-161-52-116.us-west-2.compute.amazonaws.com/ `<br>
In the Google console add the site IP for the site with http:// appended in the JavaScript Authorization url list and the above server alias url for the redirect url<br>
Add the above urls in the ` client_secrets.json ` file<br>
In the Facebook developer page add the IP address with http:// appended in the valid OAuth redirect URIs section<br>

14. Setup initial database<br>
Run the ` database_setup.py ` file<br>
` $ python database_setup.py `<br>
Run the ` initialRecipies.py ` file<br>
` $ python initailRecipies.py `<br>

Visit [http://35.161.52.116](http://35.161.52.116) to view the working application.
