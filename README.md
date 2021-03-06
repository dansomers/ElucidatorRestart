# ElucidatorRestart
Elucidator Restart Service

![alt text](https://github.com/Duke-GCB/ElucidatorRestart/raw/master/docs/FirstScreen.png "First Screen")
__User Clicks "Submit Restart Request"__
![alt text](https://github.com/Duke-GCB/ElucidatorRestart/raw/master/docs/SecondScreen.png "Second Screen")

This screen will display __Restart already scheduled.__ if a user has already requested a restart that hasn't completed yet.

## Installation
Assumes apache is already setup and running on CentOS and will be secured by Duke Shiboleth/Group Manager.
Apache should have https setup. 

### Install Shiboleth
Create /etc/yum.repos.d/duke-el-shib2.repo with the following content:
```
[duke-el-shib2]
name=Shibboleth2
baseurl=http://download.opensuse.org/repositories/security://shibboleth/CentOS_5/
enabled=1
gpgcheck=0
priority=1
```
For CentOS 6 use this URL:
```
baseurl=http://download.opensuse.org/repositories/security://shibboleth/CentOS_CentOS-6/
```
Change the baseurl for the particular version we will use.

Install package:
```
yum install shibboleth.x86_64
```

### Setup Shiboleth
Download duke shiboleth config files.
```
cd /etc/shiboleth
wget https://shib.oit.duke.edu/duke-metadata-2-signed.xml
wget https://shib.oit.duke.edu/idp_signing.crt
cp <this repo directory>/etc/shibboleth/shibboleth2.xml /etc/shibboleth/shibboleth2.xml
```

Generate certs.
```
cd /etc/shiboleth
keygen.sh
```
Update SP with value from sp-cert.pem


### Setup Apache
Turn on UseCanonicalName in /etc/httpd/conf/httpd.conf
```
UseCanonicalName On
```

Copy elrestart.conf into apache conf.d directory.
```
cd <this repo directory>
cp etc/httpd/conf.d/elrestart.conf /etc/httpd/conf.d/elrestart.conf

```

Copy CGI script into apache cgi-bin directory
```
cd <this repo directory>
cp var/www/cgi-bin/elucidator-restart.sh /var/www/cgi-bin/elucidator-restart.sh
```

### Restart Services
```
service httpd restart && service shibd restart
```

### Setup data directory structure
Directory Structure
```
elrestart/
  restart.log  # owned by apache and written to by apache and root(cron job)
  watched/  # directory owned by apache
    restart_elucidator.txt # flag file - created by apache and deleted by root(cron job)
   
```
Create directories
```
# create base directory that will hold the restart.log
mkdir /elrestart
# create apache owned subdirectory that will hold the restart_elucidator.txt flag file
# This directory will be apache owned.  
mkdir /elrestart/watched/
```


### Setup cron job
As root (since that user has permissions to restart the service) add a cron job.
```
crontab -e
```
Example that runs every 5 minutes:
```
*/5 * * * * <this repo directory>/check_restart.sh
```

