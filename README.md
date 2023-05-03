## electron-vue-app

### 预览
<img src="https://i.328888.xyz/2023/05/03/iMswTp.png" />
<img src="https://i.328888.xyz/2023/05/03/iMsYJv.png" />

### 1. 环境变量

此处使用环境变量`NODE_ENV`来切换生产和开发环境，生产环境为`NODE_ENV=production`，开发环境为`NODE_ENV=development`，若有其它如`release`等环境可在此基础上拓展。

### 2. 创建electron文件夹

在项目根目录下创建文件夹`electron`，将`main.js`和`preload.js`文件移动进来。其结构如下所示：

~~~bash
.
├── README.md
├── electron
│   ├── main.js
│   └── preload.js
...
~~~

### 3. 编辑electron/main.js

该文件主要是需要根据环境变量切换electron加载的内容，修改内容如下：

~~~js
mainWindow.loadURL(
  NODE_ENV === 'development'
  ? 'http://localhost:3000'
  :`file://${path.join(__dirname, '../dist/index.html')}`
);
~~~

### 4. 编辑package.json

首先修改`main` 属性，将`main: main.js`改为`main: electron/main.js`。

~~~json
{
  "name": "kuari",
  "version": "0.0.0",
  "main": "electron/main.js",
  // ... 
}
~~~

接着，编辑`build`属性：

~~~json
"build": {
  "appId": "com.your-website.your-app",
  "productName": "ElectronApp",
  "copyright": "Copyright © 2021 <your-name>",
  "mac": {
    "category": "public.app-category.utilities"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true
  },
  "files": [
    "dist/**/*",
    "electron/**/*"
  ],
  "directories": {
    "buildResources": "assets",
    "output": "dist_electron"
  }
}
~~~

然后，更新`scripts`属性。

此处需要先安装两个库：

- **`cross-env`**: 该库让开发者只需要注重环境变量的设置，而无需担心平台设置
- **`electron-builder`**: electron打包库

~~~bash
npm i -D cross-env electron-builder
~~~

更新后的`scripts`如下：

~~~json
{
  "dev": "vite",
  "build": "vite build",
  "serve": "vite preview",
  "electron": "wait-on tcp:3000 && cross-env NODE_ENV=development electron .", // 此处需要设置环境变量以保证开发时加载url
  "electron:serve": "concurrently -k \"yarn dev\" \"yarn electron\"",
  "electron:build": "vite build && electron-builder" // 新增打包命令
}
~~~

### 5. 打包

直接执行打包命令即可开始打包。

~~~bash
npm run electron:build
~~~

打包完成之后，会多出两个文件夹`dist`和`dist_electron`，其文件结构如下：

~~~bash
.
├── README.md
├── dist
│   ├── assets
│   ├── favicon.ico
│   └── index.html
├── dist_electron
│   ├── MyApp-0.0.0-mac.zip
│   ├── MyApp-0.0.0-mac.zip.blockmap
│   ├── MyApp-0.0.0.dmg
│   ├── MyApp-0.0.0.dmg.blockmap
│   ├── builder-debug.yml
│   ├── builder-effective-config.yaml
│   └── mac
...
~~~