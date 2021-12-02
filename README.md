# redis6-proxy-docker

> Docker部署redis6 cluster以及redis-proxy

## 准备

```
mkdir -p /opt/redis/ /opt/redis/data /opt/redis/data/6001 /opt/redis/data/6002 /opt/redis/data/6003 /opt/redis/etc /opt/redis/run
```

## /opt/redis/etc/6001.conf

```
protected-mode no
daemonize no
port 6001
cluster-enabled yes
cluster-config-file /opt/redis/etc/nodes-6001.conf
pidfile /opt/redis/run/nodes-6001.pid
cluster-node-timeout 15000
```

## /opt/redis/etc/6002.conf

```
protected-mode no
daemonize no
port 6002
cluster-enabled yes
cluster-config-file /opt/redis/etc/nodes-6002.conf
pidfile /opt/redis/run/nodes-6002.pid
cluster-node-timeout 15000
```

## /opt/redis/etc/6003.conf

```
protected-mode no
daemonize no
port 6003
cluster-enabled yes
cluster-config-file /opt/redis/etc/nodes-6003.conf
pidfile /opt/redis/run/nodes-6003.pid
cluster-node-timeout 15000
```

## /opt/redis/etc/proxy.conf

```
cluster 172.22.0.61:6001
cluster 172.22.0.62:6002
cluster 172.22.0.63:6003

port 7001
threads 8
connections-pool-size 10
connections-pool-min-size 10
daemonize no
enable-cross-slot yes
```


## docker-compose.yml

```
version: "3"
services:
  redis_proxy:
    image: kornrunner/redis-cluster-proxy:latest
    network_mode: bridge
    networks:
      redis_network:
        ipv4_address: 172.22.0.71
    ports:
      - 7001:7001
    volumes:
      - /opt/redis/etc/:/opt/redis/etc
    container_name: redis_proxy
    depends_on:
      - redis_6001
      - redis_6002
      - redis_6003
    entrypoint:
      - "/usr/local/bin/redis-cluster-proxy"
      - "-c"
      - "/opt/redis/etc/proxy.conf"

  redis_6001:
    image: redis:6.2.6
    network_mode: bridge
    networks:
      redis_network:
        ipv4_address: 172.22.0.61
    ports:
      - 6001:6001
    volumes:
      - /opt/redis/etc/:/opt/redis/etc
      - /opt/redis/run/:/opt/redis/run
      - /opt/redis/data/6001:/data
    container_name: redis_6001
    entrypoint:
      - redis-server
      - /opt/redis/etc/6001.conf

  redis_6002:
    image: redis:6.2.6
    network_mode: bridge
    networks:
      redis_network:
        ipv4_address: 172.22.0.62
    ports:
      - 6002:6002
    volumes:
      - /opt/redis/etc/:/opt/redis/etc
      - /opt/redis/run/:/opt/redis/run
      - /opt/redis/data/6002:/data
    container_name: redis_6002
    entrypoint:
      - redis-server
      - /opt/redis/etc/6002.conf

  redis_6003:
    image: redis:6.2.6
    network_mode: bridge
    networks:
      redis_network:
        ipv4_address: 172.22.0.63
    ports:
      - 6003:6003
    volumes:
      - /opt/redis/etc/:/opt/redis/etc
      - /opt/redis/run/:/opt/redis/run
      - /opt/redis/data/6003:/data
    container_name: redis_6003
    entrypoint:
      - redis-server
      - /opt/redis/etc/6003.conf

networks:
  redis_network:
    name: redis_network
    ipam:
      driver: default
      config:
        - subnet: "172.22.0.0/24"
```

## 组建cluster

```
redis-cli --cluster create 172.22.0.61:6001 172.22.0.62:6002 172.22.0.63:6003
```

