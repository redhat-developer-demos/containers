# Apache HTTPD examples

## An Apache HTTPD example based on the Red Hat Universal Base Image
This example assumes you have already done your webpack builds and moves over your static content

### OpenShift specifics
* Adding user 1001 at the end (default UBI behaviour but specificity is good)
* Setting permissions on working folder
* Copy over conf file for your configuration

### Example from base UBI 

```
FROM registry.redhat.io/ubi8
COPY build/ /var/www/html/

RUN yum update --disableplugin=subscription-manager -y && rm -rf /var/cache/yum
RUN yum install --disableplugin=subscription-manager httpd -y && rm -rf /var/cache/yum

COPY conf/httpd.conf /etc/httpd/conf/httpd.conf

RUN chgrp -R 0 /var/log/httpd /var/run/httpd /etc/httpd /var/www/html \
  && chmod -R g=u /var/log/httpd /var/run/httpd /etc/httpd /var/www/html 

EXPOSE 8080
USER 1001
CMD ["-D","FOREGROUND"]
ENTRYPOINT ["/usr/sbin/httpd"]

```

Building the container
```bash
buildah bud -f Containerfile -t httpd:latest

```

running the container
```bash
podman run -p 12000:8080 httpd:latest

curl localhost:12000
```


