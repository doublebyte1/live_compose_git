    /***************************************************************************************/

    88888888ba  88888888888        db        88888888ba,   88b           d88 88888888888  
    88      "8b 88                d88b       88      `"8b  888b         d888 88           
    88      ,8P 88               d8'`8b      88        `8b 88`8b       d8'88 88           
    88aaaaaa8P' 88aaaaa         d8'  `8b     88         88 88 `8b     d8' 88 88aaaaa      
    88""""88'   88"""""        d8YaaaaY8b    88         88 88  `8b   d8'  88 88"""""      
    88    `8b   88            d8""""""""8b   88         8P 88   `8b d8'   88 88           
    88     `8b  88           d8'        `8b  88      .a8P  88    `888'    88 88           
    88      `8b 88888888888 d8'          `8b 88888888Y"'   88     `8'     88 88888888888  

GENERAL INFORMATION
============
This is a MVP, that instantiates an SDI through a set of micro-services. The shipped services are:

* Tomcatv7/SSL support
* GNv3.0.4
* Geoserverv2.8.2
* PostgreSQLv9.3
* PostGISv2.1
* Newrelic-sysmond (monitoring)

While storage is persisted in **volumes** associated to a data-only container, the services are *disposable*, in the sense that it is relatively *cheap* to kill them and create them again.

Please refer to the [Dockerized Live wiki page](https://eos.geocat.net/redmine/projects/live/wiki/Dockerized_Live) for additional information about this system.

AUTHORS
=======
This work was originally created and maintained by joana.simoes@geocat.net

INSTALL
==========
REQUIREMENTS & INSTALLATION
------------
For supporting version '2' of the compose syntax, you will need docker-compose >= 1.6 and docker >= 1.10.0
Instructions for installing the docker engine on different OS and flavours can be found here:

https://docs.docker.com/engine/installation/

Instructions for installing docker-compose can be found here:

https://docs.docker.com/compose/install/

Note that if you are a non-Linux user, you will need to install docker-machine in addition to docker and compose.
You may check the requirements section first. After that, to build and run the system for the first time you want to go to the root directory and type:

_curl https://raw.githubusercontent.com/doublebyte1/live_compose_git/master/docker-compose.yml | docker-compose -f - up -d_

This will download and compile the necessary images, and start the micro-services.
Each service is wrapped into its own container: one for GeoNetwork, other for GeoServer and another for database. They are binded to the same host, and they are assigned fixed ports on startup.

You can stop the system, just by typing:

 _docker-compose stop_

After creating the containers, you can start the system at any time with:

 _docker-compose start_

**Note:** Before building the images, you must set some **environmental variables**:
* gitlab_user: GitLab user on Helios.
* gitlab_pass: GitLab password on Helios.
* GEOCAT_services_IP: IP address of SFTP server
* sftp_user: username of the SFTP server
* sftp_pass: password of the SFTP server
* RSYSMOND_license_key: NewRelic License key.

Data Container
-------------
As mentioned before, the _data volumes_ are dettached from the containers and mounted in a data container. This allows the micro-services containers to be stopped, restarted or killed, without any loss of data.
The volumes being exported include the data directories from GeoNetwork and GeoServer, as well as their PostgreSQL databases and the DB logs.

The data container also has an _house-keeping_ role, being responsable for the backups. For that purpose, it features a backup client (bacula-client), as well as a database client (psql), and it includes a number of backup scripts. 
This container also needs to interact with the docker daemon (in order to stop and start services during backups), and therefore it mounts the docker socket on the host.

Building Images with Different GeoNetwork and Geoserver files
-------------------------------------------------------------

Both GeoServer and GeoNetwork are setup to take as build arguments, the location and access credentials of an SFTP server and a filename. These arguments are specified in the compose file. During the build process, docker will access this server, retrieve the files and place them on the GeoNetwork or GeoServer containers. The defined values for these arguments point to the SFTP server running on GeoCat Services. You could place different files on this server, and ammend the filename variable; this would cause docker to use your files instead. You could even point docker to a different SFTP server, by modifying the server ip, username and password variables. In every case, bear in mind that __the files should be placed on the /share directory__ of the server.

Using GeoCat Live
----------------
When the system is up and running, you can access GeoNetwork on this address:

 https://[replace_me]:8441/geonetwork

Likewise you can access GeoServer on this address:

 https://[replace_me]:8442/geoserver

If you are running docker on Linux, just replace [replace_me] by _localhost_ (_e.g._: 127.0.0.1). Otherwise, replace it by the address of your docker machine. You can find this address, by typing:

 _docker-machine ip_

Since the certificates are bounded to geocat.net, you may want to avoid SSL errors by adding an entry on your hosts file (e.g.: /etc/hosts on Linux). Something like this would work:

[replace_me] sushi.geocat.net

If you want to use the backup client for jobs, you need to register the host on a bacula-server. We currently have some backup jobs configured on bacula-dir, running on helios.
You can read more about configuring and using this bacula-server instance, on this link:

https://eos.geocat.net/redmine/projects/live/wiki/Dockerized_Live#Backups

If you want to monitor the system, you will need a NewRelic key. Please sign up for a free account, to obtain a [license key](https://newrelic.com/signup). The docker images read the license key from an environmental variable, _NRSYSMOND_license_key_. To replace this key, please edit _common.env_, on the root of this folder, and replace the key value for your key.
After this, you will need to re-create the containers. Please read on, in order to to that.

You can read more about the monitoring system in place, on this link:

https://eos.geocat.net/redmine/projects/live/wiki/Dockerized_Live#Monitoring

Recreating Containers
=====================

For recreating a multi-container system, launched with docker compose, you must stop it first. Then you can remove the containers with:

 _docker-compose rm_
 
Or to do it in one go:

 _docker-compose stop && docker-compose rm -f_

And then build them again with:

 _docker-compose up_

Recreating Images
=================

For recreating images, you must remove each image individually with:

 _docker rmi [some-image]_

In order to remove an image, you must ensure *there are no containers running from this image*. If there are any containers instantiated from this image, you must stop & delete them first. 

Then you can rebuild the system with:

 _docker-compose up_

Copyright
========
Currently this product is licensed to GeoCat B.V.
