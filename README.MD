# Splunk for Docker with Solu Apps

#### Table of Contents
[Creating Persistent Data Storage](#creating-persistent-data-storage)<br>
[Upload Addons to the persistent storage](#upload-dddons-to-the-persistent-storage)<br>
[Create Splunk Docker environment](#create-splunk-docker-environment)<br>
[Docker DNS Problems FIX](#docker-dns-problems-fix)<br>


This guide will help me and others to remember how to deploy Splunk with lightning speed!

 Based on Splunks [article.](https://splunk.github.io/docker-splunk)

Requirements:
 - Docker Community Engine 17.06.2 or later
 - docker-compose

This setup comes with pre-installed apps which are used in Solution Center:
 - [Machine Learning Toolkit](https://splunkbase.splunk.com/app/2890/)
 - Nutanix
   - [TA for Prism](https://splunkbase.splunk.com/app/3103/)
   - [Nutanix Prism Dashboard for Splunk](https://splunkbase.splunk.com/app/3102/)
 - [Microsoft Teams message publication addon](https://splunkbase.splunk.com/app/4855/)
 - Tenable
   - [Tenable App](https://splunkbase.splunk.com/app/4061/)
   - [Tenable Add-on](https://splunkbase.splunk.com/app/4060/)

### Creating Persistent Data Storage

Once you've got your Docker up and running, you need to create two volumes so your config and indexing data won't
vanish with your containers.

Commands to create volumes:

```
docker volume create splunk_splunk1-etc
docker volume create splunk_splunk1-var
docker volume ls
docker volume inspect VOLUME-NAME
```

### Upload Addons to the persistent storage

I've uploaded the apps to the /opt/splunk/etc/apps_packages folder, for splunk to install from this path.
Before spinning up Splunk add them to the location:

```
docker volume inspect splunk_splunk1-etc

[
    {
        "CreatedAt": "2020-04-17T03:32:13-04:00",
        "Driver": "local",
        "Labels": {
           "com.docker.compose.project": "splunk",
            "com.docker.compose.version": "1.25.4",
            "com.docker.compose.volume": "splunk1-etc"
        },
        "Mountpoint": "/var/lib/docker/volumes/splunk_splunk1-etc/_data",
        "Name": "splunk_splunk1-etc",
        "Options": null,
        "Scope": "local"
    }
]
```

Your path to the app packages folder will be the following:

```
ls /var/lib/docker/volumes/splunk_splunk1-etc/_data
```
Create the app_packages dir:
```
mkdir /var/lib/docker/volumes/splunk_splunk1-etc/_data/app_packages
```

Upload the contents of Splunk Add-ons/Apps here:

```
ls /var/lib/docker/volumes/splunk_splunk1-etc/_data/apps_packages
microsoft-teams-messages-publication-addon_106.tgz  splunk-machine-learning-toolkit_510.tgz  tenable-add-on-for-splunk_321.tgz
nutanix-prism-dashboard-for-splunk_200.tgz          ta-for-nutanix-prism_200.tgz             tenable-app-for-splunk_210.tgz
```

### Create Splunk Docker environment

For this use docker-compose to build the whole set up and rip it down if necessary.

Follow the steps to verify your all set:
```
docker-compose -v
ls
```
You should have the docker-compose.yml in this working directory to create the Splunk instance.

**Build the setup as containers**
```
docker-compose up
```
Setup another session to test, you will see how the creation of containers will go.
If all is set, you can tear it down and set it up again as a daemon.

```
docker-compose up -d
```

Enter your enviroment http://HOST-IP:8000

Happy Splunking!


### Docker - Allow Splunk to communicate outside

Check what interface docker is using, e.g. 'docker0'
```
ip link show
```
Check available firewalld zones, e.g. 'public'
```
sudo firewall-cmd --get-active-zones
```
Check what zone the docker interface it bound to, most likely 'no zone' yet
```
sudo firewall-cmd --get-zone-of-interface=docker0
```
So add the 'docker0' interface to the 'public' zone. Changes will be visible only after firewalld reload
```
sudo nmcli connection modify docker0 connection.zone public
```
Masquerading allows for docker ingress and egress (this is the juicy bit)
```
sudo firewall-cmd --zone=public --add-masquerade --permanent
```
Optional open required incoming ports (wasn't required in my tests)
```
sudo firewall-cmd --zone=public --add-port=443/tcp
```
Reload firewalld
```
sudo firewall-cmd --reload
```
Reload dockerd
```
sudo systemctl restart docker
```
 Test ping and DNS works:
```
docker run busybox ping -c 1 172.16.0.1
docker run busybox cat /etc/resolv.conf
docker run busybox ping -c 1 yourhost.local
```
