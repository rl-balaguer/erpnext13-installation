# Frappe-ERPNext Installation
This installation uses the latest stable version of frappe framework(13.31.0) and erpnext runtime(13.32.0).
## Virtual Machine
*Note: Deploying the ERP System inside a virtual machine is the simplest and easiest procedure. Through the use of snapshots and clone, it is easier to backup and restore the ERP system image compared to containerized docker solution or a direct installation to the host operating system.*

Install VirtualBox.<br/>
Create a new virtual machine<br/>
Type: Linux, Version: Ubuntu Server(64-bit) *Note: Tested on 22.04 LTS*<br/>
Memory size: 4096 MB<br/>
File size: 60 GB<br/>
Hard disk file type: VDI<br/>
Storage on physical hard disk: Dynamically allocated<br/>
Processor count: 2, Enable PAE/NX<br/>
Video Memory: 8MB<br/>
Graphics Controller: VBoxVGA<br/>
## Guest Operating System

**Your name**: Administrator<br/>
**Computer name**: ubuntu<br/>
**username**: administrator<br/>
**password**: YOUR_PASSWORD<br/>

Wait for the setup to finish. Reboot.

## Frappe framework/ERPNext Installation
### Pre-requisites
Launch Terminal then run the following commands:

```bash
administrator$:sudo su

root#:apt-get install libffi-dev python3-pip python3-dev python3-testresources libssl-dev wkhtmltopdf gcc g++ make redis -y
```

### Database Installation and Configuration

#### Install MariaDB Server and Client
```bash
root#:apt-get install mariadb-server mariadb-client -y
```

The command above will install both server and client automatically.
```bash
root#:mysql_secure_installation
```

The above command will prompt for the following options:

Enter current password for root (enter for none): **(Enter)**<br/>
Switch to `unix_socket authentication`: **n**<br/>
Change the `root` password?: **y**<br/>
New password: **YOUR_PASSWORD**<br/>
Remove anonymous user?: **y**<br/>
Disallow root login remotely: **y**<br/>
Remove test database and access to it?: **y**<br/>
Reload privilege tables now? **y**<br/>

#### MySQL Development files
```bash
root#:apt-get install libmysqlclient-dev
```

#### MariaDB Configuration
Login to MariaDB console:
```bash
root#:mysql -u root -p
```

Verify MariaDB authentication plugin:
```SQL
MariaDB > use mysql;
MariaDB [mysql]> select User, plugin from user;
```

The result should be similar to this:
| User | plugin |
| -----| -----|
| root | mysql_native_password |

Edit the file: 
```bash
root#:nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add or modify the following lines:
```ini
[mysqld]
innodb-file-format=barracuda
innodb-file-per-table=1
innodb-large-prefix=1
character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utfmb4_unicode_ci
```

Save the file, then restart MariaDB service:

```bash
root#:systemctl restart mariadb
```

### Create a User for ERPNext

Create a new user named `erpnext`:

```bash
root#:useradd -m -s /bin/bash erpnext
```

Set password for `erpnext` user:

```bash
root#:passwd erpnext
```

Add `erpnext` user to `sudo group` so that it can execute `sudo` command:

```bash
root#:usermod -aG sudo erpnext
```

Login the `erpnext` user and set up the environment variables:

```bash
root#:su - erpnext
erpnext$:sudo nano ~/.bashrc
```

Add the following line:

```bash
PATH=$PATH:~/.local/bin/
```

Save and close the file, then activate the environment variable:

```bash
erpnext$:source ~/.bashrc
```
### Install curl
```bash
erpnext$:sudo apt install curl
```

### Install Node.js
```bash
erpnext$:curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash -

erpnext$:source ~/.bashrc

erpnext$:nvm install 14
```
### Install Yarn using NPM
```bash
erpnext$:npm install -g yarn
```

### Install ERPNext
Login `erpnext` user
```bash
su - erpnext
```

Create a new directory for `bench`.
```bash
erpnext$:sudo mkdir /opt/bench
```

Grant permission to `erpnext user`:
```bash
erpnext$:sudo chown -R erpnext:erpnext /opt/bench
```

Change directory to your newly created folder. Clone bench.
```bash
erpnext$:cd /opt/bench
erpnext$:git clone https://github.com/frappe/bench bench-repo
```

Install bench.
```bash
erpnext$:pip3 install -e bench-repo
```

Initialize frappe.
```bash
erpnext$:bench init erpnext --frappe-branch version-13
```

Change directory to the newly created folder. Create new site for your desired domain.
```bash
erpnext$:cd erpnext
erpnext$:bench new-site YOUR_SITE
```

Get the ERPNext app with version 13.
```bash
erpnext$:bench get-app --branch version-13 erpnext
```

Install erpnext app in your site.
```bash
erpnext$:bench --site YOUR_SITE install-app erpnext
```

Install `supervisor`, `nginx` and `nodejs`.
```bash
erpnext$:sudo apt-get install supervisor nginx nodejs
```

Install `frappe-bench` from `pip`.
```bash
erpnext$:sudo pip3 install frappe-bench
```

Setup for production environment. Run this twice.
```bash
erpnext$:sudo ~/.local/bin/bench setup production erpnext
```

Done.

Check `supervisor` status to verify that `frappe/erpnext` services are running.
```bash
erpnext$:sudo supervisorctl status

erpnext-redis:erpnext-redis-cache                RUNNING
erpnext-redis:erpnext-redis-queue                RUNNING
erpnext-redis:erpnext-redis-socketio             RUNNING
erpnext-web:erpnext-frappe-web                   RUNNING
erpnext-web:erpnext-node-socketio                RUNNING
erpnext-workers:erpnext-frappe-default-worker-0  RUNNING
erpnext-workers:erpnext-frappe-long-worker-0     RUNNING
erpnext-workers:erpnext-frappe-schedule          RUNNING
erpnext-workers:erpnext-frappe-short-worker-0    RUNNING
```

Create a clone to backup your successful installation snapshot.
