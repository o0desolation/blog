---
title: Electron-React环境搭建及进程通信
date: 2020-03-19 10:51:54
categories: 
- React
tags: 
- React
---

## 环境搭建

### 安装React

```
npx create-react-app electron-react
```



### 安装Electron

安装Electron

```
npm i electron --save-dev
```



可能会出现卡在node install.js的情况，原因是淘宝镜像的资源路径和官方的资源路径不同。
淘宝的镜像路径是http://npm.taobao.org/mirrors/electron/8.1.1/
而electron默认的路径是http://npm.taobao.org/mirrors/electron/v8.1.1/

+ 解决方案一：

  `Ctrl+C`结束安装进程

  ```
  cd node_modules/electron
  code install.js
  ```

  **./node_modules/electron/install.js**

  ```
  downloadArtifact({
    version,
    artifactName: 'electron',
    force: process.env.force_no_cache === 'true',
    cacheRoot: process.env.electron_config_cache,
    platform: process.env.npm_config_platform || process.platform,
    arch: process.env.npm_config_arch || process.arch,
    /*********************新增以下部分**************************/
    mirrorOptions:{
      mirror:'https://npm.taobao.org/mirrors/electron/',
      customDir:'8.1.1',//版本号根据安装版本而定
    }
    /*********************************************************/
  }).then((zipPath) => extractFile(zipPath)).catch((err) => onerror(err))
  ```

  修改后保存，执行`node install.js`，无报错即安装成功，返回根目录`cd..` `cd..`

  此种方法安装过后`package.json`中并没有`"electron":"^8.1.1"`这一项，手动添加后会导致`electron .`命令无法被正确识别。故不推荐。

* 解决方案二（**推荐**）：

  直接使用nrm 工具切换镜像源下载安装即可

  ```
  npm remove electron
  nrm use cnpm
  npm i electron --save-dev
  ```

* 其他方案：

  网上流传的比如更改系统变量、electron镜像变量之类的回答均无效！



根目录下新建`main.js`

**./main.js**

```
const { app, BrowserWindow } = require('electron')
function createWindow () {   
  // 创建浏览器窗口
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  // 并且为你的应用加载index.html
  // win.loadFile('index.html')
win.loadURL("http://localhost:3000/");

  // 打开开发者工具
  // win.webContents.openDevTools()
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(createWindow)


// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
  // 否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，
  // 通常在应用程序中重新创建一个窗口。
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow()
  }
})
```



添加electron启动命令

**./package.json**

```
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "electron": "electron ."//新增命令，以npm run electron运行
  },
```



至此环境搭建基本完成。测试运行。

**Terminal one**

```
npm start
```

**Terminal two**

```
npm run electron
```



## 进程通信

#### 目标

创建一个无边框无菜单栏的electron应用，实现标题栏的拖曳以及最小化，最大化，取消最大化和关闭按钮

#### 实现

1. 安装`electron-reload`模块，实现热加载方便调试

   ```
   npm i electron-reload --save-dev
   ```

2. 修改main.js文件

   **./main.js**

   ```
   const { app, BrowserWindow } = require("electron");
   const ipcProcess = require("./src/mainProcess/ipcProcess");   //1.导入自建的进程通信模块
   require("electron-reload")(__dirname);    //2.导入electron-reload模块实现热加载
   
   function createWindow() {
   
     const win = new BrowserWindow({
       width: 800,
       height: 600,
       frame: false,   //3.取消窗口边框，这样会得到一个空白窗口
       webPreferences: {
         nodeIntegration: true
       }
     });
    
     win.loadURL("http://localhost:3000/");
   
     ipcProcess(win);    //4.调用进程通信模块，参数为window对象
   }
   
   app.whenReady().then(createWindow);
   
   app.on("window-all-closed", () => {
     if (process.platform !== "darwin") {
       app.quit();
     }
   });
   
   app.on("activate", () => {
     if (BrowserWindow.getAllWindows().length === 0) {
       createWindow();
     }
   });
   
   ```

3. 创建并编写ipcProcess模块

   **./src/mainProcess/ipcProcess.js**

   ```
   const { ipcMain } = require("electron");
   
   const ipc = (win) => {
     ipcMain.on("minimize", () => {
       win.minimize();
     });
   
     ipcMain.on("isMaximized",(event,arg) => {
       if(win.isMaximized())
         event.returnValue = true
       else
         event.returnValue = false
     })
   
     ipcMain.on("maximize", () => {
       win.maximize();
     });
     
     ipcMain.on("unmaximize",() => {
       win.unmaximize();
     })
   
     ipcMain.on("close", () => {
       win.close();
     });
   
     //双击标题栏也可改变最大化状态，所以需要向渲染进程发送消息改变图标
     win.on("maximize",() => {
       win.webContents.send("maximized")
     })
   
     win.on("unmaximize",() => {
       win.webContents.send("unmaximized")
     })
   };
   module.exports = ipc;
   ```

   至此主进程编写完成。

4. 安装`react-icons`模块

   ```
   npm i react-icons
   ```

5. 创建并编写顶部标题栏组件

   **./src/components/layout/windowTop/index.js**

   ```
   import React, { useState, useEffect } from "react";
   import "./index.css";
   import Minimize from "react-icons/lib/md/remove";
   import Maximize from "react-icons/lib/md/crop-square";
   import Close from "react-icons/lib/md/close";
   import UnMaximize from "react-icons/lib/md/filter-none";
   const electron = window.require("electron");
   
   const WindowTop = () => {
     const [isMaximized, setIsMaximized] = useState(null);
   
     const mainProcessListner = () => {
       electron.ipcRenderer.on("maximized", () => {
         setIsMaximized(true);
       });
   
       electron.ipcRenderer.on("unmaximized", () => {
         setIsMaximized(false);
       });
     };
   
     useEffect(() => {
       setIsMaximized(electron.ipcRenderer.send("isMaximized"));
       mainProcessListner();
     }, []);
   
     const minHandler = () => {
       electron.ipcRenderer.send("minimize");
     };
     const maxHandler = () => {
       if (!isMaximized) {
         electron.ipcRenderer.send("maximize");
         setIsMaximized(true);
       } else {
         electron.ipcRenderer.send("unmaximize");
         setIsMaximized(false);
       }
     };
     const closeHandler = () => {
       electron.ipcRenderer.send("close");
     };
     return (
       <div className="top">
         <button onClick={minHandler}>
           <Minimize />
         </button>
         <button onClick={maxHandler}>
           {isMaximized ? <UnMaximize /> : <Maximize />}
         </button>
         <button onClick={closeHandler}>
           <Close />
         </button>
       </div>
     );
   };
   
   export default WindowTop;
   ```

6. 修改App组件

   **./src/App.js**

   ```
   import React from 'react';
   import WindowTop from './components/layout/windowTop/index';
   function App() {
     return (
       <WindowTop/>
     );
   }
   
   export default App;
   ```

7. 设置全局样式

   **./src/App.css**

   ```
   :root{
     margin: 0;
     padding: 0;
     --primary-color: #a8d8ea;
     --secondary-color: #aa96da;
   }
   ```

8. 添加标题栏组件样式

   **./src/components/layout/windowTop/index.css**

   ```
   @import url('../../../App.css');
   .top {
     width: 100%;
     height: 30px;
     display: flex;
     justify-content: flex-end;
     background-color:var(--primary-color);
     -webkit-app-region: drag;
   }
   
   button {
     font-size: 20px;
     border: unset;
     background-color:var(--primary-color);
     color: aliceblue;
     width: 50px;
     outline: none;
     -webkit-app-region: no-drag;
   }
   
   button:hover {
     background-color:var(--secondary-color);
   }
   ```