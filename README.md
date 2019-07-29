启动项目

```
npm start
```

打包项目

```
npm run build
```


###  create-react-app+ant-mobile+less+viewport环境搭建
简介：本文共分为4部分，第一部分安装create-react-app，第二部分安装ant-mobile,第三部分：安装less,第四部分安装viewport各部分依赖

#### 一、项目基础，安装create-react-app脚手架，创建create-react-app项目my-app

###### 1、在全局环境安装create-react-app工具
在任意文件夹执行以下命令：
```
npm isntall -g create-react-app
```
###### 2、用create-react-app工具创建我的第一个项目my-app
进入放置项目my-app的文件夹，执行以下命令：

```
/**创建项目my-app，并进入项目my-app文件夹*/
create-react-app my-app && cd my-app
```
启动我的项目my-app只需执行命令:

```
npm start
```
到这一步，其实你已经完成react大部的插件的安装，像一些：热更新、webpack、css babel等等，详细可以了解create-react-app工具。

#### 二、安装项目UI 组件库：Ant Design Mobile

###### 1、安装antd-mobile

```
npm install antd-mobile --save
```
###### 2、完成component、css按需加载需要一些配置插件
引入 react-app-rewired 并修改 package.json 里的启动配置。。由于新的 react-app-rewired@2.x 版本的关系，你需要还需要安装 customize-cra。

```
npm install react-app-rewired customize-cra --save-dev
```

```
/* package.json */
  "scripts": {
    "start": "set PORT=3006 && react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test --env=jsdom",
    "eject": "react-scripts eject"
  },
```
然后在项目根目录创建一个 config-overrides.js 用于修改默认配置。

```
module.exports = function override(config, env) {
  // do stuff with the webpack config...
  return config;
};
```
使用 babel-plugin-import, babel-plugin-import 是一个用于按需加载组件代码和样式的 babel 插件，现在我们尝试安装它并修改 config-overrides.js 文件。

```
npm install babel-plugin-import --save-dev
```

```
+ const { override, fixBabelImports } = require('customize-cra');

- module.exports = function override(config, env) {
-   // do stuff with the webpack config...
-   return config;
- };
+ module.exports = override(
+   fixBabelImports('import', {
+     libraryName: 'antd-mobile',
+     style: 'css',
+   }),
+ );
```
完成后，按需引入一个Button

```
import { Button } from 'antd-mobile';
...
    <Button type="primary">primary</Button>
...
```
彩蛋：当项目启动之后，你会发现是在port：3000端口运行的，如果需要换成其他端口，在my-app文件夹下，找到package.json文件，把

```
"start": "react-app-rewired start",
```
改成（在端口3006执行）：

```
"start": "set PORT=3006 && react-app-rewired start",
```

到这一步之后，如无需less和lib-flexible开发的同学，已经可以在my-app项目尽情开发啦，赶紧体验一把热更新带来的便捷吧！


但是我们一般都不满足的，css那种语言，怎么看都不像是程序，于是，less、sass、stylus这三个css预处理器就横空出世了，在本文中，我们重点了解less在create-react-app和ant-mobile环境下的安装。

#### 三、安装less！
依旧的，进入my-app项目目录

安装less和less-loader 

```
npm install less less-loader --save-dev
```
安装react-app-rewire-less来修改less配置：

```
npm i react-app-rewire-less --save
```

修改config-overrides.js文件为：


```
const { override, fixBabelImports, addLessLoader } = require('customize-cra');
const rewireLess = require('react-app-rewire-less');

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd-mobile',
    libraryDirectory: 'es',
    style: true,
  }),
  addLessLoader({
    javascriptEnabled: true,
    //modifyVars: { '@primary-color': '#1DA57A' }, // 这里不注释掉，那你的无法修改主题色primary-color 这里很坑的 要注意！
  }),
);
```
继续修改App.css文件名为：App.less

修改App.js文件：

```
import './App.css';
```
为

```
import './App.less';
```
启动my-app,如果看到正常页面，则less加入成功。
#### 四、添加viewport移动端自适应方案
###### 1、安装postCss插件

```
npm i --save postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano cssnano-preset-advanced
```
再安装react-app-rewire-postcss支持config-overrides重写：

```
npm i react-app-rewire-postcss --save
```

###### 2.安装完成后，在config-overrides.js加入postCss配置


```
const addCustomize = () => config => {
  require('react-app-rewire-postcss')(config, {
        plugins: loader => [
            require('postcss-flexbugs-fixes'),
            require('postcss-preset-env')({
                autoprefixer: {
                    flexbox: 'no-2009',
                },
                stage: 3,
            }),
            require('postcss-aspect-ratio-mini')({}),
            require('postcss-px-to-viewport')({
                viewportWidth: 750, // (Number) The width of the viewport.
                viewportHeight: 1334, // (Number) The height of the viewport.
                unitPrecision: 3, // (Number) The decimal numbers to allow the REM units to grow to.
                viewportUnit: 'vw', // (String) Expected units.
                selectorBlackList: ['.ignore', '.hairlines'], // (Array) The selectors to ignore and leave as px.
                minPixelValue: 1, // (Number) Set the minimum pixel value to replace.
                mediaQuery: false // (Boolean) Allow px to be converted in media queries.
            }),
            require('postcss-write-svg')({
                utf8: false
            }),
            require('postcss-viewport-units')({}),
            require('cssnano')({
                preset: "advanced",
                autoprefixer: false,
                "postcss-zindex": false
            })
        ]
    });
  return config;
}
```
此时config-overrides.js，整个配置如下：

```
const { override, fixBabelImports, addLessLoader } = require('customize-cra');
const rewireLess = require('react-app-rewire-less');


const addCustomize = () => config => {
  require('react-app-rewire-postcss')(config, {
        plugins: loader => [
            require('postcss-flexbugs-fixes'),
            require('postcss-preset-env')({
                autoprefixer: {
                    flexbox: 'no-2009',
                },
                stage: 3,
            }),
            require('postcss-aspect-ratio-mini')({}),
            require('postcss-px-to-viewport')({
                viewportWidth: 750, // (Number) The width of the viewport.
                viewportHeight: 1334, // (Number) The height of the viewport.
                unitPrecision: 3, // (Number) The decimal numbers to allow the REM units to grow to.
                viewportUnit: 'vw', // (String) Expected units.
                selectorBlackList: ['.ignore', '.hairlines'], // (Array) The selectors to ignore and leave as px.
                minPixelValue: 1, // (Number) Set the minimum pixel value to replace.
                mediaQuery: false // (Boolean) Allow px to be converted in media queries.
            }),
            require('postcss-write-svg')({
                utf8: false
            }),
            require('postcss-viewport-units')({}),
            require('cssnano')({
                preset: "advanced",
                autoprefixer: false,
                "postcss-zindex": false
            })
        ]
    });
  return config;
}

module.exports = override(
  fixBabelImports('import', {
    libraryName: 'antd-mobile',
    libraryDirectory: 'es',
    style: true,
  }),
  addLessLoader({
    javascriptEnabled: true,
    //modifyVars: { '@primary-color': '#1DA57A' }, # 这里不注释掉，那你的无法修改主题色primary-color 这里很坑的 要注意！
  }),
  addCustomize(),
);
```
###### 3、引入阿里cdn
在index.html中引入阿里cdn

```
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
```
在body中，加入如下js代码：

```
<script>
  window.onload = function () {
    window.viewportUnitsBuggyfill.init({
      hacks: window.viewportUnitsBuggyfillHacks
    });
  }
</script>
```
最终index.html如下

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="theme-color" content="#000000" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
    <script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>

    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
    <script>
      window.onload = function () {
        window.viewportUnitsBuggyfill.init({
          hacks: window.viewportUnitsBuggyfillHacks
        });
      }
    </script>
  </body>
</html>
```
重新执行npm start打开页面,在style:.App下面发现：

```
content: "viewport-units-buggyfill; width: 100vw; height: 26.667vw; line-height: 26.667vw";
```

```
特别注意
//用这个viewport方案，会有一些苹果机无法显示图片，则添加全局css即可完美显示了
img { 
    content: normal !important; 
}
```
至此，恭喜你，你手上已经多了一个可以自适应移动端+less+ant-mobile项目了，尽情发挥你的才能吧











