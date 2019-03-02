# Linux Server  

This file explains the process of creating a Linux-based server using [Amazon Lightsail](https://aws.amazon.com/lightsail/).  


The Linux distribution is Ubuntu 16.04 LTS.
The hosted web application is my Item Catalog App created in the course of my Udacity Full Stack Developer Nanodegree.

Deployed website can be found at http://3.91.87.43.xip.io or http://3.91.87.43.  

<hr>  

## Linux Server Setup and Configuration
1. [Install](https://git-scm.com/downloads) and [set up](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup) Git.  

2. Sign up for or login to [Amazon Lightsail](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start) and click create an instance in Amazon Lightsail.  
  a. Select the Linux/Unix platform.  
  b. Select OS only, then Ubuntu 16.04 LTS.  
  c. Choose an instance plan. There is an option that has one month free.   
  ![instance plans](https://github.com/mejeter/linux-server/blob/master/img/instance_plans.PNG)  
  d. Name your instance.  
  e. Copy your Public IP address and save in a note. **You will need this later.**  

3. Go to the Account page in Amazon Lightsail and download the SSH key.  
  a. Move the downloaded SSH key to the .ssh folder on your local machine and change the name to key.rsa.  
  ![download ssh key](https://github.com/mejeter/linux-server/blob/master/img/download_key.PNG)  
<br>  

### Connect to server
4. Change the permissions on the .ssh folder and the SSH key by running `$ chmod 700 ~/.ssh/` then `$ chmod 600 ~/.ssh/key.rsa` in your terminal.  

5. Connect to the server with `$ ssh -i ~/.ssh/key.rsa ubuntu@3.91.87.43` and replace 3.91.87.43 with the public IP address copied earlier.  
  a. The terminal will ask if you are sure you want to continue; type yes.  
<br>  

### Create grader user and grant access
6. Create a new user with the command `$ sudo adduser grader`.  
  a. Create a password for the user **grader** and confirm.  

7. Add a new file that will grant sudo permission `$ sudo nano /etc/sudoers.d/grader`.  
  a. Enter `grader ALL=(ALL:ALL) ALL` in the file and exit with Ctrl-X, save with Y, then Enter to confirm.  

8. Generate a new key pair with `$ ssh-keygen`.  
  a. When prompted, enter `/home/ubuntu/.ssh/grader_key` to save.  
  b. Enter a passphrase twice.  

9. Print the public key with `$ cat /home/ubuntu/.ssh/grader_key.pub`.  
  a. Copy this public key and save in a note. **You will need this later.**  

10. After running `$ cd /home/grader`, enter `$ sudo mkdir .ssh` to create a new directory.  

11. Create a new file with `$ sudo nano /home/grader/.ssh/authorized_key`.  
  a. Enter the public key copied earlier and save (Ctrl-X, Y, Enter).  

12. Change the permissions on the grader folder and the public key file with `$ sudo chmod 700 /home/grader/.ssh` and then `$ sudo chmod 644 /home/grader/.ssh/authorized_key.`  

13. Switch the owner from ubuntu to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`.  

14. Edit the configuration file: `$ sudo nano /etc/ssh/sshd_config`.  
  a. Change Port to **2200**.  
  b. Change PermitRootLogin to **no**.  
  c. Change PasswordAuthentication to **no**.  
  d. Save (Ctrl-X, Y, Enter).  

15. Run `$ sudo service ssh restart` for changes to take effect.  

16. Back in the Amazon Lightsail instance, change port number to **2200** under the Networking tab.  
![change port number](https://github.com/mejeter/linux-server/blob/master/img/port_number.PNG)  

17. Run `$ ssh -i ~/.ssh/grader_key -p 2200 grader@3.91.87.43` to restart the server on port 2200.  
<br>  

### Configure timezone and UFW
18. Update and upgrade packages: `$ sudo apt-get update` and `$ sudo apt-get upgrade`.  
  a. When prompted, select install package maintainer's version.  

19. Configure time zone: `$ sudo dpkg-reconfigure tzdata` and `$ sudo apt-get install ntp`.  
  a. When prompted, enter the region where you live.  

20. Configure Uncomplicated Firewall.  

  ```
  $ sudo ufw default deny incoming.
  $ sudo ufw default allow outgoing.
  $ sudo ufw allow 2200/tcp.
  $ sudo ufw allow 80/tcp.
  $ sudo ufw allow 123/udp.
  $ sudo ufw enable
  $ sudo ufw status
  ```  
Add the above port options to the Amazon Lightsail Firewall.  

![update port numbers](https://github.com/mejeter/linux-server/blob/master/img/port_updating.PNG)  
<br>  

### Configure Apache and mod_wsgi
21. Install Apache with: `$ sudo apt-get install apache2`.  
22. In your browser, go to http://3.91.87.43/ to ensure Apache is functioning correctly. The Apache2 Default page should be displayed.  
23. Install mod_wsgi with: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.  
24. Enable mod_wsgi: `$ sudo a2enmod wsgi`.  
25. Restart Apache  `$ sudo service apache2 restart`.  
<br>  

### Install PostgreSQL
26. Install PostgreSQL: `$ sudo apt-get install postgresql`.  
27. Ensure that PostgreSQL does not allow remote connections by running `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`. You should see:  

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
<br>  

### Create catalog user in PostgreSQL
28. Switch to **postgres** user: `$ sudo su - postgres` and connect to PostgreSQL with `$ psql`.  
29. Create user **catalog** with `# CREATE ROLE catalog WITH PASSWORD 'catalog';`.  
30. Allow **catalog** user create database permissions: `# ALTER USER catalog CREATEDB;`.  
31. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`.  
32. Connect to catalog database: `# \c catalog`.  
33. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.  
34. Grant catalog access: `# GRANT ALL ON SCHEMA public TO catalog;`.  
35. Exit psql with `# \q`. Then, switch from **postgres** user to **grader** user with `$ exit`.  
<br>  

### Create catalog user in Linux
36. Create a new user: `$ sudo adduser catalog`.  
  a. Create a password for the user **catalog** and confirm.  
37. Grant sudo access to **catalog** user: `$ sudo visudo`.  
  a. Insert `$ catalog ALL=(ALL:ALL) ALL` below the line `$ root ALL=(ALL:ALL) ALL`.  
  b. Exit and save (Ctrl-X, Y, Enter).  

38. Switch to **catalog** user: `$ sudo su - catalog`.  
39. Create catalog database with `$ createdb catalog`, then exit **catalog** user with `$ exit`.  
<br>  

### Clone catalog application
40. Install Git: `$ sudo apt-get install git`.  
41. Make a catalog directory: `$ sudo mkdir /var/www/catalog`.  
  a. Change directory: `$ cd /var/www/catalog`.  
42. Clone item catalog app: `$ sudo git clone https://github.com/mejeter/item-catalog.git catalog`.  
  a. Change owner of catalog directory: `$ sudo chown -R ubuntu:ubuntu catalog`.  
  b. Change directory: `$ cd catalog`.  
<br>  

### Edit files for Linux server compatibility
43. Rename project.py file: `$ sudo mv project.py __init__.py`.  
44. Edit **init.py** file: `$ sudo nano __init__.py`.  
  a. Replace `app.run(host='0.0.0.0', port=8000)` with **`app.run()`**.  
  b. Update path for json file from `'client_secrets.json'` to **`'/var/www/catalog/catalog/client_secrets.json'`**.  
  c. Replace `engine = create_engine('sqlite:///movies.db')` with **`engine = create_engine('postgresql://catalog:PASSWORDE@localhost/catalog')`**.  
  d. Exit and save (Ctrl-X, Y, Enter).  
45. Edit the create engine lines in both `database_setup.py` and `movies.py` files in the same manner as `__init__.py`: with **`engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`**.  
<br>  

### Edit client secrets JSON file
46. Create a new project in the Google API Developer console and download `client_secrets.json` file.  
47. Update client secrets file: `$ sudo nano client_secrets.json`.  
  a. Copy and paste contents of downloaded JSON file, and save (Ctrl-X, Y, Enter).  
48. Update `login.html` file: `$ sudo nano var/www/catalog/catalog/templates/login.html`.  
  a. Replace new client ID from downloaded JSON file and save (Ctrl-X, Y, Enter).  
<br>  

### Install packages
49. Install pip: `$ sudo apt-get install python-pip`.  
50. Install dependencies.  

  ```
  $ sudo pip install flask
  $ sudo pip install --upgrade oauth2client
  $ sudo apt-get install libpq-dev
  $ sudo pip install httplib2
  $ sudo pip install requests
  $ sudo pip install sqlalchemy
  $ sudo pip install psycopg2
  ```  
<br>  

### Configure virtual host
51. Create and configure a new virtual host: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.  
  a. Add the following:  

  ```
    <VirtualHost *:80>
       ServerName 3.91.87.43
       ServerAdmin testudacity225@gmail.com
       WSGIScriptAlias / /var/www/catalog/catalog.wsgi
       <Directory /var/www/catalog/catalog/>
           Order allow,deny
           Allow from all
           Options -Indexes
       </Directory>
       Alias /static /var/www/catalog/catalog/static
       <Directory /var/www/catalog/catalog/static/>
           Order allow,deny
           Allow from all
           Options -Indexes
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
  ```  
  b. Exit and save (Ctrl-X, Y, Enter).  
52. Enable the virtual host: `$ sudo a2ensite catalog`.  
  a. Reload Apache: `$ sudo service apache2 reload`.  
<br>  

### Configure wsgi file
53. Create and configure the .wsgi file: `$ sudo nano /var/www/catalog/catalog.wsgi`.  
  a. Add the following to the file:  

  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
  ```  
  b. Exit and save (Ctrl-X, Y, Enter).  
  c. Reload Apache: `$ sudo service apache2 reload`.  

54. Disable Apache default page: `$ sudo a2dissite 000-defualt.conf`.  
  a. Reload Apache: `$ sudo service apache2 reload`.  
<br>  

### Populate database
55. Setup database: `$ sudo python database_setup.py`.  
56. Populate database: `$ sudo python movies.py`.  
57. Reload Apache: `$ sudo service apache2 reload`.  
58. In your browser, go to http://3.91.87.43 to ensure that the application is up and running.  
  a. If error page is displayed, check Apache error logs: `$ sudo tail -100 /var/log/apache2/error.log`.  
<br>  

<hr>  

## Acknowledgements
* [golgtwins' Libraries.io](https://libraries.io/github/golgtwins/Udacity-P7-Linux-Server-Configuration)
* [yiyupan's GitHub](https://github.com/yiyupan/Linux-Server-Configuration-Udacity-Full-Stack-Nanodegree-Project)
