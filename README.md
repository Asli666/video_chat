# 高级网络_小组作业1

## 项目介绍
调用WebRTC实现基于浏览器的双人视频聊天应用，兼具文字聊天功能。
## 技术框架及依赖包
前端借助Vue.js与quasar框架，后端使用js调用WebRTC协议API。

引擎：
* node: >= 10.18.1
* npm: >= 6.13.4
* yarn: >= 1.21.1

依赖包：
* @quasar/extras: ^1.0.0
* quasar: ^1.0.0

## 使用
### 使用docker启动项目（推荐）
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
### 安装依赖包
```bash
npm install
```
#### 开发者模式启动应用
```bash
quasar dev
```
#### 搭建应用
```bash
quasar build
```
#### 自定义配置
见 [Configuring quasar.conf.js](https://quasar.dev/quasar-cli/quasar-conf-js).

### Usage
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

## API
建立一个Google的公共STUN服务器
```
const configuration = {
 iceServers: [{
   urls: 'stun:stun.l.google.com:19302'
 }]
};
```
使用ScaleDrone作为Signaling服务器
```
const drone = new ScaleDrone('CHANNEL_ID_FROM_SCALEDRONE');
```
