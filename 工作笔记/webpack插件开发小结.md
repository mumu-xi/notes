### webpack 插件开发小结

实现功能: 获取 dist 文件 -> dist 文件打包 tar.az -> 发送请求 -> 接收到后端返回 id -> 发起轮询 -> 上传状态接收 -> end

#### 基础环境搭建

为了让项目的打包速度更快，新建了一个 create-react-app。但是 CRA 将 webpack 进行了封装，不便于修改 webpack 配置。
在这里，使用了**npm eject**，会将封装在 CRA 中的配置全部反编译到当前项目，这样用户就可以完全取得 webpack 文件的控制权。

```
.
├── config
│   ├── webpack.config.js
│   └── webpackDevServer.config.js
```

接着，在项目下新建一个 cdn-upload-webpack-plugin 文件用于写插件，执行**npm init**,初始化完成后，就可以开始进行 webpack 配置啦。

#### webpack 插件基础架构

根据[官方文档](https://webpack.docschina.org/contribute/writing-a-plugin/), 一个插件由以下几部分组成：

- 一个具名 JavaScript 函数。
- 在它的原型上定义 apply 方法。
- 指定一个触及到 webpack 本身的 事件钩子。
- 操作 webpack 内部的实例特定数据。
- 在实现功能后调用 webpack 提供的 callback。

基于此

```
// index.js

class CdnUploadWebpackPlugin {
  // 在构造函数中获取用户给该插件传入的配置
  constructor(options) {
  }

  // Webpack 会调用 CdnUploadWebpackPlugin 实例的 apply 方法给插件实例传入 compiler 对象
  apply(compiler) {}
}

// 导出 Plugin
module.exports = CdnUploadWebpackPlugin;

```

然后，在/config/webpack.config.js plugins 数组中增加一个实例

```
module.exports = {
  // ... 这里是其他配置 ...

  plugins: [
      new CdnUploadWebpackPlugin({
        cdnSubPath: "/psdcloud/static",
        fileMd5: "md5"
     }),
  ]
};

```

#### [compiler 钩子](https://webpack.docschina.org/api/compiler-hooks/#afterplugins)

webpack 的 Compiler 模块是主引擎，它通过 webpack CLI 或 webpack API 或 webpack 配置文件传递的所有选项，创建出一个 compilation 实例。通过 Compiler 暴露的生命周期钩子函数，可以访问 weboack 的整个生命周期。

```
apply(compiler) {
    compiler.hooks.afterEmit.tap("customstring", async function(stats) { // 编译(compilation)完成。

    });
}
```

在这里使用了 afterEmit 这个钩子，在编译完成后去读取文件夹，进行打包。

#### dist 文件打包方案

参考： [基于 Node.js 实现压缩和解压缩](https://zhuanlan.zhihu.com/p/33783583)

Stream 接口: 因为在 HTTP 服务领域，Stream 模型会有更大的优势（ HTTP 请求本身就是一个 Request Stream），如要将一个上传文件以 tgz 压缩，使用 Stream 接口不需要将上传文件保存到本地磁盘，而是直接消费这个文件流。
同时，使用 pump 模块来配合 Stream 模式编程，由 pump 来完成这些 Stream 的清理工作。

```
function tgzFile(outputPath) {
 const tarStream = new compressing.tar.Stream();
 const destStream = fs.createWriteStream("dist.tgz");
 const fileStream = new compressing.gzip.FileStream();

 tarStream.addEntry(outputPath);

 return pump(tarStream, fileStream, destStream);
}
```

#### node 轮询方案

```
const POLLING_INTERVAL = 1000;

 polling(id) {
    timerFn();
    async function timerFn() {
      const response = await checkUploadStats(id); // 发送检测请求
        if (stillNeedPolling) { // 需要继续轮询
          setTimeout(timerFn, POLLING_INTERVAL);
        } else {
          console.error("polling error", response);
        }
      }
    }
  }
```

----------------end----------------------

