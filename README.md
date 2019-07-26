# Webpack学习笔记

反复思考如何学习webpack，最后决定根据阮一峰大神的学习demo通过阅读代码和修改demo来说学习,峰神这里使用的是webpack3,刚好我司ykit工具也兼容到了webpack3

> 命令行参数

峰神demo中定义了两条webpack命令行
```
"scripts": {
    "dev": "webpack-dev-server --open",
    "build": "webpack -p"
}
```

* `npm run dev`会执行`webpack-dev-server --open`这条命令
这里使用的是[webpack-dev-server](https://github.com/webpack/webpack-dev-server)插件，我这里简单提一嘴，我们只要记住使用webpack-dev-server启动服务，后面跟上 --open 就使用默认浏览器打开了。

* `npm run build`执行`webpack -p`会压缩混淆脚本，这个命令十分常用，多在项目beta阶段使用。

另外还有如下常用命令：
* $ webpack --config webpack.min.js //另一份配置文件

* $ webpack --display-error-details //显示异常信息

* $ webpack --watch   //监听变动并自动打包
 
* $ webpack -d    //生成map映射文件，告知哪些模块被最终打包到哪里了

## demo01
熟悉了命令，这里果断`cd demo01 && npm run dev`,浏览器看到了二字箴言`Hello World`。
```
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  }
};
```
这里也没什么总结的，在webpack.config.js中定义入口和输出就可以了。

## demo02
```
module.exports = {
  entry: {
    bundle1: './main1.js',
    bundle2: './main2.js'
  },
  output: {
    filename: '[name].js'
  }
};
```
这个demo有两个入口文件是main1.js和main2.js,选择两个文件分别打包输出,文件名使用入口中定义的名称输出文件。
模板 |	描述
--|--
[hash] |模块标识符(module identifier)的 hash
[chunkhash]|chunk 内容的 hash
[name]|模块名称
[id]|模块标识符(module identifier)
[query]|模块的 query，例如，文件名 ? 后面的字符串
* 这里可能看一下output.filename配置到底有多少种形式:
1. filename: "bundle.js" // 使用单一入口起点，静态（固定）名称如：bundle.js
2. filename: "[name].bundle.js" // 这里会使用入口名称作为文件的一部分，name就是entry的键名
3. filename: "[name].[hash].bundle.js" // 每次构建，生成威力hash
4. filename: "[chunkhash].bundle.js" // 使用基于chunk内容的hash

## demo03

在这个demo中使用了模块`module.rules`，创建模块时，匹配请求的规则数组。这些规则能够修改模块的创建方式。这些规则能够对模块(module)应用 loader，或者修改解析器(parser)。
```
module: {
    rules: [
        {
        // **Rule.test是Rule.resource.test的简写，正则匹配
        test: /\.jsx?$/,
        // **排除node_modules中的文件
        exclude: /node_modules/,
        // **指定loader
        use: {
            loader: 'babel-loader',
            options: {
            presets: ['es2015', 'react']
            }
        }
        }
    ]
}
```
这个配置就是说，匹配除node_modules文件夹外的.jsx文件，文件是react，res6标准。

## demo04

这个也是练习使用`module.rules`，和demo03差不多，只不过这里是在js文件中引用了css

```
module: {
    rules:[
        {
        test: /\.css$/,
        use: [ 'style-loader', 'css-loader' ]
        },
    ]
}
```
匹配css文件，使用style-loader和css-loader,`use: [ 'style-loader', 'css-loader' ]`的写法就是如下形式的简写。会将使css通过js在页面中生效
```
use: {
    loader: 'babel-loader',
    options: {
    presets: ['es2015', 'react']
    }
}
```

## demo05
这个demo练习如何将图片进行打包，这里的学问就大了，配置中通过url-loader将小于6.5k的jpg或png图片进行打包，也就是项目中可能存在的各种小图标，其实众多的小图标会占用http请求数，这里打包会将图片进行base64转换。但是通常会将最大限制设为默认的10000，也就是处理8k以内的图片，因为图片太大会导致转换后的字符串过长，反而进行一次http请求更为划算。

```
rules:[
{
    test: /\.(png|jpg)$/,
    use: [
        {
            loader: 'url-loader',
            options: {
                limit: 8192
            }
        }
    ]
}
```

## demo06 
看得出来，峰神的思路果然清晰，在这个练习中将之前的东西复习一下：
```
module: {
    rules:[
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['es2015', 'react']
          }
        }
      },
      {
        test: /\.css$/,
        use: [
          {
            loader: 'style-loader'
          },
          {
             loader: 'css-loader',
             options: {
               modules: true
             }
          }
        ]
      }
    ]
  }
```
这里难度不大，我们之前都已经见过，唯独不一样的是`test: /\.js[x]?$/`,这个正则会匹配js和jsx，这是一个小技巧。

## demo07
从这个练习开始，就要涉及插件的使用了。

```
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
      // **应用uglifyjs-webpack-plugin插件使用uglify压缩js文件
    new UglifyJsPlugin()
  ]
};
```
> Tips
* UglifyJS Webpack Plugin插件用来缩小（压缩优化）js文件，至少需要Node v6.9.0和Webpack v4.0.0版本。
* webpack 4之前的版本是通过webpack.optimize.CommonsChunkPlugin来压缩js，webpack 4版本之后被移除了，使用config.optimization.splitChunks来代替。

## demo08

```
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlwebpackPlugin({
      title: 'Webpack-demos',
      filename: 'index.html'
    }),
    new OpenBrowserPlugin({
      url: 'http://localhost:8080'
    })
  ]
};

```
在这个练习中，使用了两个插件，我们一个一个来看

`HtmlWebpackPlugin`主要最用有两个：
1. 为html文件中引入的外部资源如script、link动态添加每次compile后的hash，防止引用缓存的外部文件问题
2. 可以生成创建html入口文件，比如单页面可以生成一个html文件入口，配置N个html-webpack-plugin可以生成N个页面入口

更多的配置用到在学习，这里只看现在用得着的
`title`: 生成html文件的标题
`filename`: 输出的html的文件名称
生成的文件名称可以在webpack启动的服务中按照相应的路径进行访问。

`open-browser-webpack-plugin`的作用是当webpack准备好后打开浏览器到指定url
我们也发现这个demo中package.json的配置发生变化
* "dev": "webpack-dev-server ~~--open~~"

## demo09

```
// 开发环境插件，  为什么要来回转换？？？
var devFlagPlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [devFlagPlugin]
};

```
js中内容：
```
if (__DEV__) {
  document.write(new Date());
}
```
最终在页面显示出时间

## demo10

```
// **main.js
require.ensure(['./a'], function(require) {
  var content = require('./a');
  document.open();
  document.write('<h1>' + content + '</h1>');
  document.close();
});

```
这里使用了js异步加载，通过webpack来实现这一功能，也可以称之为代码分割。
在webpack看来require.ensure将会形成代码分割线，会将里面的代码再打包一个文件，而在里面require其他js文件的话也会在require.ensure完成之后的回调中请求。特别是一些体积大的三方插件类的代码，会导致首屏时间过长，这一优化很有必要，具体我会在另行学习。
借用网上的例子，不实用webpack如何实现这一过程：(以百度地图为例)
```
mapBtn.click(function() {
  //获取 文档head对象
  var head = document.getElementsByTagName('head')[0];
  //构建 <script>
  var script = document.createElement('script');
  //设置src属性
  script.async = true;
  script.src = "http://map.baidu.com/.js"
  //加入到head对象中
  head.appendChild(script);
})
```
[参考网址](https://www.jianshu.com/p/9fa38e536033)

## demo11 
```
// ** main.js
var load = require('bundle-loader!./a.js');

load(function(file) {
  document.open();
  document.write('<h1>' + file + '</h1>');
  document.close();
});
```
这个练习中还是对main.js文件进行了修改，这里使用了`bundle-loader`,其实我看了一下bundle-loader的源码，内部也是使用require.ensure来实现的，我们类比demo10就可以了。

## demo12
```
plugins: [
  // 抽离多个入口文件的公共部分，打包成公共文件
    new webpack.optimize.CommonsChunkPlugin({
      name: "commons",
      // (the commons chunk name)

      filename: "commons.js",
      // (the filename of the commons chunk)
    })
  ]
```
这一插件是用来进行公共代码抽离的，当然以上是webpack3中的写法，在**webpack4**中要像下面这样写：
```
module.exports = {
  optimization: {
      splitChunks: {
          cacheGroups: {
              commons: {
                  name: "commons",
                  chunks: "initial",
                  minChunks: 2
              }
          }
      }
  },
}
```

使用代码抽离后的打包数据：
```
     Asset       Size  Chunks                    Chunk Names
bundle2.js  448 bytes       0  [emitted]         bundle2
bundle1.js  444 bytes       1  [emitted]         bundle1
commons.js    1.35 MB       2  [emitted]  [big]  commons
```
不使用代码抽离的打包数据：
```
     Asset     Size  Chunks                    Chunk Names
bundle2.js  1.35 MB       0  [emitted]  [big]  bundle2
bundle1.js  1.35 MB       1  [emitted]  [big]  bundle1
```
效果还是很明显的，这个十分重要。

## demo13

```
module.exports = {
  entry: {
    app: './main.js',
    vendor: ['jquery'],
  },
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: 'vendor.js'
    })
  ]
};

```
这个练习是对demo12的补充，`vendor: ['jquery']`会让webpack引入jquery,并从众多引用的代码中抽离输出vendor.js。
打包数据如下：
```
    Asset       Size  Chunks                    Chunk Names
bundle.js  312 bytes       0  [emitted]         app
vendor.js     612 kB       1  [emitted]  [big]  vendor
```

## demo14
```
externals: {
  // require('data') is external and available
  //  on the global var data
  'data': 'data'
}
```
这个练习配置中使用了`externals`,字面含义就是外面的，作用就是将那些不想让webpack打包的文件引入，并且能够在全局调用，就在externals中引用。

```
// **bundle.js
"use strict";


var data = __webpack_require__(15);
var React = __webpack_require__(4);
var ReactDOM = __webpack_require__(19);

ReactDOM.render(React.createElement(
  'h1',
  null,
  data
), document.body);

/***/ }),
/* 15 */
/***/ (function(module, exports) {

module.exports = data;

```

## demo15

