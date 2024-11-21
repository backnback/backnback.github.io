---
title: "bitcamp-final 프로젝트 배포하기 2"
date: 2024-11-20 15:08:00 +0900
categories: [NCP, Docker, Server]
tags: [NCP, Docker, Server]
description: 최종 프로젝트를 리눅스 서버 생성 후 도커를 사용하여 배포하기
math: true
toc: true
pin: true
---




## (7)  Docker 엔진 설치

---

### 설치 준비

- **파일 목록 업데이트**
    - **`$ sudo yum update`**

    ```bash
    [student@bitcamp-svr1 ~]$ sudo yum update
    [sudo] password for student:
    Loaded plugins: fastestmirror, langpacks
    
    ...
    
    Updated:
      epel-release.noarch 0:7-14
    
    Complete!
    ```
    

<br>

- **apt get**
    - 기본적으로 Debian과 Ubuntu 계열의 리눅스 시스템에서 사용하는 패키지 관리 도구
    - `apt-get install <패키지명>`
    
    ```bash
    [student@bitcamp-svr1 ~]$ chmod 0755 /usr/local/bin/apt-get
    chmod: changing permissions of ‘/usr/local/bin/apt-get’: Operation not permitted
    [student@bitcamp-svr1 ~]$ sudo curl https://raw.githubusercontent.com/dvershinin/apt-get-centos/master/apt-get.sh -o /usr/local/bin/apt-get
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  1256  100  1256    0     0   1852      0 --:--:-- --:--:-- --:--:--  1849
    [student@bitcamp-svr1 ~]$ sudo chmod 0755 /usr/local/bin/apt-get
    
    ```
    

<br>

- **기존에 설치된 Docker 제거**
    
    ```bash
    sudo yum remove docker \
                      docker-client \
                      docker-client-latest \
                      docker-common \
                      docker-latest \
                      docker-latest-logrotate \
                      docker-logrotate \
                      docker-engine
    ```
    
    ```bash
    [student@bitcamp-svr1 ~]$ sudo yum remove docker \
    >                   docker-client \
    >                   docker-client-latest \
    >                   docker-common \
    >                   docker-latest \
    >                   docker-latest-logrotate \
    >                   docker-logrotate \
    >                   docker-engine
    [sudo] password for student:
    Loaded plugins: fastestmirror, langpacks
    No Match for argument: docker
    No Match for argument: docker-client
    No Match for argument: docker-client-latest
    No Match for argument: docker-common
    No Match for argument: docker-latest
    No Match for argument: docker-latest-logrotate
    No Match for argument: docker-logrotate
    No Match for argument: docker-engine
    No Packages marked for removal
    
    ```
    
    - `\` 는 옆으로 나열하지 않고 줄바꿈하는 방법

<br>

### Docker의 공식 리포지토리 추가

- **set up the repo**  (docker)
    - Docker 설치 파일을 받기 위한 저장소 등록
        - Docker는 소프트웨어 컨테이너를 관리하기 위한 도구
        - CentOS에서 Docker를 설치하려면 Docker의 공식 리포지토리를 시스템에 추가해야 한다.
    - `yum-utils`는 YUM 패키지 관리자의 여러 유용한 도구들을 포함한 패키지로,
        
        YUM 리포지토리를 관리하는 데 필요한 여러 기능을 제공
        
    - Docker의 공식 리포지토리를 CentOS에 추가
    
    ```bash
    [student@bitcamp-svr1 ~]$ sudo yum install -y yum-utils
    Loaded plugins: fastestmirror, langpacks
    Loading mirror speeds from cached hostfile
     * epel: repo.jing.rocks
    Package yum-utils-1.1.31-54.el7_8.noarch already installed and latest version
    Nothing to do
    
    [student@bitcamp-svr1 ~]$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    Loaded plugins: fastestmirror, langpacks
    adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
    grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
    repo saved to /etc/yum.repos.d/docker-ce.repo
    ```
    

<br>

### Docker Engine 설치

- **`$ sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`**
    
    ```bash
    [student@bitcamp-svr1 ~]$ sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    
    Loaded plugins: fastestmirror, langpacks
    Loading mirror speeds from cached hostfile
     * epel: repo.jing.rocks
    ...
    
    Complete!
    ```
    

<br>

### Docker engine 시작

- **`$ sudo systemctl start docker`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo systemctl start docker
    ```
    

<br>

- **Docker 설치 확인**
    - **`$ sudo docker run hello-world`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    c1ec31eb5944: Pull complete
    Digest: sha256:d211f485f2dd1dee407a80973c8f129f00d54604d2c90732e8e320e5038a0348
    Status: Downloaded newer image for hello-world:latest
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.
    
    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash
    
    Share images, automate workflows, and more with a free Docker ID:
     https://hub.docker.com/
    
    For more examples and ideas, visit:
     https://docs.docker.com/get-started/
    ```
    
<br>
<br>


## (8)  Docker 컨테이너 생성 및 실행

---

### 도커 컨테이너 생성 및 실행

- **우분투 14.04 컨테이너 생성**
    - **`$ sudo docker run -i -t ubuntu:14.04`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker run -i -t ubuntu:14.04
    Unable to find image 'ubuntu:14.04' locally
    14.04: Pulling from library/ubuntu
    2e6e20c8e2e6: Pull complete
    0551a797c01d: Pull complete
    512123a864da: Pull complete
    Digest: sha256:64483f3496c1373bfd55348e88694d1c4d0c9b660dee6bfef5e12f43b9933b30
    Status: Downloaded newer image for ubuntu:14.04
    root@afe8ed187221:/#
    
    ```
    
    - 컨테이너에 접속한 상태이다.   (`root@afe8ed187221:/#`)
        
        ```jsx
        root@afe8ed187221:/# whoami
        root
        root@afe8ed187221:/# hostname
        afe8ed187221
        root@afe8ed187221:/# ls
        bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
        boot  etc  lib   media  opt  root  sbin  sys  usr
        
        ```
        

<br>

- **컨테이너 나가기**   (**`# exit`**,    or     **ctrl + D**)
    
    ⇒   다시 리눅스 서버로 돌아온다.
    
    ⇒   헷갈리면 안된다. 우리는 현재 리눅스 서버에 접속한 상태에서 컨테이너에 접속한 것이다.  ★
    
    단, 컨테이너를 나갈 때 컨테이너도 종료된다.
    

<br>

### 도커 이미지 목록 확인하기

- **`$ sudo docker images`**
    - 현재 Docker 시스템에 다운로드되어 있는 이미지 목록
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker images
    REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
    hello-world   latest    d2c94e258dcb   18 months ago   13.3kB
    ubuntu        14.04     13b66b487594   3 years ago     197MB
    ```
    

<br>

### 도커 이미지 내려받기   (centos:7)

- **`$ sudo docker pull centos:7`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker pull centos:7
    7: Pulling from library/centos
    2d473b07cdd5: Pull complete
    Digest: sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
    Status: Downloaded newer image for centos:7
    docker.io/library/centos:7
    [student@bitcamp-svr1 ~]$ sudo docker images
    REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
    hello-world   latest    d2c94e258dcb   18 months ago   13.3kB
    centos        7         eeb6ee3f44bd   3 years ago     204MB
    ubuntu        14.04     13b66b487594   3 years ago     197MB
    
    ```
    
<br>


### 도커 이미지로 컨테이너 생성하기

- **`$ sudo docker create -i -t --name mycentos centos:7`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker create -i -t --name mycentos centos:7
    7d357dfeb0f6f7193ba11ea9e8306fe8bf9a159018182aefee82a1c76a323a96
    (컨테이너의 고유 ID)
    
    ```
    

<br>

### 도커 컨테이너 목록 보기

- **`$ sudo docker ps`**
- **`$ sudo docker ps -a`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker ps
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
    7d357dfeb0f6   centos:7       "/bin/bash"   45 seconds ago   Created                               mycentos
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   11 minutes ago   Exited (1) 8 minutes ago              laughing_haibt
    afe8ed187221   ubuntu:14.04   "/bin/bash"   19 minutes ago   Exited (0) 12 minutes ago             serene_shtern
    49cc738cd62f   hello-world    "/hello"      23 minutes ago   Exited (0) 23 minutes ago             charming_shirley
    ```
    
    - name을 지정하지 않으면 아무 2개 단어로 만든다.

<br>

- **`$ sudo docker ps -a`**    :    전체  컨테이너 출력
- **`$ sudo docker ps`**    :    현재 실행 중인 것만 나온다.
    - CONTAINER ID   :   컨테이너에 자동 할당되는 고유한 ID.
        - `$ sudo docker inspect 컨테이너이름 | grep Id`    :   컨테이너 ID 전체 보기
    - IMAGE   :   컨테이너 이미지 이름
    - COMMAND   :   컨테이너가 시작될 때 실행될 명령
    - CREATED   :   컨테이너가 생성된 후 흐른 시간
    - STATUS   :   컨테이너 상태    ( Up(실행중),  Exited(종료),  Pause(중지) )
    - PORTS    :   컨테이너가 개방한 포트와 호스트에 연결한 포트
        - 외부에 노출하도록 설정하지 않았다면 출력 내용 없음
    - NAMES   :   컨테이너의 고유한 이름
        - `--name 옵션`으로 설정한 이름.  설정하지 않으면 형용사와 명사를 사용해 무작위 생성
        - `$ sudo docker rename 무작위이름 새이름`

<br>

### 도커 컨테이너 실행하기

- **`$ sudo docker start mycentos`**
- **`$ sudo docker start 해시아이디일부분`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker start mycentos
    mycentos
    
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
    7d357dfeb0f6   centos:7       "/bin/bash"   5 minutes ago    Up 55 seconds                         mycentos
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   16 minutes ago   Exited (1) 13 minutes ago             laughing_haibt
    afe8ed187221   ubuntu:14.04   "/bin/bash"   23 minutes ago   Exited (0) 16 minutes ago             serene_shtern
    49cc738cd62f   hello-world    "/hello"      28 minutes ago   Exited (0) 28 minutes ago             charming_shirley
    
    [student@bitcamp-svr1 ~]$ sudo docker ps
    CONTAINER ID   IMAGE      COMMAND       CREATED         STATUS              PORTS     NAMES
    7d357dfeb0f6   centos:7   "/bin/bash"   5 minutes ago   Up About a minute             mycentos
    
    ```
    

<br>


### 도커 컨테이너에 접속하기

- **`$ sudo docker attach mycentos`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker attach mycentos
    [root@7d357dfeb0f6 /]#
    
    [root@7d357dfeb0f6 /]# exit
    exit
    [student@bitcamp-svr1 ~]$ sudo docker attach mycentos
    You cannot attach to a stopped container, start it first
    
    [student@bitcamp-svr1 ~]$ sudo docker start mycentos
    mycentos
    [student@bitcamp-svr1 ~]$ sudo docker attach mycentos
    [root@7d357dfeb0f6 /]#
    
    [root@7d357dfeb0f6 /]# read escape sequence   (ctrl + p, ctrl + q)
    [student@bitcamp-svr1 ~]$ sudo docker attach mycentos
    [root@7d357dfeb0f6 /]#
    ```
    
    - **exit**로 나가면 컨테이너를 종료한다.
        
        ⇒    **ctrl + P**   ⇒   **ctrl + Q** 로 단순히 컨테이너 쉘만 빠져나간다.   **(실행 유지)**    ★
        

<br>

### 한방에 모든 과정을 끝내는 법

- **`docker run -i -t`**    ★
    - 이미지 없으면 **`docker pull`**
    - **`docker create -i -t`**
    - **`docker start`**
    - **`docker attach`**    :   -i -t 옵션을 사용했을 때

- **`docker create`**
    - 이미지 없으면 **`docker pull`**


<br>

### 도커 컨테이너 rename

- **`$ sudo docker rename 무작위이름 새이름`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker rename charming_shirley hello
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                          PORTS     NAMES
    7d357dfeb0f6   centos:7       "/bin/bash"   30 minutes ago   Up 22 minutes                             mycentos
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   41 minutes ago   Exited (1) 38 minutes ago                 laughing_haibt
    afe8ed187221   ubuntu:14.04   "/bin/bash"   49 minutes ago   Exited (0) 42 minutes ago                 serene_shtern
    49cc738cd62f   hello-world    "/hello"      54 minutes ago   Exited (0) About a minute ago             hello
    
    ```
    

<br>

### 도커 컨테이너 삭제하기

- **`$ sudo docker rm 컨테이너이름`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker rm serene_shtern
    serene_shtern
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
    7d357dfeb0f6   centos:7       "/bin/bash"   31 minutes ago   Up 23 minutes                         mycentos
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   42 minutes ago   Exited (1) 39 minutes ago             laughing_haibt
    49cc738cd62f   hello-world    "/hello"      54 minutes ago   Exited (0) 2 minutes ago              hello
    
    ```
    
<br>


- **실행 중인 것**은 **멈추고 삭제**해야 한다.
    - **`$ sudo docker stop 컨테이너이름`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker stop mycentos
    mycentos
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                       PORTS     NAMES
    7d357dfeb0f6   centos:7       "/bin/bash"   32 minutes ago   Exited (137) 2 seconds ago             mycentos
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   43 minutes ago   Exited (1) 40 minutes ago              laughing_haibt
    49cc738cd62f   hello-world    "/hello"      55 minutes ago   Exited (0) 3 minutes ago               hello
    [student@bitcamp-svr1 ~]$ sudo docker rm mycentos
    mycentos
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   43 minutes ago   Exited (1) 40 minutes ago             laughing_haibt
    49cc738cd62f   hello-world    "/hello"      55 minutes ago   Exited (0) 3 minutes ago              hello
    
    ```
    
<br>


- 실행 중인 것도 **강제 삭제**하기
    - **`$ sudo docker rm -f 컨테이너명`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker rm -f mycentos2
    mycentos2
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
    f1079aa5ffc1   ubuntu:14.04   "/bin/bash"   45 minutes ago   Exited (1) 42 minutes ago             laughing_haibt
    49cc738cd62f   hello-world    "/hello"      57 minutes ago   Exited (0) 5 minutes ago              hello
    
    ```
    
<br>

- **정지된 모든 컨테이너 삭제**
    - **`$ sudo docker container prune`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker container prune
    WARNING! This will remove all stopped containers.
    Are you sure you want to continue? [y/N] y
    Deleted Containers:
    f1079aa5ffc19014f4da63d1d1df4677bece49f3e6b6650d05f3fb6fbad0d0fd
    49cc738cd62f5a140a2244eb2fc2666bdaf4e0e7f820bd40106e644fd6748077
    
    Total reclaimed space: 24B
    
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    
    ```
    
    
<br>
<br>

## (8-1)  외부에서 컨테이너 접속하기

---

- 현재는 도커가 설치된 호스트만 접속 가능한데 외부에서도 접속할 수 있게 설정

<br>

### ubuntu 컨테이너를 만들고 시작

- **`$ sudo docker run -i -t --name network_test ubuntu:14.04`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker run -i -t --name network_test ubuntu:14.04
    root@4fa012018bab:/#
    
    root@4fa012018bab:/# ifconfig
    eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
              inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:0 errors:0 dropped:0 overruns:0 frame:0
              TX packets:1 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0
              RX bytes:0 (0.0 B)  TX bytes:42 (42.0 B)
    
    lo        Link encap:Local Loopback
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:1 errors:0 dropped:0 overruns:0 frame:0
              TX packets:1 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:28 (28.0 B)  TX bytes:28 (28.0 B)
    
    ```
    
    - **`# ifconfig`**    :    현재 이 컨테이너의 IP address 나온다.

<br>

### 컨테이너에 아파치 설치하기

- **`# apt install apache2`**
    
    ```jsx
    root@4fa012018bab:/# apt install apache2
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    The following extra packages will be installed:
    .....
    
    ```
    

<br>


### 아파치 실행

- **`# service apache2 start`**
    
    ```jsx
    root@4fa012018bab:/# service apache2 status
     * apache2 is not running
    
    root@4fa012018bab:/# service apache2 start
     * Starting web server apache2          
    ```
    
    <br>

- **테스트**
    
    ```jsx
    root@4fa012018bab:/# apt install curl
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    The following extra packages will be installed:
      libcurl3
    The following NEW packages will be installed:
      curl libcurl3
    0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
    Need to get 295 kB of archives.
    After this operation, 871 kB of additional disk space will be used.
    Do you want to continue? [Y/n] y
    ... ...
    
    root@4fa012018bab:/# curl http://www.naver.com
    <html>
    <head><title>302 Found</title></head>
    <body>
    <center><h1>302 Found</h1></center>
    <hr><center> NWS </center>
    </body>
    </html>
    ```
    <br>

    
    - `curl`
        
        CLI용 HTTP 클라이언트로, HTTP/HTTPS 요청을 보내고 응답을 받을 수 있는 도구
        
        - **`# curl localhost`**   /   **`$ curl 172.17.0.2`**
            
            ```jsx
            root@4fa012018bab:/# curl localhost
            
            <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
            <html xmlns="http://www.w3.org/1999/xhtml">
              <!--
                Modified from the Debian original for Ubuntu
                Last updated: 2014-03-19
                See: https://launchpad.net/bugs/1288690
              -->
              <head>
                ....
            
            ```
        - 리눅스에서 만들었으므로 DOCKER에 당연히 접속 가능


<br>

### 외부에서 컨테이너 접속하기

- 리눅스 서버 공인 IP  (NCP)
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
    4fa012018bab   ubuntu:14.04   "/bin/bash"   19 minutes ago   Up 19 minutes             network_test
    
    [student@bitcamp-svr1 ~]$ curl 172.17.0.2
    
    ```
    
    - 리눅스 서버에서 현재 컨테이너에 접속이 되는데 외부에서는 안 된다.
        
        ⇒   포트 포워딩이 필요하다.
        

<br>
<br>

## (8-2)  포트 포워딩

---

### 포트 포워딩

- 컨테이너를 만들 때 옵션을 지정해야 한다.
- **`$ sudo docker run -i -t --name 컨테이너이름 -p 80:80 ubuntu:14.04`**
    
    ⇒    리눅스 서버에 접속하면 컨테이너에 전달하도록
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker run -i -t --name network_test2 -p 80:80 ubuntu:14.04
    [sudo] password for student:
    root@7bb74ebaa03b:/#
    
    # apt install apache2
    
    # service apache2 start
    
    ctrl P, ctrl q
    
    [student@bitcamp-svr1 ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE          COMMAND       CREATED              STATUS              PORTS                NAMES
    7bb74ebaa03b   ubuntu:14.04   "/bin/bash"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   network_test2
    4fa012018bab   ubuntu:14.04   "/bin/bash"   26 minutes ago       Up 26 minutes                            network_test
    ```
    
    ⇒   **host에 80 포트로 오면 컨테이너의 80번 포트로 전달하라.**
    
    - 인터렉션(`-i` 옵션)을 하지 않으면 시작하자 마자 exited 된다.

- 다시 브라우저를 열고 101.79.10.151 열기
    
    

- 이번엔 8888 포트를 80포트로 연결하기  (위의 과정 반복)
    - 브라우저에서 **http://101.79.10.151:8888/**
    - `$ curl ip.me`로 **서버 IP 주소**를 알아낼 수 있다.


<br>

### **index.html** **직접 수정**하기

- **`# cd var/www/html`**
- **`# sudo vi index.html`**
    
    ```jsx
    root@b39f2c13ef55:/# cd var/www/html
    root@b39f2c13ef55:/var/www/html# sudo nano index.html
    sudo: nano: command not found
    
    (nano editor가 없음)
    
    root@b39f2c13ef55:/var/www/html# sudo vi index.html
    
    (수정 작업 하기 esc + a)
    ```
    
    
    - 8888 포트로 들어온 사이트를 수정한 것


<br>

### 도커가 바인딩된 포트번호 확인하기

- **`$ sudo docker port 컨테이너이름`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker port network_test
    (바인딩도니 포트번호가 없으므로)
    
    [student@bitcamp-svr1 ~]$ sudo docker port network_test2
    80/tcp -> 0.0.0.0:80
    
    [student@bitcamp-svr1 ~]$ sudo docker port network_test3
    80/tcp -> 0.0.0.0:8888
    (컨테이너)     (호스트)
    ```
    

<br>

### 다른 방법으로 컨테이너에 접속하기

- **`$ sudo docker exec -i -t 컨테이너명 /bin/bash`**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker exec -i -t mycentos /bin/bash
    root@ea73d30ab8ea:/#
    ```
    
    - **`attach`**와 결과는 동일하다.

<br>

### Detached 모드로 컨테이너의 내부 셸을 사용하기

- **`$ sudo docker exec 컨테이너명 명령어`**
- **접속을 하지 않고 컨테이너에 명령 넣기**
    
    **⇒    컨테이너는 결과를 host os에 리턴한다.**
    
    ```jsx
    [student@bitcamp-svr1 ~]$ sudo docker exec mycentos ls
    bin
    boot
    dev
    etc
    home
    lib
    lib64
    media
    mnt
    opt
    proc
    root
    run
    sbin
    srv
    sys
    tmp
    usr
    var
    
    [student@bitcamp-svr1 ~]$ sudo docker exec mycentos ls -al
    total 4
    drwxr-xr-x   1 root root    6 Nov  7 05:22 .
    drwxr-xr-x   1 root root    6 Nov  7 05:22 ..
    -rwxr-xr-x   1 root root    0 Nov  7 05:22 .dockerenv
    drwxr-xr-x   2 root root 4096 Dec 17  2019 bin
    drwxr-xr-x   2 root root    6 Apr 10  2014 boot
    drwxr-xr-x   5 root root  360 Nov  7 05:22 dev
    drwxr-xr-x   1 root root   66 Nov  7 05:22 etc
    drwxr-xr-x   2 root root    6 Apr 10  2014 home
    drwxr-xr-x  12 root root  208 Dec 17  2019 lib
    drwxr-xr-x   2 root root   34 Dec 17  2019 lib64
    drwxr-xr-x   2 root root    6 Dec 17  2019 media
    drwxr-xr-x   2 root root    6 Apr 10  2014 mnt
    drwxr-xr-x   2 root root    6 Dec 17  2019 opt
    dr-xr-xr-x 141 root root    0 Nov  7 05:22 proc
    drwx------   2 root root   37 Dec 17  2019 root
    drwxr-xr-x   1 root root   21 Mar 25  2021 run
    drwxr-xr-x   1 root root   44 Mar 25  2021 sbin
    drwxr-xr-x   2 root root    6 Dec 17  2019 srv
    dr-xr-xr-x  13 root root    0 Nov  7 05:22 sys
    drwxrwxrwt   2 root root    6 Dec 17  2019 tmp
    drwxr-xr-x   1 root root   18 Dec 17  2019 usr
    drwxr-xr-x   1 root root   17 Dec 17  2019 var
    
    ```
    
<br>
<br>


## (9)  bitcamp-final 웹 애플리케이션 컨테이너 생성

---

- **쓸모없는 것 제거**
    
    ```jsx
    [student@bitcamp-svr-final ~]$ sudo docker container prune
    WARNING! This will remove all stopped containers.
    Are you sure you want to continue? [y/N] y
    Deleted Containers:
    cb7242e92e05b1b870af538ed311932dc3c2f0d4455300ae674f1df8d5d013ad
    1252c24fd0a43f4829ebbf4ea40de8df96be9234c1d8970f332e7b5274b379aa
    14a8c9099f593449cdb93c8682633ce1e76894337fe31ca017581d70c363202a
    a4f3d6a4d499d51e636f171e900b34671a16dd2a6dc4912e88688816ef1c7b14
    
    Total reclaimed space: 69B
    [student@bitcamp-svr-final ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE       COMMAND                  CREATED             STATUS             PORTS                   NAMES
    ```
    

<br>

### ubuntu 최신 버전 이미지로 컨테이너 생성
**
- **Ubuntu 22.04 컨테이너 생성 및 실행**
    - **`$ sudo docker run -p 8080:8080 -it --name bitcamp-final ubuntu`**
    
    ```haskell
    [student@bitcamp-svr-final ~]$ sudo docker run -p 8080:8080 -it --name bitcamp-final ubuntu
    Unable to find image 'ubuntu:latest' locally
    latest: Pulling from library/ubuntu
    ff65ddf9395b: Pull complete
    Digest: sha256:99c35190e22d294cdace2783ac55effc69d32896daaa265f0bbedbcde4fbe3e5
    Status: Downloaded newer image for ubuntu:latest
    root@506a7a2a9ecb:/# [student@bitcamp-svr-final ~]$ sudo docker ps -a
    CONTAINER ID   IMAGE                 COMMAND       CREATED      STATUS         PORTS                    NAMES
    506a7a2a9ecb   bitcamp-final:first   "/bin/bash"   2 days ago   Up 2 seconds   0.0.0.0:8080->8080/tcp   bitcamp-final
    ```
    
    - `-i -t`    =   `-it`
    - 다시 서버로 나가서 컨테이너 상태 확인

<br>

- 재접속 후 **`# apt update`**
    - **`$ sudo docker exec -it bitcamp-final /bin/bash`**
    - 컨테이너에서 `# apt update`
    
    ```bash
    [student@bitcamp-svr-final ~]$ sudo docker exec -it bitcamp-final /bin/bash
    root@506a7a2a9ecb:/# apt update
    Get:1 http://archive.ubuntu.com/ubuntu noble InRelease [256 kB]
    Get:2 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
    Get:3 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Packages [15.3 kB]
    Get:4 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [720 kB]
    Get:5 http://archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
    Get:6 http://archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
    Get:7 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [537 kB]
    Get:8 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [574 kB]
    Get:9 http://archive.ubuntu.com/ubuntu noble/multiverse amd64 Packages [331 kB]
    Get:10 http://archive.ubuntu.com/ubuntu noble/restricted amd64 Packages [117 kB]
    Get:11 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages [1808 kB]
    Get:12 http://archive.ubuntu.com/ubuntu noble/universe amd64 Packages [19.3 MB]
    Get:13 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [922 kB]
    Get:14 http://archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [542 kB]
    Get:15 http://archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Packages [18.4 kB]
    Get:16 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [792 kB]
    Get:17 http://archive.ubuntu.com/ubuntu noble-backports/universe amd64 Packages [11.8 kB]
    Fetched 26.3 MB in 7s (3805 kB/s)
    Reading package lists... Done
    Building dependency tree... Done
    Reading state information... Done
    3 packages can be upgraded. Run 'apt list --upgradable' to see them.
    ```
    
<br>

### 기타 프로그램 추가 및 설정

- **ifconfig 등 네트워킹 관련 프로그램 추가**
    - **`# apt update`**
    - **`# apt install net-tools`**

- **nano 에디터 설치**
    - **`# apt install nano`**

- **JDK 설치**
    - **`# apt install openjdk-21-jdk -y`**

- **JAVA_HOME 환경 변수** 설정
    
    ```haskell
    root@506a7a2a9ecb:/# ls -al /usr/lib/jvm
    total 8
    drwxr-xr-x 4 root root  126 Nov  7 16:36 .
    drwxr-xr-x 1 root root 4096 Nov  7 16:36 ..
    -rw-r--r-- 1 root root 1840 Jul 23 10:34 .java-1.21.0-openjdk-amd64.jinfo
    lrwxrwxrwx 1 root root   21 Jul 23 10:34 java-1.21.0-openjdk-amd64 -> java-21-openjdk-amd64
    drwxr-xr-x 9 root root  119 Nov  7 16:36 java-21-openjdk-amd64
    drwxr-xr-x 2 root root   21 Nov  7 16:36 openjdk-21
    root@506a7a2a9ecb:/# echo 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64' | tee -a /etc/bash.bashrc
    export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
    
    root@506a7a2a9ecb:/# cat /etc/bash.bashrc    (확인하는 것)
    # System-wide .bashrc file for interactive bash(1) shells.
    
    # To enable the settings / commands in this file for login shells as well,
    # this file has to be sourced in /etc/profile.
    
    # If not running interactively, don't do anything
    [ -z "$PS1" ] && return
    
    # check the window size after each command and, if necessary,
    # update the values of LINES and COLUMNS.
    shopt -s checkwinsize
    
    # set variable identifying the chroot you work in (used in the prompt below)
    if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
        debian_chroot=$(cat /etc/debian_chroot)
    fi
    
    # set a fancy prompt (non-color, overwrite the one in /etc/profile)
    # but only if not SUDOing and have SUDO_PS1 set; then assume smart user.
    if ! [ -n "${SUDO_USER}" -a -n "${SUDO_PS1}" ]; then
      PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
    fi
    
    # Commented out, don't overwrite xterm -T "title" -n "icontitle" by default.
    # If this is an xterm set the title to user@host:dir
    #case "$TERM" in
    #xterm*|rxvt*)
    #    PROMPT_COMMAND='echo -ne "\033]0;${USER}@${HOSTNAME}: ${PWD}\007"'
    #    ;;
    #*)
    #    ;;
    #esac
    
    # enable bash completion in interactive shells
    #if ! shopt -oq posix; then
    #  if [ -f /usr/share/bash-completion/bash_completion ]; then
    #    . /usr/share/bash-completion/bash_completion
    #  elif [ -f /etc/bash_completion ]; then
    #    . /etc/bash_completion
    #  fi
    #fi
    
    # sudo hint
    if [ ! -e "$HOME/.sudo_as_admin_successful" ] && [ ! -e "$HOME/.hushlogin" ] ; then
        case " $(groups) " in *\ admin\ *|*\ sudo\ *)
        if [ -x /usr/bin/sudo ]; then
            cat <<-EOF
            To run a command as administrator (user "root"), use "sudo <command>".
            See "man sudo_root" for details.
    
            EOF
        fi
        esac
    fi
    
    # if the command-not-found package is installed, use it
    if [ -x /usr/lib/command-not-found -o -x /usr/share/command-not-found/command-not-found ]; then
            function command_not_found_handle {
                    # check because c-n-f could've been removed in the meantime
                    if [ -x /usr/lib/command-not-found ]; then
                       /usr/lib/command-not-found -- "$1"
                       return $?
                    elif [ -x /usr/share/command-not-found/command-not-found ]; then
                       /usr/share/command-not-found/command-not-found -- "$1"
                       return $?
                    else
                       printf "%s: command not found\n" "$1" >&2
                       return 127
                    fi
            }
    fi
    export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
    ```
    
    - 이렇게 맨 마지막에 들어가는 것을 확인할 수 있다.
        
        **`export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64`**
        

<br>

- **환경 변수 설정 적용 후 확인**
    
    ```haskell
    root@506a7a2a9ecb:/# source /etc/bash.bashrc
    
    root@506a7a2a9ecb:/# echo $JAVA_HOME
    /usr/lib/jvm/java-21-openjdk-amd64
    ```
    

<br>

- **git 설치**
    - **`# apt install git -y`**
    
    ```jsx
    # apt install git -y
    
    root@506a7a2a9ecb:/# git --version
    git version 2.43.0
    ```
    
<br>


- **wget 설치**
    - **`# apt install wget -y`**
    
<br>

- **unzip 설치**
    - **`# apt install unzip -y`**


<br>

### bitcamp-final 저장소 복제 후 빌드 및 실행

- **bitcamp-fianl git 저장소** 가져오기
- **`# git clone …`**
    
    ```jsx
    root@506a7a2a9ecb:/# cd
    root@506a7a2a9ecb:/# pwd
    /root
    root@506a7a2a9ecb:/# mkdir git
    root@506a7a2a9ecb:/# cd git
    root@506a7a2a9ecb:/#:~/git# git clone https://github.com/backnback/bitcamp-final
    Cloning into 'bitcamp-final'...
    remote: Enumerating objects: 98, done.
    remote: Counting objects: 100% (98/98), done.
    remote: Compressing objects: 100% (71/71), done.
    remote: Total 98 (delta 15), reused 95 (delta 15), pack-reused 0 (from 0)
    Receiving objects: 100% (98/98), 77.82 KiB | 2.51 MiB/s, done.
    Resolving deltas: 100% (15/15), done.
    ```
    
<br>

- **빌드 후 실행하기**
    
    ```jsx
    
    root@506a7a2a9ecb:~/git/bitcamp-final# ./gradlew build
    bash: ./gradlew: Permission denied
    root@506a7a2a9ecb:~/git/bitcamp-final# chmod 755 ./gradlew
    root@506a7a2a9ecb:~/git/bitcamp-final# ./gradlew build
    Downloading https://services.gradle.org/distributions/gradle-8.10.2-bin.zip
    ....
    
    root@506a7a2a9ecb:~/git/bitcamp-final# java -jar ./app/build/libs/bitcamp-final.jar
    ```
    
    - 실행이 되지 않는다.    ⇒     **final.properties** 과 **email.properties** 추가 필요

<br>

- **final.properties** 과 **email.properties** 추가 후 재실행
    
    ```haskell
    root@506a7a2a9ecb:~/git/bitcamp-final# cd
    root@506a7a2a9ecb:~# mkdir config
    root@506a7a2a9ecb:~# cd config
    root@506a7a2a9ecb:~/config# nano final.properties
    root@506a7a2a9ecb:~/config# nano email.properties
    
    root@506a7a2a9ecb:~/config# cd
    root@506a7a2a9ecb:~# cd git/bitcamp-final
    root@506a7a2a9ecb:~/git/bitcamp-final# java -jar ./app/build/libs/bitcamp-final.jar
    
    ```
    

<br>
<br>

## (10)  컨테이너로 이미지 생성

---

### 컨테이너를 그대로 이미지로 만들기

- **`$ sudo docker commit 옵션 컨테이너명 REPOSITORY:TAG`**
- **`$ sudo docker commit -a "alicek106" -m "my first commit" commit_test commit_test:first`**
    - `-a author`    :   이미지 작성자에 대한 정보를 이미지에 포함시킨다.
    - `-m 커밋메시지`    :   이미지에 포함될 부가 설명을 설정한다.
    
    ```haskell
    root@506a7a2a9ecb:~/git/bitcamp-final# cd
    root@506a7a2a9ecb:~# exit
    exit
    
    [student@bitcamp-svr-final ~]$ sudo docker commit -a "backnback" -m "my first commit" bitcamp-final bitcamp-final:first
    sha256:73....
    
    [student@bitcamp-svr-final ~]$ sudo docker image ls
    REPOSITORY              TAG       IMAGE ID       CREATED          SIZE
    bitcamp-final           first     73.....        15 seconds ago   1...GB
    wordpress               latest    4c9b15c9a8ae   3 weeks ago      697MB
    ubuntu                  latest    59ab366372d5   3 weeks ago      78.1MB
    mysql                   5.7       5107333e08a8   11 months ago    501MB
    hello-world             latest    d2c94e258dcb   18 months ago    13.3kB
    centos                  7         eeb6ee3f44bd   3 years ago      204MB
    ubuntu                  14.04     13b66b487594   3 years ago      197MB
    alicek106/volume_test   latest    342a67bc9486   8 years ago      197MB
    ```
    

<br>

### 만든 이미지로 컨테이너 만들기

- **`$ sudo docker run -i -t --name commit_test2 commit_test:first`**
    - 컨테이너 변경   :    `# echo test_second! >> second`
    
    ```haskell
    [student@bitcamp-svr-final ~]$ sudo docker stop bitcamp-final
    bitcamp-final
    
    [student@bitcamp-svr-final ~]$ sudo docker run -i -t --name bitcamp-fianl2 -p 8081:8080 bitcamp-final:first
    ```
    
    - ctrl + P    ⇒   ctrl + Q 를 하면 종료 안하고 서버로 갈 수 있다.
    - 서버를 여러 개 띄울 수 있다.

