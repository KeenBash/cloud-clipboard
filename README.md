# cloud-clipboard

因为不想为了手机和电脑互传文件这种小事就扫🐴登录某个辣鸡 APP，而自己折腾出来的一个在线剪贴板。

* 支持**传输纯文本**和**一键复制**
* 支持**传输文件**（选择文件 / 拖拽 / <kbd>Ctrl+V</kbd> 粘贴截图），对于图像可以显示缩略图
* 使用 WebSocket 实现实时通知
* 前端使用 [Vue 2](https://cn.vuejs.org) 和 [Vuetify](https://vuetifyjs.com/zh-Hans/) 构建
* 后端使用 ~~[Swoole](https://www.swoole.com) 或~~ [Node.js](https://nodejs.org) ([Koa](https://github.com/koajs/koa)) 构建 ~~（两种服务端实现任选一种即可）~~

*由于这个项目是针对个人在连接到同一局域网（比如家里的路由器）的设备之间使用而设计的，因此并没有额外考虑在公开的服务器上使用时可能面对的技术和安全问题。*

test

## 截图

<details>
<summary>桌面端</summary>

![](https://ae01.alicdn.com/kf/Hfce3a9b69b3d404c8e3073ab0fffa913v.png)

</details>

<details>
<summary>移动端</summary>

![](https://ae01.alicdn.com/kf/Hbf859dd0e42c4406bf94a6b6f2f4658cf.png)

</details>

## 使用方法

### Node.js 版服务端

#### 安装和运行

~~据说 [pkg](https://github.com/vercel/pkg) 可以把 Node.js 应用打包成可执行文件，但是目前的 5.x 版还不支持 ES Modules，所以先🕊️了（~~

使用 [caxa](https://github.com/leafac/caxa) 和 GitHub Actions 打包成了可以在 Windows amd64 和 Linux amd64 使用的可执行文件，可以在[这里](https://nightly.link/TransparentLC/cloud-clipboard/workflows/ci/master)下载。

*caxa 的打包原理相当于将 Node.js 的可执行文件和所有代码一起做成了一个自解压压缩包，执行时会解压到临时文件夹，并且在退出时不会自动清空。*

配置文件是按照以下顺序尝试读取的：

* 和可执行文件放在同一目录的 `config.json`
* 在命令行中指定：`cloud-clipboard /path/to/config.json`

#### 从源代码运行

需要安装 [Node.js](https://nodejs.org)。

```bash
# 构建前端资源，只需要执行一次
# 也可以直接从 Actions 下载构建好的压缩包（static），解压到 server-node/static
cd client
npm install
npm run build

# 运行服务端
cd ../server-node
npm install
node main.js
```

配置文件是按照以下顺序尝试读取的：

* 和 `main.js` 放在同一目录的 `config.json`
* 在命令行中指定：`node main.js /path/to/config.json`

服务端默认会监听本机所有网卡的 IP 地址（也可以自己设定），并在终端中显示前端界面所在的网址，使用浏览器打开即可使用。

如果你使用的是 Node.js 17 或以上的版本，构建前端资源时可能会遇到 `Error: error:0308010C:digital envelope routines::unsupported` 的错误，在终端里设置环境变量 `NODE_OPTIONS=--openssl-legacy-provider` 可以解决这个问题。

### Docker
#### 自己打包
```bash
docker image build -t myclip .
docker container run -d -p 9501:9501 myclip
```

#### 从docker hub拉取
```bash
docker pull csmayi/lan-clip:latest
docker container run -d -p 9501:9501 csmayi/lan-clip
```

访问 [clipboard](http://127.0.0.1:9501)

### Swoole 版服务端

> 已不再维护，以下内容仅作为存档。

<details>

#### 准备环境

需要安装了 Swoole 扩展的 PHP 运行环境。

* Linux：参见 Swoole 文档的[安装说明](https://wiki.swoole.com/#/environment)
* Windows：使用官方在百毒网盘上提供的 [Cygwin 整合包](https://pan.baidu.com/s/15RodWdoIgwBLmG1I5HXzOg#list/path=%2Fsharelink2059756482-531706993208199%2Fswoole%2Fcygwin)（或者从[这里](https://files.catbox.moe/wz2ktt.zip)下载我自己搬运的 4.5.0 RC1 版本），~~也可以在 WSL(Windows Subsystem for Linux) 中安装运行环境~~

> 可能是由于 WSL 的缺陷，在 WSL 下多次刷新页面后开始有概率出现错误导致服务端挂掉：
> ```text
> WARNING swReactorEpoll_set(:178): reactor#0->set(fd=15|type=0|events=5) failed, Error: No such file or directory
> 段错误 (核心已转储)
> ```
> Cygwin 整合包下暂未发现类似的问题。

在命令行中输入 `php --ri swoole`，可以输出配置信息就代表准备完成了～

#### 安装和运行

~~实际上也不能叫安装，毕竟下载之后就可以直接用了~~

1. 前往 [Release](https://github.com/TransparentLC/cloud-clipboard/releases) 下载最新的 `cloud-clipboard.phar`。
2. 在同一目录新建配置文件 `config.json`（可以参见下面的说明），**一定要根据实际修改 IP 地址！**
3. `php cloud-clipboard.phar`
4. 打开 `http://192.168.1.136:9501`（需要替换为在配置文件中**实际设定的地址和端口**）即可使用～

#### 从源代码运行

需要安装 [Vue CLI](https://cli.vuejs.org/zh/guide/installation.html) 和 [Composer](https://getcomposer.org/download/)。

```bash
cd client
npm install
npm run build
cd ../server
composer install --no-dev

# 从源代码直接运行
php main.php

# 生成 Phar
cd ..
php build-phar.php
```

</details>

### 配置文件说明

`//` 开头的部分是注释，**并不需要写入配置文件中**，否则会导致读取失败。

```json
{
    "server": {
        // 监听的 IP 地址，省略或设为null则会监听所有网卡的IP地址
        "host": [
            "127.0.0.1",
            "::1"
        ],
        "port": 9501, // 端口号
        "key": "localhost-key.pem", // HTTPS 私钥路径
        "cert": "localhost.pem", // HTTPS 证书路径
        "forceWss": false, // 如果没有设定以上两项，但是通过 Nginx 等反向代理而使用了 HTTPS，则需要设为 true 让前端界面强制使用 WSS 连接
        "history": 10, // 消息历史记录的数量
        "auth": false // 是否在连接时要求使用密码认证，falsy 值表示不使用
    },
    "text": {
        "limit": 4096 // 文本的长度限制
    },
    "file": {
        "expire": 3600, // 上传文件的有效期，超过有效期后自动删除，单位为秒
        "chunk": 1048576, // 上传文件的分片大小，不能超过 5 MB，单位为 byte
        "limit": 104857600 // 上传文件的大小限制，单位为 byte
    }
}
```
> HTTPS 的说明：
>
> 如果同时设定了私钥和证书路径，则会使用 HTTPS 协议访问前端界面，未设定则会使用 HTTP 协议。
> 自用的话，可以使用 [mkcert](https://mkcert.dev/) 自行生成证书，并将根证书添加到系统/浏览器的信任列表中。
>
> “密码认证”的说明：
>
> 如果启用“密码认证”，只有输入正确的密码才能连接到服务端并查看剪贴板内容。
> 可以将 `server.auth` 字段设为 `true`（随机生成六位密码）或字符串（自定义密码）来启用这个功能，启动服务端后终端会以 `Authorization code: ******` 的格式输出当前使用的密码。
