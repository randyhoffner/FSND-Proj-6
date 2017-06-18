# FSND-Proj-6
## Linux Server Configuration
In this project, an Amazon Lightsail instance  is provisioned to run Apache  2 on Linux.  The server is secured; login via private key is facilitated; PostgreSQL and other required software is installed and configured; and the server is otherwise set up as detailed below.  Catalog app Randy's Liquor App, created in a previous FSND project, is cloned into the server, and modified and refactored to run on the Apache server.
## Server Access
### IP address: 34.201.75.4
### SSH port: 2200
### URL: ec2-34-201-75-4.compute-1 amazonaws.com
### grader login: ssh -i ~/.ssh/id_rsa grader@34/201.75.4 -p 2200
### grader password: 111
## Walkthrough
### 1. Create Lightsail Ubuntu 16.04 instance "randyhoffner1".
### 2. Attach static IP address 34.201.75.4 to instance.
### 3. Allow port 2200 and port 123 on Lightsail Networking tab.
### 4. SSH into instance using Lightsail browser interface.
### 5.Update Ubuntu to latest release and install automatic security updates:
```
sudo apt-get update
sudo apt-get upgrade
```
### Ubuntu is now updated.  To install and enable the unattended upgrades package:
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```
### Respond appropriately to queries.
### 7. Configure ufw (uncomplicated firewall).  First, deny all incoming.  Then, allow all outgoing. Then, allow incoming from these ports:
   * 22 (default ssh port)
   * 2200 (new ssh port)
   * 80 (web traffic)
   * 123 (NTP)

```
sudo default deny incoming
sudo default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
```
### 8. Run:
```
sudo ufw enable
sudo ufw status
```
### to enable ufw and determine which ports are open.  Verify that the above ports are open.  Make absolutely sure that port 2200 is open.
### 9. Change SSH port from 22 to 2200 by editing /etc/ssh/sshd_config.  Under the comment line "#What ports, IPs and protocols we listen for", change "Port 22" to "Port 2200".  Restart SSH service:
```
sudo service ssh restart
```
### When instance is restarted, we cannot SSH in on port 22, and we can therefore no longer use the Lightsail browser interface to SSH.  We must now SSH in on port 2200, using Mac Terminal or Windows PuTTy.
### 10. The Lightsail instance is configured to use the "newkey" key pair, which was created in Lightsail.  Add downloaded private key newkey.pem to the OSX .ssh directory:
  * Open the Home directory
  * With focus on the Home directory, press CMD-SHIFT-G
  * Enter "~/.ssh/" in the "Go to the folder:" dialog box and click GO.
  * Copy and paste "newkey.pem" to the /.ssh/ folder.
  * Assign owner read and write permissions to "newkey.pem" by running the terminal command:

```
chmod 600 ~/.ssh/newkey.pem
```
### 11. SSH in from Mac Teminal using
```
ssh -i ~/.ssh/newkey.pem ubuntu@34.201.75.4 -p 2200
```

### We will be at this prompt: "ubuntu@ip-172-26-15-127".  user is "ubuntu" and 172.26.15.127 is the private IP assigned by Lightsail.  We can also SSH in on Windows using puTTy, by using PuTTygen to convert newkey.pem to a PuTTy-compatible format, and then entering the IP address, checking connection type SSH, entering Auto-login username, and entering location of the private key file.  These SSH parameters are stored as a Saved Session in PuTTy, so that they are available for later use.
### 12. Create a user named "grader".  Give grader permission to sudo:
```
sudo useradd -m -s /bin/bash grader
sudo usermod -aG sudo grader
```
### 13. Create an SSH key pair for grader using ssh-keygen tool in on local machine in Terminal:
```
ssh-keygen -t rsa
```
### Answer dialog queries; put key pair in "~/.ssh/" directory.  On Ubuntu instance, set up grader SSH key.  As root user:
```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 70 /home/grader/ssh
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

### 14. Copy grader public key to /home/grader/.ssh/authorized_keys.  Name of grader private key shown above in Server Access section.  Private key to be copied to "Comments" section of project submission form.
### 15. Test grader SSH in:
```
ssh -i ~/.ssh/id_rsa grader@34.201.75.4 -p 2200
```
### Produces grader login: "grader@ip-172-26-15-127".
### 16. Disable root login by editing /etc/ssh/sshd_config .  Change line that reads "PermitRootLogin yes" to "PermitRootLogin no", and uncomment the line "# PasswordAuthentication no".
### 17. Set instance to UTC
```
sudo timedatectl set-timezone UTC
```

### Install NTP:
```
sudo apt-get install ntp
```
### Run CTL-D to close Ubuntu session, then SSH back in.

### 18. Install Apache:
```
sudo apt-get install apache2
```

### Verify installation by navigating to http://34.201.75.4 and seeing, "Apache2 Ubuntu Default Page"
### 20. Install mod-wsgi:
```
sudo apt-get install libapache2-mod-wsgi
```

### 21. To configure Apache2 to serve the app, edit "/etc/apache2/sites-enabled/000-default.conf" by adding this line:  

WSGIScriptAlias/var/www/html/myapp.wsgi

### just above

```
</VirtualHost>
```

Save and

```
sudo apache2ctl restart
```
### 22.Install PostgreSQL:

```ubuntu
sudo apt-get install postgresql postgresql-contrib
```
### By default PostgreSQL does not allow remote connections.  To make sure of this, edit:
```
/etc/postgresql/9.5/main/pg_hba.conf
```
### so that the only allowed IPv4 local connection is from
```
127.0.0.1
```
### and the only allowed IPv6 local connection is from
```
::1
```
### 23. Create a new database user named "catalog1" with these permissions:  superuser n, allow to create databases y, allow to create more new roles y.

```
sudo -u postgres createuser -interactive catalog1
```
### Answer queries appropriately.

### 24. Newest version of git is already installed.  Set global git configuration:

```
sudo git config -global user.name "randyhoffner"
sudo git config -global user.email "rnh@ashlandoregon.org"
```

### 25. Install the following:
```
sudo apt-get install python-psycopg2w python-flask
sudo apt-get install python-sqalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2 (requirement already satisfied)
sudo pip install flask-seasurf
```

### 26. Clone catalog app repo to instance:
```
cd /srv/
sudo mkdir FSND-Proj-5
sudo chown www-data:222-data FSND-Proj-5
sudo -u www-data git clone https://github.com/randyhoffner/FSND-Proj-5
```

### 27. Add catalog directory and .htaccess file to /var/www/:
```
cd /var/www/
sudo mkdir catalog
cd /catalog
sudo nano .htaccess
```
### To make github repo web-inaccessible, add this line to .htaccess:

```
 RedirectMatch 404 /\.git
```
### and save.



### 28. Add catalog directory to /var/www/catalog
```
cd /var/www/catalog
sudo mkdir catalog
```
### 29. Copy contents of /srv/FSND-Proj-5 to /var/www/catalog/catalog

```
cp -a /srv/FSND-Proj-5 /var/www/catalog/catalog
```
### 30. Create empty database "catalog" in /var/www/catalog/catalog, and assign appropriate access:
```
cd /var/www/catalog/catalog/
sudo -u postgres createdb -O catalog catalog
/etc/init.d/postgres restart
sudo service apache2 restart
```
### Verify that empty "catalog" database exists.  Open PostgreSQL:
```
sudo -u postgres psql
```
### At postgres prompt run:
```
\list
```
### Confirm that "catalog" database is listed.  Close PostgreSQL and restart Apache:
```
\q
sudo service apache2 restart
```
### Open PostgreSQL and determine what tables are listed in "catalog" database:
```
sudo -u postgres psql
```
### At the postgres prompt:
```
\c catalog \dt
```
### Verify that all tables in catalog dabase are listed.  Then close postgres:
```
\q
```
### 31. Create configuration file for catalog app:
```
cd /etc/apache2/sites-available
sudo nano catalog.conf
```
### 32. Put these contents into catalog.conf:
```
<VirtualHost *:80>
    ServerName mywebsite.com
    ServerAdmin rnh@ashlandoregon.org
# ServerAlias www.34.201.75.4
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
    LogLevel info
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
### 33. Create WSGI file:
```
cd /var/www/catalog
sudo nano catalog.wsgi
```
### 34. Put these contents in WSGI file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "var/www/catalog")

from catalog import app as application
application.secret_key='super_secret-key
```
### 35. Update Google sign-in information by going to Google APIs and to Randy's Liquor App, and make the following changes:

* Under Authorized Javascript Origins, add http://34.201.75.4
* Under Authorized redirect URIs, add http://ec2-34-201-75-4.compute-1.amazonaws.com
* Save, then click on DOWNLOAD JSON to download updated client data json.
* Open newly downloaded json file in Sublime Text and copy contents, then open client_secrets.json file:  
```
sudo nano /var/www/catalog/catalog/client_secrets.json
```

### Delete contents, then paste in new contents and save.

### 36. Disable the default Apache site, enable the catalog-app site, and reload Apache:
```
sudo a2 dissite 000-default.conf
sudo a2ensite catalog-app.conf
sudo service apache reload
```
### 37.Randy's Liquor App should now be available at http://34.201.75.4 and http://ec2-34-201-75-4.compute-1.amazonaws.com.  Instead, there will likely be server errors, and these are corrected by looking at the Apache error log and making the required corrections.  Eventually, the app will appear.

### 38. Randy's Liquor App will now be displayed at http://34.201.75.4.  Click on "Click Here to Login" to get to the Login page, where you may login with Google.

### Once you are logged in, you will be on the homepage of Randy's Liquor App, on which you will find a list of liquor categories, along with a description of each category.  You may add a category and description by clicking on the "Add Category" button, which will take you to the newCategory.html page where you may type in your category and description.  Any category that you create may be edited or deleted by you by clicking on the appropriate link, taking you to the editcategory.html or deletecategory.html page.

### Clicking on the category name, eg Bourbon, takes you to the Bourbon menu page, which contains a list of bourbons and their descriptions.  You may add, edit, or delete an item if you created the category.  To get back to the home page, click on the "Show All Categories" button.  

## Sources
### In addition to the Udacity classroom material, cited sources, forums, and 1:1 sessions, the following sources were consulted.
* 	  www.digitalocean.com
*     http://askubuntu.com
* 	  www.chiark.greenend.org.uk (PuTTy)
* 	  Ubuntu Documentation
* 	  For PostgreSQL:  www.digitalocean.com; killtheyak.com; PostgreSQL Documentation
* 	  Apache2 Documentation
* 	  Terminal help for /.ssh from http://apple.stackexchange.com and www.thinkplex.com
* 	  stackoverflow.com
