# NodeJS examples

## NodeJS sample from base Red Hat NodeJS Build Universal Base Image
The nodejs example is based on the NodeJS documentation as a starting point.
https://nodejs.org/en/docs/guides/nodejs-docker-webapp/

### OpenShift specifics
* Adding user 1001 at the end (default UBI behaviour but specificity is good)
* Setting permissions on working folder

### Example from base UBI 

```
FROM registry.redhat.io/ubi8/nodejs-12

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
RUN  mkdir app
COPY . .

RUN \
    chown -R 1001:0 /usr/src/app && \
    chgrp -R 0 /usr/src/app && \
    chmod -R g=u /usr/src/app

EXPOSE 8080

USER 1001

CMD [ "node", "server.js" ]

```

Building the container
```bash
buildah bud -t testnode -f Containerfile . 

```

running the container
```bash
podman run -p 12000:8080 testnode

curl localhost:12000
```


