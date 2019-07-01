# Mobius_Docker



1. 도커

2. 도커 컴포즈 

sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

git clone https://github.com/IoTKETI/Mobius.git

3. 모비우스 & npm install

4. 모비우스 db 패스워드

4. 환경설정 

    
version: '2.1'  # Docker-compose 버전 입니다. 명령어 들이 상이합니다. 

services:  # 각 가상화 소프트웨어들을 서비스로 정의합니다. 


 db:
    image: mysql:5.7    # Docker-Hub에 저장된 mysql Docker 이미지를 가져옵니다. 
    environment:        # mysql 환경변수 설정입니다. 
      MYSQL_ROOT_PASSWORD: root
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
    ports:    # 가상화된 독립적인 환경을 갖는 소프트웨어 이기에 HOST와 Port 매칭이 필요합니다.  "HOST:Docker-container" 
      - "3306:3306"
    network_mode: "host"  # 네트워크모드를 Host로 동일하게 동작합니다. 
    volumes:              # Host에서 필요한 라이브러리들을 가져옵니다.  이 경로에 mobiusdb.sql 파일 저장되어있습니다. 
      - ./docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d
    healthcheck:          # Docker-compose의 기능으로써 소프트웨어의 동작 여부를 체크해줍니다. 
            test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
            timeout: 20s
            retries: 10
 node:			# Docker-Hub에 저장된 Nodejs 이미지를 가져옵니다. 
    image: "node:7.6"
    working_dir: /home/node/app   # Nodejs 소스코드를 형성할 가상이미지 디렉토리를 설정합니다. 
    environment:
      - NODE_ENV=production
    volumes:                   # Host의 모비우스 소스코드를 가상이미지 디렉토리로 가져옵니다.
      - ./Mobius-master:/home/node/app
    network_mode: "host"
    expose:
      - "8081"
    command: "node mobius"  # 완료 후 모비우스를 실행합니다. 
    depends_on:             # Mysql이 동작 한 이후 Mobius가 동작하게 됩니다. 이를 고려하지 않을 시 DB와의 딜레이로 정상적으로 실행되지 않습니다.   
            db:
                condition: service_healthy
 mqtt:
    image: eclipse-mosquitto:latest  # Docker-Hub에 저장된 mosquitto 이미지를 가져옵니다. 
    ports:
      - 1883:1883
    network_mode: "host"
    volumes:
      - ./etc/mosquitto:/etc/mosquitto:ro
      - ./var/log/mosquitto:/var/log/mosquitto:rw
    depends_on:		    # Mysql이 동작 한 이후 mosquitto가 동작하게 됩니다. 이를 고려하지 않을 시 Mobius와의 딜레이로 정상적으로 실행되지 않습니다.   
            db:
                condition: service_healthy







docker-compose version 1.21.2, build a133471

Docker version 1.13.1, build 092cba3


Ubuntu 16.0.4 LTS  64bit

Mysql 5.7

Nodejs 7.6

Mosquitto 1.4.12
