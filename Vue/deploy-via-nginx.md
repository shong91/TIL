# Nginx 를 이용해 Vue project 배포

(pre-requisite)

1. AWS EC2 인스턴스 생성
2. [네트워크 및 보안 - 보안그룹] 의 인바운드 규칙 추가 (HTTP, HTTPS: 0.0.0.0/0)

## 1. nginx 설치

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install nginx
```

## 2. 정상 구동 여부 확인

```
$ sudo service nginx status

 nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-12-03 06:05:21 UTC; 13min ago
     Docs: man:nginx(8)
  Process: 12496 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=0/SUCCESS)
  Process: 12511 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 12500 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 12518 (nginx)
    Tasks: 2 (limit: 1140)
   CGroup: /system.slice/nginx.service
           ├─12518 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─12521 nginx: worker process
```

## 3. vue 프로젝트 빌드

vue project 경로에서 프로젝트를 빌드한다.

```
$ sudo apt install npm 
$ npm run bulid
```

```
 DONE  Build complete. The dist directory is ready to be deployed.
 INFO  Check out deployment instructions at https://cli.vuejs.org/guide/deployment.html
```

## 4. nginx 설정

### 4-1. default 파일 수정

```
$ sudo vim /etc/nginx/sites-available/default
```

```
# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /home/ubuntu/devops_front/dist;
        # Add index.php to the list if you are using PHP
        index index.html;

        server_name test.com www.test.com;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

```

### 4-2. nginx 재시작

```
$ sudo service nginx restart
```

## Troubleshooting

### error case 1

`error`: [copy-webpack-plugin] unable to locate '/home/ubuntu/devops_front/static' at '/home/ubuntu/devops_front/static'

`resolved`: Had the same trouble, place the .gitkeep file into your static folder.
