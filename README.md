# IP-Location finding website containerization with Nginx LoadBalancing.
[![Build](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

---
## Description
It's a sample IP location finding website and that the website fetched the data from ipstack website. Aso, I have containerized this website and also, I included Nginx and Redis containers. Redis act as a cache and Nginx act as a load balancer server because we are using three ipstack containers for efficient working. Also, you can easily set up these containers into your environment because I wrote the above details as a docker-compose. Furthermore, the website is created using python and it's just a demonstration. 

----
## Feature
- IP-Location finding website (Python) (Just a demonstration)
- Easy to migrate everywhere 
- Container load balanced (Nginx)
- All containers are spin up with a single command

---
## Includes
- 5 Containers for LAMP (IP-Stack, Redis, Nginx(Load Balancer))
- 1 Network (For Container interconnecting)

----
## Pre-Requests
- Need to install docker and docker-compose
- You have a basic knowledge of what is docker.
- You have a basic knowledge of what is ipstack and how it's working
- Need an IP stack Login and API for location finding
- Need to add apikey file before running the script. So, please find the IP stack URL and grab the key and change the same on "env.dev" 

-----
## How to install docker, docker-compose.
### Docker installation 
Please refer the doc for [docker installation](https://docs.docker.com/engine/install)

```sh
yum install docker -y
yum install git -y
```
### docker-compose installation
Please refer the doc for [docker-compose installation](https://docs.docker.com/compose/install/)
```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
docker-compose version 
```
> you guys can use [play with Docker](https://labs.play-with-docker.com/) which includes pre-installed docker and compose. It looks like a terminal and it's built from alpine OS. So, it's just used for learning purpose so who you guys can use 4hrs session without any additional installation.

## How to get IPstack API
> Please go through the [ipstack](https://ipstack.com/) and click "_GET FREE API KEY_" on the top right corner. 
![alt text](https://i.ibb.co/FJwjZWP/ipstack.png)

_vim env.dev_
```sh
IPSTACK_KEY="enter-your-key-here(without any double quotes)"
```

----
## How to use
```sh
git clone https://github.com/yousafkhamza/ipstack-containerization-with-nginx-lb.git
cd ipstack-containerization-with-nginx-lb.git
docker-compose --env-file env.dev up -d
```

----
## Output be like
```sh
$ docker-compose --env-file env.dev up -d
Creating network "ipstack-containerization-with-nginx-lb_ipstack" with the default driver
Pulling nginxlb (nginx:alpine)...
alpine: Pulling from library/nginx
5843afab3874: Pull complete
0dc18a5274f2: Pull complete
48a0ee941dcd: Pull complete
2446243a1a3f: Pull complete
cbf0756b41fb: Pull complete
c72750a979b9: Pull complete
Digest: sha256:91528597e842ab1b3b25567191fa7d4e211cb3cc332071fa031cfed2b5892f9e
Status: Downloaded newer image for nginx:alpine
Pulling redis (redis:latest)...
latest: Pulling from library/redis
b4d181a07f80: Pull complete
86e428f79bcb: Pull complete
ba0d0a025810: Pull complete
ba9292c6f77e: Pull complete
b96c0d1da602: Pull complete
5e4b46455da3: Pull complete
Digest: sha256:b6a9fc3535388a6fc04f3bdb83fb4d9d0b4ffd85e7609a6ff2f0f731427823e3
Status: Downloaded newer image for redis:latest
Pulling ipstack1 (yousafkhamza/ipstack:latest)...
latest: Pulling from yousafkhamza/ipstack
921b31ab772b: Pull complete
1a0c422ed526: Pull complete
ec0818a7bbe4: Pull complete
b53197ee35ff: Pull complete
8b25717b4dbf: Pull complete
acafaa4494be: Pull complete
c17f91a4a471: Pull complete
bac07450eb17: Pull complete
Digest: sha256:99ece38dc66fbdadc69a65e63f5b9d330ecfc2681a7a8819cf464180e87747c5
Status: Downloaded newer image for yousafkhamza/ipstack:latest
Creating nginx    ... done
Creating redis ... done
Creating ipstack2 ... done
Creating ipstack1 ... done
Creating ipstack3 ... done
```
_Containers list_
```sh
$ docker-compose ps
  Name                Command               State         Ports       
----------------------------------------------------------------------
ipstack1   python3 app.py                   Up                        
ipstack2   python3 app.py                   Up                        
ipstack3   python3 app.py                   Up                        
nginx      /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp
redis      docker-entrypoint.sh redis ...   Up      6379/tcp 
```

----
## How to use ipstack website after compose up (screenshots)
> _Load your site with your server_ip:80_ 
![alt text](https://i.ibb.co/8d90FRw/Screenshot-8.png)

> _Add an IP which you need to find the location. (result like this)
![alt text](https://i.ibb.co/k9qrcDH/Screenshot-7.png)

----
## Architecture
![alt text](https://i.ibb.co/DWR574g/docker-ipstack.jpg)

----
## Behind the code
_vim docker-compose.yml_
```sh
version: '3'
services:

  nginxlb:
    image: nginx:alpine
    container_name: nginx
    restart: always
    ports:
      - "80:80"
    networks:
      - ipstack
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf

  redis:
    image: redis:latest
    container_name: redis
    restart: always
    networks:
      - ipstack

  ipstack1:
    image: yousafkhamza/ipstack:latest
    container_name: ipstack1
    restart: always
    depends_on:
      - redis
    networks:
      - ipstack
    environment:
      CACHING_SERVER: redis 
      IPSTACK_KEY: "${IPSTACK_KEY}"

  ipstack2:
    image: yousafkhamza/ipstack:latest
    container_name: ipstack2
    restart: always
    depends_on:
      - redis
    networks:
      - ipstack
    environment:
      CACHING_SERVER: redis 
      IPSTACK_KEY: "${IPSTACK_KEY}"

  ipstack3:
    image: yousafkhamza/ipstack:latest
    container_name: ipstack3
    restart: always
    depends_on:
      - redis
    networks:
      - ipstack
    environment:
      CACHING_SERVER: redis 
      IPSTACK_KEY: "${IPSTACK_KEY}"
           
networks:
  ipstack:
```
_vim nginx.conf_ (work like as a container load balancer)
```sh
events {
    worker_connections   1000;
}

http {
   upstream backend {
     server ipstack1;
     server ipstack2;
     server ipstack3;
   }

   server {
       
      listen 80; 
      location / {
          proxy_pass http://backend;
      }
   }
}
```
_vim env.dev_
```sh
IPSTACK_KEY=a37f9a05417225606d66         <----------- please replace your ipstack apikey
```

----
## Conclusion
it's a sample IP-location website and anyone can easy to set up this iplocation project on your environment with the help of docker-compose and containerization. Also, this project includes 3 IP-location lookup (ipstack) containers for effectivly working and that can handle more clients, and that three containers datas store/fetch to a common cache server (Redis). Also, the 3 containers are connected with an Nginx server, and that server act as a load balancer, and this server is only connected with the outside world.

### ⚙️ Connect with Me 

<p align="center">
<a href="mailto:yousaf.k.hamza@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/yousafkhamza"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
<a href="https://www.instagram.com/yousafkhamza"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://wa.me/%2B917736720639?text=This%20message%20from%20GitHub."><img src="https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/></a><br />
