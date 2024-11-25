---
title: "bitcamp-final 프로젝트 배포하기 1"
date: 2024-11-20 15:08:00 +0900
categories: [NCP, Docker, Server]
tags: [NCP, Docker, Server]
description: 최종 프로젝트를 리눅스 서버 생성 후 도커를 사용하여 배포하기
math: true
toc: true
pin: true
---



## (1)  배포 파일 생성 및 실행 확인

---

### build.gradle 변경
- 스프링부트 실행 jar 파일을 만들 때 파일명 설정
- jar 태스크를 수행하지 않게 설정
- **[build.gradle]**
    
    ```java
    plugins {
        id 'java'
        id 'org.springframework.boot' version '2.7.18'
        id 'io.spring.dependency-management' version '1.1.6'
    }
    
    ...
    
    // 스프링부트 실행 jar 파일을 만들 때 파일명 설정
    bootJar {
        archiveFileName = "${rootProject.name}.jar"
    }
    
    // jar 태스크를 수행하지 않게 설정
    tasks.named("jar") {
        enabled = false
    }
    
    ...
    
    ```  
        
    
<br>

### Gradle 명령어
- **`$ gradle help --task bootJar`**
    - 특정 태스크(bootJar)에 대한 도움말 출력   
    <br>
- **`$ gradle build --console=plain`**
    - 프로젝트 빌드
        - `C:\Users\river\git\bitcamp-final\app\build\libs`에 bitcamp-final.jar 생긴다.  
        (settings.gradle의 `rootProject.name`을 설정한대로)
    - `--console=plain`   :   출력 형식  =  평문 설정   (깔끔하게 로그 확인)
        
        ```bash
        river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
        $ gradle build --console=plain
        > Task :app:compileJava UP-TO-DATE
        > Task :app:processResources UP-TO-DATE
        > Task :app:classes UP-TO-DATE
        > Task :app:bootJarMainClassName UP-TO-DATE
        > Task :app:bootJar UP-TO-DATE
        > Task :app:jar SKIPPED
        > Task :app:assemble UP-TO-DATE
        > Task :app:compileTestJava UP-TO-DATE
        > Task :app:processTestResources UP-TO-DATE
        > Task :app:testClasses UP-TO-DATE
        > Task :app:test UP-TO-DATE
        > Task :app:check UP-TO-DATE
        > Task :app:build UP-TO-DATE

        BUILD SUCCESSFUL in 1s
        7 actionable tasks: 7 up-to-date
        ```

<br>


### Jar 파일 실행하기
- **`$ java -jar bitcamp-final.jar`**

    ```bash
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ cd app/build/libs

    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final/app/build/libs (backnback)
    $ ls
    bitcamp-final.jar

    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final/app/build/libs (backnback)
    $ java -jar bitcamp-final.jar
    ```
    
<br>
- 단, JAR 파일로 실행 시 정적 파일을 올바르게 로드할 수 있도록 
    
    ⇒  &nbsp;  즉, 기본 경로를 사용하기 위해서 application.properties에 아래 코드가 있다면 주석 처리
    
    - `spring.web.resources.static-locations=file:src/main/resources/static`
        - 정적 리소스(static resources)의 위치를 명시적으로 지정
        - 주석 처리 시, Spring Boot는 기본적으로 `/static`, `/public`, `/resources`, `/META-INF/resources` 디렉토리에서 정적 리소스를 검색
        - 애플리케이션이 실행되는 환경에 따라 해당 파일 경로가 유효하지 않을 수 있기 때문이다.
    

<br>
<br>

## (2)  스프링부트 설정 파일을 개발과 운영으로 분리

---

### 개발 모드와 운영 모드를 분리
- application-prod.properties
- application-dev.properties


<br>


### 실행 옵션
- **JVM 아규먼트**   &nbsp; : &nbsp;   `-Dspring.profiles.active=dev`
    - 예)   `$ java -Dspring.profiles.active=dev -jar bitcamp-final.jar`
    - 예)   Gradle  :  환경변수를 통해 설정
        - `$ export SPRING_PROFILES_ACTIVE=dev`
            
            `$ gradle bootRun`
            
    - 예)   IntelliJ  :  환경변수를 통해 설정한다.
        - bootRun   &nbsp; ⇒ &nbsp;   구성   &nbsp; ⇒ &nbsp;   편집  :  `spring.profiles.active=dev`
<br>
- **프로그램 아규먼트**   &nbsp; : &nbsp;   ``--spring.profiles.active=dev``
    - 예)   `$ java -jar bitcamp-final.jar --spring.profiles.active=prod`
    - 예)   `$ gradle bootRun --args='--spring.profiles.active=dev'`
<br>
- intelliJ 에서 application.properties 이름 지정해주기
    
    bootRun의 구성 편집  &nbsp; ⇒ &nbsp;   실행 항목   &nbsp; ⇒ &nbsp;   실행 창  :  `bootRun --args='--spring.profiles.active=dev’`
    


<br>



### JVM 아규먼트로 실행하기
- dev 모드 실행 예시

    ```bash
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final/app/build/libs (backnback)
    $ cd ~/git/bitcamp-final

    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ cd app/build/libs

    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final/app/build/libs (backnback)
    $ java -Dspring.profiles.active=dev -jar bitcamp-final.jar

    ```
    

<br>



### 프로그램 아규먼트로 실행하기
- dev 모드 실행 예시

    ```bash
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final/app/build/libs (backnback)
    $ java -jar bitcamp-final.jar --spring.profiles.active=dev
    ```
    

<br>



### Gradle bootRun 실행 방법
- **루트 프로젝트**로 이동 후
    
    ```bash
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ gradle bootRun --args='spring.profiles.active=dev'
    
    ```
    

<br>
<br>


## (3)  NCP 리눅스 서버 생성

---

### ACG 설정
1. console  &nbsp; ⇒ &nbsp;   server   &nbsp; ⇒ &nbsp;   ACG  &nbsp; ⇒ &nbsp;  생성   &nbsp; ⇒ &nbsp;   bitcamp-vpc-web-acg  &nbsp;   (bitcamp-vpc 선택)
2. Inbound 설정
    1. TCP   &nbsp; : &nbsp;  접근 소스  (0.0.0.0/0)      허용 포트 (1-65535)
    2. UDP   &nbsp; : &nbsp;  접근 소스  (0.0.0.0/0)      허용 포트 (1-65535)
    3. ICMP  &nbsp; : &nbsp;  접근 소스  (0.0.0.0/0) 
3. Outbound 설정
    1. TCP   &nbsp; : &nbsp;  목적지  (0.0.0.0/0)      허용 포트 (1-65535)
    2. UDP   &nbsp; : &nbsp;  목적지  (0.0.0.0/0)      허용 포트 (1-65535)
    3. ICMP  &nbsp; : &nbsp;  목적지  (0.0.0.0/0) 
    


<br>


### 리눅스 서버 생성
- console  &nbsp; ⇒ &nbsp;   server   &nbsp; ⇒ &nbsp;   생성 (기존 콘솔 화면)   &nbsp; ⇒  &nbsp;  CentOS-7.8-64,  High CPU
    - bitcamp-vpc,    bitcamp-vpc-web-subnet 10.0.1.0/24
    - High CPU(vCPU 2개, 4GB, 50GB, g2),  시간 요금제
    - 서버 이름 &nbsp; : &nbsp; bitcamp-svr-final
    - network interface의 ip 입력  &nbsp; : &nbsp;  10.0.1.51   &nbsp;  ⇒ &nbsp;   새로운 공인 IP 할당
    - 물리 배치 그룹 미사용 / 반납 보호 해제
    - 인증키 이용
        - 새로운 인증키 생성  &nbsp;&nbsp;  (이름 예시 : bitcamp0524-95)
    - 네트워크 접근 설정 (ACG)
        - bitcamp-vpc-web-acg
    - 서버 생성


<br>


### SSH    ( telnet(원격 접속) + 보안 )
- ncp에서 만든 서버의 공인 IP를 복사하여 접속
- 서버 관리 및 설정 변경  &nbsp; ⇒  &nbsp;   서버 창에서 관리자 비밀번호 확인 클릭
    
    ⇒   우리 인증키를 주면서 비밀번호 확인하기   &nbsp; ⇒  &nbsp;  다시 git bash에서 비밀번호 넣기
    
    ```jsx
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ ssh root@211.188.48.5
    The authenticity of host '211.188.48.5 (211.188.48.5)' can't be established.
    ED25519 key fingerprint is SHA256:bmFD/+yEZhyPRF/BE85yj9e2/Fd1vFpKP6B9kqjSZHo.
    This key is not known by any other names.
    Are you sure you want to continue connecting (yes/no/[fingerprint])?
    Please type 'yes', 'no' or the fingerprint: yes
    Warning: Permanently added '211.188.48.5' (ED25519) to the list of known hosts.
    root@211.188.48.5's password:
    [root@bitcamp-svr-final ~]#
    
    ```
    
    - 관리자 모드는 **#**으로 시작한다.  &nbsp;&nbsp;   (일반 사용자 : **$**)
    

<br>
<br>


## (3-1)  일반 사용자 생성

---

### student 유저 생성
- **`# useradd student`**
    
    ```jsx
    [root@bitcamp-svr-final ~]# useradd student
    [root@bitcamp-svr-final ~]# cd ..
    [root@bitcamp-svr-final ~]# ls
    bin   dev  home   lib    media  opt   root  sbin  sys  usr
    boot  etc  home1  lib64  mnt    proc  run   srv   tmp  var
    [root@bitcamp-svr-final /]# cd home
    [root@bitcamp-svr-final home]# ls
    student
    [root@bitcamp-svr-final home]# cd student
    [root@bitcamp-svr-final student]# ls
    [root@bitcamp-svr-final student]# ls -al
    total 12
    drwx------  2 student student  62 Nov  5 14:34 .
    drwxr-xr-x. 3 root    root     21 Nov  5 14:34 ..
    -rw-r--r--  1 student student  18 Apr  1  2020 .bash_logout
    -rw-r--r--  1 student student 193 Apr  1  2020 .bash_profile
    -rw-r--r--  1 student student 231 Apr  1  2020 .bashrc
    [root@bitcamp-svr-final student]#
    ```
        

<br>

- Unix Shell에서 sh(bourne shell)와 bash(bourne again shell)의 차이
    - `bash`는 `sh`의 상위 호환으로, 더 현대적이고 스크립팅에 유용한 기능을 많이 제공한다
    
    ```haskell
    [root@bitcamp-svr-final **student]# nano**
    ```
    
    - 나가기  &nbsp; : &nbsp;  ctrl + x
    - **`nano`**  &nbsp; : &nbsp;  텍스트 파일을 편집할 때 사용.
    - **`cat`** &nbsp; : &nbsp;  텍스트 파일의 내용을 출력하거나 여러 파일을 병합할 때 사용
    
    ```jsx
    [root@bitcamp-svr-final student]# cat .bash_profile
    # .bash_profile
    
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
            . ~/.bashrc
    fi
    
    # User specific environment and startup programs
    
    PATH=$PATH:$HOME/.local/bin:$HOME/bin
    
    export PATH
    
    ```
    
    ```jsx
    [root@bitcamp-svr-final student]# cat .bashrc
    # .bashrc
    
    # Source global definitions
    if [ -f /etc/bashrc ]; then
            . /etc/bashrc
    fi
    
    # Uncomment the following line if you don't like systemctl's auto-paging feature:
    # export SYSTEMD_PAGER=
    
    # User specific aliases and functions
    
    ```
    
    - 보통 .bashrc는 손대지 않는다.


<br>


- **명령어**
    - `# cd`  &nbsp;  ⇒  &nbsp;   현재 로그인한 디렉토리로 돌아가기
    - `# whoami` &nbsp;   ⇒  &nbsp;  내가 누구야?   root
    - `# pwd`  &nbsp;  ⇒ &nbsp;    /root
    - `# exit`   &nbsp; ⇒  &nbsp;   아예 나가기


<br>


### student 유저 비밀번호 생성 
- **`# passwd student`**
    
    ```jsx
    [root@bitcamp-svr-final ~]# passwd student
    Changing password for user student.
    New password:
    Retype new password:
    passwd: all authentication tokens updated successfully.
    ```
    

<br>


### 서버에 student로 접속하기
- **`ssh student@IP`**
    ```jsx
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ ssh student@211.188.48.5
    student@211.188.48.5's password:
    [student@bitcamp-svr-final ~]$
    ```
    
    - 가능한 일반 사용자로 접속해서 사용하자 !!


<br>
<br>


## (3-2)  Root SSH 접속 제한하기

---

### root 사용자 로그인 불가하게 설정     (Root SSH 접속 제한하기)
- **`# nano /etc/ssh/sshd_config`**
    
    ```jsx
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ ssh root@211.188.48.5
    root@211.188.48.5's password:
    [root@bitcamp-svr-final ~]# cat /etc/ssh/sshd_config
    #       $OpenBSD: sshd_config,v 1.100 2016/08/15 12:32:04 naddy Exp $
    
    # This is the sshd server system-wide configuration file.  See
    # sshd_config(5) for more information.
    
    # This sshd was compiled with PATH=/usr/local/bin:/usr/bin
    
    ...
    
    # Authentication:
    
    #LoginGraceTime 2m
    PermitRootLogin yes    (이것을 찾는다)
    #StrictModes yes
    #MaxAuthTries 6
    #MaxSessions 10
    
    #PubkeyAuthentication yes
    
    # The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
    # but this is overridden so installations will only check .ssh/authorized_keys
    AuthorizedKeysFile      .ssh/authorized_keys
    
    #AuthorizedPrincipalsFile none
    
    ...
    
    ```
    
    - `PermitRootLogin yes` 를 찾는다.


<br>

- **`PermitRootLogin yes`**   &nbsp;&nbsp;  ⇒  &nbsp;&nbsp;   **`PermitRootLogin no`**  로 수정
    
    ```haskell
    [root@bitcamp-svr1 ~]# nano /etc/ssh/sshd_config
    ```
    
    - **CTRL + O**   (저장)  &nbsp;&nbsp;  ⇒  &nbsp;&nbsp;  **CTRL + X**   (나가기)


<br>


- **변경 사항 적용하기**
    
    ```haskell
    [root@bitcamp-svr-final ~]# systemctl restart sshd.service
    ```
    
    ⇒  &nbsp;  이제 직접 root로 로그인 할 수 없다.
    

<br>

### root 권한 요구
- 그러나 일반 사용자로 사용할 때 root 권한이 필요한 경우가 있다.
    
    ```jsx
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ ssh student@211.188.48.5
    student@211.188.48.5's password:
    [student@bitcamp-svr-final ~]$ yum update
    Loaded plugins: fastestmirror, langpacks
    You need to be root to perform this command.
    
    ```
    
    - **You need to be root to perform this command.**
        
        ⇒   &nbsp; root 권한으로 수행하라고 한다.
        

<br>
<br>


## (3-3)  visudo를 사용하여 일반 사용자에게 sudo 명령어 권한 주기

---

### root로 로그인 하지 않고 root 권한으로 실행하는 법
- **sudo** &nbsp;  : &nbsp; super user do
- **yum**   &nbsp; : &nbsp;  pakage manager
- **`$ sudo yum update`**   &nbsp; :  &nbsp;  yum 서버의 최신 프로그램 목록 가져오기

    ```jsx
    river@BOOK-VKBCE9E5R0 MINGW64 ~/git/bitcamp-final (backnback)
    $ ssh student@211.188.48.5
    student@211.188.48.5's password:
    [student@bitcamp-svr-final ~]$ sudo yum update

    We trust you have received the usual lecture from the local System
    Administrator. It usually boils down to these three things:

        #1) Respect the privacy of others.
        #2) Think before you type.
        #3) With great power comes great responsibility.

    [sudo] password for student:
    student is not in the sudoers file.  This incident will be reported.

    ```

<br>


### visudo 사용하기   (저장할 때 문법 체크해주는 기능)
- 루트(Root)로 전환  &nbsp; (**`$ su`**)
- **`# visudo`**  &nbsp;&nbsp; (/etc/sudoerc 파일 편집)
    - **esc** 누르고 **a**를 누르면 엔터를 입력할 수 있다.
    - 맨 마지막 줄에 한 줄 띄우고 **`student ALL=(ALL:ALL) ALL`**
    - 저장할 때 **esc** 누르고    **:**  이거 입력하고 **wq** 입력
- **`$ exit`**로 student로 다시 돌아가기
    
    ```jsx
    [student@bitcamp-svr-final ~]$ sudo yum update
    
    We trust you have received the usual lecture from the local System
    Administrator. It usually boils down to these three things:
    
        #1) Respect the privacy of others.
        #2) Think before you type.
        #3) With great power comes great responsibility.
    
    [sudo] password for student:
    student is not in the sudoers file.  This incident will be reported.
    [student@bitcamp-svr-final ~]$ su
    Password:ㅓ
    [root@bitcamp-svr-final student]# visudo
    [root@bitcamp-svr-final student]# exit
    exit
    [student@bitcamp-svr-final ~]
    
    ```

<br>    

- 다시 **`$ sudo yum update`** 실행
    
    ```jsx
    [student@bitcamp-svr-final ~]$ sudo yum update
    [sudo] password for student:
    Loaded plugins: fastestmirror, langpacks
    Determining fastest mirrors
    
    ....
    ....   (업데이트 실행 중)
    ....
    Complete!
    ```
    
    - 커널은 업데이트 하면 안된다.



<br>
<br>


## (3-4)  JDK를 설치 하기

---

### 설치 확인하기
- **`$ java -version`**

    ```jsx
    [student@bitcamp-svr-final ~]$ java -version
    -bash: java: command not found
    
    [student@bitcamp-svr-final ~]$ sudo yum install java-21-openjdk
    [sudo] password for student:
    Loaded plugins: fastestmirror, langpacks
    Loading mirror speeds from cached hostfile
    No package java-21-openjdk available.
    Error: Nothing to do
    ```
    
    - 설치가 안되어 있는 것을 확인할 수 있다.


<br>


### 설치 가능 리스트부터 확인하기
- **`$ sudo yum list java*jdk-devel`**
    ```jsx
    [student@bitcamp-svr-final ~]$ sudo yum list java*jdk-devel
    Loaded plugins: fastestmirror, langpacks
    Determining fastest mirrors
    Available Packages
    java-1.6.0-openjdk-devel.x86_64        1:1.6.0.41-1.13.13.1.el7_3         base
    java-1.7.0-openjdk-devel.x86_64        1:1.7.0.261-2.6.22.2.el7_8         base
    java-1.8.0-openjdk-devel.i686          1:1.8.0.412.b08-1.el7_9            update
    java-1.8.0-openjdk-devel.x86_64        1:1.8.0.412.b08-1.el7_9            update
    java-11-openjdk-devel.i686             1:11.0.23.0.9-2.el7_9              update
    java-11-openjdk-devel.x86_64           1:11.0.23.0.9-2.el7_9              update
    
    ```
    
    - jdk-21이 없음


<br>

### Zulu OpenJDK로 JDK 설치
- **`$ sudo yum install -y https://cdn.azul.com/zulu/bin/zulu-repo-1.0.0-1.noarch.rpm`**
        
        ⇒    Azul의 Zulu JDK 패키지를 쉽게 설치할 수 있는 **환경 설정**
        
        ```haskell
        [student@bitcamp-svr-final ~]$ sudo yum install -y https://cdn.azul.com/zulu/bin/zulu-repo-1.0.0-1.noarch.rpm
        
        Loaded plugins: fastestmirror, langpacks
        zulu-repo-1.0.0-1.noarch.rpm                             | 3.0 kB     00:00
        Examining /var/tmp/yum-root-6gw8Yd/zulu-repo-1.0.0-1.noarch.rpm: zulu-repo-1.0.0-1.noarch
        Marking /var/tmp/yum-root-6gw8Yd/zulu-repo-1.0.0-1.noarch.rpm to be installed
        Resolving Dependencies
        --> Running transaction check
        ---> Package zulu-repo.noarch 0:1.0.0-1 will be installed
        --> Finished Dependency Resolution
        
        Dependencies Resolved
        
        ================================================================================
         Package        Arch        Version        Repository                      Size
        ================================================================================
        Installing:
         zulu-repo      noarch      1.0.0-1        /zulu-repo-1.0.0-1.noarch      0.0
        
        Transaction Summary
        ================================================================================
        Install  1 Package
        
        Installed size: 0
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
          Installing : zulu-repo-1.0.0-1.noarch                                     1/1
          Verifying  : zulu-repo-1.0.0-1.noarch                                     1/1
        
        Installed:
          zulu-repo.noarch 0:1.0.0-1
        
        Complete!
        ```
        
<br>

    
- **`$ sudo yum install zulu21-jdk`**    &nbsp;&nbsp;   (`-y` 명령어로 자동 yes 처리 가능)
    
    ```haskell
    [student@bitcamp-svr-final ~]$ sudo yum install -y zulu21-jdk
    Loaded plugins: fastestmirror, langpacks
    Loading mirror speeds from cached hostfile
        * epel: repo.jing.rocks
    zulu-openjdk                                             | 2.9 kB     00:00
    zulu-openjdk/primary_db                                    | 906 kB   00:00
    Resolving Dependencies
    --> ....
    --> Finished Dependency Resolution
    
    Dependencies Resolved ...
    
    Installing:...
    Install  1 Package (+9 Dependent packages)
    
    Total download size: 132 M
    Installed size: 293 M
    Is this ok [y/d/N]: y
    Downloading packages:
    (1/10): ....
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction
        Installing : .....
    
    Installed:
        zulu21-jdk.x86_64 0:21.0.5-3
    
    Dependency Installed:
        ....
    
    Complete!
    ```
        

<br>


### JDK 설치 확인
- **`$ which javac`**   &nbsp; : &nbsp;  javac 실행 파일의 경로 출력
    
    ```jsx
    [student@bitcamp-svr-final ~]$ javac -version
    javac 21.0.5
    [student@bitcamp-svr-final ~]$ which javac
    /usr/bin/javac
    [student@bitcamp-svr-final ~]$ ls -al /usr/bin/javac
    lrwxrwxrwx 1 root root 23 Nov  5 15:43 /usr/bin/javac -> /etc/alternatives/javac
    [student@bitcamp-svr-final ~]$ echo $JAVA_HOME
    
    ```
    
    - **JAVA_HOME**이 설정 안되어 있는 것을 확인할 수 있다.



<br>


## (3-5)  JAVA_HOME 환경 변수 설정

---

### JAVA_HOME  설정하기
- **JAVA_HOME**
    ```jsx
    [student@bitcamp-svr-final ~]$ cd /usr/lib
    [student@bitcamp-svr-final lib]$ ls
    binfmt.d   games   locale          python2.7         tmpfiles.d
    cpp        gcc     modprobe.d      rpm               tuned
    crda       grub    modules         sendmail          udev
    debug      jvm     modules-load.d  sendmail.postfix  yum-plugins
    dracut     kbd     NetworkManager  sse2
    firewalld  kdump   os-release      sysctl.d
    firmware   kernel  polkit-1        systemd
    [student@bitcamp-svr-final lib]$ cd jvm
    [student@bitcamp-svr-final jvm]$ ls
    java-21-zulu-openjdk  java-21-zulu-openjdk-ca  jre  zulu21  zulu21-ca
    [student@bitcamp-svr-final jvm]$ ls -al
    total 4
    drwxr-xr-x   3 root root  107 Nov  5 15:43 .
    dr-xr-xr-x. 30 root root 4096 Nov  5 15:42 ..
    lrwxrwxrwx   1 root root   23 Nov  5 15:43 java-21-zulu-openjdk -> java-21-zulu-openjdk-ca
    drwxr-xr-x   9 root root  163 Oct 16 19:18 java-21-zulu-openjdk-ca
    lrwxrwxrwx   1 root root   21 Nov  5 15:43 jre -> /etc/alternatives/jre
    lrwxrwxrwx   1 root root    9 Nov  5 15:43 zulu21 -> zulu21-ca
    lrwxrwxrwx   1 root root   36 Nov  5 15:43 zulu21-ca -> /usr/lib/jvm/java-21-zulu-openjdk-ca
    
    [student@bitcamp-svr-final jvm]$ cd java-21-zulu-openjdk
    [student@bitcamp-svr-final java-21-zulu-openjdk]$ pwd
    /usr/lib/jvm/java-21-zulu-openjdk
    ```
    
    - 출력된 결과(`/usr/lib/jvm/java-21-zulu-openjdk`)를 **JAVA_HOME**으로 잡는다.


<br>


### `$ nano ~/.bash_profile`
- **루트 경로**로 이동 후  **`nano .bash_profile`**
- **`export JAVA_HOME=/usr/lib/jvm/java-21-zulu-openjdk`**
    
    ```haskell
    [student@bitcamp-svr-final java-21-zulu-openjdk]$ cd
    
    [student@bitcamp-svr-final ~]$ nano .bash_profile
    ```
    - **ctrl + o**  (저장) &nbsp;  ⇒ &nbsp;  **enter** &nbsp; ⇒ &nbsp;  **ctrl + x**  (나가기)

<br>

- **.bash_profile**를 바꾸면 나갔다가 들어오거나 아래의 코드를 입력 해야 한다.
    
    ```jsx
    [student@bitcamp-svr-final ~]$ source .bash_profile
    ```
    

<br>


### 환경변수 확인
- **`$ echo $JAVA_HOME`**
    ```jsx
    [student@bitcamp-svr-final ~]$ echo $JAVA_HOME
    /usr/lib/jvm/java-21-zulu-openjdk
    
    [student@bitcamp-svr-final ~]$ javac -version
    javac 21.0.5
    [student@bitcamp-svr-final ~]$ java -version
    openjdk version "21.0.5" 2024-10-15 LTS
    OpenJDK Runtime Environment Zulu21.38+21-CA (build 21.0.5+11-LTS)
    OpenJDK 64-Bit Server VM Zulu21.38+21-CA (build 21.0.5+11-LTS, mixed mode, sharing)
    
    ```
    

<br>


## (3-6)  리눅스 서버에 프로젝트 폴더를 가져오기 (GIT CLONE)

---

### 프로젝트 가져오기
- 사용자 홈폴더에서

    ```jsx
    [student@bitcamp-svr-final ~]$ mkdir git
    [student@bitcamp-svr-final ~]$ cd git
    [student@bitcamp-svr-final git]$ git clone https://github.com/backnback/bitcamp-final
    Cloning into 'bitcamp-final'...
    remote: Enumerating objects: 98, done.
    remote: Counting objects: 100% (98/98), done.
    remote: Compressing objects: 100% (71/71), done.
    remote: Total 98 (delta 15), reused 95 (delta 15), pack-reused 0 (from 0)
    Unpacking objects: 100% (98/98), done.
    ```
    
<br>


### gradle은 따로 설치할 필요 없다.
- **`$ ./gradlew build`**
    ```jsx
    [student@bitcamp-svr-final git]$ cd bitcamp-final
    [student@bitcamp-svr-final bitcamp-final]$ ./gradlew build
    -bash: ./gradlew: Permission denied
    [student@bitcamp-svr-final bitcamp-final]$ chmod +x gradlew
    [student@bitcamp-svr-final bitcamp-final]$ ./gradlew build
    Downloading https://services.gradle.org/distributions/gradle-8.10.2-bin.zip
    .............10%.............20%.............30%.............40%.............50%.............60%.............70%.............80%.............90%.............100%
    
    Welcome to Gradle 8.10.2!
    
    Here are the highlights of this release:
     - Support for Java 23
     - Faster configuration cache
     - Better configuration cache reports
    
    For more details see https://docs.gradle.org/8.10.2/release-notes.html
    
    Starting a Gradle Daemon (subsequent builds will be faster)
    
    BUILD SUCCESSFUL in 1m 25s
    5 actionable tasks: 5 executed
    
    ```
    
    - permission denied가 뜨는 경우 권한을 바꿔줘야 한다.   (`chmod`)


<br>

- build 후 JAR 파일 실행 시 실행 안된다.
    
    ⇒    **ncp.properties** 파일이 서버에 존재하지 않기 때문에 실행할 수 없다.
    
    - 우리의 경우는 **final.properties**와 **email.properties**


<br>
<br>


## (4)  NCP 보안 파일 생성

---

### 서버에 NCP 보안 파일 만들기
- 사용자 홈폴더에서
    - **`$ mkdir config`**
    - **`$ cd config`**
    - **`$ nano final.properties`**      (**로컬 파일**의 내용 복사 후 붙여넣기)
    
    ```jsx
    [student@bitcamp-svr-final bitcamp-final]$ cd
    [student@bitcamp-svr-final ~]$ mkdir config
    [student@bitcamp-svr-final ~]$ cd config
    [student@bitcamp-svr-final config]$ nano final.properties
    ```
    - **ctrl + o**  (저장)  &nbsp; ⇒ &nbsp;  **enter**  &nbsp; ⇒ &nbsp;  **ctrl + x**  (나가기)

<br>

- **email.properties**도 같은 방법으로 처리
    
    ```jsx
    [student@bitcamp-svr-final config]$ nano email.properties
    ```
    
    - **ctrl + o**  (저장)  &nbsp; ⇒ &nbsp;  **enter**  &nbsp; ⇒ &nbsp;  **ctrl + x**  (나가기)


<br>
<br>


## (5)  애플리케이션 배치 및 실행

---

### 애플리케이션 배치 및 실행
- Git 저장소를 가져와서 build 후 JAR 파일 실행
    ```haskell
    ~$ mkdir git
    ~$ cd git
    ~/git$ git clone https://github.com/backnbakc/bitcamp-final
    ~/git$ cd bitcamp-final
    ~/git/bitcamp-myapp$ ./gradlew build
    ~/git/bitcamp-myapp$ java -jar ./app/build/libs/bitcamp-final.jar
    ```
    
    - 현재는 **DB**에 접속 포트가 허용되지 않아서 db의 데이터를 가져오지 못한다.
    
    
<br>
<br>


## (6)  MySQL ACL에 접속 서버의 IP 등록

---

### NCP 콘솔에서 DB 허용 추가하기
- 서버  &nbsp; ⇒ &nbsp;    acg   &nbsp; ⇒ &nbsp;  mysql  &nbsp; ⇒ &nbsp;   **inbound 규칙** 수정
    - (TCP : **211.188.48.5/32** ,   허용 포트 : **1-65535**)