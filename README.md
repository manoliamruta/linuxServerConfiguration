# LINUX SERVER CONFIGURATION

## Project Description
This project takes a baseline installation of a Linux distribution on a virtual machine and prepares it to host the web application called [Catalog App](https://github.com/manoliamruta/CatalogProject) on Amazon Lightsail instance, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

### 1. Configured Server Details

The IP address is 13.127.41.80.

The SSH port used is 2200.

The URL to the hosted webpage is: http://ec2-13-127-41-80.ap-south-1.compute.amazonaws.com/.

### 2. Softwares Installed

-   Apache2
-   mod_wsgi
-   PostgreSQL
-   git
-   pip
-   virtualenv
-   httplib2
-   Python Requests
-   oauth2client
-   SQLAlchemy
-   Flask
-   libpq-dev
-   psycopg2

### 3. Configuring the web server

#### Step 1. Create an instance with Amazon Lightsail.

1)    Sign in to Amazon Lightsail using an Amazon Web Services account

2)    Follow the 'Create an instance' link

3)    Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

4)    Choose a payment plan

5)    Give the instance a unique name and click 'Create'

6)    Wait for the instance to start up


#### Step 2. Connect to the Amazon Lightsail instance on local machine.
Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal.

1)    Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

2)    Click on 'Download default key'

3)    A file called LightsailDefaultPrivateKey.pem or LightsailDefaultPrivateKey-YOUR-AWS-REGION.pem will be downloaded; open this in a text editor

4)    Copy the text and put it in a file called lightrail_key.rsa in the local ~/.ssh/ directory

5)    Run `chmod 600 ~/.ssh/lightrail_key.rsa`

6)    Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance (Note: `ubuntu` is the default user for Amazon Lightsail instance.)

#### Step 3. Update all currently installed packages.

1)    Check package updates are available by running `sudo apt-get update`.

2)    Download available package updates by running `sudo apt-get upgrade`.

#### Step 4. Configure the Uncomplicated Firewall (UFW)
1.  Start by changing the SSH port from `22` to `2200` (open up the /etc/ssh/sshd_config file, change the port number on line 5 to `2200`, then restart SSH by running `sudo service ssh restart`; restarting SSH is a very important step!)
    
2.  Check to see if the ufw (the preinstalled ubuntu firewall) is active by running `sudo ufw status`
    
3.  Run `sudo ufw default deny incoming` to set the ufw firewall to block everything coming in
    
4.  Run `sudo ufw default allow outgoing` to set the ufw firewall to allow everything outgoing
    
5.  Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH
    
6.  Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port `2200` so that SSH will work
    
7.  Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server
    
8.  Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP
    
9.  Run `sudo ufw deny 22` to deny port `22` (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port `2200`)
    
10.  Run `sudo ufw enable` to enable the ufw firewall
    
11.  Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:
    
    ```
    To                         Action      From
    --                         ------      ----
    22                         DENY        Anywhere
    2200/tcp                   ALLOW       Anywhere
    80/tcp                     ALLOW       Anywhere
    123/udp                    ALLOW       Anywhere
    22 (v6)                    DENY        Anywhere (v6)
    2200/tcp (v6)              ALLOW       Anywhere (v6)
    80/tcp (v6)                ALLOW       Anywhere (v6)
    123/udp (v6)               ALLOW       Anywhere (v6)
    
    ```
    
12.  Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports `80`(TCP),Custom `123`(UDP), and Custom `2200`(TCP) should be allowed; make sure to deny the default port `22`)
    
13.  Now, to login, open up the Terminal and run:
    
    `ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance
    

#### Step 5. Create a new user named `grader`
1.  Run `sudo adduser grader`.
    
2.  Enter in a new UNIX password (twice) when prompted.
    
3.  Fill out information for the new `grader` user.
    
4.  To switch to the `grader` user, run `su - grader`, and enter the password.

#### Step 6. Give `grader` user sudo permissions
1.  Run `sudo visudo`
    
2.  Search for a line that looks like this:
    
    `root ALL=(ALL:ALL) ALL`
    
3.  Add the following line below this one:
    
    `grader ALL=(ALL:ALL) ALL`
    
4.  Save and close the visudo file
    
5.  To verify that `grader` has sudo permissions, `su` as `grader` (run `su - grader`), enter the password, and run `sudo -l`; after entering in the password (again), a line like the following should appear, meaning `grader` has sudo permissions:
    
    ```
    Matching Defaults entries for grader on
        ip-XX-XX-XX-XX.ec2.internal:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User grader may run the following commands on
        ip-XX-XX-XX-XX.ec2.internal:
        (ALL : ALL) ALL
    
    ```
#### Step 7. Create an SSH key pair for `grader` using the `ssh-keygen` tool.
1.  Run `ssh-keygen` on the local machine
    
2.  Choose a file name for the key pair (such as grader_key)
    
3.  Enter in a passphrase twice (two files will be generated; the second one will end in .pub)
    
4.  Log in to the virtual machine
    
5.  Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)
    
6.  Run `touch .ssh/authorized_keys`
    
7.  On the local machine, run `cat ~/.ssh/insert-name-of-file.pub`
    
8.  Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine
    
9.  Run `chmod 700 .ssh` on the virtual machine
    
10.  Run `chmod 644 .ssh/authorized_keys` on the virtual machine
    
11.  Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)
    
12.  Log in as the grader using the following command:
    
    `ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`
    


#### Step 8. Configure the local timezone to UTC.
1.  Run `sudo dpkg-reconfigure tzdata`, and follow the instructions (UTC is under the 'None of the above' category)
    
2.  Test to make sure the timezone is configured correctly by running`date`

#### Step 9. Install and configure Apache to serve a Python mod_wsgi application.
1.  Run `sudo apt-get install apache2` to install Apache
    
2.  Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load
3.  Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) use the following command:
    
    `sudo apt-get install libapache2-mod-wsgi`
    
4.  Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`

#### Step 10. Install and configure PostgreSQL:
1.  Install PostgreSQL by running `sudo apt-get install postgresql`
    
2.  Open the /etc/postgresql/9.5/main/pg_hba.conf file
    
3.  Make sure it looks like this (comments have been removed here for easier reading):
    
    ```
    local   all             postgres                                peer
    local   all             all                                     peer
    host    all             all             127.0.0.1/32            md5
    host    all             all             ::1/128                 md5
    
    ```
#### Create a new PostgreSQL user named `catalog` with limited permissions

1.  Switch to the `postgres` user by running `sudo su - postgres`.
    
2.  Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`
    
3.  Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`
    
4.  Next, give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`
    
5.  Finally, give the `catalog` user a password by running `\password catalog`
    
6.  Check to make sure the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:
    
    ```
                       List of roles
     Role name |                         Attributes                         | Member of 
    -----------+------------------------------------------------------------+-----------
     catalog   | Create DB                                                  | {}
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
    
    ```
    
7.  Exit psql by running `\q`
    
8.  Switch back to the `ubuntu` user by running `exit`
    

#### Create a Linux user called `catalog`

1.  Create a new Linux user called `catalog`:
    
    -   run `sudo adduser catalog`
    -   enter in a new UNIX password (twice) when prompted
    -   fill out information for `catalog`
2.  Give the `catalog` user sudo permissions:
    
    -   run `sudo visudo`
        
    -   search for a line that looks like this: `root ALL=(ALL:ALL) ALL`
        
    -   add the following line below this one: `catalog ALL=(ALL:ALL) ALL`
        
    -   save and close the visudo file
        
    -   to verify that `catalog` has sudo permissions, `su` as `catalog` (run `sudo su - catalog`), and run `sudo -l`
        
    -   after entering in the UNIX password, a line like the following should appear (meaning `catalog` has sudo permissions):
        
        ```
         User catalog may run the following commands on
            ip-XX-XX-XX-XX.ec2.internal:
             (ALL : ALL) ALL
        
        ```
        
3.  While logged in as `catalog`, create a database called catalog by running `createdb catalog`
    
4.  Run `psql` and then run `\l` to see that the new database has been created
    
5.  Switch back to the `ubuntu` user by running `exit`
#### Step 11. Install `git` and clone Catalog Project
1.  Run `sudo apt-get install git`
    
2.  Create a directory called 'CatalogProject' in the /var/www/ directory
    
3.  Change to the 'CatalogProject' directory, and clone the catalog project:
    
    `sudo git clone https://github.com/manoliamruta/CatalogProject.git`
      
4.  Change the ownership of the 'CatalogProject' directory to `ubuntu` by running (while in /var/www):
    
    `sudo chown -R ubuntu:ubuntu CatalogProject/`
    
5.  Change to the /var/www/CatalogProject/CatalogProject directory
       
7.  In the catalogApp.py file do these changes.
    
    `app.run(host='0.0.0.0', port=8000)`
    
    Change this line to:
    
    `app.run()`
    
#### Edit client_secrets.json file

 Authenticate login through Google:
    
    -   Create a new project on the Google API Console
        
    -   Create an OAuth Client ID (under the Credentials tab), and make sure to add [http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com](http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com) as authorized JavaScript origins
        
    -   Add [http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/login](http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/login), [http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/gconnect](http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/gconnect), and [http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/oauth2callback](http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com/oauth2callback) as authorized redirect URIs
        
    -   Edit the file called client_secrets.json in the /var/www/CatalogProject/CatalogProject/ directory.
        
    -   Google will provide a client ID and client secret for the project; download the JSON file, copy and paste the contents into the client_secrets.json file
        
    -   Edit the templates/login.html file in the project directory, and change the clientID to the new one.
        
    -   Add the complete file path for the client_secrets.json file in the catalogApp.py file; change it from 'client_secrets.json' to '/var/www/CatalogProject/CatalogProject/client_secrets.json'.

Switch the database in the application from SQLite to PostgreSQL
Replace in database_setup.py, lotsofitems.py and catalogApp.py with the following:
```
engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')

```
#### Set up a virtual environment and install dependencies

1.  Start by installing pip (if it isn't installed already) with the following command:
    
    `sudo apt-get install python-pip`
    
2.  Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`
    
3.  Change to the /var/www/CatalogProject/CatalogProject/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running `virtualenv venv` .
    
4.  Activate the new environment, `venv`, by running `. venv/bin/activate`
    
5.  With the virtual environment active, install the following dependencies (note: with the exception of the libpq-dev package, make sure to _not_ use `sudo` for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):
    
    `pip install httplib2`
    
    `pip install requests`
    
    `pip install --upgrade oauth2client`
    
    `pip install sqlalchemy`
    
    `pip install flask`
    
    `sudo apt-get install libpq-dev`
    
    `pip install psycopg2`
    
6.  In order to check everything is working correctly, run `python catalogApp.py`; the following (among other things) should be returned:
    
    `* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`
    
7.  Deactivate the virtual environment by running `deactivate`

#### Set up and enable a virtual host

1.  Create a file in /etc/apache2/sites-available/ called catalogProject.conf
    
2.  Add the following into the file:
    
    ``` 
        <VirtualHost *:80>
                    ServerName 13.127.41.80
                    ServerAdmin manoliamruta@gmail.com
                    DocumentRoot /var/www/CatalogProject/CatalogProject

                    <Directory /var/www/CatalogProject/CatalogProject/>
                            Order allow,deny
                            Allow from all
                            Options -Indexes
                    </Directory>

                    Alias /static /var/www/CatalogProject/CatalogProject/static

                    <Directory /var/www/CatalogProject/CatalogProject/static/>
                            Order allow,deny
                            Allow from all
                            Options -Indexes
                    </Directory>
                    # do not allow .git version control files to be issued
                    <Directorymatch "^/.*/\.git+/">
                            Order deny,allow
                            Deny from all
                    </Directorymatch>
                    <Files ~ "^\.git">
                            Order allow,deny
                            Deny from all 
                    </Files>

                    WSGIScriptAlias / /var/www/CatalogProject/catalogProject.wsgi

                    ErrorLog ${APACHE_LOG_DIR}/error.log
                    LogLevel warn
                    CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost> 

    ```

3.  Run `sudo a2ensite catalogProject` to enable the virtual host
    
    The following prompt will be returned:
    
    ```
    Enabling site catalogProject.   
    To activate the new configuration, you need to run:
      service apache2 reload
    
    ```
    
4.  Run `sudo service apache2 reload`

#### Write a .wsgi file
1.  Apache serves Flask applications by using a .wsgi file; create a file called catalogProject.wsgi in /var/www/CatalogProject
    
2.  Add the following to the file:
    
    ```
    activate_this = '/var/www/CatalogProject/CatalogProject/venv/bin/activate_this.py'
    execfile(activate_this, dict(__file__=activate_this))
    
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/CatalogProject/CatalogProject")
    
    from catalogApp import app as application
    application.secret_key = '12345'
    
    ```
3.  Resart Apache: `sudo service apache2 restart`

#### Disable the default Apache site

1.  At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`
    
    The following prompt will be returned:
    
    ```
    Site 000-default disabled.
    To activate the new configuration, you need to run:
      service apache2 reload
    
    ```
2.  Run `sudo service apache2 reload`

#### Change the ownership of the project direcotries
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

```
sudo chown -R www-data:www-data CatalogProject/

```
#### Set up the database schema and populate the database

1.  While in the /var/www/CatalogProject/CatalogProject/ directory, activate the virtualenv by running `. venv/bin/activate`
    
2.  Then run `python lotsofitems.py`
    
3.  Deactivate the virtualenv (run `deactivate`)
    
4.  Resart Apache again: `sudo service apache2 restart`
    
5.  Now open up a browser and check to make sure the app is working by going to [http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com](http://ec2-XX-XX-XX-XX.region.compute.amazonaws.com)

### Sources

Below is a list of sources I used to complete this project.

Udacity course: [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

Udacity course: [Linux Command Line Basics](https://www.udacity.com/course/linux-command-line-basics--ud595)

SQLAlchemy [documentation](http://docs.sqlalchemy.org/en/rel_1_0/orm/basic_relationships.html)

Flask [documenation](http://flask.pocoo.org/) 

Amazon Lightsail [documentation](https://aws.amazon.com/lightsail/)

Stack overflow

Udacity Videos and Forum
