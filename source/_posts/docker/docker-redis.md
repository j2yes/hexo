---
title: docker redis clustering
categories:
  - docker
tags:
  - docker
  - container
  - swarm
  - redis
  - sentinel
date: 2019-06-29 16:00:28
---


docker를 이용한 redis sentinel clustering을 구성

# redis clustering

- tutorial : https://redis.io/topics/cluster-tutorial
- spec : https://redis.io/topics/cluster-spec
- redis sentinel : https://redis.io/topics/sentinel
- zookeeper를 이용한 clustering : http://d2.naver.com/helloworld/294797

Redis는 여러가지 방법으로 HA를 구성할 수 있다. 

## 멀티 마스터를 위한 Redis Cluster 방식

- Redis Cluster는 샤딩 방식으로 데이터가 나뉘어 저장된다.
- Redis Cluster Node 는 2개 포트가 필요하다. 
  - client를 위한 포트와 data를 위한 포트(client 포트 + 10000)
  - data 포트가 node 사이의 채널 역할을 한다.
- **docker에서 redis cluster를 사용하려면 host 네트워크를 사용해야 한다.**

## sentinel 을 이용하는 방법

- 3 + 2n 의 sentinel instance를 추천
- 기본 포트는 26379
- client에서 바라보는 end point는 sentinel instance이며, sentinel이 master 접속정보를 client에 전달한다. client는 해당 정보로 다시 master로 요청을 보냄.

### docker 에서 1master + 1slave + 3sentinel 사용 예제 :

- docker : https://github.com/mdevilliers/docker-rediscluster
- docker-compose : https://github.com/AliyunContainerService/redis-cluster
```sbtshell
sudo apt-get install redis-server
sudo apt-get purge --auto-remove redis-server
```

sentinel 실행 command : `redis-server /path/to/sentinel.conf --sentinel`


### docker swarm 에서 sentinel 사용
- https://github.com/thomasjpfan/redis-cluster-docker-swarm

docker-compose.yml 
```sbtshell
version: '3.1'

services:

  redis-sentinel:
    image: thomasjpfan/redis-sentinel:${TAG:-latest}
    environment:
      - REDIS_IP=redis-zero
      - REDIS_MASTER_NAME=redismaster
    deploy:
      replicas: 3
    networks:
      - redis

  redis:
    image: thomasjpfan/redis-look:${TAG:-latest}
    environment:
      - REDIS_SENTINEL_IP=redis-sentinel
      - REDIS_MASTER_NAME=redismaster
      - REDIS_SENTINEL_PORT=26379
    deploy:
      replicas: 3
    networks:
      - redis

networks:
  redis:
    external: tru
```

### 같은 네트워크에 접속해서 테스트해보기 : 같은 네트워크에서 ip주소 대신 docker service 이름으로 접속할 수 있다.

```sbtshell
#네트워크 접속
sudo docker run -it --network redis redis /bin/bash

#redis ip 정보 얻기
docker ps inspect continerID

#네트워크에 속한 redis에 요청보내기
redis-cli -h 10.0.0.7 -p 6379 set foo bb

#redis-sentinel
redis-cli -h redis-sentinel -p 26379 sentinel master redismaster

```

### 삭제할 때 stack 도 지워야 함

```sbtshell
docker stack rm stachID
```

### 서버종료 대비 / manager node가 2개이상 이어야 함, 2대 중 1대 서버에서 장애가 나면 복구할 수 없음
- manager node 는 static ip를 갖어야 함
- **2개 노드만 사용하는 경우 둘다 manager로 사용하기** : leader 서버가 재시작하는 경우 자동 복구가 가능해짐
  - https://docs.docker.com/engine/swarm/admin_guide/