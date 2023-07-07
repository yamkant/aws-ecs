# Docker file 실행방법

### 각 컨테이너의 역할
**nginx**
- 정적 리로스의 요청을 처리합니다.
- Django 서버를 프록시합니다.

**web**
- gunicorn(python wsgi)를 사용하여 동적인 요청을 처리합니다.
- 서버사이드 요청을 wsgi를 통해 Django로 전달합니다.

### 도커 이미지 빌드 및 실행
- root 경로는 현재 디렉토리입니다.
- my-net이라는 네트워크에 nginx, web을 연결하여 nginx는 host와 포트를 개방시키고 내부에서 web으로 요청을 전달합니다.
- 저희가 구성할 ECS의 EC2 리눅스 환경은 amd cpu를 사용합니다.
- 저는 M1을 사용하기 떄문에 빌드 플랫폼을 변경하지만, 로컬환경에서 테스트하는 경우 build platform 관련 명령어는 제거해도 됩니다.

**Network 구성**  
- Docker Bridge Network 모드를 사용하기 위해 네트워크를 생성합니다.
    ```sh
    $ docker network create \   
        --driver=bridge \
        --subnet=172.20.0.0/16 \
        --gateway=172.20.0.1 \
        my-net
    ```

**Nginx**
- Nginx 이미지를 빌드하고, 위에서 생성한 네트워크를 연결하여 실행합니다.
    ```sh
    $ docker buildx build --platform=linux/amd64  \
        -t my-nginx --load ./nginx
    $ docker run --network my-net -d -p 80:80 \
        --name nginx my-nginx  
    ```

**Web**
- Web 이미지를 빌드하고, 위에서 생성한 네트워크를 연결하여 실행합니다.
    ```sh
    $ docker buildx build --platform=linux/amd64  \
        -t my-web --load ./web 
    $ docker run --network my-net -d \
        --env-file ./web/.env \
        --name web my-web  
    ```
- 주의: `.env.sample` 형태로 `web/.env`를 생성해주어야합니다.

### 참고
**wsgi**
- Callable Object를 통해, 요청에 대한 정보를 Application에 전달합니다.
- HTTP Request에 대한 정보와 Callback 함수 두가지를 Callable Object를 통해 정보를 전달해야합니다. 
- WSGI Middleware는 Web Application의 실행 전/후에 기능을 추가합니다.
- 따라서, Authentication, Routing, Session, Cookie, Error Page render와 같이 공통적으로 Application에 적용되어야할 기능을 Middleware를 통해 처리합니다.