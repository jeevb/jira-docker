# SSL certificates

Proper certificates will have to be placed in this directory before building the Docker images. To generate a self-signed certificate:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -sha256 -keyout jira.key -out jira.crt
```
