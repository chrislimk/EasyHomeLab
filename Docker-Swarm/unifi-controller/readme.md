# Create unifi-controller application

## Create container folders
```
mkdir -p /volume1/Docker/Unifi/unifi-app
mkdir -p /volume1/Docker/Unifi/unifi-mongo
```

## Create init-mongo.js
```
vi /volume1/Docker/Unifi/unifi-mongo/init-mongo.js
```
Paste and save
```
db.getSiblingDB("unifi").createUser({user: "unifi", pwd: "P@ssw0rd", roles: [{role: "dbOwner", db: "unifi"}]});
db.getSiblingDB("unifi_stat").createUser({user: "unifi", pwd: "P@ssw0rd", roles: [{role: "dbOwner", db: "unifi_stat"}]});
```

## Deploy stack
```
version: "3.2"
services:
  unifi-network-application:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi-network-application
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Singapore
      - MONGO_USER=unifi
      - MONGO_PASS=P%40ssw0rd #text replacement because @ is not supported
      - MONGO_HOST=unifi-db
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi
#      - MEM_LIMIT=1024 #optional
#      - MEM_STARTUP=1024 #optional
#      - MONGO_TLS= #optional
#      - MONGO_AUTHSOURCE= #optional
    deploy:
      placement:
        constraints:
          - "node.role==worker"
    volumes:
      - unifi_config:/config
    ports:
      - 8443:8443
      - 3478:3478/udp
      - 10001:10001/udp
      - 8081:8080
      - 8880:8880
#      - 1900:1900/udp #optional
#      - 8843:8843 #optional
#      - 8880:8880 #optional
#      - 6789:6789 #optional
#      - 5514:5514/udp #optional
    restart: unless-stopped
    depends_on:
    - unifi-db

  unifi-db:
    image: docker.io/mongo:4.4.27
    deploy:
      placement:
        constraints:
          - "node.role==worker"
    volumes:
      - unifi_mongo_config:/data/db
      - /volume1/Docker/Unifi/unifi-mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    restart: unless-stopped

# Bind Local volume
volumes:
  unifi_config:
    driver: local
    driver_opts:
      type: "none"
      o: "bind"
      device: "/volume1/Docker/Unifi/unifi-app"

  unifi_mongo_config:
    driver: local
    driver_opts:
      type: "none"
      o: "bind"
      device: "/volume1/Docker/Unifi/unifi-mongo"

# If using NFS
#volumes:
#  unifi_config:
#    driver: local
#    driver_opts:
#      type: nfs
#      o: addr=192.168.4.2,rw,soft
#      device: ":/mnt/SSD/Docker/unifi-network-app"

#  unifi_mongo_config:
#    driver: local
#    driver_opts:
#      type: nfs
#      o: addr=192.168.4.2,rw,soft
#      device: ":/mnt/SSD/Docker/unifi-mongo"
```
