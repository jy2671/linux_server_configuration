# Udacity Project - Linux Server Configuration 


## Project Overview
The project requests to take a baseline installation of a Linux server and prepare it to host web applications. Additionally, the project requires securing the server from a number of attack vectors, installing and configuring a database server, and deploying a web application onto it. It requests to turn a brand-new, bare bones, Linux server into the secure and efficient web application host the applications need.

## Information
	• Public IP: 3.84.49.72
	• Accessible SSH port: 2200
	• Application URL: http://ec2-3-84-49-72.compute-1.amazonaws.com/	
	
## Project Details

### Amazon Lightsail Server Set Up

Use [Amazon Lightsail](https://amazonlightsail.com/) to create a Linux server instance
  * Create an AWS account and log into [Amazon Lightsail](https://amazonlightsail.com/)
  * Create an instance
    * Login and create an instance
    * Instance image: Ubuntu OS only - Ubuntu 16.04 LTS
 	* Start the instance    
  * Add port 123 and 2200 to Firewall on the instance to accept connections (22 and 80 are default)
      
## Server Configuration
1. SSH into the server

    *SSH to the server before setting up firewall (ufw)*
   
    Download the default private key from the Account page to local desktop
   
    * `$ mv LightsailDefaultKey-us-east-1.rsa ~/.ssh/`
    * `$ chmod 700 ~/.ssh`
    * `$ chmod 644 ~/.ssh/LightsailDefaultKey-us-east-1.rsa`
    * `$ ssh -i ~/.ssh/LightsailDefaultKey-us-east-1.rsa ubuntu@3.84.49.72`
		
2. Create a new user `grader`
   * SSH into the server as ubuntu 
   
      `$ ssh -i ~/.ssh/LightsailDefaultKey-us-east-1.rsa ubuntu@3.84.49.72`
      
   * Sudo and become a root user
    
      `$ sudo su`
      
    * Add a new user `grader`
    
      `$ sudo adduser grader`
      
    * Give `grader` sudo permission
    
      * Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`
      * Add `grader ALL=(ALL) NOPASSWD:ALL"` and save
      
    * Edit `/etc/hosts` to prevent the error - sudo: unable to resolve host
     
      * `$ sudo nano /etc/hosts`
      * Add the host: `127.0.1.1 ip-10-20-37-65` under '127.0.0.1 localhost'
      
3. Update all packages and install `finger`:
     * `$ sudo apt-get update`
     * `$ sudo apt-get upgrade`
     * `$ sudo apt-get install finger` (finger is a utility software to check user's status)
    
4. Configure the key-based authentication for the user `grader`
    * Keygen to generate private and public key pair for the user `grader`
    * Open a new terminal on the local machine
    
      * `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`
      * `$ cat ~/.ssh/udacity_key.rsa.pub` (copy the public key)
      
    * SSH to the server as `grader`, sudo to root, and create
    
      * `$ touch /home/grader/.ssh/authorized_keys`
      * Paste the `udacity_key.pub` which was copied from local terminal `~/.ssh/udacity_key.rsa.pub` to the                   `/home/grader/.ssh/authorized_keys` file
      
     * Change permissions:
      
        * `$ sudo chmod 700 /home/grader/.ssh`
        * `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
        * `$ sudo chown -R grader:grader /home/grader/.ssh` (Change the owner from root to grader)
			
      * Log into the server through SSH:
      
        `$ ssh -i ~/.ssh/udacity_key.rsa grader@3.84.49.72`
      
5. Enforce key-based authentication

      * `$ sudo nano /etc/ssh/sshd_config` -> Edit `PasswordAuthentication to no`.
		
      * Restart the ssh service: `$ sudo service ssh restart`
    
6. Change the SSH port from 22 to 2200

      * `$ sudo nano /etc/ssh/ssdh_config`  -> change Port from 22 to 2200
      * `$ sudo service ssh restart` -> restart SSH  
      * SSH into the server: `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@3.84.49.72`
    
7. Disable ssh login for root user

    * `$ sudo nano /etc/ssh/sshd_config` -> edit PermitRootLogin to no. 
    * `$ sudo service ssh restart` -> restart SSH
    
8. Configure the local timezone to `UTC`

    * `$ sudo dpkg-reconfigure tzdata -> choose UTC`
    
9. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

       *Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.*
      
    * `$ sudo ufw allow 2200/tcp`
    * `$ sudo ufw allow 80/tcp`
    * `$ sudo ufw allow 123/udp`
    * `$ sudo ufw enable`
		
## Deploy Item Catalog Project - Virtual Machine, Apache2, and PostgreSQL to host the application

1. SSH into the server as `grader`
2. Install and configure Apache to serve Python mod_wsgi application

    * `$ sudo apt-get install apache2`
    * `$ sudo apt-get install libapache2-mod-wsgi python-dev`

    *mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications*
    
    * Enable mod_wsgi -> `$ sudo a2enmod wsgi` -> enable mod_wsgi
    * `$ sudo service apache2 start`
    
    *Input the public IP address and should be able to see Apache2 Ubuntu Default Page*
    
3. Install Git

    * `$ sudo apt-get install git`
	
4. Clone the Item Catalog app from Github

    * `$ cd /var/www`
    * `$ sudo mkdir catalog`
    *  `sudo chown -R grader:grader catalog` -> change owner for the catalog folder
    * `$ cd /catalog`
    * `$ git clone https://github.com/jy2671/item-catalog.git catalog`-> clone item catalog repository from Github
    * `$sudo nano catalog.wsgi` to create a `catalog.wsgi` file which serves the application over mod_wsgi
    * Add the following into the `catalog.wsgi` file:
    
    
      ```python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0, "/var/www/catalog/")

      from catalog import app as application
      application.secret_key = 'super_secret_key'
      ```
		
5. Install the virtual machine (/var/www/catalog/catalog)

    * $ sudo pip install virtualenv
    * $ sudo virtualenv venv
    * $ source venv/bin/activate
    * $ sudo chmod -R 777 venv
    
6. Install Flask and dependencies

     * `$ sudo apt-get install python-pip`
     * `$ sudo pip install Flask`
     * `$ sudo pip install httplib2`
     * `$ sudo pip install sqlalchemy`
     * `$ sudo pip install sqlaclemy_utils`
     * `$ sudo pip install requests`
     * `$ sudo pip install render_template`
     * `$ sudo pip install redirect`
     * `$ sudo pip install psycopg2-binary`
     * `$ sudo -H pip install oauth2client`

7. Rename the application.py to `__init__.py` in /var/www/catalog/catalog

      * modify `__init__.py` and add a path for client_secrets.json
      
      ``` python
      CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id'] APPLICATION_NAME = "Item Catalog Application"


      # Upgrade the authorization code into a credentials object
      oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
      oauth_flow.redirect_uri = 'postmessage' credentials = oauth_flow.step2_exchange(code)
    
      ```
      * change the host and port
      
      ```python
      if __name__ == '__main__':
          app.secret_key = 'super_secret_key'
          app.debug = True
          app.run(host='3.84.49.72', port=80)

      ```

8. Configure and enable the virtual host

      * `$ sudo nano /etc/apache2/sites-available/catalog.conf` -> create a virtual host config file
      * add the code below into the catalog.conf file
      
          ```
            <VirtualHost *:80>
            ServerName 3.84.49.72
            ServerAlias ec2-3-84-49-72.compute-1.amazonaws.com
            ServerAdmin admin@3.84.49.72
            WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python2.7/site-pa$
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
                
      * Enable the new virtual host: `$ sudo a2ensite catalog`
      
   *Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)*

9. Install and configure PostgreSQL

      * `$ sudo apt-get install libpq-dev python-dev`
      * `$ sudo apt-get install postgresql postgresql-contrib`
      * `$ sudo su - postgres -i`
      
      * `# CREATE USER catalog WITH PASSWORD 'password';` -> create a new user called 'catalog' with password
      * `# ALTER USER catalog CREATEDB;` -> grant the catalog user CREATDB
      * `# CREATE DATABASE catalog WITH OWNER catalog;` -> create the catalog database owned by the catalog user
      * `# \c catalog` -> connect to the database
      * `# REVOKE ALL ON SCHEMA public FROM public;' -> revoke all rights
      * `# GRANT ALL ON SCHEMA public TO catalog;` -> only let the catalog role create tables
      * `# \q` -> log out form PostgreSQL
      * `$ exit` -> return to the grader user
      * Modify the python files and update the database connection to 
      
        `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
      * `$ python /var/www/catalog/catalog/setup_database.py` -> setup the database
      
10. Restart Apache to launch the app -> `$ sudo service apache2 restart`
      
## Troubleshooting
 
  * Error: pg_config executable not found while installing `$ sudo pip install psycopg2`
  
    Solution: `$ sudo pip install psycopg2-binary`
  
  * wsgi:crit: The mod_python module cannot be used on conjunction with mod_wsgi 4.0+. Remove the mod_python module from the Appache configuration
  
    Solution: `$ sudo apt-get remove libapache2-mod-python` -> removed libapache2-mod-python; fixed the issue
    
  * Error: Target WSGI script '/var/www/flaskApp/items-catalog/items-catalog.wsgi' cannot be loaded as Python module in /var/log/apache2/error.log
  
    Solution: `$ sudo -H pip install oauth2client` -> add '-H' and reinstall oauth2client; fixed the issue
    
 *Special thanks to [iliketomatoes](https://github.com/iliketomatoes) for a very helpful README*
