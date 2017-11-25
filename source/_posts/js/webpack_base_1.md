---
title: webpack基本配置备忘录
date: 2017-09-20 17:46:25
updated: 2017-11-25 09:40:00
categories: 学习
tags: [JS,webpack]
---
项目中用到webpack，但照着文档设置后就没有再动过。
<!--more-->
#### 入口[Entry]

> webpack 创建应用程序所有依赖的关系图(dependency graph)。图的起点被称之为入口起点(entry point)。入口起点告诉 webpack 从哪里开始，并根据依赖关系图确定需要打包的内容。可以将应用程序的入口起点认为是根上下文(contextual root) 或 app 第一个启动文件。

实际项目中为了实现多平台化，用到了gulp，用一个参数(clientEntry)来指定，并在最后赋值给entry
```javascript
var clientEntry ={
  main: path.resolve('./src', 'client/bootstrap/main.js'),
  vendor:['react','react-dom','react-css-modules','react-redux', 'react-router', 'react-router-redux']
};
//plugin
new webpack.optimize.CommonsChunkPlugin({
   name: "vendor",
   minChunks: Infinity
})
将部分第三方库提取到vendor中，方便缓存。
```
#### 出口[Output]

将所有的资源(assets)归拢在一起后，还需要告诉 webpack 在哪里打包应用程序。webpack 的 output 属性描述了如何处理归拢在一起的代码(bundled code)

```javascript
output: {
    filename: '[name]-[hash].js',
    chunkFilename: '[name]-[chunkhash].js'
}
```
#### Loader

webpack 把每个文件(.css, .html, .scss, .jpg, etc.) 都作为模块处理。然而 webpack 自身只理解 JavaScript。
webpack loader 在文件被添加到依赖图中时，其转换为模块。
在更高层面，在 webpack 的配置中 loader 有两个目标。

识别出(identify)应该被对应的 loader 进行转换(transform)的那些文件。(test 属性)
转换这些文件，从而使其能够被添加到依赖图中（并且最终添加到 bundle 中）(use 属性)
```javascript
{
     test: /\.css$/,
     modules: false,
     include: [path.resolve(rootdir, 'node_modules')],
     use: ['style-loader', 'css-loader']
 },
 {
     test: /\.s(c|a)ss$/,
     include: [path.resolve(rootdir, 'src')],
     use: [
         {loader: 'style-loader'},
         {loader: 'css-loader'},
         {
             loader: 'postcss-loader',
             options: {
                 config: {
                     ctx: {
                         autoprefixer: { browsers: ['last 10 Chrome versions', 'last 5 Firefox versions', 'Safari >= 6', 'ie > 8'] }
                     }
                 }
             }
         },
         {
             loader: 'sass-loader',
             options:{
                 sourceMap:true
             }
         }
     ]
 },
 {
     test: /\.less$/,
     // include: [path.resolve(rootdir, 'src')],
     use: [
         {loader: 'style-loader'},
         {loader: 'css-loader'},
         {
             loader: 'less-loader',
             options:{
                 sourceMap:true
             }
         }
     ]
 },
 {
     test: function (path) {
         return !path.match(/content\/(.*)\.png$/) && path.match(/\.png$/);
     },
     include: [path.resolve(rootdir, 'src')],
     use: [
       {
           loader:'url-loader',
           options:{
               limit:10240,
               name:'assets/[name]-[hash:20].[ext]'
           }
       },
     ]
 },
```
#### 插件(Plugins)

插件是 wepback 的支柱功能。webpack 自身也是构建于，你在 webpack 配置中用到的相同的插件系统之上！
插件目的在于解决 loader 无法实现的其他事。

由于插件可以携带参数/选项，必须在 webpack 配置中，向 plugins 属性传入 new 实例。
```javascript
plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: "vendor",
                minChunks: Infinity
            }),
            new webpack.LoaderOptionsPlugin({
                options: {
                    context: '/',
                    postcss:{
                        includePaths: [
                            path.resolve(rootdir, 'src/client'),
                            path.resolve(rootdir, 'src'),
                            path.resolve(rootdir, 'node_modules')
                        ]
                    },
                    sassLoader: {
                        includePaths: [
                            path.resolve(rootdir, 'src/client'),
                            path.resolve(rootdir, 'src'),
                            path.resolve(rootdir, 'node_modules')
                        ]
                    }
                }
            }),
            new webpack.ContextReplacementPlugin(/moment[\/\\]locale$/, /zh-cn/),
            new webpack.NoEmitOnErrorsPlugin(),
            new webpack.BannerPlugin(config.copyright),
            new ExtractTextPlugin({
                filename:'[contenthash:20].[name].css',
                allChunks: true})
        ],
```
CommonsChunkPlugin 插件，是一个可选的用于建立一个独立文件(又称作 chunk)的功能，这个文件包括多个入口 chunk 的公共模块。通过将公共模块拆出来，最终合成的文件能够在最开始的时候加载一次，便存起来到缓存中供后续使用。这个带来速度上的提升，因为浏览器会迅速将公共的代码从缓存中取出来，而不是每次访问一个新页面时，再去加载一个更大的文件。

loader-options-plugin和其他插件不同，它用于将 webpack 1 迁移至 webpack 2。在 webpack 2中，对 webpack.config.js 的结构要求变得更加严格；不再开放扩展给其他的 loader/插件。webpack 2 推荐的使用方式是直接传递 options 给 loader/插件（换句话说，配置选项将不是全局/共享的）。
不过，在某个 loader 升级为依靠直接传递给它的配置选项运行之前，可以使用 loader-options-plugin 来抹平差异。你可以通过这个插件配置全局/共享的 loader 配置，使所有的 loader 都能收到这些配置。

上下文替换插件(ContextReplacementPlugin) 与一个带表达式的 require 语句 相关，例如 require(‘./locale/‘ + name + ‘.json’)。遇见此类表达式时，webpack 查找目录 (‘./locale/‘) 下符合正则表达式 (/^.*.json$/)的文件。由于 name 在编译时(compile time)还是未知的，webpack 会将每个文件都作为模块引入到 bundle 中。

代码中用到了moment库，主要用来获取当前时间的。

在编译出现错误时，使用 NoEmitOnErrorsPlugin来跳过输出阶段。这样可以确保输出资源不会包含错误。

BannerPlugin 为每个 chunk 文件头部添加 banner。

ExtractTextPlugin会将所有的入口 chunk(entry chunks)中引用的 *.css，移动到独立分离的 CSS 文件。因此，你的样式将不再内嵌到 JS bundle 中，而是会放到一个单独的 CSS 文件（即 styles.css）当中。 如果你的样式文件大小较大，这会做更快提前加载，因为 CSS bundle 会跟 JS bundle 并行加载。

此外 在项目中还用到了一些插件：
```javascript
// jscs:disable disallowVar
/* eslint-disable no-var */
var path = require('path');
var webpack = require('webpack');
var autoprefixer = require('autoprefixer');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var rootdir = process.cwd();
module.exports = function (config) {
    var client = {
        name: config.env || 'app' + '-client-' + config.locale || 'default',
        target: 'web',
        devtool: 'source-map',
        output: {
            filename: '[name]-[hash].js',
            chunkFilename: '[name]-[chunkhash].js'
        },
        module: {
            rules: [
                {
                    test: /\.jsx?$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: [{loader:'babel-loader'}]
                },
                {
                    test: /\.css$/,
                    modules: true,
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                        {loader:'style-loader'},
                        {
                            loader: 'css-loader',
                            options:{
                                root:'./client'
                            }
                        }
                    ]
                },
                {
                    test: /\.css$/,
                    modules: false,
                    include: [path.resolve(rootdir, 'node_modules')],
                    use: ['style-loader', 'css-loader']
                },
                {
                    test: /\.s(c|a)ss$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                        {loader: 'style-loader'},
                        {loader: 'css-loader'},
                        {
                            loader: 'postcss-loader',
                            options: {
                                config: {
                                    ctx: {
                                        autoprefixer: { browsers: ['last 10 Chrome versions', 'last 5 Firefox versions', 'Safari >= 6', 'ie > 8'] }
                                    }
                                }
                            }
                        },
                        {
                            loader: 'sass-loader',
                            options:{
                                sourceMap:true
                            }
                        }
                    ]
                },
                {
                    test: /\.less$/,
                    // include: [path.resolve(rootdir, 'src')],
                    use: [
                        {loader: 'style-loader'},
                        {loader: 'css-loader'},
                        {
                            loader: 'less-loader',
                            options:{
                                sourceMap:true
                            }
                        }
                    ]
                },
                {
                    test: function (path) {
                        return !path.match(/content\/(.*)\.png$/) && !path.match(/wx_mp\/img/) && path.match(/\.png$/);
                    },
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                      {
                          loader:'url-loader',
                          options:{
                              limit:10240,
                              name:'assets/[name]-[hash:20].[ext]'
                          }
                      },
                    ]
                },
                {
                    test: /content\/(.*)\.(png|svg|gif|jpe?g)$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: [{
                        loader:'file-loader',
                        options: {
                            name: 'assets/content/[name].[ext]'
                        }
                    }]
                },
                {
                    test: function (path) {
                        return !path.match(/content\/(.*)\.(gif|svg|jpe?g)$/) && !path.match(/wx_mp\/img/) && path.match(/\.(gif|jpe?g|svg)$/);
                    },
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                        {
                            loader:'file-loader',
                            options:{
                                name:'assets/[name]-[hash:20].[ext]'
                            }
                        }
                    ]
                },
                {
                    test: /\.ico$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                        {
                            loader:'file-loader',
                            options: {
                                name: '[name].[ext]'
                            }
                        }
                    ]
                },
                {
                    test: /\.woff|ttf|eot$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: [
                        {
                            loader:'url-loader',
                            options: {
                                limit:1024,
                                name: 'assets/fonts/[name].[ext]'
                            }
                        }
                    ]
                }
            ],
            noParse: /node_modules\/quill\/dist/
        },
        resolve: {
            extensions: ['.web.js', '.js', '.json'],
            modules: [  path.resolve(rootdir, 'src/client'), path.resolve(rootdir, 'src'), path.resolve(rootdir, 'node_modules')]
        },
        resolveLoader: {
            modules: [
                path.resolve(rootdir, 'webpack/loaders'),
                path.resolve(rootdir, 'node_modules')
            ]
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: "vendor",
                minChunks: Infinity
            }),
            new webpack.LoaderOptionsPlugin({
                options: {
                    context: '/',
                    postcss:{
                        includePaths: [
                            path.resolve(rootdir, 'src/client'),
                            path.resolve(rootdir, 'src'),
                            path.resolve(rootdir, 'node_modules')
                        ]
                    },
                    sassLoader: {
                        includePaths: [
                            path.resolve(rootdir, 'src/client'),
                            path.resolve(rootdir, 'src'),
                            path.resolve(rootdir, 'node_modules')
                        ]
                    }
                }
            }),
            new webpack.ContextReplacementPlugin(/moment[\/\\]locale$/, /zh-cn/),
            new webpack.NoEmitOnErrorsPlugin(),
            new webpack.BannerPlugin(config.copyright),
            new ExtractTextPlugin({
                filename:'[contenthash:20].[name].css',
                allChunks: true})
        ],
        devServer: {
            host: '0.0.0.0',
            proxy: {
                '/api/*': {
                    target: 'http://139.196.34.65/',
                    secure: false,
                    changeOrigin: true
                }
            }
        },
        uglify: true,
        offline: false,
        html: ''
    };
    var server = {
        name: config.env || 'app' + '-server-' + config.locale || 'default',
        target: 'node',
        devtool: 'source-map',
        output: {
            filename: '[name].js',
            chunkFilename: '[name].[chunkhash].js'
        },
        module: {
            rules: [
                {
                    test: /\.jsx?$/,
                    include: [path.resolve(rootdir, 'src')],
                    use: ['babel-loader']
                }
            ]
        },
        resolve: {
            alias: {
                'any-promise': path.resolve(rootdir, 'src/server/shim/any-promise'),
                'busboy': path.resolve(rootdir, 'src/server/shim/busboy')
            },
            modules: [path.resolve(rootdir, 'src'), path.resolve(rootdir, 'src/server'), path.resolve(rootdir, 'node_modules')]
        },
        plugins: [
            new webpack.NoEmitOnErrorsPlugin(),
            new webpack.BannerPlugin(config.copyright),
            new webpack.DefinePlugin({
                'process.env': {
                    'NODE_PG_FORCE_NATIVE': 'undefined'
                }
            })
        ],
        node: {
            __filename: true,
            __dirname: true
        }
    };
    return [client, server];
};
```