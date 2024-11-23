---
title: "MySQL 설치 및 설정"
date: 2024-11-23 08:08:00 +0900
categories: [MySQL, 기초]
tags: [MySQL, 설치 및 설정]
description: MySQL을 로컬에서 설치하고 설정하는 방법
math: true
toc: true
pin: false
media_subpath: /assets/img/posts/mysql-install/
---



## MySQL 설치 방법


### MySQL을 설치하는 방법

1. **MySQL 사이트**  :  [https://www.mysql.com/](https://www.mysql.com/)  
    <br>

2. 메뉴바의 **Downloads** 클릭  &nbsp; ⇒ &nbsp;  <span class="i1">**MySQL Community (GPL) Downloads**</span> 클릭
    ![](image1.png){: width="600" height="300" }  
    <br>

3. **MySQL Community Server  8.4.3 버전 <span class="i2">LTS</span>**   &nbsp; ⇒ &nbsp;   Windows (x86, 64-bit), MSI Installer 다운로드
    ![](image2.png){: width="600" height="300" }  
    
    <br>

4. **"No thanks, just start my download"** 클릭 &nbsp; (로그인 필수 아님)
    
    <br>


5. 다운로드 받은 MSI Installer 설치 시작  &nbsp; (**Complete 버전** 설치)
    
    <br>


6. **MySQL Configurator**  
    next  ⇒  next  ⇒  show advanced and logging options 체크  ⇒  next  ⇒  비밀번호 1111  ⇒  next  ⇒  next 
    ⇒  next  ⇒  next  ⇒  next  ⇒  Sample Databases(둘 다 체크하기 (Sakila, World))  ⇒  next
    ⇒  (다른 버전 있다면 삭제)  ⇒  execute 클릭  ⇒  next  ⇒  finish
    

<br>
<br>
<br>


### MySQL 환경 설정 및 시작하기  (CLI 방식)


1. 명령 프롬프트  
    - **`$ mysql`**
    
    ```bash
    $ mysql
    ```
    ⇒ &nbsp;&nbsp;  **(x)** &nbsp; 경로를 찾지 못한다.

    <br>

2. 시스템 환경 변수  &nbsp; ⇒ &nbsp;  환경 변수    &nbsp; ⇒ &nbsp;   시스템 변수    &nbsp; ⇒ &nbsp;   path

    <br>

    
3. 새로 만들기  &nbsp; : &nbsp;   `C:\Program Files\MySQL\MySQL Server 8.4\bin`
    - 설치된 MySQL 폴더 내부의 bin 폴더 위치

    <br>
 
    
4. **명령 프롬프트 <span class="i2"><u>창 닫고 다시 열기</u></span>**
    - 환경 변수를 변경하면 다시 열어야 수정된 환경 변수를 활용할 수 있다.

    <br>

    
5. 명령 프롬프트
    
    ```bash
    $ mysql
    
    ==>  ERROR 1045 (28000): Access denied for user 'ODBC'@'localhost' (using password: NO)
    ```
    
    - 이렇게 나오는 것이 **정상**이다.

    <br>

6. 명령 프롬프트    
    - **`$ mysql -u root -p`** &nbsp; &nbsp; (`-p`  &nbsp; : &nbsp;  비밀번호 직접 입력)

    ```bash
    $ mysql -u root -p
    
    Enter password: ****   (설치할 때 설정한 비밀번호 : 1111)
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 15
    Server version: 8.4.2 MySQL Community Server - GPL
    
    Copyright (c) 2000, 2024, Oracle and/or its affiliates.
    
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    ```
    
    - 위의 명령어를 입력하면 비밀번호를 입력하라고 한다. 
    - 설치할 때 설정한 비밀번호를 입력하면 접속할 수 있다.

    <br>

7. 명령 프롬프트
    - **`$ quit`** or **`$ exit`**
    
    ```bash
    $ quit
    ```
    
    - 나가기 명령어