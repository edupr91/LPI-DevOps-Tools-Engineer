# Setup your custom image
Run the following to create your Dockerfile
```bash
~]$ cat <<EOF > Dockerfile
# set base image
FROM debian:latest
# Set main maintainer
MAINTAINER usuario@correo.es
# Update and Install apache
RUN apt-get update && apt-get install -y locales locales-all apache2
RUN locale-gen es_ES.UTF-8
# Set that twe will expose the port 80
EXPOSE 80
VOLUME /var/www
# Apache2 variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_LOG_DIR /var/log/apache
RUN mkdir -p \$APACHE_RUN_DIR \$APACHE_LOCK_DIR \$APACHE_LOG_DIR
# We content
COPY index.html /var/www/html
CMD ["/usr/sbin/apache2", "-D", "FOREGROUND"]

EOF
```

---

Create a `index.html` file, replace the text with whatever fits you best
```bash
~]$ cat <<EOF > index.html
Hello IT
Have you tried turning it off and on again...?
EOF
```
---

Create the image
```bash
~]$ docker build --rm --no-cache --pull -t debian_apache2 .
## <- there are some missing lines. This is just to make my point
Successfully built 3f60df14d330
Successfully tagged debian_apache2:latest
```

Now with a `docker images` we can see that we have the debian_apache2 image created a few seconds ago.
```bash
~]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian_apache2      latest              3f60df14d330        15 seconds ago      484MB
```

Using the new image
```bash
~]$ docker run -dtiP --name web debian_apache2
0c0b7ca3554069899f8b4f6bde751ca9b7f563e0ca4e76c89a7c555581e09721
# lets find out which port has been exposed
~]$ docker ps -l
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                   NAMES
0c0b7ca35540        debian_apache2      "/usr/sbin/apache2 -…"   About a minute ago   Up About a minute   0.0.0.0:32768->80/tcp   web
~]$ curl localhost:32768
Hello IT
Have you tried turning it off and on again...?
~]$
```

Change the `index.html` and create new image version (`1.2`)
```bash
~]$ cat <<EOF > index.html
I like being weird. Weird's all i got
EOF
~]$ docker build --rm --no-cache --pull -t debian_apache2:1.2 .
~]$ docker run -dtiP --name web2 debian_apache2:1.2
16f3255cfac8f88a1c0ae7b6435fba922cbb07ecc39263ca512735a93b3a6020
~]$ docker ps -l
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                   NAMES
16f3255cfac8        debian_apache2:1.2   "/usr/sbin/apache2 -…"   10 seconds ago      Up 9 seconds        0.0.0.0:32769->80/tcp   web2
~]$ curl localhost:32769
I like being weird. Weird's all i got
```
