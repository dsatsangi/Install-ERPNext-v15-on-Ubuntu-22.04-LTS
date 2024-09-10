# Install-ERPNext-v15-on-Ubuntu-22.04-LTS
A complete Guide on How to Install Frappe/ERPNext version 15 in Ubuntu 22.04 LTS


<h1 align="center">
  <br>
  <a href="https://erpnext.com/"><img src="https://erpnext.com/files/erpnext-logo-blue-v2.png" alt="Markdownify" width="200"></a>
  <br>
  ERPNEXT v15
  <br>
</h1>

## Steps
* Install ERPNext
* Setup Production
* Setup Multitenancy
* Add a Domain
* Install SSL Certificate

## Pre-requisites
* Ubuntu 22.04 LTS
* Python 3.11+
* Node.js 20+
* A user with sudo privileges
* MariaDB 10.3.x

## Step 1 - Update and Upgrade Packages
```bash
sudo apt-get update -y
```
```bash
sudo apt-get upgrade -y
```

## Step 2 - Create a New User
```bash
sudo adduser frappeuser
```
```bash
sudo usermod -aG sudo frappeuser
```
```bash
su frappeuser
```
```bash
cd /home/frappeuser
```
* Ensure you have replaced [frappeuser] with your username. eg. sudo adduser frappe

# Install Required Packages
- For setting up ERPNext 15, we need to install several software packages first.
- Install Python ERPNext version 15 requires Python version 3.11+.

## Step 3 - Adding PPA repository and Install Python 3.11
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
```
```bash
sudo apt-get install python3-dev python3.11-dev python3-setuptools python3-pip python3-distutils
```
## Step 4 - Install Python Virtual Environment
```bash
sudo apt install python3.10-venv python3.11-venv 
```
## Step 5 - Set Default Python Version to 3.11
```bash
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.11 1
```
## Step 6 - Install Redis Server
```bash
sudo apt-get install redis-server
```

### Step 7 - Install and Setup MariaDB
```bash
sudo apt-get install software-properties-common
```
```bash
sudo apt install mariadb-server mariadb-client
```
```bash
sudo mysql_secure_installation
```

Upon running the last command, you’ll encounter a series of prompts on the server. Make sure to follow the subsequent Steps carefully to ensure the setup is configured properly.

Enter current password for root: (Enter your SSH root user password) (Press blank Enter if for the first time)

Switch to unix_socket authentication [Y/n]: Y

Change the root password? [Y/n]: Y

Remove anonymous users? [Y/n] Y

Disallow root login remotely? [Y/n] Y

Remove test database and access to it? [Y/n] Y

Reload privilege tables now? [Y/n] Y


### Step 8 - Create my.cnf MYSQL config file
```bash
sudo nano /etc/mysql/mariadb.conf.d/my.cnf
```
##### Add these lines
    [mysqld]
    innodb-file-format=barracuda
    innodb-file-per-table=1
    innodb-large-prefix=1
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci
    
    [mysql]
    default-character-set = utf8mb4

#restart mariadb to update my.cnf
```bash
systemctl restart mariadb
```

###  Step 9 - Install other packages
```bash
sudo apt-get install xvfb libfontconfig
```
```bash
sudo apt-get install libmysqlclient-dev
```
```bash
sudo apt-get install fontconfig libxrender1 xfonts-75dpi
```
```bash
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb && sudo dpkg --install wkhtmltox_0.12.6.1-2.jammy_amd64.deb;
```
#Test the install: wkhtmltopdf 0.12.6 (with patched qt)
```bash
wkhtmltopdf --version
```
    
###  Step 10 - Install CURL, Node.js, NPM, and Yarn
#### CURL
```bash
sudo apt install curl
```
##### Node.js
```bash
sudo apt install curl 
```
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
```bash
source ~/.bashrc
```
```bash
nvm install 20
```
##### npm
```bash
sudo apt-get install npm
```
##### Yarn
```bash
sudo npm install -g yarn
```
    
### Step 10 - Install Frappe Bench
```bash
sudo pip3 install frappe-bench
```

### Step 11 - Initialize Frappe Bench
```bash
bench init frappe-folder --frappe-branch version-15
```
```bash
cd frappe-folder/
```
    
##### Add the node-sass package
```bash
yarn add node-sass
```
    
#### Change user directory permissions
```bash
sudo chmod -R o+rx /home/frappeuser
```
    
### Step 12 - Create a New Site
```bash
bench new-site [site-name]
```
### Step 13 - Install ERPNext and other Apps
```bash
bench get-app erpnext --branch version-15
```
```bash
bench get-app hrms --branch version-15
```
```bash
bench get-app --branch version-15 https://github.com/resilient-tech/india-compliance.git
```
```bash
bench get-app https://github.com/The-Commit-Company/Raven.git
```
```bash  
bench --site [site-name] install-app erpnext
bench --site [site-name] install-app hrms
bench --site [site-name] install-app india_compliance
bench --site [site-name] install-app raven
```

#### Lastly (For Development)
```bash
bench use [site-name]
```
```bash
bench start
```

# Setting ERPNext for Production
### Step 1 - Enable Scheduler
    bench --site [site-name] enable-scheduler
### Step 2 - Disable maintenance mode
    bench --site [site-name] set-maintenance-mode off
### Step 3 - Setup production config
    sudo bench setup production [frappe-user]
### Step 4 - Setup NGINX to apply the changes
```bash
bench setup nginx
```
### Step 5 - Restart Supervisor and Launch Production Mode
```bash
sudo supervisorctl restart all
```
```bash
sudo bench setup production frappeuser
```

# Restore Database
#### Restore
bench --site [site-name] --force restore [path to database backup file] --with-private-files [relative-path-to-private-files-backup-file] --with-public-files [relative-path-to-public-files-backup-file]
#### Migrate
bench --site [site-name] migrate

# Setup Multitenancy
#### DNS based multitenancy 
You can name your sites as the hostnames that would resolve to it. Thus, all the sites you add to the bench would run on the same port and will be automatically selected based on the hostname.

To make a new site under DNS based multitenancy, perform the following Steps.

### Step 1 - Switch on DNS based multitenancy (once)
```bash
bench config dns_multitenant on
```
    
### Step 2 - Create a new site
```bash
bench new-site site2name
```
    
### Step 2 - Re generate nginx config
```bash
bench setup nginx
```

### Step 3 - Reload nginx
```bash
sudo service nginx reload
```
    
# Adding a Domain with SSL to your Site
### Add Domain
    bench setup add-domain [desired-domain]
### Install Let's Encrypt Certificate
#### Install snapd on your machine
```bash
sudo apt install snapd
```
#### Update snapd
```bash
sudo snap install core;
```
```bash
sudo snap refresh core
```
#### Remove existing installations of certbot
```bash
sudo apt-get remove certbot
```
#### Install certbot
```bash
sudo snap install --classic certbot
```
```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### Get Certificate
```bash
sudo -H bench setup lets-encrypt [site-name]
```
You will be faced with several prompts, respond to them accordingly. This command will also add an entry to the crontab of the root user (this requires elevated permissions), that will attempt to renew the certificate every month.

## Credits

This how to has been prepared in reference with

- [kalungia/How-to-Install-ERPNext-on-Ubuntu-22.04-LTS](https://github.com/kalungia/How-to-Install-ERPNext-on-Ubuntu-22.04-LTS)
