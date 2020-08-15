# PHP examples

## PHP example based on the UBI image
Based on this example from developers.redhat.com: https://developers.redhat.com/blog/2020/03/24/red-hat-universal-base-images-for-docker-users/

### OpenShift specifics
* Adding user 1001 at the end (default UBI behaviour but specificity is good)
* Setting permissions on working folder
* Configures httpd.conf change to port 8080 as we are not running as root
* Extra sed to change www.conf file # to ; as # is deprecated
* Added LoadModules lines to rever to older PHP behaviour for a single development container (not best practice for production)

### Sample file

```
FROM registry.access.redhat.com/ubi8/ubi:8.1

RUN microdnf -y install httpd php \
  && microdnf clean all

ADD index.php /var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf \
  sed -i 's/"#"/";"/' /etc/php-fpm.d/www.conf \
  && mkdir /run/php-fpm \
  && chgrp -R 0 /var/log/httpd /var/run/httpd /run/php-fpm \
  && chmod -R g=u /var/log/httpd /var/run/httpd /run/php-fpm

RUN sed -i 's/\#/\;/' /etc/php-fpm.d/www.conf

RUN echo 'LoadModule mpm_prefork_module modules/mod_mpm_prefork.so' > /etc/httpd/conf.modules.d/00-mpm.conf
RUN sed -i 's|# LoadModule php7_module modules/libphp7-zts.so|LoadModule php7_module modules/libphp7-zts.so|' /etc/httpd/conf.modules.d/15-php.conf


EXPOSE 8080
USER 1001
CMD php-fpm & httpd -D FOREGROUND
```


Building the container
```bash
buildah bud -t testphp -f Containerfile . 

```

running the container
```bash
podman run -p 12000:8080 localhost/testphp

curl localhost:12000
```
