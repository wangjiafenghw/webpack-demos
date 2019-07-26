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
