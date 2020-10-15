# Quasar App (video_chat)

A Quasar Framework app

## Install and start From Docker (Recommended)
```
(host)$ cd <host_absolute_dir_path>
(host)$ git clone git@github.com:Asli666/video_chat.git
(host)$ docker pull node:latest
(host)$ docker run -it -p 8080:8080 -v <host_absolute_dir_path>/video_chat:/workspace/video_chat node:latest /bin/bash
(container)$ npm install -g @quasar/cli
(container)$ cd /worksapce/video_chat
(container)$ npm install
(container)$ quasar dev
(host) > open your chrome, input: http://localhost:8080/#/
```

## Install the dependencies
```bash
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)
```bash
quasar dev
```


### Build the app for production
```bash
quasar build
```

### Customize the configuration
See [Configuring quasar.conf.js](https://quasar.dev/quasar-cli/quasar-conf-js).

## Usage
- 新建房间：任意输入号码，点击‘创建’按钮即可。跳转页面后记下Room ID, 将该号码告诉对方。
- 登陆房间：输入一个房间的Room ID，点击登录即可。

要注意的是，http协议的url可能无法使用getUserMedia等方法，所以双方都须在chrome浏览器下启动flags设置，
输入下列url到chrome浏览器

```
chrome://flags/#unsafely-treat-insecure-origin-as-secure
```

在Insecure origins treated as secure项中的输入框输入
```
http://<前端服务器所在的内网ip>：8080
```
