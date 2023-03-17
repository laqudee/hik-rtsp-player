<template>
  <div class="header">HIK Rtsp Video Player For Vue3 & Vite</div>
  <div class="video-box">
    <div id="divPlugin" class="plugin"></div>
  </div>
  <div class="btns">
    <button @click="videoReadPlay">start play</button>
    <button @click="videoStopPlay">stop play</button>
  </div>
</template>

<script setup>
import { reactive, onMounted, onBeforeUnmount } from 'vue'

// 这里需要修改
const ipInfo = reactive({
  ip: 'xx.xxx.xx.1xx', // 自行配置ip地址
  port: 80, // 设置端口号
  user: 'xxxx', // 设置用户
  password: 'xxxxxx' // 设置密码
})
const player = reactive({
  g_iWndIndex: 0,
  channels: [],
  deviceport: '',
  szDeviceIdentify: '',
  iRtspPort: ''
})

// 初始化
const initVideoPlugin = () => {
  let isSupport = window.WebVideoCtrl.I_SupportNoPlugin();
  // 检查插件是否已经安装过
  let iRet = window.WebVideoCtrl.I_CheckPluginInstall();
  if (-1 == iRet) {
    alert("您还未安装过插件，双击开发包目录里的WebComponentsKit.exe安装！");
    return;
  }
  WebVideoCtrl.I_InitPlugin(500, 300, {
    bWndFull: true, // 窗口双击全屏
    iPackageType: 2, // 封装格式 无插件只能是2
    iWndowType: 1, // 分屏类型 1*1 2*2 ....
    bNoPlugin: true, // 开启无插件模式
    cbSelWnd: function (xmlDoc) {
      player.g_iWndIndex = parseInt($(xmlDoc).find("SelectWnd").eq(0).text(), 10);
      console.log('所选的编号：', player.g_iWndIndex);
    },
    cbInitPluginComplete: function () {
      console.log("初始化成功！");
      WebVideoCtrl.I_InsertOBJECTPlugin("divPlugin");
      // 检查插件是否最新
      if (-1 == WebVideoCtrl.I_CheckPluginVersion()) {
        alert("检测到新的插件版本，双击开发包目录里的WebComponentsKit.exe升级！");
        return;
      }
    }
  })
}

// 登录
const handleLogin = () => {
  if (!ipInfo.ip || !ipInfo.port) {
    return
  }
  player.szDeviceIdentify = ipInfo.ip + "_" + ipInfo.port
  WebVideoCtrl.I_Login(ipInfo.ip, 1, ipInfo.port, ipInfo.user, ipInfo.password,
    {
      success: function (xmlDoc) {
        setTimeout(function () {
          getDevicePort() // 获取端口 （影响不大）
        }, 10)
      },
      error: function (status, xmlDoc) {
        console.log('登录失败', status, xmlDoc);
      }
    })
}

// 登出
const handleLogout = () => {
  if (!player.szDeviceIdentify) {
    return;
  }
  let iRet = WebVideoCtrl.I_Logout(player.szDeviceIdentify);
  if (0 == iRet) {
    console.log('退出成功');
  } else {
    console.log('退出失败');
  }
}

// 获取通道
const getChannelInfo = () => {
  if (!player.szDeviceIdentify) {
    return
  }
  // 模拟通道
  WebVideoCtrl.I_GetAnalogChannelInfo(player.szDeviceIdentify, {
    async: false,
    success: function (xmlDoc) {
      let oChannels = $(xmlDoc).find("VideoInputChannel");
      $.each(oChannels, function (i) {
        let id = $(this).find("id").eq(0).text(),
          name = $(this).find("name").eq(0).text();
        if (!name) {
          name = "Camera " + (i < 9 ? "0" + (i + 1) : (i + 1));
        }
        player.channels.push({
          id: id,
          name: name
        })
      });
    },
    error: function (status, xmlDoc) {
      console.log('获取模拟通道失败', status, xmlDoc)
    }
  });
  // 数字通道
  WebVideoCtrl.I_GetDigitalChannelInfo(player.szDeviceIdentify, {
    async: false,
    success: function (xmlDoc) {
      let oChannels = $(xmlDoc).find("InputProxyChannelStatus");
      $.each(oChannels, function (i) {
        let id = $(this).find("id").eq(0).text(),
          name = $(this).find("name").eq(0).text(),
          online = $(this).find("online").eq(0).text();
        if ("false" == online) {// 过滤禁用的数字通道
          return true;
        }
        if ("" == name) {
          name = "IPCamera " + (i < 9 ? "0" + (i + 1) : (i + 1));
        }
        player.channels.push({
          id: id,
          name: name
        })

      });
      videoReadPlay()
    },
    error: function (status, xmlDoc) {
      console.log(player.szDeviceIdentify + " 获取数字通道失败！", status, xmlDoc);
    }
  });
  // 零通道
  WebVideoCtrl.I_GetZeroChannelInfo(player.szDeviceIdentify, {
    async: false,
    success: function (xmlDoc) {
      let oChannels = $(xmlDoc).find("ZeroVideoChannel");

      $.each(oChannels, function (i) {
        let id = $(this).find("id").eq(0).text(),
          name = $(this).find("name").eq(0).text();
        if ("" == name) {
          name = "Zero Channel " + (i < 9 ? "0" + (i + 1) : (i + 1));
        }
        player.channels.push({
          id: id,
          name: name
        })
      });
    },
    error: function (status, xmlDoc) {
      console.log(player.szDeviceIdentify + " 获取零通道失败！", status, xmlDoc);
    }
  });
}

// 获取端口
const getDevicePort = () => {
  if (!player.szDeviceIdentify) {
    return
  }
  var oPort = WebVideoCtrl.I_GetDevicePort(player.szDeviceIdentify);
  if (oPort != null) {
    player.deviceport = oPort.iDevicePort;
    player.iRtspPort = oPort.iRtspPort;
    getChannelInfo() // 获取模拟通道
  }
}

// 开始渲染
const videoReadPlay = () => {
  console.log('开始渲染');

  let oWndInfo = WebVideoCtrl.I_GetWindowStatus(player.g_iWndIndex)
  let iChannelID = player.channels[0].id
  if (!player.szDeviceIdentify) {
    return;
  }
  var startRealPlay = function () {
    WebVideoCtrl.I_StartRealPlay(player.szDeviceIdentify, {
      iRtspPort: parseInt(player.deviceport, 10), // RTSP端口必须是int
      iStreamType: 1, // 码流类型：1-主码流 必须int
      iChannelID: parseInt(iChannelID, 10), // 播放通道 必须int
      bZeroChannel: false, // 是否播放零通道 默认false
      success: function () {
        console.log("预览成功")
      },
      error: function (status, xmlDoc) {
        console.log("预览失败", status, xmlDoc)
      }
    });
  };
  if (oWndInfo != null) {// 已经在播放了，先停止
    WebVideoCtrl.I_Stop({
      success: function () {
        startRealPlay();
      }
    });
  } else {
    startRealPlay();
  }
}

// 停止预览
const videoStopPlay = () => {
  let oWndInfo = WebVideoCtrl.I_GetWindowStatus(player.g_iWndIndex)
  let szInfo = ""

  if (oWndInfo != null) {
    WebVideoCtrl.I_Stop({
      success: function () {
        szInfo = "停止预览成功！";
        console.log(szInfo);
        // handleLogout() // todo
      },
      error: function () {
        szInfo = "停止预览失败！";
        console.log(szInfo);
      }
    });
  }
}

onMounted(() => {
  initVideoPlugin()
  handleLogin()
})

onBeforeUnmount(() => {
  videoStopPlay()
  handleLogout()
})

</script>

<style lang="less" scoped>
.header {
  font-size: 20px;
  padding: 10px 0;
}

.video-box {
  width: 500px;
  height: 300px;
  background: #000;

  .plugin {
    width: 100%;
    height: 100%;
  }
}

.btns {
  margin-top: 20px;

  button {
    margin: 0 10px;
  }
}
</style>
