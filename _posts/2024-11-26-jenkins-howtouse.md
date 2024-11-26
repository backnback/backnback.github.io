---
title: "Jenkins 사용법"
date: 2024-11-26 11:00:00 +0900
categories: [Docker, Jenkins]
tags: [Jenkins, Docker, 배포 자동화]
description: Jenkins 사용 방법
math: true
toc: true
pin: false
media_subpath: /assets/img/posts/jenkins-howtouse/
---



# Jenkins 설치 및 기본 설정

## (1)  Jenkins 서버 생성 및 Jenkins 네트워크 브릿지 생성
### Jenkins용 리눅스 서버 생성
- console  &nbsp; ⇒ &nbsp;   server   &nbsp; ⇒ &nbsp;   생성 (기존 콘솔 화면)   &nbsp; ⇒  &nbsp;  CentOS-7.8-64,  High CPU
    - bitcamp-vpc,    bitcamp-vpc-web-subnet 10.0.1.0/24
    - High CPU(vCPU 2개, 4GB, 50GB, g2),  시간 요금제
    - 서버 이름 &nbsp; : &nbsp; bitcamp-svr-final2
    - network interface의 ip 입력  &nbsp; : &nbsp;  10.0.1.53   &nbsp;  ⇒ &nbsp;   새로운 공인 IP 할당
    - 물리 배치 그룹 미사용 / 반납 보호 해제
    - 인증키 이용
        - 새로운 인증키 생성  &nbsp;&nbsp;  (이름 예시 : bitcamp0524-95)
    - 네트워크 접근 설정 (ACG)
        - bitcamp-vpc-web-acg
    - 서버 생성


<br>
<br>


### 네트워크 브릿지 생성

- **젠킨스 도커 컨테이너**에서 사용할 **<span class="i1">브릿지 네트워크</span>**를 **준비**한다.
    - **`# docker network ls`**  &nbsp; : &nbsp;  <span class="i2">네트워크 목록</span> 확인
    - **`# docker network create jenkins`**  &nbsp; : &nbsp;  **Jenkins**와 **컨테이너**가 <span class="i2">통신할 네트워크</span> 생성
    
    ```bash
    [student@bitcamp-svr-final ~]$ sudo docker network ls
    [sudo] password for student:
    NETWORK ID     NAME      DRIVER    SCOPE
    f5549640db92   bridge    bridge    local
    01ee24acf253   host      host      local
    e7496018f6dd   none      null      local
    
    [student@bitcamp-svr-final ~]$ sudo docker network create jenkins
    9d944f354e333a0f86aa3ef26e5dc4d67ddc23ede8b17460eaec4eb50b5500b6
    
    [student@bitcamp-svr-final ~]$ sudo docker network ls
    NETWORK ID     NAME      DRIVER    SCOPE
    f5549640db92   bridge    bridge    local
    01ee24acf253   host      host      local
    9d944f354e33   jenkins   bridge    local
    e7496018f6dd   none      null      local
    ```
    

<br>

<br>

<br>

## (2) Jenkins Docker 컨테이너 생성 및 실행

### Jenkins Docker 이미지 다운로드

- **<span class="i1">젠킨스 도커 이미지</span>** **가져오기**
    - **`# docker pull jenkins/jenkins:jdk21`**  &nbsp; : &nbsp;  Jenkins Docker 이미지 다운로드
    - **`# docker image ls`**  &nbsp; : &nbsp;  `# docker images`와 같은 명령어지만 새로운 명령 구조
    
    ```bash
    [student@bitcamp-svr-final ~]$ sudo docker pull jenkins/jenkins:jdk21
    [sudo] password for student:
    jdk21: Pulling from jenkins/jenkins
    b2b31b28ee3c: Already exists
    cf5313b8439f: Pull complete
    131ccd8ae6df: Pull complete
    98c40a8886f1: Pull complete
    9f48c0cbf990: Pull complete
    3eba3e0aca54: Pull complete
    a3d4294ce7e3: Pull complete
    0908c948a73e: Pull complete
    cd72c4b0ee34: Pull complete
    9124c27d77a0: Pull complete
    3c3b9bf6160a: Pull complete
    8fe50c533fb6: Pull complete
    Digest: sha256:6bda81af6ef786f7d3687a9ca3009ba932a44d1c199086cec2b6b05f0e004aea
    Status: Downloaded newer image for jenkins/jenkins:jdk21
    docker.io/jenkins/jenkins:jdk21
    
    [student@bitcamp-svr-final ~]$ sudo docker image ls
    REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
    bitcamp-final-front   first     b8dc875a08da   18 hours ago    510MB
    jenkins/jenkins       jdk21     9ad7ab5685af   6 days ago      485MB
    backtoback/bitcamp    0.1       d1eb90b730dc   7 days ago      1.35GB
    bitcamp-final         first     d1eb90b730dc   7 days ago      1.35GB
    <none>                <none>    d959fe30f57d   7 days ago      1.35GB
    ubuntu                latest    fec8bfd95b54   5 weeks ago     78.1MB
    hello-world           latest    d2c94e258dcb   19 months ago   13.3kB
    centos                7         eeb6ee3f44bd   3 years ago     204MB
    ubuntu                14.04     13b66b487594   3 years ago     197MB
    ```
    

<br>

<br>

### **<span class="i1">도커 이미지 만들기</span>  &nbsp; : &nbsp;  젠킨스  &nbsp; + &nbsp; JDK21 &nbsp; + &nbsp; 도커 클라이언트**

- **<span class="i3">작업 디렉토리</span>** **생성 후 <span class="i3">install-docker.sh 파일</span>** **생성**
    
    ```bash
    [student@bitcamp-svr-final ~]$ mkdir jenkins
    
    [student@bitcamp-svr-final ~]$ cd jenkins
    
    [student@bitcamp-svr-final jenkins]$ vi install-docker.sh
    ```
    
    - **<span class="i3">install-docker.sh 파일</span> 내용 <span class="i2">복사 후 붙여 넣기</span>**
        
        ```bash
        #!/bin/sh
        
        apt-get update
        
        apt-get -y install apt-transport-https \
             apt-utils \
             ca-certificates \
             curl \
             gnupg2 \
             zip \
             unzip \
             acl \
             software-properties-common
        
        curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey
        
        add-apt-repository \
           "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
           $(lsb_release -cs) \
           stable" && \
        
        apt-get update
        
        apt-get -y install docker-ce
        ```
        
        - 작성 완료 후 `ESC`  클릭 후 `:wq` 입력

<br>

- **<span class="i1">도커 빌드 파일</span>** **생성**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ vi Dockerfile
    ```
    
    - **<span class="i2">Dockerfile</span> 내용 <span class="i2">복사 후 붙여 넣기</span>**
        
        ```bash
        FROM jenkins/jenkins:jdk21
        
        USER root
        
        COPY install-docker.sh /install-docker.sh
        RUN chmod +x /install-docker.sh
        RUN /install-docker.sh
        
        RUN usermod -aG docker jenkins
        RUN setfacl -Rm d:g:docker:rwx,g:docker:rwx /var/run/
        
        USER jenkins
        ```
        

<br>

- **<span class="i2">도커 이미지</span> 생성**
    - **`# docker build -t backtoback/bitcamp:jenkins .`**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker build -t backtoback/bitcamp:jenkins .
    [sudo] password for student:
    [+] Building 54.9s (11/11) FINISHED                              docker:default
     => [internal] load build definition from Dockerfile                       0.0s
     => => transferring dockerfile: 275B                                       0.0s
     => [internal] load metadata for docker.io/jenkins/jenkins:jdk21           0.0s
     => [internal] load .dockerignore                                          0.0s
     => => transferring context: 2B                                            0.0s
     => [internal] load build context                                          0.0s
     => => transferring context: 563B                                          0.0s
     => [1/6] FROM docker.io/jenkins/jenkins:jdk21                             0.1s
     => [2/6] COPY install-docker.sh /install-docker.sh                        0.0s
     => [3/6] RUN chmod +x /install-docker.sh                                  0.3s
     => [4/6] RUN /install-docker.sh                                          47.5s
     => [5/6] RUN usermod -aG docker jenkins                                   0.3s
     => [6/6] RUN setfacl -Rm d:g:docker:rwx,g:docker:rwx /var/run/            0.2s
     => exporting to image                                                     6.4s
     => => exporting layers                                                    6.4s
     => => writing image sha256:aa2cd2c9ac9a8691ae423ed5721f5d6b0e8f0baabb160  0.0s
     => => naming to docker.io/backtoback/bitcamp:jenkins  
    ```
    

<br>

- **도커 이미지**를 **<span class="i1">도커 허브 사이트</span>에 <span class="i2">업로드</span> 하기**
    - **`# docker login`**  &nbsp; : &nbsp;  입력 후 username과 password를 입력하면 로그인 된다.
    - **`# docker push backtoback/bitcamp:jenkins`**  &nbsp; : &nbsp;  로컬의 이미지를 도커 허브에 업로드
        - `[사용자 이름]/[저장소 이름]:[태그]`
        - **사용자 이름**과 **저장소 이름**은 **<span class="i1">도커 허브 사이트</span>**에 있는 것을 사용해야 한다.
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker login
    [sudo] password for student:
    Authenticating with existing credentials...
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    
    Login Succeeded
    
    [student@bitcamp-svr-final jenkins]$ sudo docker push backtoback/bitcamp:jenkins
    The push refers to repository [docker.io/backtoback/bitcamp]
    8e837c36d4f1: Pushed
    5f69b67c07c9: Pushed
    4851e4b39029: Pushed
    a48de29c78e8: Pushed
    b337f6f3395b: Pushed
    c57854c6754d: Pushed
    d453c9a3f980: Pushed
    200df0fdd6da: Pushed
    a47fce14f185: Pushed
    fc0b714bfc86: Pushed
    e2b50ada3c38: Pushed
    ad429ebafa13: Pushed
    452ea6c210d0: Pushed
    a65090797361: Pushed
    c18f5e54294a: Pushed
    a07cdd6d45d1: Pushed
    24b5ce0f1e07: Pushed
    jenkins: digest: sha256:99425d9ccf948a355476118a7ebef6f38459325ec8d98d3982799010fe1d772e size: 3875
    
    ```
    
    ![](image.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

- **컨테이너** **<span class="i1">생성 및 실행</span>하기**  **&nbsp;** **(DooD 방식)**
    - **<span class="i2">DooD(Docker out of Docker)</span>** **방식**은 도커 컨테이너 내부에서 호스트 머신의 Docker 데몬(Docker Engine)을 직접 사용하는 방식이다.
        - 컨테이너 내부에서 docker 명령어 실행 시, 실제로 호스트 머신의 Docker 데몬이 작업을 처리한다.
    - **`# docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home -p 8080:8080 -p 50000:50000 --restart=on-failure --network="jenkins" --name docker-jenkins backtoback/bitcamp:jenkins`**
        - **`--privileged`**
            - 컨테이너에 추가적인 권한을 부여   &nbsp; (DooD 방식 관련)
        - **`-v /var/run/docker.sock:/var/run/docker.sock`**
            - 호스트의 Docker 데몬 소켓(`/var/run/docker.sock`)을 컨테이너 내부에 마운트하는 것
            - 컨테이너에서 도커 명령을 실행하면, 호스트의 Docker 데몬과 직접 통신   &nbsp; (DooD 방식)
        - **`-v jenkins_home:/var/jenkins_home`**
            - `jenkins_home`라는 이름의 도커 볼륨을 생성
        - **`p 8080:8080`**
            - Jenkins의 웹 인터페이스는 기본적으로 컨테이너의 `8080` 포트에서 실행된다.
        - **`p 50000:50000`**
            - Jenkins에서 노드 간 통신을 위한 포트
        - **`--restart=on-failure`**
            - 컨테이너가 실패로 종료되었을 때 자동으로 재시작  &nbsp; (오류로 종료된 경우에만)
        - **`--network="jenkins"`**
            - `jenkins`라는 이름의 Docker 네트워크를 사용
                
                ⇒   컨테이너가 같은 네트워크 안에 있는 다른 컨테이너와 통신 가능
                
        - **`--name docker-jenkins backtoback/bitcamp:jenkins`**
            - `backtoback/bitcamp:jenkins` 이미지로 `docker-jenkins` 컨테이너 생성
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home -p 8081:8080 -p 50000:50000 --restart=on-failure --network="jenkins" --name docker-jenkins backtoback/bitcamp:jenkins
    [sudo] password for student:
    7665e5d7a3263b4b24d8b4343a6e720400f4cf6b6e70b64a018fc9cb1ba15b11
    
    [student@bitcamp-svr-final jenkins]$ sudo docker container ls
    CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                              NAMES
    7665e5d7a326   backtoback/bitcamp:jenkins   "/usr/bin/tini -- /u…"   13 seconds ago   Up 12 seconds   0.0.0.0:50000->50000/tcp, 0.0.0.0:8081->8080/tcp   docker-jenkins
    e51e35b65c89   bitcamp-final-front:first    "docker-entrypoint.s…"   19 hours ago     Up 19 hours     0.0.0.0:3000->3000/tcp                             bitcamp-final-front
    506a7a2a9ecb   bitcamp-final:first          "/bin/bash"              7 days ago       Up 17 hours     0.0.0.0:8080->8080/tcp                             bitcamp-final
    
    ```
    
    - **`8080`으로 하는 것이 맞으나 현재 `8080` 포트를 사용 중이여서 `8081`로 설정했다.**

<br>

- **<span class="i2">(주의!)</span> 젠킨스 컨테이너를 <span class="i1">재생성 할 때</span>, <span class="i1">기존 젠킨스 볼륨</span>을 <span class="i2">제거</span>하기**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker volume ls
    DRIVER    VOLUME NAME
    local     jenkins_home
    
    [student@bitcamp-svr-final jenkins]$ sudo docker rm jenkins_home
    
    ```


   

<br>

<br>

### 젠킨스 설정

- **젠킨스 잠금을 풀기 위해 <span class="i2">관리자 암호</span> 찾아서 넣기**
    - **`# docker logs docker-jenkins`**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker logs docker-jenkins
    [sudo] password for student:
    Running from: /usr/share/jenkins/jenkins.war
    webroot: /var/jenkins_home/war
    2024-11-26 02:52:14.828+0000 [id=1]     INFO    winstone.Logger#logInternal: Beginning extraction from war file
    2024-11-26 02:52:15.985+0000 [id=1]     WARNING o.e.j.ee9.nested.ContextHandler#setContextPath: Empty contextPath
    2024-11-26 02:52:16.055+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: jetty-12.0.14; built: 2024-09-30T14:22:54.197Z; git: e77516598a07cca826d27fa8a4f7c70e953921a6; jvm 21.0.5+11-LTS
    2024-11-26 02:52:16.558+0000 [id=1]     INFO    o.e.j.e.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.ee9.jsp.JettyJspServlet
    2024-11-26 02:52:16.621+0000 [id=1]     INFO    o.e.j.s.DefaultSessionIdManager#doStart: Session workerName=node0
    2024-11-26 02:52:17.115+0000 [id=1]     INFO    hudson.WebAppMain#contextInitialized: Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
    2024-11-26 02:52:17.282+0000 [id=1]     INFO    o.e.j.s.handler.ContextHandler#doStart: Started oeje9n.ContextHandler$CoreContextHandler@a518813{Jenkins v2.486,/,b=file:///var/jenkins_home/war/,a=AVAILABLE,h=oeje9n.ContextHandler$CoreContextHandler$CoreToNestedHandler@43d38654{STARTED}}
    2024-11-26 02:52:17.294+0000 [id=1]     INFO    o.e.j.server.AbstractConnector#doStart: Started ServerConnector@772861aa{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
    2024-11-26 02:52:17.305+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: Started oejs.Server@422c3c7a{STARTING}[12.0.14,sto=0] @3308ms
    2024-11-26 02:52:17.306+0000 [id=31]    INFO    winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
    2024-11-26 02:52:17.469+0000 [id=30]    INFO    jenkins.model.Jenkins#<init>: Starting version 2.486
    2024-11-26 02:52:17.553+0000 [id=39]    INFO    jenkins.InitReactorRunner$1#onAttained: Started initialization
    2024-11-26 02:52:17.573+0000 [id=37]    INFO    jenkins.InitReactorRunner$1#onAttained: Listed all plugins
    2024-11-26 02:52:18.502+0000 [id=37]    INFO    jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
    2024-11-26 02:52:18.506+0000 [id=37]    INFO    jenkins.InitReactorRunner$1#onAttained: Started all plugins
    2024-11-26 02:52:18.516+0000 [id=38]    INFO    jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
    2024-11-26 02:52:18.736+0000 [id=38]    INFO    jenkins.InitReactorRunner$1#onAttained: System config loaded
    2024-11-26 02:52:18.737+0000 [id=38]    INFO    jenkins.InitReactorRunner$1#onAttained: System config adapted
    2024-11-26 02:52:18.737+0000 [id=38]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
    2024-11-26 02:52:18.738+0000 [id=38]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
    2024-11-26 02:52:18.913+0000 [id=53]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
    2024-11-26 02:52:19.310+0000 [id=37]    INFO    jenkins.install.SetupWizard#init:
    
    *************************************************************
    *************************************************************
    *************************************************************
    
    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:
    
    5cddc3eca44e42ddbbcc05cf5d406feb
    
    This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
    
    *************************************************************
    *************************************************************
    *************************************************************
    
    2024-11-26 02:52:24.977+0000 [id=37]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
    2024-11-26 02:52:25.002+0000 [id=30]    INFO    hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
    2024-11-26 02:52:26.493+0000 [id=53]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
    2024-11-26 02:52:26.493+0000 [id=53]    INFO    hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
    ```
    
    - `/var/jenkins_home/secrets/initialAdminPassword` 파일에 저장된 암호가 보인다.

<br>

- **<span class="i1">젠킨스 플러그인</span> 설치**
    1. **Jenkins 웹 UI에 접속한다.**   **&nbsp;** **&nbsp;  <span class="i2">(http://서버IP:8081)</span>**
        - 포트는 컨테이너 <span class="i2">생성 시 지정한 포트</span>이다.
        
        ![](image 1.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
    2. **위에서 찾은 <span class="i3">관리자 암호</span>를 넣어준다.**
    3. **"Install suggested plugins" 클릭**
        
        ![](image 2.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        

<br>

- **<span class="i1">첫 번째 젠킨스 관리자</span> 등록**
    
    ```bash
    계정명: admin
    암호: 1111
    암호확인: 1111
    이름: admin
    이메일주소: xxx@xxx.xxx
    ```
    
    ![](image 3.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

- **<span class="i2">젠킨스 루트 URL</span> 설정**
    - **`http://서버IP:8081`**
    
    ![](image 4.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    
    - 자동 입력되어 있다.

<br>

- **Start Using Jenkins 클릭**
    
    ![](image 5.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

<br>

<br>

## (3) 젠킨스 환경 설정

### **<span class="i1">JDK 및 Gradle</span> 설정**

- **Dashboard / <span class="i2">Jenkins 관리</span>**
    
    ![](image 6.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    
    - **System Configuration / Tools**
        
        ![](image 7.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
        - **JDK**
            - `Add JDK` 클릭
            - Name: **OpenJDK-21**
            - JAVA_HOME: **/opt/java/openjdk**
            - Apply 클릭
            
            ![](image 8.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            
        
        <br>
        
        - **Gradle**
            - `Add Gradle` 클릭
            - Name: **Gradle 8.7**
            - **Install automatically: 체크**
            - Version: **8.7 선택**
            - Apply 클릭
            
            ![](image 9.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            

<br>

- **Node.js 설정**
    - Dashboard / Jenkins 관리
        - **플러그인 관리**
            
            ![](image 10.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            
            - Available plugins 선택
                
                ![](image 11.png){: width="300" height="150" .left }  
                <div class="clear"></div>
                
                - `Nodejs` 검색
                - `NodeJS` 체크
                - `Install without restart` 클릭
                
                ![](image 12.png){: width="300" height="150" .left }  
                <div class="clear"></div>
                
        
        <br>
        
        - **System Configuration** / Global Tool Configuration  (= **Tools**)
            - **NodeJS**
                - `Add NodeJS` 클릭
                - Name: NodeJS 18.16.0
                - Install automatically: 체크
                - Version: 18.16.0 선택
                - Apply 클릭
                
                ![](image 13.png){: width="300" height="150" .left }  
                <div class="clear"></div>
                

<br>

<br>

### **<span class="i2">**(주의!)</span> jenkins/jenkins:lts-jdk11 을 설치했을 경우

- 우리는 `$ sudo docker pull jenkins/jenkins:jdk21` 이것을 설치했다.

<br>

- JDK 17 설치
    - root 사용자로 젠킨스 컨테이너에 접속하기
        
        ```bash
        호스트# docker exec -itu 0 jenkins-jdk11 bash
        컨테이너/# apt-get update
        컨테이너/# apt-get install openjdk-17-jdk -y
        ```
        

<br>

- 젠킨스에 JDK 17 경로 등록
    - Jenkins 관리
        - Global Tool Configuration
            - JDK
                - `Add JDK` 클릭
                    - Name: `OpenJDK-17`
                    - JAVA_HOME: `/usr/lib/jvm/java-17-openjdk-amd64`
            - SAVE 클릭

<br>

<br>

<br>

## (4) Jenkins에 프로젝트 등록 및 자동 배포하기

### github.com의 프로젝트 연동

- **Dashboard**    &nbsp; ⇒  &nbsp;  **새로운 Item**
    - Enter an item name  &nbsp; :  &nbsp; **<span class="i2">`bitcamp-final`</span>**
    - **`Freestyle project`** 클릭
    - OK 클릭
    
    ![](image 14.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

<br>

### **설정**

- **<span class="i2">General</span>**
    - **설명**  &nbsp; :  &nbsp; **<span class="i2">`bitcamp-final 빌드`</span>**
    - **`GitHub project`** 체크
        - **Project url   &nbsp; :  &nbsp;  ``https://github.com/backnback/bitcamp-final.git``**
    
    ![](image 15.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

- **General   ⇒   <span class="i2">소스 코드 관리</span>**
    - **<span class="i3">`Git`</span> 선택**
        - **Repository url   &nbsp; :  &nbsp;  ``https://github.com/backnback/bitcamp-final.git``**
        - **Credentials   &nbsp; :  &nbsp;  <span class="i1">(push를 할 경우)</span>**
            
            ![](image 16.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            
            - **Add 버튼** 클릭  &nbsp; :  &nbsp; <span class="i3">**`Add Jenkins`**</span> 선택
            
            <br>
            
            - **<span class="i3">`Username with Password`</span>** 선택
                - **Username    &nbsp; :  &nbsp;**   깃허브 사용자이름
                - **Password    &nbsp; :  &nbsp;**  깃허브 토큰
                
                ![](image 17.png){: width="300" height="150" .left }  
                <div class="clear"></div>
                
                - **<span class="i3">'Add'</span>** 클릭

                <br>

            - **<span class="i3">`Username/토큰`</span>** 선택
                
                ![](image 18.png){: width="300" height="150" .left }  
                <div class="clear"></div>
                
        
        <br>
        
        - **Branch Specifier**   &nbsp; :  &nbsp;    **<span class="i3">*/main</span>**
            
            ![](image 19.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            

<br>

- **General  ⇒   <span class="i2">Triggers</span>    &nbsp; (빌드 유발)**
    - **<span class="i3">`GitHub hook trigger for GITScm polling`</span> 선택**
    
    ![](image 20.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

- **General  ⇒   <span class="i2">Environment</span>    &nbsp; (빌드 환경)**
    - **<span class="i1">`Provide Node & npm bin/ foler to PATH`</span> 체크**
    - **<span class="i3">`NodeJS 18.16.0`</span> 선택**
    
    ![](image 21.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    

<br>

- **General  ⇒   <span class="i2">Build Steps</span>**
    - <span class="i1">**`Invoke Gradle script`**</span> **선택**
        
        ![](image 22.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
        - **<span class="i3">`Invoke Gradle`</span> 선택**
            - **Gradle Version**    &nbsp; :  &nbsp;  <span class="i3">**Gradle 8.7**</span> **선택**
        - **Tasks**
            - **<span class="i3">`clean build`</span> 입력**
        
        ![](image 23.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
    
    <br>
    
    - **저장**

<br>

- <span class="i2">**지금 빌드**</span> **클릭**
    
    ![](image 24.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    
    <br>
    
    - **<span class="i1">Console Output</span> 확인**
        
        ![](image 25.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
        ```bash
        Started by user admin
        Running as SYSTEM
        Building in workspace /var/jenkins_home/workspace/bitcamp-final
        The recommended git tool is: NONE
        using credential b278040e-7116-4257-b06e-f8644b4be52c
        Cloning the remote Git repository
        Cloning repository https://github.com/backnback/bitcamp-final
         > git init /var/jenkins_home/workspace/bitcamp-final # timeout=10
        Fetching upstream changes from https://github.com/backnback/bitcamp-final
         > git --version # timeout=10
         > git --version # 'git version 2.39.5'
        using GIT_ASKPASS to set credentials 
         > git fetch --tags --force --progress -- https://github.com/backnback/bitcamp-final +refs/heads/*:refs/remotes/origin/* # timeout=10
         > git config remote.origin.url https://github.com/backnback/bitcamp-final # timeout=10
         > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
        Avoid second fetch
         > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
        Checking out Revision 64f00f28f271fee84be00d134599a0d1e00cf84e (refs/remotes/origin/main)
         > git config core.sparsecheckout # timeout=10
         > git checkout -f 64f00f28f271fee84be00d134599a0d1e00cf84e # timeout=10
        Commit message: "Merge pull request #121 from backnback/orbit"
        First time build. Skipping changelog.
        Unpacking https://nodejs.org/dist/v18.16.0/node-v18.16.0-linux-x64.tar.gz to /var/jenkins_home/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/NodeJS_18.16.0 on Jenkins
        [Gradle] - Launching build.
        Unpacking https://services.gradle.org/distributions/gradle-8.7-bin.zip to /var/jenkins_home/tools/hudson.plugins.gradle.GradleInstallation/Gradle_8.7 on Jenkins
        [bitcamp-final] $ /var/jenkins_home/tools/hudson.plugins.gradle.GradleInstallation/Gradle_8.7/bin/gradle clean build
        
        Welcome to Gradle 8.7!
        
        Here are the highlights of this release:
         - Compiling and testing with Java 22
         - Cacheable Groovy script compilation
         - New methods in lazy collection properties
        
        For more details see https://docs.gradle.org/8.7/release-notes.html
        
        Starting a Gradle Daemon (subsequent builds will be faster)
        > Task :app:clean UP-TO-DATE
        
        > Task :app:compileJava
        Note: /var/jenkins_home/workspace/bitcamp-final/app/src/main/java/bitcamp/project/service/impl/UserServiceImpl.java uses or overrides a deprecated API.
        Note: Recompile with -Xlint:deprecation for details.
        
        > Task :app:processResources
        > Task :app:classes
        > Task :app:bootJarMainClassName
        > Task :app:bootJar
        > Task :app:jar SKIPPED
        > Task :app:assemble
        > Task :app:compileTestJava
        > Task :app:processTestResources
        > Task :app:testClasses
        > Task :app:test
        > Task :app:check
        > Task :app:build
        
        BUILD SUCCESSFUL in 2m 16s
        8 actionable tasks: 7 executed, 1 up-to-date
        Build step 'Invoke Gradle script' changed build result to SUCCESS
        Finished: SUCCESS
        ```
        
         
        

<br>

- **리눅스 서버**에서 **확인**
    - **`$ sudo docker exec -itu 0 docker-jenkins bash` 접속**
    - **`# cd /var/jenkins_home/workspace/`**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo docker exec -itu 0 docker-jenkins bash
    [sudo] password for student:
    root@7665e5d7a326:/#
    
    root@7665e5d7a326:/# cd /var/jenkins_home/workspace/
    root@7665e5d7a326:/var/jenkins_home/workspace# ls
    bitcamp-final  bitcamp-final@tmp
    ```
    

<br>

<br>

### github webhook 연동

- **깃허브 [Repository]**     &nbsp;   &nbsp; **⇒**   &nbsp;   &nbsp;    **Settings**    &nbsp;   &nbsp;   **⇒**     &nbsp;   &nbsp;  **Webhooks**
    
    ![](image 26.png){: width="300" height="150" .left }  
    <div class="clear"></div>
    
    <br>
    
    - **<span class="i1">Add webook</span> 클릭**
        - **Payload URL    &nbsp; :   &nbsp;    `http://젠킨스서버주소:8081/github-webhook/`**
        - **Content type**    &nbsp; :   &nbsp;   <span class="i3">**`application/json`**</span>
        
        ![](image 27.png){: width="300" height="150" .left }  
        <div class="clear"></div>
        
        - **저장**

<br>

<br>

### 스프링부트 애플리케이션 docker 이미지 생성 및 도커 허브에 push 하기

- **Jenkins의 <span class="i1">Dashboard</span>**
    - **<span class="i2">bitcamp-final Freestyle 아이템</span> 선택**
        - **<span class="i1">구성 탭</span> 선택**
            
            ![](image 28.png){: width="300" height="150" .left }  
            <div class="clear"></div>
            
            - **<span class="i3">Build Steps</span>**
                - **`Add build step`    &nbsp; :   &nbsp;   <span class="i2">Execute shell</span> 클릭**
                    
                    ![](image 29.png){: width="300" height="150" .left }  
                    <div class="clear"></div>
                    
                    - **`docker build -t [dockerHub UserName]/[dockerHub Repository]:[version] .`**
                        - Dockerfile을 사용하여 backtoback/bitcamp-final이라는 이름의 이미지 빌드
                    - **`docker login -u '도커허브아이디' -p '도커허브비번' docker.io`**
                    - **`docker push [dockerHub UserName]/[dockerHub Repository]:[version]`**
                    
                    ```bash
                    docker build -t backtoback/bitcamp-final:1.0 .
                    docker login -u backtoback -p 비밀번호 docker.io
                    docker push backtoback/bitcamp-final:1.0
                    ```
                    
                    <br>
                    
                - **저장**
                

<br>

- **<span class="i1">/var/run/docker.sock</span>의 <span class="i2">permission denied</span> 발생하는 경우**
    ```bash
    Started by GitHub push by backnback
    Running as SYSTEM
    Building in workspace /var/jenkins_home/workspace/bitcamp-final
    The recommended git tool is: NONE
    using credential b278040e-7116-4257-b06e-f8644b4be52c
    > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/bitcamp-final/.git # timeout=10
    Fetching changes from the remote Git repository
    > git config remote.origin.url https://github.com/backnback/bitcamp-final # timeout=10
    Fetching upstream changes from https://github.com/backnback/bitcamp-final
    > git --version # timeout=10
    > git --version # 'git version 2.39.5'
    using GIT_ASKPASS to set credentials 
    > git fetch --tags --force --progress -- https://github.com/backnback/bitcamp-final +refs/heads/*:refs/remotes/origin/* # timeout=10
    > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
    Checking out Revision 6f2200860b02e5c58f22c5af171aa2ef53cf30e2 (refs/remotes/origin/main)
    > git config core.sparsecheckout # timeout=10
    > git checkout -f 6f2200860b02e5c58f22c5af171aa2ef53cf30e2 # timeout=10
    Commit message: "Merge pull request #122 from backnback/backnback"
    > git rev-list --no-walk 64f00f28f271fee84be00d134599a0d1e00cf84e # timeout=10
    [Gradle] - Launching build.
    [bitcamp-final] $ /var/jenkins_home/tools/hudson.plugins.gradle.GradleInstallation/Gradle_8.7/bin/gradle clean build
    Starting a Gradle Daemon (subsequent builds will be faster)
    > Task :app:clean

    > Task :app:compileJava
    Note: /var/jenkins_home/workspace/bitcamp-final/app/src/main/java/bitcamp/project/service/impl/UserServiceImpl.java uses or overrides a deprecated API.
    Note: Recompile with -Xlint:deprecation for details.

    > Task :app:processResources
    > Task :app:classes
    > Task :app:bootJarMainClassName
    > Task :app:bootJar
    > Task :app:jar SKIPPED
    > Task :app:assemble
    > Task :app:compileTestJava
    > Task :app:processTestResources
    > Task :app:testClasses
    > Task :app:test
    > Task :app:check
    > Task :app:build

    BUILD SUCCESSFUL in 17s
    8 actionable tasks: 8 executed
    Build step 'Invoke Gradle script' changed build result to SUCCESS
    [bitcamp-final] $ /bin/sh -xe /tmp/jenkins12607953437281659352.sh
    + docker build -t backtoback/bitcamp-final:1.0 .
    ERROR: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
    Build step 'Execute shell' marked build as failure
    Finished: FAILURE
    ```
    - **git push** 후 **Jenkins UI**에서 보니 <span class="i2">/var/run/docker.sock: connect: permission denied</span>가 발생했다.

    <br>

    - **`호스트# chmod 666 /var/run/docker.sock`**
    
    ```bash
    [student@bitcamp-svr-final jenkins]$ sudo chmod 666 /var/run/docker.sock
    [sudo] password for student:
    ```



<br>

- <span class="i2">**Dockerfile**</span>**이 없다는** <span class="i2">**오류 발생**</span>
    
    ```bash
    .....
    BUILD SUCCESSFUL in 18s
    8 actionable tasks: 8 executed
    Build step 'Invoke Gradle script' changed build result to SUCCESS
    [bitcamp-final] $ /bin/sh -xe /tmp/jenkins3289070697202538191.sh
    + docker build -t backtoback/bitcamp-final:1.0 .
    #0 building with "default" instance using docker driver
    
    #1 [internal] load build definition from Dockerfile
    #1 transferring dockerfile: 2B done
    #1 DONE 0.0s
    ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
    Build step 'Execute shell' marked build as failure
    Finished: FAILURE
    ```
    



<br>

- <span class="i1">**bitcamp-final**</span>**의** <span class="i2">**Dockerfile**</span> **만들기**
    
    ```bash
    FROM openjdk:21-jdk
    
    ARG JAR_FILE=app/build/libs/bitcamp-final.jar 
    
    COPY ${JAR_FILE} bitcamp-final.jar
    
    ENTRYPOINT [ "java", "-jar", "bitcamp-final.jar" ]
    ```
    

<br>


- <span class="i2">**Git Push**</span> **후** <span class="i3">**Jenkins UI**</span>**와** <span class="i3">**Docker Hub**</span>**를 확인하면 성공한 결과를 볼 수 있다.**
    
    ```bash
    Started by GitHub push by backnback
    Running as SYSTEM
    Building in workspace /var/jenkins_home/workspace/bitcamp-final
    
    ...
    
    Starting a Gradle Daemon (subsequent builds will be faster)
    > Task :app:clean
    
    > Task :app:compileJava
    
    ...
    
    BUILD SUCCESSFUL in 17s
    8 actionable tasks: 8 executed
    Build step 'Invoke Gradle script' changed build result to SUCCESS
    [bitcamp-final] $ /bin/sh -xe /tmp/jenkins6351348835620207006.sh
    + docker build -t backtoback/bitcamp-final:1.0 .
    #0 building with "default" instance using docker driver
    
    #1 [internal] load build definition from Dockerfile
    #1 transferring dockerfile: 194B done
    #1 DONE 0.0s
    
    #2 [internal] load metadata for docker.io/library/openjdk:21-jdk
    #2 DONE 2.6s
    
    #3 [internal] load .dockerignore
    #3 transferring context: 2B done
    #3 DONE 0.0s
    
    #4 [1/2] FROM docker.io/library/openjdk:21-jdk@sha256:af9de795d1f8d3b6172f6c55ca9ba1c5768baa11bb2dc8af7045c7db9d4c33ac
    #4 resolve docker.io/library/openjdk:21-jdk@sha256:af9de795d1f8d3b6172f6c55ca9ba1c5768baa11bb2dc8af7045c7db9d4c33ac 0.0s done
    #4 sha256:af9de795d1f8d3b6172f6c55ca9ba1c5768baa11bb2dc8af7045c7db9d4c33ac 1.04kB / 1.04kB done
    #4 sha256:c67f402f77197f2e6ae84ff1fca868699ce3b38bfa78604524051420fa2e4383 954B / 954B done
    #4 sha256:079114de2be199f2ae0f7766ac0187d24a0c3a2d658fc51bffc6af5b8bd85469 4.42kB / 4.42kB done
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 0B / 44.96MB 0.2s
    #4 sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 0B / 15.03MB 0.2s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 0B / 203.93MB 0.2s
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 10.49MB / 44.96MB 0.5s
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 16.78MB / 44.96MB 0.6s
    #4 ...
    
    #5 [internal] load build context
    #5 transferring context: 62.71MB 0.5s done
    #5 DONE 0.7s
    
    #4 [1/2] FROM docker.io/library/openjdk:21-jdk@sha256:af9de795d1f8d3b6172f6c55ca9ba1c5768baa11bb2dc8af7045c7db9d4c33ac
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 26.21MB / 44.96MB 0.8s
    #4 sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 10.49MB / 15.03MB 0.8s
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 31.46MB / 44.96MB 0.9s
    #4 sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 15.03MB / 15.03MB 0.9s
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 37.75MB / 44.96MB 1.0s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 11.53MB / 203.93MB 1.0s
    #4 sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 44.96MB / 44.96MB 1.1s done
    #4 sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 15.03MB / 15.03MB 1.1s done
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 22.02MB / 203.93MB 1.2s
    #4 extracting sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 0.1s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 34.60MB / 203.93MB 1.4s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 54.53MB / 203.93MB 1.7s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 66.06MB / 203.93MB 1.9s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 79.69MB / 203.93MB 2.1s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 92.27MB / 203.93MB 2.3s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 111.15MB / 203.93MB 2.6s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 124.78MB / 203.93MB 2.8s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 140.51MB / 203.93MB 3.1s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 150.99MB / 203.93MB 3.3s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 170.92MB / 203.93MB 3.7s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 182.45MB / 203.93MB 3.9s
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 195.04MB / 203.93MB 4.1s
    #4 extracting sha256:5262579e8e45cb87fdc8fb6182d30da3c9e4f1036e02223508f287899ea434c0 2.9s done
    #4 sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 203.93MB / 203.93MB 4.4s done
    #4 extracting sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 0.1s
    #4 extracting sha256:0eab4e2287a59db00ae2d401e107a120e21ac3a291b097faffb1af38a1bc773c 0.6s done
    #4 extracting sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 0.1s
    #4 extracting sha256:7c002e8f606286a649b6f6cc6420c9056f7d3075fe3094b9cc33a715ff609335 4.9s done
    #4 DONE 10.2s
    
    #6 [2/2] COPY app/build/libs/bitcamp-final.jar bitcamp-final.jar
    #6 DONE 1.1s
    
    #7 exporting to image
    #7 exporting layers
    #7 exporting layers 0.3s done
    #7 writing image sha256:ff5b7389237cc2b023712d537cd30de204534ffee8bfe8d3470607c544a551fc 0.0s done
    #7 naming to docker.io/backtoback/bitcamp-final:1.0 done
    #7 DONE 0.4s
    + docker login -u backtoback -p dlrkfka8796!! docker.io
    WARNING! Using --password via the CLI is insecure. Use --password-stdin.
    WARNING! Your password will be stored unencrypted in /var/jenkins_home/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credential-stores
    
    Login Succeeded
    + docker push backtoback/bitcamp-final:1.0
    The push refers to repository [docker.io/backtoback/bitcamp-final]
    0fcc4731f586: Preparing
    10359c5dc4ba: Preparing
    601b48657e0c: Preparing
    b42107e74152: Preparing
    601b48657e0c: Mounted from library/openjdk
    10359c5dc4ba: Mounted from library/openjdk
    b42107e74152: Mounted from library/openjdk
    0fcc4731f586: Pushed
    1.0: digest: sha256:fd2ab8a6624a4adfc8fe411d953eb6ab4d70116358fa5f635a265ff890aa2a83 size: 1166
    Finished: SUCCESS
    ```
    
    ![](image 30.png){: width="300" height="150" .left }  




<br>

<br>

<br>

## (5) 스프링부트 애플리케이션 컨테이너 실행하기

### 작업 디렉토리 만들기

- **작업 디렉토리 만들기**
    - **`root@bitcamp-jenkins:~# mkdir springboot`**
    - **`root@bitcamp-jenkins:~# cd springboot`**
    - **`root@bitcamp-jenkins:~/springboot#`**
    
    ```bash
    [student@bitcamp-svr-final ~]$ mkdir springboot
    [student@bitcamp-svr-final ~]$ cd springboot
    [student@bitcamp-svr-final springboot]$
    ```
    

<br>

### <span class="i2">**ssh key**</span> **생성**

- **docker-jenkins 도커 컨테이너에서 실행**
    - **`root@bitcamp-jenkins:~/springboot# docker exec -itu 0 docker-jenkins bash`**
        
        ```bash
        [student@bitcamp-svr-final springboot]$ sudo docker exec -itu 0 docker-jenkins bash
        [sudo] password for student:
        root@7665e5d7a326:/#
        ```
        
    
    <br>
    
    - **`root@젠킨스컨테이너:/# ssh-keygen -t rsa -C "docker-jenkins-key" -m PEM -P "" -f /root/.ssh/docker-jenkins-key`**
        
        ```bash
        root@7665e5d7a326:/# ssh-keygen -t rsa -C "docker-jenkins-key" -m PEM -P "" -f /root/.ssh/docker-jenkins-key
        Generating public/private rsa key pair.
        Your identification has been saved in /root/.ssh/docker-jenkins-key
        Your public key has been saved in /root/.ssh/docker-jenkins-key.pub
        The key fingerprint is:
        SHA256:5hFiRhI1t3tJv39++4fq6oXmNN6p9jxw2q3htvZCUEA docker-jenkins-key
        The key's randomart image is:
        +---[RSA 3072]----+
        |    oo+ ..E.     |
        |     o o .  .    |
        |      + o ..     |
        |     o . +.o     |
        |        S o..    |
        |       o o..o.   |
        |        . =*+. . |
        |         =o**+o +|
        |         o*B@Bo+B|
        +----[SHA256]-----+
        
        ```
        
    
    <br>
    
    - **`root@젠킨스컨테이너:/# ls ~/.ssh/`**
        - **결과**  &nbsp; **:** &nbsp;     **docker-jenkins-key**  **docker-jenkins-key.pub**
        
        ```bash
        root@7665e5d7a326:/# ls ~/.ssh/
        docker-jenkins-key  docker-jenkins-key.pub
        ```
        
    
    <br>
    
    - **`root@젠킨스컨테이너:/# cat ~/.ssh/docker-jenkins-key`**
        - <span class="i3">**개인키**</span> **내용 출력**
        
        ```bash
        root@7665e5d7a326:/# cat ~/.ssh/docker-jenkins-key
        -----BEGIN RSA PRIVATE KEY-----
        MIIG4wIBAAKCAYEA1o4NZmOLHINy+vV7UzPmbDoCZ5jhH+xCXEGNHQ0u9P6eH/XO
        
        .......
        
        Vu15mf6dOrxaMhe2ix1XaGhFJecHjiUMoXeGtQj/GrRxzkrpuPOv
        -----END RSA PRIVATE KEY-----
        ```
        
    
    <br>
    
    - **`root@젠킨스컨테이너:/# cat ~/.ssh/docker-jenkins-key.pub`**
        - <span class="i3">**공개키**</span> **내용 출력**
        
        ```bash
        
        root@7665e5d7a326:/# cat ~/.ssh/docker-jenkins-key.pub
        ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDWjg1mY4scg3L69XtTM+ZsOg
        
        ......
        
        rITFA8= docker-jenkins-key
        ```
        

<br>

- <span class="i1">**SSH-KEY 개인키 파일**</span>**을** <span class="i3">**젠킨스 홈 폴더**</span>**에 두기**
    - **`root@젠킨스컨테이너:/# cp ~/.ssh/docker-jenkins-key /var/jenkins_home/`**
        - <span class="i1">**Jenkins**</span>에서 <span class="i2">**SSH 키**</span>를 **사용 가능**하게 하는것
    - **`root@젠킨스컨테이너:/# chmod +r /var/jenkins_home/docker-jenkins-key`**
        - 파일 **읽기 권한** 추가
        
        ```bash
        root@7665e5d7a326:/# cp ~/.ssh/docker-jenkins-key /var/jenkins_home/
        root@7665e5d7a326:/# chmod +r /var/jenkins_home/docker-jenkins-key
        ```
        
    

<br>

- <span class="i1">**docker-jenkins 컨테이너**</span>의 <span class="i3">**SSH-KEY 공개키 파일**</span>을 <span class="i2">**Host**</span>로 **복사**해오기
    - **`root@bitcamp-jenkins:~/springboot# docker cp docker-jenkins:/root/.ssh/docker-jenkins-key.pub ./docker-jenkins-key.pub`**
        - **Docker 컨테이너**에서 **파일**을 <span class="i1">**호스트 시스템**</span>으로 **복사**하는 명령어
    - **`root@bitcamp-jenkins:~/springboot# cat docker-jenkins-key.pub`**
        - <span class="i3">**공개키**</span> **내용 출력**
    
    ```bash
    root@7665e5d7a326:/# read escape sequence
    
    [student@bitcamp-svr-final springboot]$ sudo docker cp docker-jenkins:/root/.ssh/docker-jenkins-key.pub ./docker-jenkins-key.pub
    [sudo] password for student:
    Successfully copied 2.56kB to /home/student/springboot/docker-jenkins-key.pub
    
    [student@bitcamp-svr-final springboot]$ cat docker-jenkins-key.pub
    ssh-rsa AAAAB3NzaC1  ....  IIOACrITFA8= docker-jenkins-key
    
    ```
    

<br>

<br>

### 스프링부트 서버에 SSH-KEY 공개키 등록하기

- <span class="i2">**bitcamp-final 서버**</span>**로 이동**
    - **`일반사용자@bitcamp-springboot:~$ sudo -i`**
    - **`root@bitcamp-springboot:~# mkdir .ssh`**
    - **`root@bitcamp-springboot:~# cd .ssh`**
    - **`root@bitcamp-springboot:~/.ssh# vi authorized_keys`**
        - **Jenkins 컨테이너의 공개키 복사**
    
    ```bash
    
    ```
    

<br>

<br>

### Jenkins에 Publish Over SSH 플러그인 설정

- **플러그인 설치**
    - <span class="i2">**Jenkins UI**</span>   &nbsp; ⇒  &nbsp;  **Jenkins 관리**
        - **plugins**
            - <span class="i2">**Available plugins**</span> **탭**
                - **`Publish Over SSH` 플러그인 설치**

<br>

- **플러그인 연동**
    - <span class="i2">**Jenkins UI**</span>    &nbsp; ⇒ &nbsp;   **Jenkins 관리**
        - <span class="i1">**System(시스템 설정)**</span>
            - <span class="i3">**Publish over SSH**</span>
                - Passphrase  &nbsp; : &nbsp;  접속하려는 컨테이너의 root 암호 &nbsp; <===  &nbsp; 암호로 접속할 때
                - Path to key  &nbsp; : &nbsp; docker-jenkins-key  &nbsp; <===  &nbsp; SSH-KEY로 접속할 때(private key 파일 경로, JENKINS_HOME 기준)
                - Key &nbsp; : &nbsp; 개인키 파일의 내용  &nbsp; <===  &nbsp; SSH-KEY 개인키 파일의 내용을 직접 입력할 때
                - **추가 버튼** 클릭
                - <span class="i3">**SSH Servers**</span>
                    - **Name**  &nbsp; : &nbsp; 임의의서버 이름
                    - **Hostname** &nbsp; : &nbsp; 스프링부트서버의 IP 주소
                    - **Username**  &nbsp; :  &nbsp; 사용자 아이디
                    - **`Test Configuration`** 버튼 클릭
                        - <span class="i1">**Success**</span> **OK!**

<br>

<br>

### Jenkins 서버에서 스프링부트 서버를 제어하여 스프링부트 컨테이너 실행하기

- <span class="i2">**bitcamp-final 서버**</span>에서 **docker pull 및 run**
    - <span class="i2">**Jenkins UI**</span>    ⇒    **Dashboard**
        - <span class="i2">**bitcamp-final Freestyle 아이템**</span> **선택**
            - **구성 탭 선택**
                - <span class="i1">**빌드 환경**</span>
                    - <span class="i3">**Send files or execute commands over SSH after the build** runs</span> **선택**
                    - **Transfers**
                        - **Exec command**
                            - **`docker login -u '도커허브아아디' -p '도커허브비번' docker.io`**
                            - **`docker pull [dockerHub UserName]/[dockerHub Repository]:[version]`**
                            - **`docker ps -aq --filter name=[containerName] | grep -q . && docker rm -f $(docker ps -aq --filter name=[containerName])`**
                            - **`docker run -d --name [containerName] -p 80:80 -v /root/config:/root/config [dockerHub UserName]/[dockerHub Repository]:[version]`**
    