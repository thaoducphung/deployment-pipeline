
# Migrating to docker-compose.yml #

Even with a simple image, we've already been dealing with plenty of command line options in both building, pushing and running the image.
 
Next we will switch to a tool called docker-compose to manage these. 

docker-compose is designed to simplify running multi-container applications to using a single command.

In the folder where we have our Dockerfile with the following content:

```dockerfile
FROM ubuntu:18.04

WORKDIR /mydir

RUN apt-get update
RUN apt-get install -y curl python
RUN curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
RUN chmod a+x /usr/local/bin/youtube-dl

ENV LC_ALL=C.UTF-8

ENTRYPOINT ["/usr/local/bin/youtube-dl"]
```

we create a file called `docker-compose.yml`:

```yaml
version: '3' 

services: 
    youtube-dl-ubuntu:  
      image: <username>/<repositoryname>
      build: . 
``` 

The version setting is not very strict, it just needs to be above 2 because otherwise the syntax is significantly different. See <https://docs.docker.com/compose/compose-file/> for more info. The key `build:` value can be set to a path (ubuntu), have an object with `context` and `dockerfile` keys or reference a `url of a git repository`.

Now we can build and push with just these commands: 

```console
$ docker-compose build
$ docker-compose push
```

## Volumes in docker-compose ##

To run the image as we did previously, we will need to add the volume bind mounts. Volumes in docker-compose are defined with the following syntax `location-in-host:location-in-container`. Compose can work without an absolute path:

```yaml
version: '3.5' 

services: 

    youtube-dl-ubuntu: 
      image: <username>/<repositoryname> 
      build: . 
      volumes: 
        - .:/mydir
      container_name: youtube-dl
``` 

We can also give the container a name it will use when running with container_name. And the service name can be used to run it: 

```console
$ docker-compose run youtube-dl-ubuntu https://imgur.com/JY5tHqr
```

{% include_relative exercises/2_1.html %}

## Web services in docker-compose ##

Compose is really meant for running web services, so let's move from simple binary wrappers to running a HTTP service. 

<https://github.com/jwilder/whoami> is a simple service that prints the current container id (hostname). 

```console
$ docker container run -d -p 8000:8000 jwilder/whoami 
  736ab83847bb12dddd8b09969433f3a02d64d5b0be48f7a5c59a594e3a6a3541 
```

Navigate with a browser or curl to localhost:8000, they both will answer with the id. 

Take down the container so that it's not blocking port 8000.

```console
$ docker container stop 736ab83847bb
$ docker container rm 736ab83847bb  
```

Let's create a new folder and a docker-compose file `whoami/docker-compose.yml` from the command line options.

```yaml
version: '3.5'  

services: 
    whoami: 
      image: jwilder/whoami 
      ports: 
        - 8000:8000 
``` 

Test it: 

```console
$ docker-compose up -d 
$ curl localhost:8000 
```

Environment variables can also be given to the containers in docker-compose.

```yaml
version: '3.5'

services:
  backend:
      image:  
      environment:
        - VARIABLE=VALUE
        - VARIABLE 
```

{% include_relative exercises/2_2.html %}
{% include_relative exercises/2_3.html %}
