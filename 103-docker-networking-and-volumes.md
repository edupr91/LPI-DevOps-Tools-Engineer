# Docker 103

## Working with networks
List, create and deploy a container within an specific network
```bash
# docker List networks
~]$ docker network  list
NETWORK ID          NAME                DRIVER              SCOPE
85d465ac83a8        bridge              bridge              local
6c2ddf563aee        host                host                local
0da7a5f9559b        none                null                local

# Create network
~]$ docker network create --subnet 192.168.100.0/24 --ip-range  192.168.100.100/30 --gateway 192.168.100.100 prod
dab18f6729282127c763413630728c55ed8caaf4476103c5f9068eedcdabe055

# Now you can see the new network listed
~]$ docker network  list
NETWORK ID          NAME                DRIVER              SCOPE
85d465ac83a8        bridge              bridge              local
6c2ddf563aee        host                host                local
0da7a5f9559b        none                null                local
dab18f672928        prod                bridge              local


# Deploy a container with a random IP within the new network
~]$ docker run -dti --network prod  --name web1 nginx

# Inspect the network settings
~]$ docker inspect -f "{{json .NetworkSettings.Networks.prod}}" web1 | jq
{
  "IPAMConfig": null,
  "Links": null,
  "Aliases": [
    "abe4d4e148bf"
  ],
  "NetworkID": "dab18f6729282127c763413630728c55ed8caaf4476103c5f9068eedcdabe055",
  "EndpointID": "6318c483a66840213cd4059214992559b5b290a3d2d1aede302eb0c0a988e1ae",
  "Gateway": "192.168.100.100",
  "IPAddress": "192.168.100.101",
  "IPPrefixLen": 24,
  "IPv6Gateway": "",
  "GlobalIPv6Address": "",
  "GlobalIPv6PrefixLen": 0,
  "MacAddress": "02:42:c0:a8:64:65",
  "DriverOpts": null
}

# Deploy a container with a specific IP within the new network
~]$ docker run -dti --network prod --ip 192.168.100.110 --name web2 nginx
84f1daaa0178ab101a3adf8176dd93c5f164ada14ed1d5e29ed8237f8b17d2ee

# Inspect the network settings
~]$ docker inspect -f "{{json .NetworkSettings.Networks.prod}}" web2 | jq
{
  "IPAMConfig": {
    "IPv4Address": "192.168.100.110"
  },
  "Links": null,
  "Aliases": [
    "84f1daaa0178"
  ],
  "NetworkID": "dab18f6729282127c763413630728c55ed8caaf4476103c5f9068eedcdabe055",
  "EndpointID": "fd7ed4183dc2bd8135a82c3c77da034ad5ce43565e1cc7bd594a1e8cf92434f3",
  "Gateway": "192.168.100.100",
  "IPAddress": "192.168.100.110",
  "IPPrefixLen": 24,
  "IPv6Gateway": "",
  "GlobalIPv6Address": "",
  "GlobalIPv6PrefixLen": 0,
  "MacAddress": "02:42:c0:a8:64:6e",
  "DriverOpts": null
}

```

---

## Volumes
Deploy nginx with your custom html folder
```bash
mkdir /var/www/html
~]$ cat <<EOF > /var/www/html/index.html
Hello IT
Have you tried turning it off and on again...?
EOF

# Carefull, HERE you need to set the full directory from your local machine
# docker run --name web3 -v /folder/full/path:/usr/share/nginx/html:ro -d nginx
~]$ docker run --name web3 -v /var/www/html:/usr/share/nginx/html:ro -d nginx

# Now check the <--
~]$ docker inspect -f "{{json .Mounts}}" web3| jq
[
  {
    "Type": "bind",
    "Source": "/home/vagrant/html", # <---
    "Destination": "/usr/share/nginx/html",
    "Mode": "ro",
    "RW": false,
    "Propagation": "rprivate"
  }
]
~]$ docker inspect -f "{{json .NetworkSettings.Networks }}" web3| jq
{
  "bridge": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": null,
    "NetworkID": "fd3598c8c3ef91bac172f3179cb28c72b62f69978ec8cfd2f3e94c15e9a3cca6",
    "EndpointID": "7627352629e2b31b92c0e7b0584e0469e9e6a5f1ebac04d5e069c87bde82eccc",
    "Gateway": "172.17.0.1",
    "IPAddress": "172.17.0.2", # <---
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:11:00:02",
    "DriverOpts": null
  }
}
# Final test
~]$ curl 172.17.0.2
Hello IT
Have you tried turning it off and on again...?

```

List volumes and interact with them....
```bash
~]$ docker volume list
DRIVER              VOLUME NAME
~]$ docker volume create --name web4_www
web4_www
~]$ docker volume list
DRIVER              VOLUME NAME
local               web4_www
~]$ docker volume inspect web4_www
[
    {
        "CreatedAt": "2020-11-12T17:50:24Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/web4_www/_data",
        "Name": "web4_www",
        "Options": {},
        "Scope": "local"
    }
]
## AS ROOT
~]# cat <<EOF > /var/lib/docker/volumes/web4_www/_data/index.html
It’s not magic, it’s talent and sweat.
Gilfoyle - Silicon Valley
EOF

# Now hwere we can just use the name of the volume (web4_www)
docker run --name web4 -v web4_www:/usr/share/nginx/html:ro -d nginx
# We asume that we will get the IP 172.17.0.3, just check with an inspect first....
~]$ curl 172.17.0.3
It’s not magic, it’s talent and sweat.
Gilfoyle - Silicon Valley
```


On the fly creates a volume my_vol and it gets mounted in /var/lib/my_vol

**NOTE:** it does weird things if you try to mount the folder in a non root directories
It must be smth like `/html`, `/app`, `/foo`, etc
```bash
~]$ docker run -d --name web5 --mount source=my_vol,target=/html nginx
~]$ docker volume inspect my_vol
[
    {
        "CreatedAt": "2020-11-12T18:09:04Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my_vol/_data",
        "Name": "my_vol",
        "Options": null,
        "Scope": "local"
    }
]
~]$ docker inspect -f "{{json .Mounts}}" web5| jq
[
  {
    "Type": "volume",
    "Name": "my_vol",
    "Source": "/var/lib/docker/volumes/my_vol/_data",
    "Destination": "/html",
    "Driver": "local",
    "Mode": "z",
    "RW": true,
    "Propagation": ""
  }
]

```

Connect two containers, MySQL & Wordpress
```bash
~]$ docker volume create vol_mysql
vol_mysql
~]$ docker inspect vol_mysql
[
    {
        "CreatedAt": "2020-11-12T18:19:14Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/vol_mysql/_data",
        "Name": "vol_mysql",
        "Options": {},
        "Scope": "local"
    }
]
~]$ docker run -dti --name wp_mysql -e MYSQL_ROOT_PASSWORD=Qwerty00 -v vol_mysql:/var/lib/mysql mysql:5.7
5dc6a4dbea3ddc5657c3c4858c1a3cc012efa79c8cd449091850b463b4f7f742
~]$ docker run -dti --name wp_web -p 8000:80 --link wp_mysql:mysql wordpress
ccfb3006dbb612a6dd6457ae8d88214bc1cb4cea2ccd820066ec8b239eb6b363
~]$ docker inspect -f "{{ json .HostConfig.Links }}" wp_web | jq
[
  "/wp_mysql:/wp_web/mysql"
]
~]$ curl --head localhost:8000
HTTP/1.1 302 Found
Date: Thu, 12 Nov 2020 18:28:13 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.4.12
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
X-Redirect-By: WordPress
Location: http://localhost:8000/wp-admin/install.php # <---
Content-Type: text/html; charset=UTF-8

```

To check environment variables and networking defined in an specific container:
```bash
~]$ docker exec -ti wp_mysql env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=5dc6a4dbea3d
TERM=xterm
MYSQL_ROOT_PASSWORD=Qwerty00
GOSU_VERSION=1.12
MYSQL_MAJOR=5.7
MYSQL_VERSION=5.7.32-1debian10
HOME=/root

~]$ docker exec -ti wp_web cat /etc/hosts
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.4  mysql 5dc6a4dbea3d wp_mysql
172.17.0.6  ccfb3006dbb6

~]$ docker exec -ti wp_web env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=ccfb3006dbb6
TERM=xterm
MYSQL_PORT=tcp://172.17.0.4:3306
MYSQL_PORT_3306_TCP=tcp://172.17.0.4:3306
MYSQL_PORT_3306_TCP_ADDR=172.17.0.4
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_PORT_33060_TCP=tcp://172.17.0.4:33060
MYSQL_PORT_33060_TCP_ADDR=172.17.0.4
MYSQL_PORT_33060_TCP_PORT=33060
MYSQL_PORT_33060_TCP_PROTO=tcp
MYSQL_NAME=/wp_web/mysql
MYSQL_ENV_MYSQL_ROOT_PASSWORD=Qwerty00
MYSQL_ENV_GOSU_VERSION=1.12
MYSQL_ENV_MYSQL_MAJOR=5.7
MYSQL_ENV_MYSQL_VERSION=5.7.32-1debian10
PHPIZE_DEPS=autoconf    dpkg-dev    file    g++     gcc     libc-dev    make  pkg-config    re2c
PHP_INI_DIR=/usr/local/etc/php
APACHE_CONFDIR=/etc/apache2
APACHE_ENVVARS=/etc/apache2/envvars
PHP_EXTRA_BUILD_DEPS=apache2-dev
PHP_EXTRA_CONFIGURE_ARGS=--with-apxs2 --disable-cgi
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_LDFLAGS=-Wl,-O1 -pie
GPG_KEYS=42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312
PHP_VERSION=7.4.12
PHP_URL=https://www.php.net/distributions/php-7.4.12.tar.xz
PHP_ASC_URL=https://www.php.net/distributions/php-7.4.12.tar.xz.asc
PHP_SHA256=e82d2bcead05255f6b7d2ff4e2561bc334204955820cabc2457b5239fde96b76
WORDPRESS_VERSION=5.5.3
WORDPRESS_SHA1=61015720c679a6cbf9ad51701f0f3fedb51b3273
HOME=/root
```

