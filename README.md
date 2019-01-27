# Linux Server Configuration (Udacity Full-Stack Nanodegree)
Detailed instructions on configuring and securing an Ubuntu Linux Server and then deploying a Flask web application: specifically [*Calla Grocers*](https://github.com/antzie/catalog) an item catalogue application developed for Udacity's Full-Stack Nanodegree.
___
Public IP Address: http://3.104.111.195

URL: http://ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com

SSH Port: 2200
___
## Software Summary
- Python - 2.7.12
- Python-flask -  0.10.1-2build2
- Postgresql - 9.5
- Apache2 -  2.4.18
- Libapache2-mod-wsgi - 4.3.0-1.1build1 
- Ubuntu - 16.04.1
- AWS Lightsail Server 
---
## Summary
### Configure Server
 1) Set Up Amazon Light Sail Server
 2) Log-in & Create User: Grader
 3) Update Default Lightsail Server
 4) SSH-Keys/File Authorisation
 5) Disable Root Log-in/Change SSH Port
 6) Check UTC TimeZone
 7) Configure UFW Firewall
 8) Login as Grader
### Deploy Project
 1) Install Apache/Mod-WSGI/PostgreSQL
 2) Setup PostgreSQL/Disable Remote Connections
 3) Install Python Libraries/Git etc.
 4) Clone Git Repository 
 5) Setup Git Files (Postgresql Connection et al.)
 6) Configure WSGI
 7) Update Google Authentication
 8) Configure Apache2 Virtualhost
 9) Run Application
### Miscellaneous
1) Potential Errors
2) Documentation List
---
# Configuring Server
## 1) Set Up Amazon Lightsail
Login to Amazon Lightsail: https://lightsail.aws.amazon.com

Follow instructions to set up an AWS account if do not already have one.

### Create an Instance:
At AWS Lightsail homepage: choose 'Create an Instance'

Choose the following options:
 - Platform : Linux
 - Blueprint: OS Only -Ubuntu 16.04 LTS
 - Instance Plan: e.g. $3.50/month 
 -Identify your instance: e.g. Ubuntu-512MB-Sydney-3
Create Instance!

### Acquire AWS Public-Private Key 
From Lightsail Instance main page, access link to default private key from Account page.

Download SSH key pair to  local machine.

Move ```.pem public key``` file into the ```.ssh``` folder at the root of local user

*Location on Windows*

``` C:\Users\<YOUR USER NAME ON YOUR MACHINE>\.ssh ```

*Example* ```C:\Users\antzie\.ssh```

Secure the key: 
``` chmod 600 ~/.ssh/<YOUR KEY NAME> ```

### Configure LightSail FireWall

From instance main page - click on networking.

Add two (2) ports:

 CUSTOM --- UDP --- 123 
 
 CUSTOM --- TCP --- 2200 
 
*Remember to save port changes.*

## 2). Log-In & Create User: Grader
### Login to AWS Lightsail Instance from local machine
Access command line. (Windows: GIT BASH)
Log-in!
``` ssh -i ~/.ssh/<YOUR AWS KEY> ubuntu@<YOUR PUBLIC IP ADDRESS> ```
*Example*
``` ssh -i ~/.ssh/LightsailDefaultKey-ap-southeast-2.pem ubuntu@3.104.111.195 ```
You should be logged in as user, ubuntu
### Create New User
Switch to root user
```$ sudo su - ```
Create User
```$ sudo adduser grader  ``` 
*Set Password for Grader*
Grant Sudo Access
``` $ sudo nano /etc/sudoers.d/grader```
Insert into sudoers.d/grader
```grader ALL=(ALL:ALL) ALL ```

**Potential Error** '*Fix Resolve Host Error*' 
``` $ sudo nano /etc/hosts```
Add following below 127.0.1.1:localhost
```127.0.1.1 ip-10-20-37-65```
## 3) Update Default Lightsail Server
As root user
``` 
$ apt-get update 
$ apt-get upgrade
$ apt-get install finger
$ reboot
```
**Potential Error**:  'the following packages have been kept back'
``` $ sudo apt-get install <list of packages kept back>```

## 4) SSH-Keys for Grader/File Authourisation
### Generate Keys
On **local** machine run
``` $ ssh-keygen ```
```
Enter file in which to save the key (/c/Users/<YOUR LOCAL USERNAME>/.ssh/id_rsa): eg: /c/users/antzie/.ssh/graderKey
```
### Setup Keys on Server
Read public key (still on local machine)
``` $ cat ~/.ssh/<NAME OF YOUR KEY>.pub ```
*Example* ``` $ cat ~/.ssh/graderKey.pub ```
Copy public key -- *include* ssh-rsa and *exclude* user@user'sComputerName

On **Lightsail Server**:
As root user go to grader.
``` $ cd /home/grader ```
 Create both the  ```.ssh``` directionary and ```.ssh/authorized_keys```;
 ``` $ mkdir .ssh ```
``` $ nano .ssh/authorized_keys```
 Paste public key into ```authorized_keys```
 
### File authorisations
```
$ chown grader:grader /home/grader/.ssh
$ chmod 700 /home/grader/.ssh
$ chown grader:grader /home/grader/.ssh/authorized_keys
$ chmod 644 /home/grader/.ssh/authorized_keys
```
**Potential Error**: If for any reason you wish to delete a user, the following command will delete both the user and its associated directory.
``` $ userdel -r grader ```

**Login as Grader with Key**
``` ssh -i ~/.ssh/<PRIVATE KEY NAME> grader@3.104.111.195```
*Example* ``` ssh -i ~/.ssh/graderKey grader@3.104.111.195```
## 5) Disable Root Log-In/Change SSH Port
***WARNING***. Before changing SSH port, ***ensure*** that the LightSail instance has been configured to port 2200. See above - 1) Configure Lightsail Firewall.
Open file for editing
``` $  nano /etc/ssh/sshd_config ```
Disable Root Log-In
```PermitRootLogin permit-password to: PermitRootLogin no```
Ensure 
``` PasswordAuthentication no ```
Change SSH Port 
``` Port 22 change to  Port 2200 ```
Restart ssh for changes to take effect
``` $ service ssh restart ``` 
## Log into server as Grader and specify port 2200, e.g:
``` $ ssh -i ~/.ssh/graderKey grader@3.104.111.195 -p 2200  ```
## 6) Check UTC Timezone
``` $ date +%Z ``` 
## 7) Configure UFW FireWall
***WARNING***. Before configuring UFW, ***ensure*** that the LightSail instance has been configured to allow the correct connections. See above - 1) Configure Lightsail Firewall.
Block all incoming connections
``` $ sudo ufw default deny incoming ```
Allow all outgoing connections
``` $ sudo ufw default allow outgoing ```
Specific Configurations
``` $ sudo ufw allow 2200/tcp ```
``` $ sudo ufw allow www ```
``` $ sudo ufw allow ntp ```
Check to see if rules have been added **especially** *2200/tcp*
``` $ sudo ufw show added ```
Enable Firewall 
``` sudo ufw enable ```
Check Status
``` sudo ufw status ```
Should look like below:
|  To    | Action | From  |
| ------------- |-----------| -----|
| 2200/tcp   | ALLOW  | Anywhere |
| 80/tcp  | ALLOW  |   Anywhere|
| 123    | ALLOW  | Anywhere |
| 2200/tcp (v6)  | ALLOW  |   Anywhere (v6) |
| 80/tcp (v6)   | ALLOW  | Anywhere (v6) |
| 123 (v6)  | ALLOW  |   Anywhere (v6) |                           

# Deploy Project
## 1) Install Apache and Mod-WSGI and 
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install postgresql postgresql-contrib
```
## 2) Setup Postgresql/Disable Remote Connections
Create user **catalog**  with database creation rights and a prompt for entering a password (-P)
``` $ sudo -u postgres createuser -P catalog ```
Create Database **catalog** with 'catalog' as owner. 
```  $ sudo -u postgres createdb -O catalog catalog ```
Disable Remote Connections to **postgresql**
``` $ nano /etc/postgresql/9.3/main/pg_hba.conf ```
Ensure that the only connections are:
|   Connections |  | 
| ------------- |-----------|
| IPv4   | 127.0.0.1  | 
| IPv6  | ::1   |   
*Docs*: 
 [Ubuntu - PostgreSQL](https://help.ubuntu.com/community/PostgreSQL),  
 [Ramiro - Blog PostgreSQL](http://ramiro.org/blog/postgresql-cheat-sheet-ubuntu/)
 [Digital Ocean - Securing PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
## 3) Install Python Libraries, Git, etc.
```
$ sudo apt-get install python-psycopg2 python-flask
$ sudo apt-get install python-sqlalchemy python-pip
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
$ sudo apt-get install git
```
Upgrade Python Packages
Upgrade pip
``` $ sudo pip install --upgrade pip ```
List outdated packages
``` $ pip list --outdated ```
Install pipdate
``` $ sudo pip install pipdate ```
Upgrade all Packages
``` $ sudo -H pipdate ```
**Potenial Error** : AttributeError: 'module' object has no attribute 'SSL_ST_INIT'
``` $ sudo python -m easy_install --upgrade pyOpenSSL ```
*Docs*:
https://fosstack.com/tips/upgrade-all-pip-packages/index.html
https://stackoverflow.com/questions/43267157/python-attributeerror-module-object-has-no-attribute-ssl-st-init
## 4) Clone Git Repository
Go to /srv directory 
``` cd /srv ```
Change owner to www-data and git clone [Catalog Repository](https://github.com/antzie/catalog)
``` $ sudo mkdir flaskApp ```
``` $ sudo chown www-data:www-data flaskApp/ ```
``` $ sudo -u www-data git clone https://github.com/antzie/catalog.git ```
Directory should look like this:
/srv/flaskApp/catalog
-- > contents of git directory
## 5) Setup Git Files
### Rename app.py
 If not already done so, rename main python application file.
e.g. rename app.py to '_ _init__.py' 
(*required for mod_wsgi*)
``` $ cd /srv/flaskApp/catalog ```
``` $ sudo mv app.py __init__.py ```
### Add Path to Client Secrets 
Find and copy the absolute path of client_secrets file.
e.g. client_secret.json
``` $ cd /srv/flaskApp/catalog ```
``` $ readlink -f client_secret.json```
Paste into __ init__.py
``` $ sudo nano __init__.py```
Replace all references to client_secret.json
*Example*
CLIENT_ID = json.loads(open('/srv/flaskApp/catalog/client_secret.json', 'r').read())['web']['client_id']
### Postgresql Database Connection
``` $ cd srv/flaskApp/catalog ```
Change engine in *all* the following files:
``` $ sudo nano __init__.py```
``` $ sudo nano db_setup.py```
``` $ sudo nano db_populate.py ```
Change
``` engine = create_engine('sqlite:///grocerwithusers.db'```
To
``` engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')```
Where PASSWORD is the password for catalog created when postgresql was set up.
### Change host and port in __ init__.py
Change Host to Lightsail Public IP and Port to 80.
*Example*:
if __name__ == '__main__':
    app.secret_key = 'SECRET HAHA'
    app.debug = True
    app.run(host='3.104.111.195', port=80)
    
## 6) Configure WSGI file
Return to flaskApp directory
``` $ cd /srv/flaskApp/ ```
Create wsgi file:
``` $ sudo nano catalog.wsgi ```
Paste the following:
>#!/usr/bin/env python2
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/srv/flaskApp')
from catalog import app as application

>application.config['OAUTH_SECRETS_LOCATION'] = '/srv/flaskApp/catalog/'
application.config['ALLOWED_EXTENSIONS'] = set(['jpg', 'jpeg', 'png', 'gif'])
application.secret_key = 'super_secret_key'
>
*Docs*: http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

## 7) Update Google Authentication
Find AWS hostname/domain
http://www.hcidata.info/host2ip.cgi
Example: `ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com`
### Update Google Developers Console
https://console.developers.google.com.
Add AWS domain to authorized domains in *Oauth Consent Screen* 
Add Public IP and Domain to *Authorized JavaScript origins* in *Credentials* -> *Web Client*
e.g ```  	http://3.104.111.195  ```
```  	http://ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com ```
Add Domain to *Authorized Redirect URIs* in *Credentials* -> *Web Client*
e.g. ```http://ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com/callback ```
Download *client_secrets* file.
Copy contents of newly adjusted client_secrets file
### Update Client_Secret file
Paste contents of file into *client_secret.json* on server
``` $ sudo nano /srv/flaskApp/catalog/client_secret.json ```
## 8) Configure Apache2 
Create a configuration file.
``` $ sudo nano /etc/apache2/sites-available/catalogApp.conf```
Paste the following 
```
<VirtualHost *:80>
        # Set Name, Alias and Admin. 
        ServerName 3.104.111.195
        ServerAdmin admin@3.104.111.195
        ServerAlias ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com

        # Define WSGI parameters - the daemon process runs as the www-data user.
        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /srv/flaskApp/catalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /srv/flaskApp/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /srv/flaskApp/static

        # Allow Apache to serve the files from the static directory
        <Directory  /srv/flaskApp/static/>
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Disable default virtualHost
``` $ sudo a2dissite 000-default.conf ```
Enable catalogApp virtualHost
``` $ sudo a2ensite catalogApp.conf ```
Reload Apache
``` $ sudo service apache2 reload ```
*Docs* Setting Up VirtualHost
https://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html
http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config
## 9) Run Application
Open http://3.104.111.195 or ec2-3-104-111-195.ap-southeast-2.compute.amazonaws.com
Pray.
---
# Miscellaneous
## 1) Potential Errors List
Inexhaustive list of *some* of the errors that arose in production and their solutions:
Command for isolating server errors: 
``` $ sudo tail -20 /var/log/apache2/error.log ```

**ERROR** '*Fix Resolve Host Error*' 
``` $ sudo nano /etc/hosts```
Add following below 127.0.1.1:localhost
```127.0.1.1 ip-10-20-37-65```

**ERROR**:  *'the following packages have been kept back'*
``` $ sudo apt-get install <list of packages kept back>```

**ERROR** : *AttributeError: 'module' object has no attribute 'SSL_ST_INIT'*
``` $ sudo python -m easy_install --upgrade pyOpenSSL ```
## 2) Documentation List

- Postgresql
- https://help.ubuntu.com/community/PostgreSQL,
- http://ramiro.org/blog/postgresql-cheat-sheet-ubuntu/
- https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
 Package Python Update/Install
- https://www.cyberciti.biz/faq/debian-ubuntu-linux-apt-get-aptitude-show-package-version-command/
- https://fosstack.com/tips/upgrade-all-pip-packages/index.html
- https://stackoverflow.com/questions/43267157/python-attributeerror-module-object-has-no-attribute-ssl-st-init
 Mod_WSGI
- http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
Setting Up VirtualHost
- https://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html
- http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
- https://www.the-art-of-web.com/system/apache-authorization/
- Github Alumni
- https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config
- https://github.com/chuanqin3/udacity-linux-configuration

