---
title: docker
date: 2019-02-24 20:58:52
tags:
- docker
- container
- swarm
- docker compose
- vagrant
categories:
- docker
---

Vagrant를 이용해 가상으로 서버를 생성하여 docker를 연습해볼 수 있는 환경을 만들어보자.

# vagrant 환경설정 
1. virtual box 설치 : https://www.virtualbox.org/wiki/Downloads
2. vagrant 설치 : https://www.vagrantup.com/downloads.html
3. vagrant box 검색 : https://app.vagrantup.com/boxes/search

## vagrant 기본 command

```sbtshell
#ubuntu 16.04 설치하기
vagrant init ubuntu/xenial64

# 재시작 (실행 중 변경된 환경설정 적용) --provision 옵션을 주면 새로 프로비져닝한다.
vagrant reload
  
#정지 상태로 변경
vagrant suspend

#os 종료
vagrant halt

#파일삭제 후 os 종료
vagrant destroy
  
#재시작 or 시작
vagrant up

#vagrant os 접속 : init한 디렉토리에서 실행
#기본 id/password => vagrant/vagrant
vagrant ssh 
```
 
## vagrant 설정 파일 수정

- port forwarding : config.vm.network "forwarded_port", guest: 6379, host: 1234
- 아이피 주소 지정 : config.vm.network "private_network", ip: "192.168.33.10"

# docker 설치하기

ubuntu(16.04)에 docker 설치하기 : https://docs.docker.com/install/linux/docker-ce/ubuntu/

## installation command

```sbtshell
# 1
sudo apt-get update

# 2
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# 3
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 4
sudo apt-key fingerprint 0EBFCD88

# 5
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# 6
sudo apt-get update

# 7-1 (latest 버전 설치하기)
sudo apt-get install docker-ce

# 7-2 (지정한 버전 설치하기)
sudo apt-get install docker-ce=<VERSION>

# 8 버전확인하기
sudo docker --version

# 9 hello-world 실행해보기
sudo docker run hello-world

# (option) start on boot 설정
#enable
sudo systemctl enable docker
#disable
sudo systemctl disable docker
```

# docker basic command

```sbtshell
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Excecute Docker image : docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG…]
## 옵션
## -d    detached mode 흔히 말하는 백그라운드 모드
## -p    호스트와 컨테이너의 포트를 연결 (포워딩)
## -v    호스트와 컨테이너의 디렉토리를 연결 (마운트)
## -e    컨테이너 내에서 사용할 환경변수 설정
## –name    컨테이너 이름 설정
## –rm    프로세스 종료시 컨테이너 자동 제거
## -it    -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
## –link    컨테이너 연결 [컨테이너명:별칭]
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls (= docker ps)
docker container ls -all #종료된 컨테이너 포함
docker container ls -a -q

## start, stop, remove start Docker container
docker start containerID
docker stop containerID
docker rm containerID

## docker log 확인하기
## 옵션
## --tail
## -f
docker logs -f containerID

##실행중인 container에 접속하기
docker exec -it 053abaac4f93 /bin/bash
```

# docker 실행 & 접속하기

1. 이미지 찾기 : https://hub.docker.com/explore/
2. ubuntu 이미지 실행하기 : `docker run -it ubuntu:16.04 /bin/bash`
  - 실행중인 프로세스가 없으면 컨테이너가 바로 종료됨
3. redis 이미지 실행하기 : `docker run -d -p 6379:6379 redis`
4. redis에 데이터 저장, 읽기 
```sbtshell
telnet localhost 6379
set foo bar
get foo
## to exit telnet
ctrl + ] -> ctrl + d
```

# swarm 설정하기

1. manager swarm initialization : swarm이 생성되면 자동으로 ingress라는 overlay 네트워크가 만들어짐

```sbtshell
# swarm 모드를 active 하면 ingress overlay network 가 생성됨
docker swarm init --advertise-addr ip_address_to_connect_with_worker

#example
#docker swarm init --advertise-addr 192.168.33.10

#swarm 해제하기
docker swarm leave
```

2. worker swarm 만들기

```sbtshell
docker swarm join --token token_value manager_node_ip_address:2377

#example
#docker swarm join --token SWMTKN-1-63g2es09cpi3fgu3js0jhxc42e4hzp2npcuub8gxdv8ozra1ih-csp26l0gdhoeajgi8yxq7cm7c 192.168.33.10:2377

```

3. 생성된 node 확인하기

```sbtshell
docker node ls
```

4. 서비스 실행하기

```sbtshell
docker service create --name redis-m -p 6379:6379 redis

#실행된 서비스 확인하기 : manager에서만 command 가능
docker service ls

#생성된 컨테이너 확인
## host의 컨테이너 확인
docker container ls 
docker ps

## node에서 실행중인 컨테이너 확인 : manager에서만 command 가능
docker node ps 
```

5. HA(LB) 설계 : https://docs.docker.com/engine/swarm/ingress/#configure-an-external-load-balancer
6. TEST

```sbtshell
#192.168.33.11 서버
telnet localhost 6379
set test test

#192.168.33.10 서버
telnet localhost 6379
get test
```

7. 서버 스케일링하기 : 매니저 노드에서 실행

```sbtshell
docker service scale redis-m=2
## *redis와 같은 data store는 data sync를 위해 솔루션에 맞는 추가설정을 해야 함 ##
```

# docker network : https://docs.docker.com/network/

-  driver : host (호스트와 다른 설정이 필요할 때) / bridge (한 호스트 여러 컨테이너) / overlay (여러 호스트 여러 컨테이너, 다양한 애플리케이션) / macvlan / none
-  network scope : local (호스트안에서 연결을 제공) / swarm (swarm 클러스터에서 연결이 가능)


## overlay 네트워크 생성 : [network-turorial-overlay](https://docs.docker.com/network/network-tutorial-overlay/#walkthrough)
swarm init과 함께 생성되는 ingress overlay 네트워크는 customize를 위해 삭제하고 원하는 옵션(서브넷, 게이트웨이…)을 주고 다시 만들 수 있다.

1. swarm initialization

```sbtshell
docker swarm init
```

2. 네트워크 생성 : user-defined overlay / overlay network for standalone containers

```sbtshell
#overlay network 생성 on manager
docker network create -d overlay nginx-net

#생성한 network 에 서비스 실행
#example (-p host port/geust port)
#docker service create --name redis-net-test --network redis-test --replicas 1 -p 6379:6379 redis

#attachable을 이용해서 overlay 네트워크에 service가 아닌 container를 직섭 실행할 수 있다. 
docker network create --driver=overlay --attachable test-net

#목록
docker network ls
```

# Dockerizing : 도커 이미지 만들기 

## image build or commit

```sbtshell
docker build
docker commit
```

## push to registry

```sbtshell
docker push
```

## docker file 설정 : [define Dockerfile](https://docs.docker.com/get-started/part2/#your-new-development-environment)

```sbtshell
#example
FROM node:4.6.1

ADD ./ /app
WORKDIR /app
RUN npm config set registry http://registry.npmjs.org
RUN npm install pm2@latest -g
RUN npm install

ENV PORT 80
EXPOSE 80

#CMD ["node", "api.js"
```

# docker compose + swarm 실행하기 : [compose](https://docs.docker.com/compose/overview/)

-  docker compose 는 여러 컨테이너를 정의하고 실행하기 위한 툴이다.
-  docker compose 버전 3에서 swarm 서비스 배포가 가능하다.

## 설치하기

```sbtshell
#download
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

#실행권한주기 
sudo chmod +x /usr/local/bin/docker-compose

#버전확인
docker-compose --version
```

## example docker-compose.yml file : [Example](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp.md)

```sbtshell
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

## 실행

`docker stack deploy --compose-file docker-compose.yml DEPLOYED_NAM`

## redis master + slave 생성

1. 파일 작성 : docker-compose.yml

```sbtshell
version: ‘3.3'
services:
  redis-master:
    image: redis
    #ports:
      #- "6379:6379" #host:container
networks:
     - some-network
  redis-slave:
    image: redis
    command: redis-server --slaveof redis-master 6379
deploy:
      replicas: 3
    networks:
     - some-network


networks:
  some-network:
driver: overlay
    attachable: true
```

2. 실행

```sbtshell
#overlay network가 새로 생성되면서 서비스 2개가 생김
#port가 swarm외부에 열려있지 않음
docker stack deploy --compose-file docker-compose.yml redis-clu

#service 확인 : 서비스에서 보이는 포트는 외부에서 swarm으로 들어가는 포트 정보
docker service ls

#컨테이너 확인 : 컨테이너의 포트정보는 현재 컨테이너에서 실행중인 포트정보
docker ps
```

# reference
- docker 무작정 따라하기 : https://www.slideshare.net/pyrasis/docker-fordummies-44424016
- docker command : https://docs.docker.com/edge/engine/reference/commandline/docker/
- swam : https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html
- docker를 이용한 ci stack 만들기 : https://github.com/marcelbirkner/docker-ci-tool-stack
- docker network architecture : https://success.docker.com/article/Docker_Reference_Architecture-_Designing_Scalable,_Portable_Docker_Container_Networks
- docker 이미지 만들기 1 : http://www.nurinamu.com/dev/2016/07/04/create-a-docker-image/
- docker 이미지 만들기 2 : https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html
- vagrant : https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html
