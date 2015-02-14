# docker-mariadb-10.0-galera
Multi Master Replication using MariaDB 10.0 and Galera inside Docker.

Requires at least 3 nodes.

Let's say server1, server2, server3. You will have 2 data containers (data+config) and 1 container running MariaDB and Galera on each hosts.

# 1 - Config and data containers
### Server1
##### Make a config container that will be accessible from the host and the container.
```
docker run --name server1-mariadb-config -v /var/configs/mariadb/conf.d:/etc/mysql/conf.d busybox true
                                            ^^^^^^                      ^^^^^^
                                            Host directory               Container directory
```

##### Make a data container that will be accessible from the host and the container.
```
docker run --name server1-mariadb-config -v /var/data/mariadb:/data busybox true
                                            ^^^^^^^                      ^^^^^^
                                            Host directory               Container directory
```

##### 


### Server2
##### Config container
```
docker run --name server2-mariadb-config -v /var/configs/mariadb/conf.d:/etc/mysql/conf.d busybox true
```

##### Data container
```
docker run --name server2-mariadb-config -v /var/data/mariadb:/data busybox true
```

### Server3
##### Config container
```
docker run --name server3-mariadb-config -v /var/configs/mariadb/conf.d:/etc/mysql/conf.d busybox true
```

##### Data container
```
docker run --name server3-mariadb-config -v /var/data/mariadb:/data busybox true
```

# 2 - Initial startup

Start the first server with :
```
docker run -d --net=host --privileged=true --volumes-from server1-mariadb-config --volumes-from server1-mariadb-data --name mariadb-srv-1 factual/mariadb-galera /bin/start new
```

Start server2 and server 3 with :
```
# server 2
docker run -d --net=host --privileged=true --volumes-from server2-mariadb-config --volumes-from server2-mariadb-data --name mariadb-srv-2 factual/mariadb-galera /bin/start new

# server 3
docker run -d --net=host --privileged=true --volumes-from server3-mariadb-config --volumes-from server3-mariadb-data --name mariadb-srv-3 factual/mariadb-galera /bin/start new
```

# 3 - Restart server1 in "node mode"
It is very important to restart the first node just like the other. Otherwise if you stop and start your container, you will create a new cluster each time.

```
docker stop mariadb-srv-1
docker rm mariadb-srv-1
docker run -d --net=host --privileged=true --volumes-from server1-mariadb-config --volumes-from server1-mariadb-data --name mariadb-srv-1 factual/mariadb-galera /bin/start node
```

# 4 - Debug
If anything goes wrong, you can always debug via the error.log
```
tail -f /var/data/mariadb/error.log
```
