# Introduction
I'll guide you trough how you can install Seafile on a Ubuntu 16.04 LTS machine, in this document I'll only adapt best practices.
To tighten up your security on your Ubuntu server please se the NohaTech-Ubuntu-secure-server.md file.

***All the path's in this guide are starting at /opt/nohatech/ make sure that you change them to the path that your using.***

# Installation
Now we going to start the installation.

### Create user
Best is to have a "clean" Ubuntu installation to install Seafile on, but if you don't have a clean one it's recommended that Seafile are running with it's own user - ***you should not run Seafile with root or sudo command.***

### Update Ubuntu
We need to make sure that Ubuntu are up to date, use this command below to do that.
```
 sudo apt-get update && sudo apt-get upgrade -y
```
Now reboot your server

### Install MariaDB
We are going to use MariaDB and what we want to do is that we want to run the latest stable version to do that we need to change some files and do some installation.
```
 sudo apt-get install software-properties-common
 sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
```
And then we need to add some lines to /etc/apt/sources.list
```
 sudo nano /etc/apt/sources.list
```
Then add this two lines in the bottom of the sources.list
```
# http://downloads.mariadb.org/mariadb/repositories/
deb [arch=amd64,i386] http://ftp.ddg.lth.se/mariadb/repo/10.2/ubuntu xenial main
deb-src http://ftp.ddg.lth.se/mariadb/repo/10.2/ubuntu xenial main
```
As you can se when I write this document the latest stable version of MariaDB is 10.2, to make sure that it's still the latest version please visit http://downloads.mariadb.org/mariadb/repositories/ 

Now it's time to install MariaDB.
```
 sudo apt-get update
 sudo apt-get install mariadb-server -y
```
And now to finish the MariaDB setup we need to run some security, this is something that many people are missing and it's really important to do this beacuse otherwise it's easy to get hacked.
```
 sudo mysql_secure_installation
```
(If you using auth_socket in MariaDB you just press enter when it's asking for the password, if your not using it then type your root password for MariaDB.)
This setup are relly easy to do, now you will be asked if you want to change your administration password for MairaDB here you can choose whatevery you want. But for the other quastions you should answer Y on everyone of them.

#### Create the databases
Now we need to add some databases to MariaDB, we are going to do that manually because of the new unix_socket auth that MariaDB have been start using, because of that the "setup-seafile-mysql.sh" will not work depending of what version of MariaDB your running - so better safe then sorry.
```
 sudo mysql -u root -p
```
(If you using auth_socket in MariaDB you just press enter when it's asking for the password, if your not using it then type your root password for MariaDB.)
Now we are in MariaDB and we need to create the databases for Seafile.
```
 create database `ccnet-db` character set = 'utf8';
 create database `seafile-db` character set = 'utf8';
 create database `seahub-db` character set = 'utf8';
```
Now we need to create the Seafile user for MariaDB and give it access to the databases that we have been creating.
Change the "identified by 'seafile';" to a password of your choosing rather then seafile use a strong and long password, you should write your password inside the '' signs and make sure not to delete them or the ; sign.
```
 create user 'seafile'@'localhost' identified by 'seafile';
 
 GRANT ALL PRIVILEGES ON `ccnet-db`.* to `seafile`@localhost;
 GRANT ALL PRIVILEGES ON `seafile-db`.* to `seafile`@localhost;
 GRANT ALL PRIVILEGES ON `seahub-db`.* to `seafile`@localhost;
```
Now we are all done, so just do the following.
```
 flush privileges;
 quit;
```

### The last installation
Now it's time to do the installation of the rest of the needed things, as you can see it's deverted in to blocks so run one command at the time to make sure that everything are installing correctly.
```
 sudo apt-get install python -y
 
 sudo apt-get install python2.7 libpython2.7 python-setuptools python-imaging python-ldap python-urllib3 ffmpeg python-pip python-mysqldb python-memcache memcached libmemcached-dev zlib1g-dev -y
 
 sudo -H pip install pillow moviepy pylibmc django-pylibmc
```
You will be prompted to upgrade pip during the installations above, so we are going to do that - if you don't get prompted about it you can still run this command to be sure that you have the latest version.
```
 sudo -H pip install --upgrade pip
```
And now we need to upgrade Pillow to the latest version so we can get the best experience with thumbnails and viewing our pictures from the webbrowser.
```
 sudo -H pip install --upgrade Pillow
```
And we can also upgrade this two pip package.
```
 sudo -H pip install --upgrade python-memcached==1.57
 sudo -H pip install --upgrade pytz==2016.7
```

# Download and setup Seafile
In this section we are going to download, install and setup Seafile.
To see what's the latest version of Seafile CE please visit this link: https://www.seafile.com/en/download/
We are going to use the /opt folder to install Seafile in, and we are going to use a folder named "nohatech".
You can change the "nohatech" folder name to what ever you want I just using it as a example.

### Download Seafile
Now it's time to download Seafile. As you can see in this example I'm using the version 6.2.2 change the filenames to the version you are installing.
```
 cd /opt
 wget https://download.seadrive.org/seafile-server_6.2.2_x86-64.tar.gz
```
And now we need to unpack it.
```
 tar -xvzf seafile-server_6.2.2_x86-64.tar.gz
```
And now we need to create some folders, change the "nohatech" folders name to a name that you prefer.
```
 mkdir nohatech
 mkdir nohatech/installed
```
Now we need to move the seafile-server_6.2.2 folder and the seafile-server_6.2.2_x86-64.tar.gz file.
```
 mv seafile-server_6.2.2_x86-64.tar.gz nohatech/installed
 mv seafile-server_6.2.2 nohatech
```

### Install Seafile
Now it's time to run the setup script for Seafile.
```
 cd /opt/nohatech/seafile-server_6.2.2
 
 ./setup-seafile-mysql.sh
```
During the setup you should choose
```
 [2] Use existing ccnet/seafile/seahub databases
```
And then read everything in most cases it's just run at default except when it asks you for your username and password for the databases then you should write the ones that you did choose in the MariaDB installation step above.
The setup script will ask you the names of the databases, and it's the same as the one you did create in the MariaDB installation also, if you don't rememeber what you did choose take a look at the MariaDB installation section above.

Now you should have new folders in the nohatech/ folder.
So what we need to do now is to run Seafile for the first time and then you will be prompted to create the Administrator when you do run the ./seahub.sh start command, but it's importent that you are running the ./seafile.sh start command before the seahub as both are needed to run Seafile.
And also remember that you should never start Seafile with the sudo command or change any of the files in your Seafile folder with the sudo command.
```
 cd /opt/nohatech/seafile-server-latest
 
 ./seafile.sh start
 ./seahub.sh start
```
Now things are up an running for you, good! What we need to do now is to do some modifications and install some add-ons to make it run perfectly.
But you can try to login on it trough your webbrowser: http://yourlocalip:8000/
Make sure that you have opend port 8000 in the UFW firewall if you have it activated, but UFW should be disable as default.

### Add Memcached
It's recommended to setup Memcached for Sefile to increase the performance, we have already installed all of the necessary components.
So what we need to do is to add some lines to the seahub_settings.py file.
```
 nano /opt/nohatech/conf/seahub_settings.py
```
Then add the following lines in the bottom of the file.
```
 CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
    }
 }
```
#### Memcached trough unix_socket
This is recommended but it's not completly tested yet so for now we will just use the http socket option, but I'll update this section as soon as possible, performence should increase with about 30% according to some benchmarks.

### Start Seafile on system boot
We want Seafile to autostart on boot of our server, so now we going to fixa that.
So now we need to create some new files.
Remember to change the path to your own path for Seafile also remember to change the user and group name if your user and group name are not Seafile as in this example.
```
 sudo nano /etc/systemd/system/seafile.service
```
And then add the following to the file.
```
 [Unit]
 Description=Seafile
 After=network.target mysql.service

 [Service]
 Type=oneshot
 ExecStart=/opt/nohatech/seafile-server-latest/seafile.sh start
 ExecStop=/opt/nohatech/seafile-server-latest/seafile.sh stop
 RemainAfterExit=yes
 User=seafile
 Group=seafile

 [Install]
 WantedBy=multi-user.target
```
Now we need to create a second file.
```
 sudo nano /etc/systemd/system/seahub.service
````
Then add this to the file.
```
 [Unit]
 Description=Seafile hub
 After=network.target seafile.service

 [Service]
 ExecStart=/opt/nohatech/seafile-server-latest/seahub.sh start
 ExecStop=/opt/nohatech/seafile-server-latest/seahub.sh stop
 User=seafile
 Group=seafile
 Type=oneshot
 RemainAfterExit=yes

 [Install]
 WantedBy=multi-user.target
```
And now we are on the last step, now we need to enable this as a service.
```
 sudo systemctl enable seafile.service
 sudo systemctl enable seahub.service
```

### Seafile GC
Seafile GC are going to clean up the deleted files etc. to free space on your server. But on the CE (free version) of Seafile it can't run as long as Seafile are running.
So we are going to make a script and add it to crontab to make it autorun and solve this little issue for us.

First we need to create the file.
```
 nano /opt/nohatech/seafile/cleanupScript.sh
```
Then add the following to the file.
```
 #!/bin/bash
 # stop the server
 echo Stopping the Seafile-Server...
 systemctl stop seafile.service

 echo Giving the server some time to shut down properly....
 sleep 10

 # run the cleanup
 echo Seafile cleanup started...
 sudo -u seafile /opt/nohatech/seafile-server-latest/seaf-gc.sh -r

 echo Giving the server some time....
 sleep 3

 # start the server again
 echo Starting the Seafile-Server...
 systemctl start seafile.service

 echo Seafile cleanup done!
```
Make sure that the script has been given execution rights, to do that run this command
```
 sudo chmod +x /opt/nohatech/seafile/cleanupScript.sh
```
Now we need to add the script to crontab so we can autorun it.
```
 sudo nano crontab -e
```
Then add the following line at the end on the crontab file.
```
 0 2 * * Sun /opt/nohatech/seafile/cleanupScript.sh
```
This means that every Sunday at 02:00 this script will run.

### Install NGINX
We need NGINX so we can access Seafile trough 443 port and use SSL also NGINX are going to work as a revers proxy for us.
Before we start we need to stop Seafile.
```
 cd /opt/nohatech/seafile-server-latest/
 ./seafile.sh stop
 ./seahub.sh stop
```
Now we are ready to go!

#### Latest version

#### Install

#### Configuration

#### Self signed cert

#### Free SSL cert from Let's Encrypt (recommended)

### Configuration for Seafile

#### ccnet.conf

#### seafile.conf

#### seahub_settings.py

# UFW Firewall
A Firewall is something that everyone wants and needs these days, so I'm going to guide you trough it.
As default Ubuntu should have UFW installed but if not, then installed it trough this command.
```
 sudo apt-get install ufw
```
The ports that are needed for Seafile to work is 80 and 443, but we also want the SSH port to be opened and it's port 22 as standard, if you have change it by following the NohaTech-Ubuntu-secure-server.md document you just change it from 22 to the port number that you have choosed for it.
So let us open the ports.
```
 sudo ufw allow 80/tcp
 sudo ufw allow 443/tcp
 sudo ufw allow 22/tcp
```
And we also need to make some default rules, what we going to do is block all incoming ports that we haven't opend, and open every outgoing port as we want to send traffic and we can't get hacked by the traffic that we are sending out.
```
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
```
And before we going to enable the UFW firewall we need to make sure that every port that we want to have open is open so we don't lock our selfs out.
```
 sudo ufw status numbered
```
We are going to se that port 80,443,22/tcp are blocked both for IPv4 and IPv6 and that's normal.
Now we can enable the UFW firewall.
```
 sudo ufw enable
```
Now we are finished and you have a firewall installed and activated.

# Fail2Ban
Fail2Ban are searching trough your logfiles and are blocking IP's that are trying to hack your server in differnet ways.
First we need to install Fail2Ban.
```
 sudo apt-get install fail2ban
```
### Latest version
As everything in this document we are using the latest stable version of the software and Fail2Ban is no differnet, but as version 10 have some different rules etc. we will still be at version 9 as that's the most stable one.
So now, let us download and install the newest version.
```
 wget http://se.archive.ubuntu.com/ubuntu/pool/universe/f/fail2ban/fail2ban_0.9.7-2_all.deb
```
then we need to install it.
```
 sudo dpkg -i fail2ban_0.9.7-2_all.deb
```

Now we have the latest version installed and we can continue with the configuration of Fail2Ban.

### For Seafile

### For NGINX