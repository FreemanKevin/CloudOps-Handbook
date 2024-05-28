

## 实验条件

> 国内网站 80/443 端口必须备案才可测试，推荐使用国外云服务。

- 公网域名（国外）

- 公网虚拟机（国外）
- 公网邮箱
- 域名映射（DNS 服务商）




## 物理方式部署

> 若需要离线依赖包，请自行处理。

### 1. 安装依赖

- Debian 系列操作系统

```shell
 sudo apt install nginx 
 sudo apt install certbot python3-certbot-nginx
```



- Centos 系列操作系统

```shell
# 使用 yum
sudo yum install epel-release
sudo yum install nginx
sudo yum install certbot python3-certbot-nginx

# 使用 dnf (适用于 CentOS 8 及更高版本)
sudo dnf install epel-release
sudo dnf install nginx
sudo dnf install certbot python3-certbot-nginx
```



### 2. 启动 NGINX

```shell
sudo systemctl start nginx
sudo systemctl enable nginx
```





### 3. 申请证书

示例命令，包含所有必要的参数：

```bash
sudo certbot --nginx -d example.com -d www.example.com -m youremail@example.com --agree-tos --non-interactive
```

详细解释一下这些参数：

- `--nginx`：使用 Nginx 插件。
- `-d example.com -d www.example.com`：指定要为其获取证书的域名，可以指定多个域名。
- `-m youremail@example.com`：指定联系用的邮箱地址。
- `--agree-tos`：自动同意 Let’s Encrypt 的服务条款。
- `--non-interactive`：在非交互模式下运行，适用于自动化脚本。

完整的命令可以如下：

```bash
sudo certbot --nginx -d example.com -d www.example.com -m youremail@example.com --agree-tos --non-interactive
```

替换 `example.com` 和 `www.example.com` 为你的实际域名，`youremail@example.com` 为你的实际邮箱地址。

**例子**

假设你要为 `mydomain.com` 和 `www.mydomain.com` 获取证书，并且你的邮箱是 `admin@mydomain.com`，命令如下：

```bash
sudo certbot --nginx -d mydomain.com -d www.mydomain.com -m admin@mydomain.com --agree-tos --non-interactive
```

这样，Certbot 会自动处理域名验证和证书安装，无需任何进一步的用户交互。



### 4 申请后的变化

申请完证书后，会有以下几个主要变化：

#### 4.1. 证书生成和安装

- **证书文件**：
  - Let’s Encrypt 会生成并安装 SSL 证书文件。这些文件通常包括：
    - `fullchain.pem`：包含完整的证书链，包括服务器证书和中间证书。
    - `privkey.pem`：服务器私钥。
  - 这些文件通常位于 `/etc/letsencrypt/live/<your_domain>/` 目录下。

#### 4.2. Nginx 配置更新

- **Nginx 配置自动更新**：
  
  - Certbot 的 Nginx 插件会自动更新 Nginx 的配置文件，以使用新生成的 SSL 证书。
  - 例如，原本的 HTTP 配置可能会被修改为包含 HTTPS 配置。Nginx 配置文件中会添加或修改以下部分：
    ```nginx
    server {
        listen 443 ssl;
        server_name example.com www.example.com;
    
        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    
        # 其他配置项...
    }
    ```

#### 4.3. HTTPS 启用

- **启用 HTTPS**：
  - 你的网站将开始通过 HTTPS 提供服务，确保数据传输的加密和安全。
  - 访问者在浏览器中访问你的网站时，会看到地址栏中的锁定图标，表示连接是安全的。

#### 4.4. 自动重定向（可选）

- **HTTP 到 HTTPS 的重定向**：
  
  - 如果选择自动重定向，Certbot 会自动在 Nginx 配置中添加从 HTTP 重定向到 HTTPS 的规则。例如：
    ```nginx
    server {
        if ($host = www.example.com) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    
        if ($host = example.com) {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    
        listen 80;
        server_name example.com www.example.com;
        return 404; # managed by Certbot
    }
    ```

#### 4.5. 定期续订

##### 4.5.1 自动续订测试

- Certbot 会设置一个系统任务（通常是 `cron` 或 `systemd` timer），以自动续订证书。你可以通过以下命令检查续订情况：
  ```bash
  sudo certbot renew --dry-run
  ```
  
  

`sudo certbot renew --dry-run` 是一个测试命令，用于检查 Certbot 的续订过程是否能够成功完成，而不会实际更新或安装任何证书。这是一个非常有用的命令，可以确保在实际续订过程中不会出现问题。



`--dry-run` 参数的作用

- **模拟续订过程**：该命令会模拟证书续订过程，但不会实际更新现有的证书。
- **验证环境配置**：通过运行该命令，你可以验证当前的 Certbot 配置和环境是否正确，确保续订过程能够顺利进行。
- **检测潜在问题**：如果有任何配置错误或其他可能阻碍续订的问题，`--dry-run` 运行时会报告这些问题。



##### 4.5.2 实际续订

在生产环境中，Certbot 会自动设置一个定时任务（通常是 `cron` 或 `systemd` timer），确保证书在过期前自动续订。这个任务通常每 12 小时运行一次，检查是否有需要续订的证书，并在需要时自动续订。



##### 4.5.3 检查自动续订任务

1. 使用 `cron`

如果 Certbot 使用 `cron` 进行自动续订，你可以通过以下命令查看定时任务：

```bash
sudo crontab -l
```

你可能会看到类似以下的条目：

```plaintext
0 */12 * * * certbot renew --quiet
```

这表示 `certbot renew --quiet` 命令会每 12 小时运行一次，以静默模式检查并续订证书。



2. 使用 `systemd`

如果 Certbot 使用 `systemd` timer 进行自动续订，你可以通过以下命令查看：

```bash
systemctl list-timers | grep certbot
```

这将显示 Certbot 自动续订 timer 的状态，例如：

```plaintext
Mon 2024-05-28 12:00:00 CEST  3h 58min left n/a                          n/a        certbot.timer
```



##### 4.5.4 确认自动续订任务 

为了确认自动续订任务是否已正确配置并运行，你可以手动检查 `cron` 或 `systemd` 日志：

1. 检查 `cron` 日志

查看 `/var/log/syslog` 或 `/var/log/cron`（具体日志文件路径可能因系统不同而异）：

```bash
grep certbot /var/log/syslog
```



2. 检查 `systemd` 日志

使用 `journalctl` 查看 `certbot` 的日志：

```bash
journalctl -u certbot.timer
```



3. 检查 `certbot` 的日志目录

Certbot 可能会将日志文件保存在它自己的目录中，通常位于 `/var/log/letsencrypt`：

```
sudo ls /var/log/letsencrypt
sudo cat /var/log/letsencrypt/letsencrypt.log
```



##### 4.5.5 结论

`sudo certbot renew --dry-run` 只是一个测试命令，用于确认续订过程是否能够顺利进行。实际的自动续订是由定时任务（`cron` 或 `systemd` timer）管理的，这些任务会定期运行 `certbot renew` 命令，确保你的证书在过期前自动续订。通过检查这些定时任务的配置和日志，你可以确保自动续订功能正常工作。

#### 4.6. 安全和性能提升

- **提升安全性**：
  - 启用 HTTPS 后，数据在传输过程中将被加密，防止中间人攻击和数据窃取。
  - 现代浏览器和搜索引擎会优先考虑使用 HTTPS 的网站，提升用户信任和搜索排名。

- **HTTP/2 支持**：
  - 启用 HTTPS 后，你可以配置 Nginx 使用 HTTP/2 协议，该协议能显著提升网站性能，特别是在高延迟网络环境下。

#### 4.7. 总结

申请和安装 Let’s Encrypt 证书后，你的网站将通过 HTTPS 提供服务，数据传输将被加密，Nginx 配置文件会自动更新以使用新的证书，并且系统会设置自动续订任务，确保证书不会过期。这些变化将提升你网站的安全性和用户信任度。



## 容器化方式部署

### 1. 安装依赖

核心包：`Docker-ce` / `Docker-compose`，具体过程不再赘述，若需离线包，请自行解决。

### 2. 启动 NGINX

使用下面的 `Compose` 配置模版，建议文件名：`docker-compose-before.yaml`

```yaml
version: "3"
services:
  nginx:
    container_name: nginx
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./default-before.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certbot/letsencrypt:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
  certbot:
    container_name: certbot
    image: certbot/certbot:latest
    depends_on:
      - nginx
    command: >- 
             certonly --reinstall --webroot --webroot-path=/var/www/certbot
             --email youremail@example.com --agree-tos --no-eff-email
             -d example.com
    volumes:
      - ./certbot/letsencrypt:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
```



使用这里的配置文件模版： `default-before.conf`

```shell
server {
    listen [::]:80;
    listen 80;
    server_name example.com;
    location ~/.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }
}
```



启动 NGINX

```shell
mkdir -p ./certbot/{letsencrypt,www}
sudo docker-compose  -f  docker-compose-before.yaml up nginx -d   ## 访问域名会出现 404 ，对证书申请与签发不影响，不用管
```



### 3. 申请证书

```shell
sudo docker-compose -f ./docker-compose-before.yaml up certbot
```



类似日志输出：

```shell
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo docker-compose  -f  docker-compose-before.yaml up certbot
[+] Running 2/0
 ⠿ Container nginx    Running                                                                                                                                                    0.0s
 ⠿ Container certbot  Created                                                                                                                                                    0.1s
Attaching to certbot
certbot  | Saving debug log to /var/log/letsencrypt/letsencrypt.log
certbot  | Account registered.
certbot  | Requesting a certificate for example.com
certbot  | 
certbot  | Successfully received certificate.
certbot  | Certificate is saved at: /etc/letsencrypt/live/example.com/fullchain.pem
certbot  | Key is saved at:         /etc/letsencrypt/live/example.com/privkey.pem
certbot  | This certificate expires on 2024-08-26.
certbot  | These files will be updated when the certificate renews.
certbot  | NEXT STEPS:
certbot  | - The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.
certbot  | 
certbot  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
certbot  | If you like Certbot, please consider supporting our work by:
certbot  |  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
certbot  |  * Donating to EFF:                    https://eff.org/donate-le
certbot  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
certbot exited with code 0
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo docker-compose  -f  docker-compose-before.yaml ps -a 
NAME                COMMAND                  SERVICE             STATUS              PORTS
certbot             "certbot certonly --…"   certbot             exited (0)          
nginx               "/docker-entrypoint.…"   nginx               running             0.0.0.0:80->80/tcp, :::80->80/tcp
```



### 4. 关闭所有初始化容器

```shell
sudo docker-compose  -f  docker-compose-before.yaml down
```



### 5. 启动 NGINX

> 当前文档中使用了 `wiki.js` 做为测试后端，因此有了相关配置。

此时 NGINX 的配置与服务均是正式使用的，即有完整的代理配置，重定向配置，证书配置。

NGINX 配置文件，文件名：`default-https.conf`

```shell
server {
    listen [::]:80;
    listen 80;

    server_name example.com;

    return 301 https://$host$request_uri;
}

server {
    listen [::]:443 ssl;
    listen 443 ssl;

    server_name example.com; 

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }

    access_log  /var/log/nginx/access.log;
    error_log   /var/log/nginx/error.log;

    location / {
        proxy_pass http://wikijs:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```



Compose 服务编排脚本，文件名：`docker-compose.yaml`

```yaml
version: "3"

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    environment:
      - TZ=Asia/Shanghai
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./default-https.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certbot/letsencrypt:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
      - ./log/nginx:/var/log/nginx
    depends_on:
      - wikijs

  wikijs:
    image: requarks/wiki:2
    container_name: wikijs
    restart: always
    environment:
      DB_TYPE: "postgres"
      DB_HOST: "db"
      DB_PORT: 5432
      DB_USER: "wikijs"
      DB_PASS: "wikijs_password"
      DB_NAME: "wikijs"
      WIKI_ADMIN_EMAIL: "your@example.com"
      WIKI_ADMIN_PASSWORD: "your_password"
    depends_on:
      - db
    volumes:
      - wikijs-data:/wiki/data

  db:
    image: postgres:13
    container_name: wikijs-db
    restart: always
    environment:
      POSTGRES_DB: "wikijs"
      POSTGRES_USER: "wikijs"
      POSTGRES_PASSWORD: "wikijs_password"
    volumes:
      - db-data:/var/lib/postgresql/data

  certbot-cron:
    image: certbot/certbot
    restart: always
    container_name: certbot-cron
    volumes:
      - ./certbot/letsencrypt:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
      - ./certbot/log:/var/log
    entrypoint: /bin/sh -c "trap exit TERM; while :; do certbot renew --webroot --webroot-path=/var/www/certbot; sleep 12h & wait $${!}; done"
    depends_on:
      - nginx


volumes:
  wikijs-data:
  db-data:
```





完整的文件目录树结构，应类似于：

```shell
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo tree .
.
|-- certbot
|   |-- letsencrypt
|   |   |-- accounts
|   |   |   `-- acme-v02.api.letsencrypt.org
|   |   |       `-- directory
|   |   |           `-- 83a270e756e902ac92bdc315893c6770
|   |   |               |-- meta.json
|   |   |               |-- private_key.json
|   |   |               `-- regr.json
|   |   |-- archive
|   |   |   `-- example.com
|   |   |       |-- cert1.pem
|   |   |       |-- chain1.pem
|   |   |       |-- fullchain1.pem
|   |   |       `-- privkey1.pem
|   |   |-- live
|   |   |   |-- README
|   |   |   `-- example.com
|   |   |       |-- README
|   |   |       |-- cert.pem -> ../../archive/example.com/cert1.pem
|   |   |       |-- chain.pem -> ../../archive/example.com/chain1.pem
|   |   |       |-- fullchain.pem -> ../../archive/example.com/fullchain1.pem
|   |   |       `-- privkey.pem -> ../../archive/example.com/privkey1.pem
|   |   |-- renewal
|   |   |   `-- example.com.conf
|   |   `-- renewal-hooks
|   |       |-- deploy
|   |       |-- post
|   |       `-- pre
|   |-- log
|   |   `-- letsencrypt
|   |       |-- letsencrypt.log
|   |       |-- letsencrypt.log.1
|   |       |-- letsencrypt.log.2
|   |       |-- letsencrypt.log.3
|   |       |-- letsencrypt.log.4
|   |       |-- letsencrypt.log.5
|   |       `-- letsencrypt.log.6
|   `-- www
|-- default-before.conf
|-- default-https.conf
|-- docker-compose-before.yaml
|-- docker-compose.yaml
`-- log
    `-- nginx
        |-- access.log
        `-- error.log

21 directories, 27 files
```



启动服务：

```shell
sudo docker-compose up -d
```



关闭服务(若需要)：

```shell
sudo docker-compose down
```



查看服务，应类似于：

```shell
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo docker-compose ps -a 
NAME                COMMAND                  SERVICE             STATUS              PORTS      
certbot-cron        "/bin/sh -c 'trap ex…"   certbot-cron        running             80/tcp, 443/tcp
nginx               "/docker-entrypoint.…"   nginx               running             0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp
wikijs              "docker-entrypoint.s…"   wikijs              running             3000/tcp, 3443/tcp
wikijs-db           "docker-entrypoint.s…"   db                  running             5432/tcp
```



查看证书自动续期状态，应类似于：

```shell
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo docker-compose logs -f --tail 1000 certbot-cron
certbot-cron  | Saving debug log to /var/log/letsencrypt/letsencrypt.log
certbot-cron  | 
certbot-cron  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
certbot-cron  | Processing /etc/letsencrypt/renewal/example.com.conf
certbot-cron  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
certbot-cron  | Certificate not yet due for renewal
certbot-cron  | 
certbot-cron  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
certbot-cron  | The following certificates are not due for renewal yet:
certbot-cron  |   /etc/letsencrypt/live/example.com/fullchain.pem expires on 2024-08-26 (skipped)
certbot-cron  | No renewals were attempted.
certbot-cron  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
^Ccanceled
```



本地测试强制更新证书，只测试不实际更新证书，应类似于：

```shell
admin@ip-172-31-3-211:~/wikijs-nginx$ sudo docker run --rm -v $(pwd)/certbot/letsencrypt:/etc/letsencrypt -v $(pwd)/certbot/www:/var/www/certbot certbot/certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/example.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
^CExiting due to user request.
```





## 故障处理

### 1. 端口占用

网站服务默认使用：`80、443` 端口，需要确保不能被占用。



### 2. 网站重定向 301

DNS 服务商那边的控制规则可能会影响网站正常 `https` 重定向，比如无限循环重定向 301 ，需要自行排查确认。