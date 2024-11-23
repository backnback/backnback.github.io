---
title: "MySQL 사용 방법"
date: 2024-11-23 14:08:00 +0900
categories: [MySQL, 기초]
tags: [MySQL, 사용방법, DDL]
description: MySQL을 사용하는 방법
math: true
toc: true
pin: false
media_subpath: /assets/img/posts/mysql-howtouse/
---



# ▶  데이터베이스와 DBMS
- mysql-settings

    ```bash
    # MySQL 설치 및 설정
    
    ## MySQL 설치

    [windows OS]
    scoop 패키지 관리자를 이용하여 mysql 설치
    > scoop install MySQL
    
    서비스에 등록
        mysqld --install "서비스명"
    > mysqld --install "MySQL"
    
    서비스 시작
        net start 서비스명
    > net start MySQL    (windows)
    
    서비스 종료
        net stop 서비스명
    > net stop MySQL
    
    [mac]
    mysql 설치
    > brew install mysql
    
    mysql 설치 후 root 암호 변경
    > sudo mysql_secure_installation
    
    mysql 실행
    > brew services start mysql
    > brew services stop mysql
    또는
    > mysql.server start
    > mysql.server stop
    
    [linux]
    mysql 설치
    > sudo apt update
    > sudo apt install mysql-server
    > sudo systemctl status mysql
    > mysql -V
    
    mysql 설치 후 root 암호 변경
    > sudo mysql_secure_installation
    
    mysql 실행
    > service mysql start
    > service mysql stop
    > service mysql status

    ## mysql 서버에 접속하기
    로컬 MySQL 서버에 접속
    > mysql -u root -p
    > Enter password: 암호입력
    
    원격 MySQL 서버에 접속
    > mysql -h 서버주소 -u root -p
    > Enter password: 암호입력
    
    ## mysql root 암호 변경
    > alter user 'root'@'localhost' identified by '1111';
    
    ## MySQL 사용자 추가
    > CREATE USER '사용자아이디'@'서버주소' IDENTIFIED BY '암호';
    
    로컬에서만 접속할 수 있는 사용자를 만들기:
    > CREATE USER 'study'@'localhost' IDENTIFIED BY '1111';
      => 이 경우 stidu 사용자는 오직 로컬(서버를 실행하는 컴퓨터)에서만 접속 가능한다.
      => 다른 컴퓨터에서 실행하는 MySQL 서버에 접속할 수 없다는 것을 의미한다.
    
    원격에서만 접속할 수 있는 사용자를 만들기:
    > CREATE USER 'study'@'%' IDENTIFIED BY '1111';
      => 이 경우 study 사용자는 원력에서만 접속 가능하다.
    
    ## MySQL 사용자 목록 조회
    > select user from 데이터베이스명.테이블명;
    > select user from mysql.user;
    
    ## MySQL 데이터베이스 생성
    > CREATE DATABASE 데이터베이스명
      DEFAULT CHARACTER SET utf8
      DEFAULT COLLATE utf8_general_ci;
    
    > CREATE DATABASE studydb
      DEFAULT CHARACTER SET utf8
      DEFAULT COLLATE utf8_general_ci;
      
    ## MySQL 사용자에게 데이터베이스 사용 권한 부여
    > GRANT ALL ON 데이터베이스명.* TO '사용자아이디'@'서버주소';
    > GRANT ALL ON studydb.* TO 'study'@'localhost';
    
    ## 데이터베이스 목록 조회
    > show databases;
    
    ## 사용자 교체
    > quit    (프로그램 종료 후)
    > mysql -u study -p   (다시 실행)
    
    ## 기본으로 사용할 데이터베이스 지정하기
    > use 데이터베이스명
    > use studydb;
    
    ## 데이터베이스의 전체 테이블 목록 조회
    > show tables;
    ```
    

## ※  DBMS 도입   -   소개

![](image.jpg)


- **DBMS**  &nbsp; : &nbsp;  Database 관리 S/W
    - RDBMS &nbsp; (관계형 DBMS)
        - Oracle,  MS-SQL,  DB2,  MYSQL(오픈소스),  MariaDB,  PostgreSQL,  Tibero,  Cubrid,  Altibase
    - MongoDB,  Redis

<br>

- **데이터베이스**  &nbsp; (DB  :  DataBase)
    - 데이터베이스는 데이터의 집합이다.
    - 검색, 갱신이 쉽도록  구조화
        - 실시간 접근
        - 동시 공유
        - 데이터 중복 최소화
        - 일관성, 무결성, 보안성

<br>

- 일관성  &nbsp; (규칙 준수)
    - 규칙이나 제약 조건을 준수   &nbsp; ⇒ &nbsp;   데이터 신뢰성 확보


<br>
<br>
<br>

## ※  DBMS 도입   -   MySQL 설치

---

![](image 1.jpg){: width="600" height="300" .left }  
<div class="clear"></div>


<br>


- [MySQL DBMS 설치 및 설정](https://www.notion.so/MySQL-003a6b2c180945b3ac782976b9695015?pvs=21)

- MySQL 서버에 직접 접근할 수 없다.   (만약 할 수 있었다면 외부 해커도 가능하니까)
    
    ⇒  &nbsp; Linux 서버를 통해 MySQL 서버로 접근한다.   (외부 접근은 Linux가 방화벽으로 차단한다)
    
    ⇒ &nbsp; 그런데 Linux 서버에서는 CLI로 제어한다. 그러므로 개발자도 CLI 방식을 알아야 한다.
    

<br>
<br>
<br>


## ※  DBMS 도입   -   DBMS 사용

### 1️⃣  로그인   (서버 접속)


- **로컬 MySQL 서버**에 접속
    
    ![](image 2.jpg){: width="300" height="150" .left }  
    <div class="clear"></div>

    <br>
    
    - **명령프롬프트**
        - **`mysql -u root -p`**
        
            ```bash
            C:\Users\BITCAMP>mysql -u root -p
            Enter password: ****
            Welcome to the MySQL monitor.  Commands end with ; or \g.
            Your MySQL connection id is 16
            Server version: 8.4.2 MySQL Community Server - GPL
            
            Copyright (c) 2000, 2024, Oracle and/or its affiliates.
            
            Oracle is a registered trademark of Oracle Corporation and/or its
            affiliates. Other names may be trademarks of their respective
            owners.
            
            Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
            ```
            
            - `mysql`   :   DBMS 클라이언트
            - `-u`   :   user ID
            - `-p`   :   암호 직접 입력


<br>

- **원격 MySQL 서버**에 접속
    
    ![](image 3.jpg){: width="250" height="125" .left }  
    <div class="clear"></div>

    
    ```bash
    mysql -h 서버주소 -u root -p
    ```
    
<br>


- **mysql root 암호 변경**
    
    ```bash
    alter user 'root'@'localhost' identified by '1111';
    ```
    
<br>

- 서비스 시작 및 종료
    - 명령프롬프트  (관리자 권한 실행)
        
        ```bash
        system32$ net stop MySQL84
        
        system32$ net start MySQL84
        
        or  (서비스 창에서 start, stop)
        ```
        

<br>
<br>

### 2️⃣  DBMS 사용자 등록

![](image 4.jpg){: width="450" height="225" .left }  
<div class="clear"></div>

- 똑같은 아이디를 만들 수 있다
- **호스트**  &nbsp; : &nbsp;   접속 허용 서버  &nbsp; (도메인,  IP주소,  % (All : 외부 모든 서버))

<br>

- 로컬에서만 접속할 수 있는 사용자를 만들기
    
    ```bash
    CREATE USER 'study'@'localhost' IDENTIFIED BY '1111';
    ```
    
    - 이 경우 **study 사용자**는 오직 **로컬**(서버를 실행하는 컴퓨터)에서만 **접속 가능**한다.
    - 다른 컴퓨터에서 실행하는 MySQL 서버에 <u>접속할 수 없다</u>는 것을 의미한다.
    - 로그인
        
        ```bash
        $ mysql -u study -p   (비번 1111)
        ```
        
<br>

- **원격**에서만 <u>접속할 수 있는 사용자</u>를 만들기
    
    ```bash
    CREATE USER 'study'@'%' IDENTIFIED BY '2222';   
    CREATE USER 'study'@'%' IDENTIFIED BY '1111';
    ```
    
    - 이 경우 study 사용자는 원격에서만 접속 가능하다.
    - 로그인
        
        ```bash
        $ mysql -h 192.168.0.7 -u study -p   (비번 2222)
        ```
        
<br>

- **사용자 교체**
    
    ```bash
    > quit    (프로그램 종료 후)
    > mysql -u study -p   (다시 실행)
    ```
    

<br>

- **사용자 삭제**

    ![](image 5.jpg){: width="200" height="100" .left }  
    <div class="clear"></div>
    
    ```bash
    $ mysql -u root -p  (1111)
    $ select user, host from mysql.user;
    
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    +------------------+-----------+
    | user             | host      |
    +------------------+-----------+
    | study            | %         |
    | mysql.infoschema | localhost |
    | mysql.session    | localhost |
    | mysql.sys        | localhost |
    | root             | localhost |
    | study            | localhost |
    +------------------+-----------+
    


    $ drop user 'study'@'localhost';
    $ drop user 'study'@'%';
    $ select user, host from mysql.user;
    
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    +------------------+-----------+
    | user             | host      |
    +------------------+-----------+
    | mysql.infoschema | localhost |
    | mysql.session    | localhost |
    | mysql.sys        | localhost |
    | root             | localhost |
    +------------------+-----------+
    ```
    

<br>
<br>

### 3️⃣  데이터베이스 생성


![](image 6.jpg){: width="900" height="450" .left }  
<div class="clear"></div>


```bash
> CREATE DATABASE studydb DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

> CREATE DATABASE studydb
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;
```

- `Collate` &nbsp; : &nbsp; 문자열 정렬 규칙
- `utf8_general_ci`  &nbsp; : &nbsp;  UTF-8 인데 대소문자 구분 없음

<br>

- **데이터베이스 삭제**
    
    ![](image 7.jpg){: width="300" height="150" .left }  
    <div class="clear"></div>
    
    ```bash
    > drop database studydb;
    ```
    

<br>
<br>

### 4️⃣  데이터베이스 사용 권한 설정


![](image 8.jpg){: width="500" height="250" .left }  
<div class="clear"></div>


```bash
> GRANT ALL ON studydb.* TO 'study'@'localhost';

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
Query OK, 0 rows affected (0.01 sec)
```


<br>
<br>

### 5️⃣  데이터베이스 목록 조회

![](image 9.jpg){: width="200" height="100" .left }  
<div class="clear"></div>

- **root 사용자**의 데이터베이스
    
    ```bash
    $ mysql -u root -p   (비번 1111)
    $ show databases;

    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sakila             |
    | studydb            |
    | sys                |
    | world              |
    +--------------------+
    ```
    
<br>

- **study 사용자**의 데이터베이스
    
    ```bash
    $ mysql -u study -p  (1111)
    $ show databases;

    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | performance_schema |
    | studydb            |
    +--------------------+
    ```
    

<br>
<br>

### 6️⃣  데이터베이스 사용 설정

![](image 10.jpg){: width="200" height="100" .left }  
<div class="clear"></div>


- **기본**으로 **사용**할 데이터베이스 **지정**하기
    
    ```bash
    > use studydb;      (이제 부터 쓸거야)
    ```
    
    - <u>지정하지 않으면 테이블 생성 등을 할 수 없다.</u>


<br>
<br>


### 7️⃣  테이블 목록 조회

![](image 11.jpg){: width="150" height="75" .left }  
<div class="clear"></div>


- 기본으로 지정한 **데이터베이스의 테이블**
    
    ```bash
    > show tables
    ```
    

<br>
<br>

### 8️⃣  테이블 상세 조회

![](image 12.jpg){: width="200" height="100" .left }  
<div class="clear"></div>


```bash
> describe 테이블명

> desc 테이블명   (실무에서 쓰는 것)
```


<br>
<br>


### 9️⃣  사용자, 호스트 조회

![](image 13.jpg){: width="400" height="200" .left }  
<div class="clear"></div>

- 로컬 사용자의 mysql 데이터베이스의 user 테이블
    
    ⇒  &nbsp; 현재까지의 사용자를 알 수 있다.
    
    ```bash
    $ select user, host from user;

    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

    +------------------+-----------+
    | user             | host      |
    +------------------+-----------+
    | study            | %         |
    | mysql.infoschema | localhost |
    | mysql.session    | localhost |
    | mysql.sys        | localhost |
    | root             | localhost |
    | study            | localhost |
    +------------------+-----------+**
    ```
    
<br>

- 테이블에서 **원하는 컬럼 조회**
    
    ![](image 14.jpg){: width="500" height="250" .left }  
    <div class="clear"></div>
    
    ```bash
    > select user from 데이터베이스명.테이블명;
    
    > select user, host from mysql.user;
    ```
    
    - 현재 데이터베이스면 **<u>데이터베이스명</u> 생략 가능**