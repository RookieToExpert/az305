#### 打包Nginx html镜像
1. 创建打包文件：

   ```mkdir -p ~/timeweb```
2. 将html文件移动到对应打包文件：

   ```sudo cp /var/www/timeapp/index.html ~/timeweb```
3. 编辑nginx配置文件:

   ```cat >default.conf.template <<'stop'```
   ```bash
   server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / { try_files $uri /index.html; }

    resolver 168.63.129.16 1.1.1.1 ipv6=off valid=60s;

    # 反代 /api 到后端；用环境变量控制
    location /api/ {
        proxy_http_version 1.1;

        # 环境变量（容器启动时会被替换）
        set $backend_base   "${BACKEND_SCHEME}://${BACKEND_HOST}";
        set $backend_host   "${BACKEND_HOST}";

        # 如果是 HTTPS 外网域名（ACA），需要下面两行；内网 http 也保留，不影响
        proxy_set_header Host $backend_host;
        proxy_ssl_server_name on;

        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 二选一：是否保留 /api 前缀
        # —— 是否保留 /api 前缀 ——
        # 保留 /api（后端本来就有 /api 前缀时用，不需要改写）：
        # proxy_pass $backend_base;
   
        #不保留 /api
        rewrite ^/api/?(.*)$ /$1 break;
        proxy_pass $backend_base;
        }
      }
     ```

4. 创建dockerfile：

   ```cat >dockerfile <<'docker'```
   ```docker
   FROM nginx:alpine
   COPY ./index.html /usr/share/nginx/html/index.html
   # 放到 /etc/nginx/templates/ 且以 .template 结尾，启动时自动 envsubst
   COPY ./default.conf.template /etc/nginx/templates/default.conf.template
   # 千万不要覆盖 entrypoint / CMD，保持默认即可
   ```
   
5. 构建docker image：

   ```sudo docker build -t rayrayye/timeweb:latest```
6. push到dockerhub:

  ```sudo docker login```
  ```sudo docker push rayrayye/timeweb:latest```

#### 本地测试
1. 用8080端口跑一下image：
后台运行docker，前端可以测试：

```sudo docker run --rm -d -p 8080:80 rayrayye/webapp:latest```

进入docker，查看文件配置等：

```sudo docker run -it -p 8080:80 rayrayye/webapp:latest```

进入docker跑一个命令：

```docker exec -it web sh -lc 'cat /etc/nginx/conf.d/default.conf'```

> --rm 容器退出就自动删除（包括匿名卷的可写层）。适合本地调试，省得手动 docker rm
> -i 保持 STDIN 打开，允许交互输入。
> -t 分配一个伪 TTY，让交互界面像在终端一样工作（比如有颜色、光标）。-it通常一起使用
> -p 端口映射Host:Container
> -d 后台运行container
> exec 不会新建容器，而是在已运行容器内开一个进程
> -l (login shell）：把这个 shell 当成“登录 shell”，会先读取 /etc/profile、~/.profile 等来初始化环境变量（PATH 等）。
> -c 让 shell 执行后面这段命令字符串，然后退出.
2. 测试一下反应头：

```curl -I http://localhost:8080/```
> 后端未配置前，/api/* 可能 502；等 ACA 里把两个环境变量填上就通

3. 查看log：
```sudo docker log <containerID/name> | tail```
