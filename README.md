# OpenShift Container Cheat Sheet

The purpose of this repository is to create a single repository describing OpenShift container best practices with some ready examples for at least the following set of containers:

- [Node.js](nodejs)
- Spring Boot
- Quarkus
- [Python](python)
- PHP
- [GO](go)
- [C# Dotnet Core](dotnet)
- JBoss EAP
- WebSphere Liberty (now that Runtimes includes Liberty) 
- HTTPD example(apache_httpd)

## Creating Images for OpenShift

You can find some best practices documented at [openshift.com creating images section](https://docs.openshift.com/container-platform/4.5/openshift_images/create-images.html)

This is a **cheat sheet** so we will get right to the important bits. These two guidelines will solve most of your issues.

#### Non-Root containers: Permissions
OpenShift runs containers using arbitrarily assigned user IDs by default. This provides extra security if a process should excape the container by preventing escalated permissions on the host node.

Directories and files that may be written to by processes in the running image should be owned by the root group and be read/writable by that group. Files to be executed should also have group execute permissions.

The command to set this is:
```
RUN chgrp -R 0 /some/directory && \
    chmod -R g=u /some/directory
```

See example [Python](https://raw.githubusercontent.com/redhat-developer-demos/containers/master/python/flask/Dockerfile)

Alternatively you can use the Red Hat language specific UBI images as per the [Go Example](https://raw.githubusercontent.com/redhat-developer-demos/containers/master/go/ContainerFile) or [.net Example](https://raw.githubusercontent.com/redhat-developer-demos/containers/master/dotnet/ContainerFile), these have the right permissions set if you follow the conventions.

There are some caveats for sensitive areas as per normal Linux system e.g. /etc/passwd.

### Non-Root Containers: Ports
Processes running in the container must not listen on privileged ports (ports below 1024), since they are not running as a privileged user.

## Questions to answer

- Who is looking for this content? 
Developers who wishes to build containers to run on Red Hat provided Kubernetes following good practices.

- What problem(s) are the developers who need this content trying to solve? There are loads off bad examples on the internet that causes the 'it worked on my laptop' problem. We believe this should not be an issue in the container world.

- What pain points are they trying to address? Provide simple easy copy & paste examples that follow good practices.

## What will you find here

Samples container build files that include

- Documentation
- Test and usage instructions
- Highlight differences between OCP and non-best practice builds


## Quick Tips


Testing container builds against your OCP clusters (in my case CRC). Assumes you are logged in. 
```bash
oc new-build --name mybuild --binary --strategy docker
oc start-build mybuild --from-dir=.

oc logs -f build/mybuild-1   

# Verify your image exists
oc new-app --search mybuild

# Deploy your image
oc new-app mybuild

#Verifty image
oc get pods

# Get app details, use service to expose route
oc status

oc expose svc/mybuild

# This will show route
oc status

# in our example
curl http://mybuild-test.apps-crc.testing

```

You can access the internal registry after logging into OCP
```bash
oc login
podman login -u $(oc whoami) -p $(oc whoami -t) default-route-openshift-image-registry.apps-crc.testing --tls-verify=false

podman tag localhost/imagename  default-route-openshift-image-registry.apps-crc.testing/[yourproject]/imagename
podman push default-route-openshift-image-registry.apps-crc.testing/[yourproject]/imagename --tls-verify=false

```

# podman and buildah: We encourage buildah
podman uses buildah underneath for builds (same code logic), but with buildah developers have extra tools for building images, is more flexible by allowing developers not only to depend on a Dockerfile, but source files, and even bash scripts.