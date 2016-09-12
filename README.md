WebLogic and Apache Docker Images
=====

The steps below will build Docker images to run Apache and WebLogic 12.1.3

Build a Docker image containing Oracle JDK (Server JRE specifically):

# Building base images

The first base image to build is the Oracle JDK image. Dockerfile for this particular image is stored on `OracleJDK` folder.

## Java 8
[Download Server JRE 8](http://www.oracle.com/technetwork/java/javase/downloads/server-jre8-downloads-2133154.html) `.tar.gz` file and drop it inside folder `java-8`.

Build it:

```
$ cd java-8
$ docker build -t oracle/jdk:8 .
```

## WebLogic
**IMPORTANT:** you have to download the binary of WebLogic and put it in place (see `.download` files inside **12.1.3** folder). Then go into the **OracleWebLogic** folder and run the `buildDockerImage.sh` script as root.

```
$ sh buildDockerImage.sh -v 12.1.3 -d
```

First make sure you have built **oracle/weblogic:12.1.3-developer**. Then execute the build using the command below. The password must be at least 8 alphanumeric characters with at least one number or special character.

```
$ cd 1213-domain
$ docker build -t 1213-domain --build-arg ADMIN_PASSWORD=<define> .
```

To start the Admin Server, run:

```
$ docker run -d --name wlsadmin --hostname wlsadmin -p 8001:8001 1213-domain
```

**(Optional Step if domain mode is required)** To start a Managed Server to self-register with the Admin Server above, run:

```
$ docker run -d --link wlsadmin:wlsadmin -p 7001:7001 1213-domain createServer.sh
```

After create the domain image, the application need to get deployed using another layer using on top of the the image **1213-domain**. Move to the folder **1213-appdeploy** and execute:

```
$ docker build -t 1213-appdeploy .
```

```
$ docker run -d -p 8001:8001 1213-appdeploy
```

To access the sample application, go to http://localhost:8001/sample

## Apache

For Apache we will use the image provided by Red Hat registry. To execute that you must run in a host with RHEL already registered on RHN with entitlements. To know more check the `subscription-manager` command. Then pull the image and execute:

```
$ docker pull rhscl/httpd-24-rhel7
```

And then execute the container.

```
$ docker run -d -p 80:80 registry.access.redhat.com/rhscl/httpd-24-rhel7
```


## Final images

This is how the images should looks like at the end of all Docker image builds.

```
REPOSITORY                                        TAG                 IMAGE ID            CREATED             SIZE
1213-appdeploy                                    latest              d482a3c3b771        22 hours ago        1.299 GB
1213-domain                                       latest              3ff2a8c5b943        2 days ago          1.299 GB
oracle/weblogic                                   12.1.3-developer    53e95bd4c42a        2 days ago          1.297 GB
oracle/jdk                                        8                   fdad3941a423        2 days ago          592.4 MB
registry.access.redhat.com/rhscl/httpd-24-rhel7   latest              306b915e136e        6 weeks ago         439.9 MB
registry.access.redhat.com/rhel7                  latest              1988eb5b3fc6        7 weeks ago         278.2 MB
```
