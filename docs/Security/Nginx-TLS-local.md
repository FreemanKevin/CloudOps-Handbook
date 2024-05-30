
`minica` 是一个轻量级的工具，用于生成和管理自签名的证书，特别适合在本地开发环境中使用。以下是使用 `minica` 为本地开发环境生成自签名证书的步骤：

### 1. 安装 `minica`

首先，你需要安装 `minica`。可以从 [GitHub](https://github.com/jsha/minica) 下载并安装。

#### 通过 Homebrew (适用于 macOS)

```sh
brew install minica
```

#### 通过源代码安装

1. 克隆 `minica` 仓库：
    ```sh
    git clone https://github.com/jsha/minica.git
    cd minica
    ```

2. 编译 `minica`：
    ```sh
    go build
    ```

### 2. 生成根证书和私钥

运行以下命令来生成一个根证书和私钥：

```sh
minica
```

这将在当前目录下生成两个文件：

- `minica.pem`：根证书
- `minica-key.pem`：根证书的私钥

### 3. 为你的本地开发域生成证书

假设你要为 `example.local` 生成证书，可以运行以下命令：

```sh
minica --domains example.local
```

这将生成以下文件：

```shell
.
├── example.local
│   ├── cert.pem         # example.local 域名的 SSL/TLS 证书
│   └── key.pem          # example.local 域名证书的私钥
├── minica-key.pem       # 根证书的私钥
└── minica.pem           # 根证书
```



当然，下面是各个文件的详细解释：

1. `minica.pem`

- **类型**：根证书（CA 证书）
- **用途**：这是由 `minica` 生成的根证书，用于签署其他证书。它是你创建的所有本地开发证书的信任基础。你需要将它添加到你的操作系统或浏览器的受信任的根证书存储中，以便浏览器能够信任你生成的所有证书。



2. `minica-key.pem`

- **类型**：根证书的私钥
- **用途**：这是与 `minica.pem` 根证书配对的私钥。它用于签署其他证书（例如 `example.local` 的证书）。这个文件必须保密，因为任何拥有它的人都可以生成被信任的证书。



3. `example.local/cert.pem`

- **类型**：域名证书
- **用途**：这是为 `example.local` 生成的 SSL/TLS 证书。它由 `minica.pem` 根证书签署，并且应该配置在你的本地开发服务器上，以提供 HTTPS 服务。



4. `example.local/key.pem`

- **类型**：域名证书的私钥
- **用途**：这是与 `example.local` 证书配对的私钥。它用于加密通信并确保只有拥有私钥的服务器能够解密流量。这个文件同样需要保密，并且在配置 HTTPS 服务器时与 `cert.pem` 一起使用。



### 4. 安装根证书

为了让你的操作系统和浏览器信任生成的自签名证书，你需要将根证书 `minica.pem` 安装到受信任的根证书存储中。

#### 在 macOS 上安装根证书
1. 打开钥匙串访问（Keychain Access）应用。
2. 从菜单中选择 `文件 -> 导入项目`，选择 `minica.pem`。
3. 找到导入的证书，右键点击并选择 `获取信息`。
4. 展开 `信任` 部分，将 `使用此证书时` 设置为 `始终信任`，然后关闭窗口。

#### 在 Windows 上安装根证书
1. 双击 `minica.pem` 文件，打开证书安装向导。
2. 选择 `本地计算机`，点击 `下一步`。
3. 选择 `将所有的证书放入下列存储`，点击 `浏览`，选择 `受信任的根证书颁发机构`。
4. 完成向导并确认安装。

#### 在 Linux 上安装根证书
将 `minica.pem` 复制到 `/usr/local/share/ca-certificates/` 目录，并更新证书存储：

```sh
sudo cp minica.pem /usr/local/share/ca-certificates/minica.crt
sudo update-ca-certificates
```

### 5. 配置你的本地开发服务器

最后，你需要配置你的本地开发服务器使用生成的证书和私钥。例如，在一个简单的 `NGINX` 服务器中，你可以这样配置：

```shell
server {
    listen 443 ssl;
    server_name example.local;

    ssl_certificate /etc/nginx/ssl/example.local/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/example.local/key.pem;

    location / {
        proxy_pass http://localhost:3000;  # 或者你实际的本地服务器地址和端口
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name example.local;
    return 301 https://$host$request_uri;
}
```

然后拷贝证书到相关目录下，并重启 `NGINX` 或 重载配置文件即可。

### 6. 修改 `hosts` 文件

为了使你的浏览器能够解析 `example.local`，你需要在 `hosts` 文件中添加一行。

编辑 `hosts` 文件

```sh
sudo vim /etc/hosts
```

添加以下内容：
```
127.0.0.1 example.local
```

### 7. 访问你的本地网站

现在，你可以在浏览器中访问 `https://example.local`。如果你已经将 `minica.pem` 根证书添加到受信任的根证书存储中，你不应该看到任何证书警告。



### 总结

通过上述步骤，你可以使用 `minica` 为本地开发环境生成并配置自签名证书。这将有助于在本地开发中使用 HTTPS，使开发环境更接近生产环境。