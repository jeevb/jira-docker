# JIRA Docker Configuration

##### Getting Started
These instructions assume that Docker is already installed and running on the machine in question.

Clone the JIRA Docker Configuration to the machine:
```
$ git clone git@bitbucket.org:jeevb/jira-docker.git
```

Edit the following environment variables in `.env` using your favorite editor:
```
POSTGRES_DB=jiradb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

Add SSL certificates to `nginx/ssl/`. This should be easy as moving them to this folder. For testing purposes, generating self-signed certificates would suffice:
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -sha256 -keyout jira-docker/nginx/ssl/jira.key -out jira-docker/nginx/ssl/jira.crt
```
**NOTE: Do not use self-signed certificates in production.**

Generate a strong DHE parameter:
```
$ openssl dhparam -out jira-docker/nginx/ssl/dhparam.pem 4096
```

Build and run the containers in detached mode using `docker-compose`:
```
$ cd jira-docker
$ docker-compose up -d
```

Navigate to the JIRA instance at [https://jira.example.com](https://jira.example.com) on your favorite browser to start setting up.

##### Setting up the database
Choose the option to use a PostgreSQL database, and set the host to `postgres`. Leave the port unchanged. All other options will be as specified in the `.env` file.

##### Increase Available Memory
To keep JIRA as snappy as it can possibly be, it should be run with at least 2 cores and 4GB ram. `t2.medium` on AWS EC2 would be a good candidate for such a configuration. With more ram, more heap space can be allocated to the Java Virtual Machine (JVM) on which JIRA will be run. To do so,

Run the `admin` container using `docker-compose`:
```
$ docker-compose run --rm admin
```

Open up the required configuration file:
```
$ vim /opt/atlassian/jira/bin/setenv.sh
```

Edit the `JVM_MINIMUM_MEMORY` and `JVM_MAXIMUM_MEMORY` lines to look as follows:
```
JVM_MINIMUM_MEMORY="1024m"
JVM_MAXIMUM_MEMORY="2048m"
```

Restart the containers:
```
$ docker-compose restart
```

##### Mismatched URL Scheme
Since Nginx will be used as a reverse proxy server in front of JIRA, the following change will have to be made:

Run the `admin` container using `docker-compose`:
```
$ docker-compose run --rm admin
```

Open up JIRA server configuration file:
```
$ vim /opt/atlassian/jira/conf/server.xml
```

Add the following lines to the file within the `Connector` node:
```
...
    <Service name="Catalina">

        <Connector port="8080"

                   maxThreads="150"
                   minSpareThreads="25"
                   connectionTimeout="20000"

                   enableLookups="false"
                   maxHttpHeaderSize="8192"
                   protocol="HTTP/1.1"
                   useBodyEncodingForURI="true"
                   redirectPort="8443"
                   acceptCount="100"
                   disableUploadTimeout="true"
<!-- Lines added to solve the URL scheme mismatch -->
                   scheme="https"
                   proxyName="jira.example.com"
                   proxyPort="443"/>
...
```

Restart the containers:
```
$ docker-compose restart
```

### Todos

- Add instructions for [migrating from a JIRA cloud instance](https://confluence.atlassian.com/jira/migrating-from-jira-cloud-to-jira-server-299569790.html).
- Add support for backups to AWS S3.
