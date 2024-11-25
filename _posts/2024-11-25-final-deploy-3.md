---
title: "bitcamp-final 프로젝트 배포하기 3"
date: 2024-11-25 19:08:00 +0900
categories: [NCP, Docker, React, Frontend]
tags: [NCP, Docker, Server]
description: 리엑트 배포하기
math: true
toc: true
pin: true
---




## (11)  bitcamp-final-front 프로젝트 컨테이너 만들기

### Dokcerfile 생성

- **프로젝트 Root 디렉토리**에 **<span class="i2">Dockerfile</span>** **생성**하기
    
    ```bash
    # 1. Node.js 기반의 Alpine 이미지를 사용
    FROM node:18-alpine
    
    # 2. 작업 디렉토리 설정
    WORKDIR /app
    
    # 3. package.json과 package-lock.json을 먼저 복사
    COPY package*.json ./
    
    # 4. 의존성 설치 (npm 사용)
    RUN npm install
    
    # 5. 애플리케이션 소스 코드 복사
    COPY . .
    
    # 6. 포트 설정
    EXPOSE 3000
    
    # 7. 앱 실행 (npm start)
    CMD ["npm", "start"]
    ```
    
    - Git push 하여 업데이트 해준다.

<br>

- **<span class="i1">리눅스 서버</span>**에서 bitcamp-final-front(**프로젝트 Repo.**)를 **clone**해준다.
    
    ```bash
    [student@bitcamp-svr-final ~]$ cd git
    [student@bitcamp-svr-final git]$ ls
    bitcamp-final
    [student@bitcamp-svr-final git]$ git clone https://github.com/backnback/bitcamp-final-front
    Cloning into 'bitcamp-final-front'...
    remote: Enumerating objects: 2137, done.
    remote: Counting objects: 100% (24/24), done.
    remote: Compressing objects: 100% (23/23), done.
    remote: Total 2137 (delta 1), reused 6 (delta 1), pack-reused 2113 (from 1)
    Receiving objects: 100% (2137/2137), 14.82 MiB | 15.65 MiB/s, done.
    Resolving deltas: 100% (1501/1501), done.
    ```
    

<br>

- **<span class="i2">Docker 이미지</span> 빌드**
    - bitcamp-final-front의 루트 디렉토리로 이동하고  <span class="i2">Dockerfile</span>을 이용하여 이미지를 빌드한다.
    - **`sudo docker build -t bitcamp-final-front:first .`**
        - `-t [이미지이름]:[태그]`   :   태그(tag) 지정 옵션
        - `.`   :   현재 디렉토리   (Dockerfile을 찾는 위치)
    
    ```bash
    [student@bitcamp-svr-final git]$ cd bitcamp-final-front
    [student@bitcamp-svr-final bitcamp-final-front]$ sudo docker build -t bitcamp-final-front:first .
    [+] Building 37.8s (11/11) FINISHED                                                                 docker:default
     => [internal] load build definition from Dockerfile                                                          0.0s
     => => transferring dockerfile: 423B                                                                          0.0s
     => [internal] load metadata for docker.io/library/node:18-alpine                                             2.4s
     => [auth] library/node:pull token for registry-1.docker.io                                                   0.0s
     => [internal] load .dockerignore                                                                             0.0s
     => => transferring context: 2B                                                                               0.0s
     => [1/5] FROM docker.io/library/node:18-alpine@sha256:7e43a2d633d91e8655a6c0f45d2ed987aa4930f0792f6d9dd3bff  3.4s
     => => resolve docker.io/library/node:18-alpine@sha256:7e43a2d633d91e8655a6c0f45d2ed987aa4930f0792f6d9dd3bff  0.0s
     => => sha256:7000d2e73f938c4f62fdda6d398d7dffd50e6c129409ae2b1a36ccebf9289ffe 1.72kB / 1.72kB                0.0s
     => => sha256:870e987bd79332f29e9e21d7fa06ae028cfb507bb8bbca058f6eb60f7111cae9 6.20kB / 6.20kB                0.0s
     => => sha256:aa6f657bab0c137ad8a7930532dbd7c53338023e828890090a4070f47a2ed65d 40.09MB / 40.09MB              1.0s
     => => sha256:f477ea663f1c2a017b6f95e743b5ad72f8ff8bf861b0759a86ba1985f706308f 1.39MB / 1.39MB                0.7s
     => => sha256:43c47a581c29baa57713ee0da7af754f3994227b64949cb5e9e4dbbc2108c6cd 449B / 449B                    0.9s
     => => sha256:7e43a2d633d91e8655a6c0f45d2ed987aa4930f0792f6d9dd3bffc7496e44882 7.67kB / 7.67kB                0.0s
     => => extracting sha256:aa6f657bab0c137ad8a7930532dbd7c53338023e828890090a4070f47a2ed65d                     2.1s
     => => extracting sha256:f477ea663f1c2a017b6f95e743b5ad72f8ff8bf861b0759a86ba1985f706308f                     0.1s
     => => extracting sha256:43c47a581c29baa57713ee0da7af754f3994227b64949cb5e9e4dbbc2108c6cd                     0.0s
     => [internal] load build context                                                                             0.0s
     => => transferring context: 37.40kB                                                                          0.0s
     => [2/5] WORKDIR /app                                                                                        0.5s
     => [3/5] COPY package*.json ./                                                                               0.1s
     => [4/5] RUN npm install                                                                                    25.5s
     => [5/5] COPY . .                                                                                            0.2s
     => exporting to image                                                                                        5.6s
     => => exporting layers                           
     
     
     .......
    ```
    

<br>

- **이미지 빌드 확인**
    - **`$ sudo docker images`**
        
        ```bash
        [student@bitcamp-svr-final bitcamp-final-front]$ sudo docker images
        REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
        bitcamp-final-front   first     b8dc875a08da   12 seconds ago   510MB
        backtoback/bitcamp    0.1       d1eb90b730dc   7 days ago       1.35GB
        bitcamp-final         first     d1eb90b730dc   7 days ago       1.35GB
        <none>                <none>    d959fe30f57d   7 days ago       1.35GB
        ubuntu                latest    fec8bfd95b54   5 weeks ago      78.1MB
        hello-world           latest    d2c94e258dcb   19 months ago    13.3kB
        centos                7         eeb6ee3f44bd   3 years ago      204MB
        ubuntu                14.04     13b66b487594   3 years ago      197MB
        
        ```
        
        - bitcamp-final-front 이미지가 존재하는 것을 확인할 수 있다.

<br>

- **<span class="i1">컨테이너 실행하기</span>**
    - **`sudo docker run -d -p 3000:3000 --name bitcamp-final-front bitcamp-final-front:first`**
        - `-d`   :   백그라운드로 실행하는 것이다.
        
        ```bash
        [student@bitcamp-svr-final bitcamp-final-front]$ sudo docker run -d -p 3000:3000 --name bitcamp-final-front bitcamp-final-front:first
        e51e35b65c899ad8f23ba69bb2fd5069d267264b1125f9635bb6130ca2c2c0af
        
        [student@bitcamp-svr-final bitcamp-final-front]$ sudo docker ps -a
        CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS          PORTS                    NAMES
        e51e35b65c89   bitcamp-final-front:first   "docker-entrypoint.s…"   18 seconds ago   Up 17 seconds   0.0.0.0:3000->3000/tcp   bitcamp-final-front
        506a7a2a9ecb   bitcamp-final:first         "/bin/bash"              6 days ago       Up 50 minutes   0.0.0.0:8080->8080/tcp   bitcamp-final
        
        ```
        
        - **`http://211.188.48.5:3000` 으로 접속하면 첫 화면이 보인다.**

<br>

- **접속 후 <span class="i1">벡엔드</span>와 <span class="i2"><u>연결이 되지 않는다면</u></span>**
    
    **[WebConfig.java]**
    
    ```bash
    package bitcamp.project;
    
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
    import org.springframework.web.servlet.config.annotation.CorsRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    
    import javax.sql.DataSource;
    
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
                    .allowedOrigins("http://211.188.48.5:3000", "http://localhost:3000")  // 리액트 앱이 실행되는 주소
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("Authorization", "Content-Type")
                    .allowCredentials(true);
        }
    
    }
    ```
    
    - allowedOrigins 설정이 올바르게 되어 있는지 확인한다.
    - 위에서 설정한 것처럼 `http://211.188.48.5:3000`을 허용해줘야 벡엔드와 프론트엔드가 연결될 수 있다.
        
        설정했다면 꼭 다시 git pull로 최신 상태로 업데이트하고 반드시 build 후 실행해야 한다.
        

<br>
<br>

### % 참고 %

- **Nginx를 사용하여 배포하기**
    
    ```bash
    # 1. Node.js 기반의 이미지를 사용
    FROM node:18 AS build
    
    # 2. 작업 디렉토리 설정
    WORKDIR /app
    
    # 3. package.json과 package-lock.json을 먼저 복사
    COPY package*.json ./
    
    # 4. 의존성 설치
    RUN npm install
    
    # 5. 애플리케이션 빌드
    COPY . .
    RUN npm run build
    
    # 6. Nginx 이미지로 애플리케이션을 서빙
    FROM nginx:alpine
    
    # 7. 빌드된 파일을 Nginx의 기본 웹 서버 디렉토리로 복사
    COPY --from=build /app/build /usr/share/nginx/html
    
    # 8. Nginx 서버를 실행할 포트 지정
    EXPOSE 80
    
    # 9. Nginx 실행
    CMD ["nginx", "-g", "daemon off;"]
    ```