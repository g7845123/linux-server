# Overview

This is a project for [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/nd004) provided by [Udacity](https://www.udacity.com). 

In this project, I took a baseline installation of a Linux distribution on a virtual machine and prepare it to host my web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

# Server Infomation

IP address: 52.88.185.81

SSH port: 2200

URL: [ec2-52-88-185-81.us-west-2.compute.amazonaws.com](ec2-52-88-185-81.us-west-2.compute.amazonaws.com) 

Use the path /munin to monitor server status with username `munin` and password `UMzV3bx8`

# Configuration Steps

* Create new user "grader" and login 

    ```sh
    # adduser grader
    # su - grader
    ```
* Generate key pair for grader

    ```sh
    $ mkdir ~/.ssh
    $ chmod 700 ~/.ssh
    $ ssh-keygen -t rsa
    $ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
    ```
* Disable remote root login

    ```sh
    $ sudo vim /etc/ssh/sshd_config
    ```
    Set `PermitRootLogin` to `no`, and restart ssh service
    ```sh
    $ sudo service ssh restart
    ```
* Change ssh port to 2200

    ```sh
    $ sudo vim /etc/ssh/sshd_config
    ```
    Set `Port` to `2200`, and restart ssh service
    ```sh
    $ sudo service ssh restart
    ```
* Setup firewall

    ```sh
    $ sudo ufw limit 2200
    $ sudo ufw allow http
    $ sudo ufw allow ntp
    $ sudo ufw enable
    ```
    Here, 2200 is set to `limit` instead of `allow` to monitor for repeat unsuccessful login attempts and ban potential attackers
* Update packages

    ```sh
    $ sudo apt-get upgrade
    ```
* Setup weekly automatic package update

    ```sh
    $ sudo vim /etc/cron.weekly/apt-security-updates
    ```
    Add the forllowing commands
    ```sh
    echo "**************" >> /var/log/apt-security-updates
    date >> /var/log/apt-security-updates
    aptitude update >> /var/log/apt-security-updates
    aptitude safe-upgrade -o Aptitude::Delete-Unused=false --assume-yes --target-release `lsb_release -cs`-security >> /var/log/apt-security-updates
    echo "Security updates (if any) installed"
    ```
    Then set the script executable
    ```sh
    $ sudo chmod +x /etc/cron.weekly/apt-security-updates
    ```
* Install PostgreSQL and set password "postgres"

    ```sh
    $ sudo apt-get install postgresql
    $ sudo passwd postgres
    ```
* Configure database "moviecollection" and user "catalog"

    ```sh
    $ sudo su - postgres
    $ psql
    postgres=# \password postgres
    postgres=# CREATE DATABASE moviecollection;
    postgres=# CREATE USER catalog WITH PASSWORD '******';
    postgres=# \q
    $ psql moviecollection
    moviecollection=# GRANT ALL PRIVILEGES ON DATABASE moviecollection TO catalog;
    moviecollection=# \q
    $ exit
    ```
* Install apache2 server

    ```sh
    $ sudo apt-get install apache2
    ```
* Install git and clone project 3

    ```sh
    $ sudo apt-get install git
    $ cd /var/www
    $ sudo git clone https://github.com/g7845123/movie-collection
    ```
    Set permission 755 for folders, 644 for static files, 700 for dynamic files

* Install packages required for project 3

    ```sh
    $ apt-get -qqy install python-sqlalchemy
    $ apt-get -qqy install python-pip
    $ sudo pip install werkzeug
    $ sudo pip install flask
    $ sudo pip install oauth2client
    $ sudo pip install requests
    $ sudo pip install httplib2
    $ sudo pip install dict2xml
    $ sudo pip install flask-seasurf
    ```
* Modify project 3 code to migrate from sqlite to PostgreSQL

    ```py
    SQLALCHEMY_DATABASE_URI = "postgresql://catalog:******@localhost/moviecollection"
    engine = create_engine(SQLALCHEMY_DATABASE_URI)
    ```
* Initialize database

    ```sh
    $ python database_setup.py
    ```
* Fill database with popular movies in themoviedb.org

    ```sh
    $ python init_databases.py
    ```
* Install WSGI

    ```sh
    $ sudo apt-get install libapache2-mod-wsgi
    ```
* Create WSGI application "myapp.wsgi"

    ```py
    import logging, sys, os
    sys.path.append(os.path.dirname(__file__))
    logging.basicConfig(stream=sys.stderr)

    from moviecollection import app as application

    application.secret_key = 'super_secret_key'
    application.debug = True

    if __name__ == '__main__':
        application.run(host='0.0.0.0')
    ```
* Configure WSGI

    ```sh
    $ sudo vim /etc/apache2/sites-enabled/000-default.conf
    ```
    Add the following config entries: 
    ```sh
    WSGIScriptAlias / /var/www/moviecollection/myapp.wsgi
    <Directory /var/www/moviecollection>
        WSGIProcessGroup moviecollection
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>
    # Block public access to .git folder 
    <Directory /var/www/moviecollection/.git>
        Order allow,deny
        Deny from all
    </Directory>
    ```
    Restart server
    ```sh
    $ sudo apache2ctl restart
    ```
* Install Munin to monitor server status

    ```sh
    $ sudo apt-get install munin-node
    $ sudo apt-get install munin
    $ sudo apt-get install apache2-utils
    $ sudo htpasswd -c /etc/munin/munin-htpasswd munin
    sudo vim /etc/munin/plugin-conf.d/munin-node
    ```
    Set `env.PGPASSWORD` to the database password under `[postgres_*]` section and restart the service. 
    ```sh
    $ sudo service munin-node restart
    ```

# References

All Web sites, books, forums, blog posts, github repositories etc. that I referred to or used are listed in `references.txt`. 
