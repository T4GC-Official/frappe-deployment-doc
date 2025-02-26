# Frappe Framework Version 15.x Installation Guide

This guide provides step-by-step instructions to install the Frappe Framework on your system.

## Prerequisites

Before you begin, ensure you have the following installed on your system:

- Python 3.6+ python3-dev python3-pip
- Node.js 20.x
- Redis 6.x
- MariaDB 10.6.6+
- yarn
- git
- wkhtmltopdf (with specific version requirements)

## Add a frappe user and give sudo privileges
```sh
    sudo adduser frappe
    sudo usermod -a -G sudo frappe
```
## Login to the frappe user
```sh
    su - frappe
```
Note: It will ask for the frappe password.

## Update the OS
Note: Before installing any software, it is recommended to update the OS to the latest version.
```sh
    sudo apt-get update
```

##  Install Git
Description: Git is a free and open-source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
```sh
    sudo apt-get install git
```
## Install Redis
Description: Redis is an open-source, in-memory data structure store, used as a database, cache, and message broker.

```sh
    sudo apt-get install redis-server -y
```
## Install Python 3.6+
Description: Python is a programming language that lets you work quickly and integrate systems more effectively.

```sh
    sudo apt-get install python3 python3-dev python3-pip python3.10-venv -y
```
## Install MariaDB
Description: MariaDB is a community-developed, commercially supported fork of the MySQL relational database management system.
```sh
    sudo apt-get update
    sudo apt-get install mariadb-server -y
    sudo apt install software-properties-common
    sudo mysql_secure_installation
    
```
Note: After running the above command, you will be prompted with which user to access (Press Enter), set a root password(Y), remove anonymous users(Y), disallow root login remotely(N), remove the test database(Y), and reload privileges(Y). 
#### Edit Configuration File if frappe version is less than v15.21.x 
#### Note: If you are using frappe version 15.21.x or above, you can skip this step.
```sh
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
```vim
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```
#### Restart MariaDB
```sh
    sudo systemctl restart mariadb
```

## Install Node.js 20.x
Description: Node.js is an open-source, cross-platform, back-end JavaScript runtime environment that runs on the V8 engine and executes JavaScript code outside a web browser.
#### Install node using nvm (Node Version Manager)
```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
    source ~/.profile 
    nvm install 20
    nvm use 20

```
#### Check Node version 
Note: The version should be 21.x or above. for latest version of frappe framework 15.x
```sh
    node -v
```

## Install Yarn using npm
Description: Yarn is a package manager that doubles down as project manager. Whether you work on one-shot projects or large monorepos, as a hobbyist or an enterprise user, we've got you covered.

```sh
    npm install -g yarn
```

#### Install **xvfb** is an X server that can run on machines with no display hardware and no physical input devices. It emulates a dumb framebuffer using virtual memory.
```sh
    sudo apt-get install -y xvfb 
```

#### Install **libfontconfig** is a library designed to provide system-wide font configuration, customization, and application access.
```sh
    sudo apt-get install -y libfontconfig
```

## Install wkhtmltopdf
**Description**: **wkhtmltopdf** and wkhtmltoimage are open source (LGPLv3) command line tools to render HTML into PDF and various image formats using the Qt WebKit rendering engine.
Download and install wkhtmltopdf package from https://wkhtmltopdf.org/downloads.html

```sh
  sudo apt-get install -y wkhtmltopdf
```

# Dependencies for Production Setup

#### Install Nginx
```sh
    sudo apt-get install nginx -y
```
## Add www-data to the frappe user group
```sh
    sudo adduser www-data frappe
```
#### Install Supervisor
```sh
    sudo apt-get install supervisor -y
```
#### Enable Nginx, Supervisor and mariadb
```sh
    sudo systemctl enable nginx && sudo systemctl enable mariadb && sudo systemctl enable supervisor
```
#### Install Fail2ban
```sh
    sudo apt-get install fail2ban -y
```
####
```sh
    sudo apt-get install ansible -y
```

#### Install Certbot
```sh
    sudo apt-get install certbot python3-certbot-nginx -y
```

## Install Frappe Framework
Description: Frappe is a full-stack web application framework written in Python, JavaScript, HTML/CSS with MySQL as the backend. It was developed by Frappe Technologies Pvt. Ltd. and is released under the MIT license.

```sh
    pip3 install frappe-bench
```
## Run the install frappe framework with sudo aswell
```sh
    sudo pip3 install frappe-bench
```
 
Congratulations! You have successfully installed the Frappe Framework on your system.
## Source the file
```sh
    source ~/.profile
```

## Setup the frappe-bench directory
```sh
    bench init frappe-bench
```
<hr>
##################### Now the bench is ready to create new sites for development #####################
<hr>

## Move into the frappe-bench directory
```sh
    cd frappe-bench
```

#### To enable multi_tenancy
```sh
    bench config dns_multitenant on
```
#### Create a new site
```sh
    bench new-site <site-name> --admin-password <site-admin-password> --db-root-password <mariadb-root-password> --db-root-username <mariadb-root-password>
```
#### Setup Redis Cache,Redis Queue and socketio
```sh
    bench setup redis
    bench setup socketio
```
#### Setup Nginx
```sh
    bench setup nginx
```

#### Setup Supervisor
```sh
    bench setup supervisor
```
#### Create symlinks for NGINX and SUPERVISOR (For manual configuration. Run bench setup production for automatic creation)
```sh 
    cd /etc/nginx/conf.d
    sudo ln -s /home/frappe/frappe-bench/config/nginx.conf nginx.conf
    cd ../../supervisor/conf.d
    sudo ln -s /home/frappe/frappe-bench/config/supervisor.conf supervisor.conf
```
#### Start Supervisor
```sh
    sudo supervisorctl reload
    sudo systemctl reload nginx
    sudo supervisorctl status all
    sudo supervisorctl start all
```

#### Enable scheduler
```sh
    bench --site <site-name> enable-scheduler
```

#### Setup Let's Encrypt 
```sh
    sudo -H bench setup lets-encrypt <site-name>
```
# For 2nd site
```sh
    bench new-site <site-name2> --admin-password <site-admin-password> --db-root-password <mariadb-root-password> --db-root-username <mariadb-root-username>
```
#### Setup Nginx again
```sh
    bench setup nginx
```

#### Enable scheduler for the 2nd site
```sh
    bench --site <site-name2> enable-scheduler
```
#### Setup Let's Encrypt 
```sh
    sudo -H bench setup lets-encrypt <site-name>
```

#### Reload Nginx
```sh
    sudo systemctl reload nginx
```
Note: Until the multi_tenancy is not on, lets-encrypt will throw an error.


# Install Frappe Apps
#### To install an app from the Frappe App Store
```sh
    bench get-app <app-name>
    bench --site <site-name> install-app <app-name>
```
#### To install an app from a custom repository
```sh
    bench get-app <app-repo-url>
    bench --site <site-name> install-app <app-name>
```
#### To setup Fail2ban
```sh
    sudo bench setup fail2ban
```

#### To setup Production
```sh
    sudo bench setup production frappe
```

# Additional Important Commands

#### To uninstall an app
```sh
    bench --site <site-name> uninstall-app <app-name>
```
#### To list the apps on a site
```sh
    bench --site <site-name> list-apps
```
#### To migrate a site
```sh
    bench --site <site-name> migrate
```

#### To take a backup of a site
```sh
    bench --site <site-name> backup
```

#### To restore a site
```sh
    bench --site <site-name> restore --db-root-username <username> --db-root-password <password>
```
#### To drop a site
```sh
    bench drop-site <site-name>
```

#### To Update the bench
```sh
    bench update
```
Note: It runs;
1. Updates Bench – Pulls the latest changes for the bench repository.
2. Updates Apps – Pulls updates for Frappe and any installed apps (like ERPNext) from their respective Git repositories.
3. Runs Migrations – Applies database migrations for updated apps.
4. Builds Assets – Recompiles JS, CSS, and other assets.
5. Restarts Services – Restarts frappe processes and related services.

#### To update UI-related changes
```sh 
    bench build
```
Note:
1. Compiles JS & CSS – Processes files from apps/*/public/ and builds them into sites/assets/.
2. Minifies Assets – Optimizes files for production.
3. Updates Webpack Bundles – Rebuilds JS/CSS bundles for Frappe and other apps.
4. Cleans Up Old Files – Removes unused or outdated assets.

# Security Implementations

## Restrict direct SSH access to frappe user
```sh
    sudo sh -c "echo 'DenyUsers frappe' >> /etc/ssh/sshd_config && systemctl restart sshd"
```
## Remove sudo privileges for frappe
```sh
    sudo deluser frappe sudo
```
# Localhost Development Setup
#### To enable developer mode
```sh
    bench set-config developer_mode 1
```
#### To enable auto-reload
```sh
    bench watch
```
#### To start the development server
```sh
    sudo supervisorctl restart all
```