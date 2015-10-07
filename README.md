# Introduction
This docker container provides a JIRA installation. As far as possible it's configurable through environment variables. The initialization and setup scripts are largely inspired by the superb gitlab container from sameersbn.

# Versions
## Jira
Current Jira version: 6.4

## MySQL Driver
Current MySQL Driver Version: 5.1.34

# Hardware and Software Requirements
These are the official hardware and software requirements by Atlassian. They apply to the current version, but you should be fine running any of the older versions on this hardware.

## Software
### Client
#### Web Browser
| Browser                     | Supported Versions                                        | Notes                                     |
|-----------------------------|-----------------------------------------------------------|-------------------------------------------|
| Chrome                      | Latest stable version supported                           |                                           |
| Microsoft Internet Explorer | 9.0, 10.0, 11.0                                           | 'Compatibility View' is **not supported** |
| Mozilla Firefox             | Latest stable version supported                           |                                           |
| Safari                      | Latest stable version supported on Mac OS X only          |                                           |
| Mobile Safari               | iOS, iPod touch and iPhone only --- Latest stable version |                                           |
| Android                     | The default browser on Android 4.0.3 (Ice Cream Sandwich) |                                           |

### Server
#### Java
The correct Java version is already included in the container. Currently it runs on Java 1.8.0_40.

#### Application Server
Jira runs on Apache Tomcat.

#### Database
Jira supports most popular relational database servers. However, this container only supports MySQL. If you would like to run this container using any other database server, please let us know, so we might add the feature.

## Hardware
The hardware required to run Jira depends on a number of different JIRA configurations (eg. projects, issues, custom fields, permissions, etc.) as well as the maximum number of concurrent requests that the system will experience during peak hours. See the [Jira Sizing Guide](https://confluence.atlassian.com/display/ENTERPRISE/JIRA+Sizing+Guide) for more information.

At an absolute minimum you will need:

* Dual Core CPU
* 8GB RAM
* 10GB Disk Space

# Quickstart Guide
To quickly run Jira using a MySQL Docker container run these commands:

Start MySQL

```sudo docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=pw mysql
sudo docker exec -i -t mysql /bin/bash```

This will start a MySQL container and a bash terminal within. Now we can create the database scheme for Jira:

```mysql -u root -ppw
create database jira;
exit
exit```

Now we can run Jira. /opt/jira-home will be linked to a docker volume. In order for Jira to be able to write to that directory, we need to change the ownership accordingly. The easiest way to do this is with a jira-data container. Jira is - by default - run as the jira user with UID:GID 5000:5000.
The Jira container will be linked to the MySQL container. Jira will automatically detect this linked container and configure the database connection accordingly:

```sudo docker run -d -v /opt/jira-home --name jira-data busybox chown -R 5000:5000 /opt/jira-home
sudo docker run -p 8080:8080 -d --name jira --volumes-from jira-data --link mysql:mysql -e "DB_USER=root" -e "DB_PASS=pw" -e "DB_NAME=jira" quay.io/ukhomeofficedigital/docker-jira```

After about 1 minute you should be able to access and configure Jira on [http://localhost:8080](http://localhost:8080)

# Configuration
The following features can be configured through the use of environment variables.

## Data Storage
Jira stores it's information in /opt/jira-home. You should mount a volume from the host or another docker container (see example in Quickstart Guide above). It is recommended to backup the content of this volume on a regular basis.

Volumes can be mounted to a docker container with the -v (directory in host) or the --volumes-from (volume in another docker container) option.

## Database connection
Jira stores most of it's data in a database. Current this container only supports MySQL as database backend.

### External MySQL Server
The connection to an external database server can be configured using the following environmental variables:

| Variable | Description                                   | Default         | Example                 |
|----------|-----------------------------------------------|-----------------|-------------------------|
| DB_TYPE  | One of: mysql                                 |                 | mysql                   |
| DB_HOST  | Hostname or IP Address of the database server |                 | localhost, 123.456.78.9 |
| DB_PORT  | Port on which the database server listens     |                 | 3306                    |
| DB_NAME  | Name of the database for Jira                 | jira_production | jira_production         |
| DB_USER  | User with which to connect to the database    | jira            | jira                    |
| DB_PASS  | Password for the user DB_USER                 |                 | some_very_secret_value  |
| DB_POOL  | Number of database connections to open        | 20              | 30                      |

### Linked MySQL Container
If the container detects a linked mysql container with name "mysql" it will use that database connection. The variables DB_TYPE, DB_HOST and DB_PORT will be set to the corresponding values. MySQL needs to run on the default MySQL port 3306 in the linked container.

## Jira User
The UID and GID of the user jira can be configured through USERMAP_UID und USERMAP_GID.

## Reverse Proxy
The container supports the use of SSL and non SSL reverse proxies. When configuring a reverse proxy, both name and port are required. It is invalid to provide both SSL and non SSL proxy configuration. The context path of the reverse proxy may only be provided if a reverse proxy was configured. The context path variable is optional.

The reverse proxy name should be the FQDN of Jira's base url.

### SSL
SSL reverse proxy can be configured with SSL_REVERSE_PROXY_NAME and SSL_REVERSE_PROXY_PORT

For example, if your Jira installation is reachable through https://example.com:8443/jira the variables should be set to:

SSL_REVERSE_PROXY_NAME=example.com
SSL_REVERSE_PROXY_PORT=8443

### non SSL
Non SSL reverse proxy can be configured with REVERSE_PROXY_NAME and REVERSE_PROXY_PORT

For example, if your Jira installation is reachable through http://example.com:8080/jira the variables should be set to:

SSL_REVERSE_PROXY_NAME=example.com
SSL_REVERSE_PROXY_PORT=8080

### Context Path
The context path of the reverse proxy can be configured with REVERSE_PROXY_CONTEXT_PATH. It should start with a `/`.

In the examples above, REVERSE_PROXY_CONTEXT_PATH should be set to `/jira`.

# Quick Reference
* **DB_TYPE**: The database type. Possible values: `mysql`
* **DB_HOST**: The database server hostname or IP address
* **DB_PORT**: The database server port
* **DB_NAME**: The database name. Defaults to `jira_production`
* **DB_USER**: The database user. Defaults to `jira`
* **DB_PASS**: The database user's password
* **DB_POOL**: The size of the database connection pool. Defaults to 20
* **USERMAP_UID**: The UID of the user jira. Defaults to 5000
* **USERMAP_GID**: The GID of the user jira. Defaults to 5000
* **SSL_REVERSE_PROXY_NAME**: The FQDN of the reverse proxy
* **SSL_REVERSE_PROXY_PORT**: The port on which the reverse proxy accepts connections for Jira
* **REVERSE_PROXY_NAME**: The FQDN of the reverse proxy
* **REVERSE_PROXY_PORT**: The port on which the reverse proxy accepts connections for Jira
* **REVERSE_PROXY_CONTEXT_PATH**: The context path of Jira's base URL

