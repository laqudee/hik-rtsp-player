# Vue 3 + Vite + HIK Rtsp

> 目的：在Vue3项目中使用海康`Web无插件开发包`，播放视频，需要配合nginx代理一起开发调试。

> 备注：详细功能请查看`Web无插件开发包`，本项目只作为如何在Vue3项目中正确使用该插件开发包的demo，所以只包含最简单的功能。

## 开发指南

#### Install dependencies
```
  pnpm install
```

#### Compiles and hot-reloads for development
```
  pnpm dev
```

#### Compiles and minifies for production
```
  pnpm build
```

## WEB无插件开发包使用流程

#### 1. 在public中引入静态JS资源

#### 2. 在`index.html`中引入
```html
  <script src="/static/hikvision/jquery-1.12.1.min.js"></script>
  <script src="/static/hikvision/jsPlugin-1.2.0.min.js"></script>
  <script src="/static/hikvision/webVideoCtrl.js"></script>
  <script src="/static/hikvision/encryption/AES.js"></script>
  <script src="/static/hikvision/encryption/cryptico.min.js"></script>
  <script src="/static/hikvision/encryption/crypto-3.1.2.min.js"></script>
  <script src="/static/hikvision/playctrl/AudioRenderer.js"></script>
  <script src="/static/hikvision/playctrl/SuperRender.js"></script>
```

#### 3. 在`vite.config.js`中添加`base`
```js
export default defineConfig({
  plugins: [vue()],
  base: '/'
})
```

#### 4. 在需要的vue文件中开始使用`WEB无插件开发包`

#### 5. `pnpm build`打包，将打包后的`dist`复制到`nginx`下的`html`中

#### 6. 启动`nginx`打开`localhost:8004`查看效果

## 监控渲染流程

#### 1. 初始化 initVideoPlugin()
- 注意这一步，`divPlugin`元素要有具体的width和height
- 这一步可以设置`width * height`的屏等其他配置选项

#### 2. 登录 handleLogin()
- 通过后端接口或已经配置好的`ip`，`port`，`username`，`password`进行登录操作

```js
const ipInfo = reactive({
  ip: 'xx.xxx.xx.1xx', // 自行配置ip地址
  port: 80, // 设置端口号
  user: 'xxxx', // 设置用户
  password: 'xxxxxx' // 设置密码
})
```

#### 3. 获取通道 getChannelInfo()
- 模拟通道
- 数字通道
- 零通道

#### 4. 获取设备端口 getDevicePort()

#### 5. 开始渲染 videoReadPlay（）
- 可以设置一些渲染的配置
  - isStreamType
  - bZeroChannel
  - 等

#### 6. 停止预览 videoStopPlay（）

#### 7. 登出 handleLogout()

## 注意
> 若存在手动切换页面的情况，需要在销毁页面前执行`停止预览`和`登出`，否则会出现再次进入视频播放页面无法播放的情况，可能是已登录，后续动作就不在执行

```js
onBeforeUnmount(() => {
  videoStopPlay()
  handleLogout()
})
```

> 还要注意一些UI组件和WEB无插件开发包的样式兼容性问题

> Vue2和Vue3在public中引入静态资源会有些许不同注意区分
```html
    <!-- Vue 2 -->
    <script src="./static/hikvision/jquery-1.12.1.min.js" ></script>
    <script src="./static/hikvision/jsPlugin-1.2.0.min.js"></script>
    <script src="./static/hikvision/webVideoCtrl.js"></script>
    <script src="./static/hikvision/encryption/AES.js"></script>
    <script src="./static/hikvision/encryption/cryptico.min.js"></script>
    <script src="./static/hikvision/encryption/crypto-3.1.2.min.js"></script>
    <script src="./static/hikvision/playctrl/AudioRenderer.js"></script>
    <script src="./static/hikvision/playctrl/SuperRender.js"></script>
```

> WEB无插件开发包的开发调试需要nginx的配合，最终效果也需要打包放到nginx环境下才能看出

## 扩展
> WEB无插件开发包里面有许多别的用处的方法，可以根据业务需求进行添加和修改


