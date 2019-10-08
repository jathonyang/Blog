因为自己的项目是基于`vue-cli3`进行开发，所以这里只讨论这种情况下的解决办法 
在进行多页面开发的时候，项目刚开始阶段，因为文件较少，所以代码编译速度还行，但是随着项目逐渐增大，`webpack`编译的速度越来越慢，并且经常出现内存溢出的情况。
下面就是几种尝试的方法，加快编译的速度
## 增加`Node`运行内存
在`Node`中通过`JavaScript`使用内存时只能使用部分内存（`64`位系统下约为`1.4 GB`，`32`位系统下约为`0.7 GB`）。所以不管电脑实际的运行内存是多少，`Node`在运行代码编译的时候，使用内存大小不会发生变化。这样就可能导致因为原有的内存不够，导致内存溢出。所以可以增加`Node`的运行内存，下面是两种方法
### 更改`cmd`
在`node_modules/.bin/vue-cli-server.cmd`把下面代码复制上去
```cmd
@IF EXIST "%~dp0\node.exe" (
  "%~dp0\node.exe" --max_old_space_size=4096 "%~dp0\..\@vue\cli-service\bin\vue-cli-service.js" %*
) ELSE (
  @SETLOCAL
  @SET PATHEXT=%PATHEXT:;.JS;=;%
  node --max_old_space_size=4096  "%~dp0\..\@vue\cli-service\bin\vue-cli-service.js" %*
)
```
### 更改`package.json`
把启动`Node`服务的更改下：
```cmd
node --max_old_space_size=4096 node_modules/@vue/cli-service/bin/vue-cli-service.js serve
```
本质上没啥区别，都是通过强行增加`Node`的运行内存，来解决内存溢出的问题。但是这种方法不治本，虽然不会造成内存溢出，但是编译速度还是挺慢的，编译完成还是需要等很久。

## 配置需要编译的文件
这种就是按需配置需要编译的文件，为什么出现内存溢出，本质上还是因为需要编译的文件太多了，那我们就可以减少需要编译的页面就可以解决这个问题。   
首先所有的页面配置都是放在`page.config.js`,如果我们需要对某些特定页面进行配置，就需要过滤所有的页面，获取需要编译的页面，下面是编译文件的写法
```js
const path = require('path');
const fs = require('fs');
const pages = require('../pages.config');

const params = JSON.parse(process.env.npm_config_argv).original;
const buildPath = params[params.length - 1].match(/[a-zA-Z0-9]+/)[0] || '';

let buildConfig = {
  pages: [],
};


if (!/(test|online|serve)/gi.test(buildPath)) {
  const configJsPath = path.resolve(__dirname, `${buildPath}.js`);

  // 如果该路径存在
  if (fs.existsSync(configJsPath)) {
    // eslint-disable-next-line import/no-dynamic-require
    buildConfig = require(configJsPath);
  } else if (pages[buildPath]) {
    buildConfig.pages = buildPath.split(',');
  } else {
    throw new Error('该路径不存在');
  }
} else {
  buildConfig = require('./default');
}
module.exports = buildConfig.pages;
```
大多数情况下，一个产品都是由多个业务线构成，每次可能需要更改的就是某一条业务线，就完全可以单独创建一个这条业务线的配置文件，然后再这个文件写入你需要编译的页面名称，就可以单独编译这个页面，或者说在调用通过传入的字符串来编译那些页面
## 使用`webbpack-dev-serve`钩子进行单独编译
在webpack进行热更新的时候，实际上是使用了`webpack-dev-server`这个的服务，然后是否有钩子能够给我们提供，如果我们访问哪个页面他就编译那个页面的代码。幸运的是找到，在`devServer`中存在这么一个钩子函数`before`， 那就可以在`vue.config.js`中修改
```js
const compiledPages = [];
before(app) {
      app.get('*.html', (req, res, next) => {
        const result = req.url.match(/[^/]+?(?=\.)/);
        const pageName = result && result[0];
        const pagesName = Object.keys(multiPageConfig);

        if (pageName) {
          if (pagesName.includes(pageName)) {
            if (!compiledPages.includes(pageName)) {
              const page = multiPageConfig[pageName];
              fs.writeFileSync(`dev-entries/${pageName}.js`, `import '../${page.tempEntry}'; // eslint-disable-line`);
              compiledPages.push(pageName);
            }
          } else {
            // 没这个入口
            res.writeHead(200, { 'content-type': 'text/html; charset=utf-8' });
            res.end('<p style="font-size: 50px;">不存在的入口</p>');
          }
        }
        next();
      });
    },
```
`multPageConfig`是多页面的配置，在开发环境中，做了一下修改，配置如下:
```js
{
    pageName: {
        entry: entryPath,
        chunks: [array]
    }
}
const fs = require('fs');
const util = require('util');

const outputFile = util.promisify(fs.writeFile);
async function main() {
  const tasks = [];
  if (!fs.lstatSync('dev-entries').isDirectory()) {
    fs.mkdirSync('dev-entries');
  }
  Object.keys(pages).forEach((key) => {
    const entry = `dev-entries/${key}.js`;
    pages[key].tempEntry = pages[key].entry; // 暂存真正的入口文件地址
    pages[key].entry = entry;
    tasks.push(outputFile(entry, ''));
  });
  await Promise.all(tasks);
}

if (process.env.NODE_ENV === 'development') {
  main();
}

module.exports = pages;
```
在上面文件中，我们首先需要更改多页面的配置，创建一个目录，包含所有的页面的`js`文件，但是需要注意的是这些文件都是空的文件，什么都没有，然后在`vue.config.js`中多页面的配置为
```js
pages: multiPageConfig, // 配置多页应用
```
因为所有的页面`js`已经被置为空了，所以编译的速度非常快。然后再访问页面的时候，`webpack`已经拦截访问的页面，也就是需要更改的页面，这时候就手动往`dev-entries`目录下写入需要编译的文件，从而实现了访问某个页面就编译哪个页面的代码

## 升级`html-webpack-plugin`版本
多页面出现内存溢出的问题是因为在编译的时候，实际是一次更改，编译了多个文件，这是`html-webpack-plugin`的问题。因为没生成一个页面，就需要调用一下`new htmlWebpackPlugin()`，多个页面的时候内存就不够用了。所以改一下这个这个`webpack`插件的版本,升级到`4.0.0-beta.8`这个版本。然后再`vue.config.js`中添加下面的配置，这样也不会造成内存溢出。
```js
const htmlPlugins = [];
Object.keys(multiPageConfig).forEach((key) => {
    htmlPlugins.push(multiPageConfig[key])
})
configureWebpack: {
    plugins: [
      ...htmlPlugins,
    ],
}
```

## 其他加快编译的技巧
`webpack`的插件还是很方便的，网上有啥`happypack`类似的插件。由于运行在 Node.js 之上的 Webpack 是单线程模型的，所以`Webpack`需要处理的事情需要一件一件的做，不能多件事一起做。
我们需要`Webpack`能同一时间处理多个任务，发挥多核`CPU`电脑的威力，`HappyPack`就能让`Webpack` 做到这点，它把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。可能是我电脑太烂了，装上没啥太大的提升，具体使用方法可以参照这篇文章[webpack优化之HappyPack 实战](https://www.jianshu.com/p/b9bf995f3712)。还有一些细节的地方比如说有些包需要加入编译，但是一般我们在调试的时候只需要在`chrome`上进行调试，开发环境就不用加入编译，多处使用的代码单独打包，这些也就不说了，大家多多尝试

这几种解决多页面内存溢出的方法各有优缺点，读者可根据自己的项目自行决定使用哪种方法，可能有时还需要多种方式组合使用，就看看那个好使好用了。  
解决这个问题顺便研究了一下`webpack`，配置果然博大精深，难怪市面有流传`webpack`配置工程师。推销一波自己的[github](https://github.com/skychenbo/Blog)最近在抓紧学习，会持续更新文章，希望大家多多关注。

