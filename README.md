# Linux Server Configuration Project


## TL:DR

Host Name: http://ec2-18-217-235-171.us-east-2.compute.amazonaws.com/

IP Address: 18.217.235.171

## Amazon Lightsail Server Set Up

1. [Visit Amazon Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0) and create a new AWS account:


2. After you log in, click 'Create Instance';

3. Select Platform and  blueprint

4. Scroll down to name your instance and click 'Create'

5. The instance needs +or-2 mins to set up. After it is set up, you will see 'running' in the left corner of the status card. Take note of the public IP.

6. Click the status card and navigate to networking

7. Download your private key which is a .pem file.

8. Click the 'Networking' tab and find the 'Add another' at the bottom. Add port 123 and 2200.

## Server Configuration

1. Save the downloaded `.pem` public key file into .ssh folder which is based in the home directory

2. Secure public key while also making it accessible `$ chmod 600 ~/.ssh/YourAWSKey.pem`

N.B. From here take note of the use of first and second instances of terminals. The first terminal will be the server which will be same as a direct operation on the AWS terminal while the second terminal will be used to create the grader's account

3. Open the first terminal and use this key to log into our Amazon Lightsail Server: `$ ssh -i ~/.ssh/YourAWSKey.pem ubuntu@13.58.109.116`

4. Log as root user`$ sudo su -`

5. Then type  `$ sudo adduser grader` to create another user 'grader'

6. Create a new file in the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. And give grader the super permisssion `grader ALL=(ALL:ALL) ALL`. In nano save with (control X, then type `yes`, then hit the return key on your keyboard)

7. In order to prevent the `$ sudo: unable to resolve host` error, edit the hosts by `$ sudo nano /etc/hosts`, and then add `127.0.0.1 ip-10-20-37-65` under `127.0.0.1:localhost`

8. Run the following commands to update all packages and install finger package:
- `$ sudo apt-get update`
- `$ sudo apt-get upgrade`
- `$ sudo apt-get install finger`

9. Open a new(now the second) Terminal window (Command+N) and input `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`

10. Stay on the same(i.e the second) Terminal window, input `$ cat ~/.ssh/udacity_key.rsa.pub` to read the public key. Copy the public key.

11. Return to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by `$ cd /home/grader`

12. Create a .ssh directory (in same first terminal): `$ mkdir .ssh`

13. Create a file to store the public key(still in first terminal...use of first terminal will end in #18): `$ touch .ssh/authorized_keys`

14. Edit the authorized_keys file `$ nano .ssh/authorized_keys`

15. Change the permission: `$ sudo chmod 700 /home/grader/.ssh` and `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`

16. Change the owner from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`

17. Restart the ssh service: `$ sudo service ssh restart`

18. Type `exit` to disconnect from Amazon Lightsail server

19. Log into the server as grader(that is log in via the second terminal window): `$ ssh -i ~/.ssh/udacity_key.rsa grader@13.58.109.116`

20. Enforce the key-based authentication: `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and change text to `no`. After this, restart ssh again: `$ sudo service ssh restart`

21. Change the ssh port from 22 to 2200: `$ sudo nano /etc/ssh/ssdh_config` Find the *Port* line and change `22` to `2200`. Restart ssh: `$ sudo service ssh restart`

22. Disconnect the server by `^c` and then log back through port 2200: `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.58.109.116` now loggin in via port 2200

23. Disable ssh login for *root* user to prevent attackers from making a fondering attept with root: `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit to `no`. Restart ssh `$ sudo service ssh restart`

24. Configure Uncomplicated Firewall:
- `$ sudo ufw allow 2200/tcp`
- `$ sudo ufw allow 80/tcp`
- `$ sudo ufw allow 123/udp`
- `$ sudo ufw enable`


## Deploy Catalog Application

Ensure you are logged in as grader. Should at anypoint a ubuntu password is requested simply ^d and use `sudo` to re-execute that command.

1. Install required packages
- `$ sudo apt-get install apache2`
- `$ sudo apt-get install libapache2-mod-wsgi python-dev`
- `$ sudo apt-get install git`

2. Enable mod_wsgi (mod_wsgi package implements an Apache module which can host any Python web application which supports the Python WSGI specification.)`$ sudo a2enmod wsgi` and start the web server by `$ sudo service apache2 start` or `$ sudo service apache2 restart`
3. Enter your public IP address in your browser now and the apache2 default page should be loaded.

4. Create catalog folder to keep app and make grader owner and group of the folder
- `$ cd /var/www`
- `$ sudo mkdir catalog`
- `$ sudo chown -R grader:grader catalog`
- `$ cd catalog`

5. Clone the project from Github: `$ git clone [your link] catalog` (so folder path to app will become `var/www/catalog/catalog`)

(The Web Server Gateway Interface (WSGI) is a specification for simple and universal interface between web servers and web applications or frameworks for the Python programming language.)

6. Create a .wsgi file in `/var/www/catalog/`: `$sudo nano catalog.wsgi` and add the following into this file
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'your_secret_key'
```

7. In /var/www/catalog/catalog Rename the `app.py` to `__init__.py` as follows `mv app.py __init__.py`

(The venv module provides support for creating lightweight “virtual environments” with each virtual environment having its own Python binary. It allows the app have its own independent set of installed Python packages in its site directories.)

8. Install virtual environment
- `$ sudo apt-get install python-pip`
- `$ sudo pip install virtualenv`
- `$ sudo virtualenv venv`
- `$ source venv/bin/activate`
- `$ sudo chmod -R 777 venv`

You should see a `(venv)` appears before your username in the command line.

9. Install the Flask and other packages needed for this application
- `$ sudo pip install Flask`
- `$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests`
N.B. You may need to re-install outside the venv if some modules are missing such as `ImportError: No module named sqlalchemy`...sometimes there are some minimal compatibility issues between venv and some module installation


10. Use the `nano __init__.py` command to change the `client_secrets.json` line to `/var/www/catalog/catalog/client_secrets.json` as follows `CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`
    Ensure to look through `__ini__.py` for every instance of this change and replace as stated.
    Also replace
    `if __name__ == '__main__':
    app.secret_key = 'gamers_secret_hoops'
    app.debug = True
    app.run(0.0.0.0, port=5000)`

    with

    `if __name__ == '__main__':
    app.secret_key = 'gamers_secret_hoops'
    app.debug = True
    app.run()`

11. Comfigure and enable the virtual host
N.B `sites-available/: This is an apache2 directory that contains all of the virtual host files that define different web sites. These will establish which content gets served for which requests.`

- `$ sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste the following code and save
```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS] e.g 18.217.235.171
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME] eg http://ec2-18-217-235-171.us-east-2.compute.amazonaws.com/
    ServerAdmin admin@[YOUR PUBLIC IP ADDRESS]
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
```
You can find your host name in this link: http://www.hcidata.info/host2ip.cgi

12. Now we need to set up the database
- `$ sudo apt-get install libpq-dev python-dev`
- `$ sudo apt-get install postgresql postgresql-contrib`
- `$ sudo -u postgres -i`

You should see the username changed again in command line, and type `$ psql` to get into postgres command line

13. Create a user to create and set up the database. My database for example is named `catalog` and the user I am creating is also called `catalog`
- `$ CREATE USER catalog WITH PASSWORD [your password];`
- `$ ALTER USER catalog CREATEDB;`
- `$ CREATE DATABASE catalog WITH OWNER catalog;`
- Connect to database `$ \c catalog`
- `$ REVOKE ALL ON SCHEMA public FROM public;`
- `$ GRANT ALL ON SCHEMA public TO catalog;`
- Quit postgres as follows: `$ \c` and then `$ exit`

List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

14. use `sudo nano` command to change all engine to `engine = create_engine('postgresql://catalog:[your password]@localhost/catalog)` e.g engine = create_engine('postgresql://catalog:grader@localhost/catalog')
Base.metadata.bind = engine.
Ensure to do this in the database_setup.py and the populatedDatabase.py(i.e. in 3 places)

15. Initiate the database if you have a script to do so: `python database_setup.py `

16. Restart Apache server `$ sudo service apache2 restart` and enter your public IP address or host name into the browser. Hooray! Your application should be online now!

17. Lastly but very importantly, if default apache2 page persists, it means you have not enabled your site then execute `sudo a2ensite [name of app]`...'Name of app' here would be the folder the app is saved in inside /var/www/catalog/catalog for example mine is `sudo a2ensite catalog`

## Appendix:
N.B.

I. Some apache2 testing tools to check appropriateness of workflow
`apache2ctl -t`
`apache2ctl -S`

II. Project Folder(s) structure
```
/var/www/catalog
    |-- catalog.wsgi
    |__ /catalog
         |-- __init__.py
         |-- gamersnba.py
         |-- client_secrets.json
         |-- fb_client_secret.json
         |-- db_setup.py
         |-- /static
         |-- /templates
         |-- /venv

```

III.  `Fail2Ban`
`Fail2Ban` is an intrusion prevention software framework that protects computer servers from brute-force attacks.
- Install Fail2Ban: `sudo apt-get install fail2ban`.
- Install sendmail for email notice: `sudo apt-get install sendmail iptables-persistent`.
- Create a copy of a file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- Change the settings in `/etc/fail2ban/jail.local` file:
  ```
  set bantime = 600
  destemail = useremail@domain
  action = %(action_mwl)s
  ```
- Under `[sshd]` change `port = ssh` by `port = 2200`.
- Restart the service: `sudo service fail2ban restart`.

IV. Automatic Updates Installation
The `unattended-upgrades` package can be used to automatically install important system updates.
- Enable automatic (security) updates: `sudo apt-get install unattended-upgrades`.
- Edit `/etc/apt/apt.conf.d/50unattended-upgrades`, uncomment the line `${distro_id}:${distro_codename}-updates` and save it.
- Modify `/etc/apt/apt.conf.d/20auto-upgrades` file so that the upgrades are downloaded and installed every day:
  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::AutocleanInterval "7";
  APT::Periodic::Unattended-Upgrade "1";
  ```
- Restart Apache: `sudo service apache2 restart`.

## Reference
Profound gratitude to:
- https://github.com/callforsky/udacity-linux-configuration
- https://github.com/twhetzel/ud299-nd-linux-server-configuration
- https://github.com/boisalai/udacity-linux-server-configuration

Other Resources:
- https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- Official Ubuntu Documentation, [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).
- Ubuntu Wiki, [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates).
- DigitalOcean, [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04).
- [Fail2Ban Official website](http://www.fail2ban.org/wiki/index.php/Main_Page).



