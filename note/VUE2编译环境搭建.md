
>这个编译环境是在使用laravel5.2和vue2重构newssdk项目的时候搭建的,主要用到了gulp、webpack等工具，其中gulp的主要作用是任务的拆分，监视文件改动并重新编译，webpack的作用是按照vue官方提供的配置对vue2编写的代码进行编译，并提供vue多页面编译的解决方案

## 依赖文件package.json
```json
{
  "name": "Project-List-Template",
  "version": "1.0.0",
  "description": "laravel5.2 with vue2 building environment",
  "main": "index.js",
  "scripts": {
    "build": "gulp webpack --env prod",
    "dev": "gulp webpack --env dev"
  },
  "author": "caoqiang",
  "license": "MIT",
  "dependencies": {
    "babel": "^6.23.0",
    "babel-core": "^6.23.1",
    "babel-loader": "^6.3.2",
    "babel-polyfill": "^6.23.0",
    "babel-preset-es2015": "^6.24.1",
    "browser-sync": "^2.18.8",
    "css-loader": "^0.26.1",
    "express": "^4.15.3",
    "extract-text-webpack-plugin": "^2.0.0-rc.3",
    "file-loader": "^0.10.0",
    "gulp": "^3.9.1",
    "less": "^2.7.2",
    "less-loader": "^2.2.3",
    "optimize-css-assets-webpack-plugin": "^1.3.0",
    "style-loader": "^0.13.1",
    "uglify-js": "mishoo/UglifyJS2#harmony",
    "uglifyjs-webpack-plugin": "^0.4.3",
    "url-loader": "^0.5.7",
    "vue": "^2.1.10",
    "vue-loader": "^11.1.0",
    "vue-template-compiler": "^2.1.10",
    "webpack": "^2.2.1",
    "webpack-stream": "^3.2.0"
  },
  "devDependencies": {
    "copy-webpack-plugin": "^4.0.1",
    "element-ui": "^1.3.6",
    "glob": "^7.1.2",
    "gulp-task-loader": "^1.4.4",
    "html-webpack-plugin": "^2.28.0",
    "minimist": "^1.2.0",
    "ora": "^1.3.0",
    "rimraf": "^2.6.1",
    "shelljs": "^0.7.8",
    "webpack-merge": "^4.1.0"
  },
  "engines": {
    "node": ">= 4.0.0",
    "npm": ">= 3.0.0"
  }
}

```

## gulp 入口文件
```javascript
'use strict';
var gulp = require("gulp");
var path = require("path");
var extend = require('extend');
var minimist = require('minimist');

//接收命令行传递的参数，默认是dev
var knownOptions = {
  string: 'env',
  default: { env: process.env.NODE_ENV || 'dev' }
};

var options = minimist(process.argv.slice(2), knownOptions);


var config = {
    base: path.join(__dirname),                                            //项目根目录
    env: options.env,                                                      //编译环境
    pkg: require('./package.json'),                                        //package.json内容
    version: "0.01",                                                       //版本号（这里可以省略，直接从pkg中读取即可）
    source: path.join(__dirname, 'resources/assets'),                      //要编译的VUE源码根目录
    test: path.join(__dirname, 'test'),                                    //测试目录
    dist: path.join(__dirname, 'public'),                                  //编译后的目录
    assets_custom_static: path.join(__dirname, 'resources/assets/static'), //静态文件目录
    assets_sub_directory: '',                                              //
    assets_public_path: '/'                                                //编译后根目录
};
var envConfig = require('./gulp-tasks/config/index.config')(config);       //获取配置
require('gulp-task-loader')(extend(config, envConfig));
```

这里面用到了`gulp-task-loader`插件，这个插件的作用是可以将gulp的任务拆分成多个文件放到一个目录下，默认是gulp-tasks目录

## webpack任务
> 目录结构如下
│  webpack.js    ----入口程序
├─config        ----存放配置的目录
│      -index.config ----环境信息配置
│      -utils.tool ----生成vue2的webpack配置的工具类
└─webpack-config ----存放webpack配置的目录
│      -vue-loader.config ----vue2的webpack配置
│      -webpack.base.config ----webpack的基本配置
│      -webpack.config ----多页面+vue2的webpa配置

### 入口程序webpack.js文件
```javascript
'use strict';

var ora = require('ora');        //实现node.js 命令行环境的 loading效果， 和显示各种状态的图标等
var rm = require('rimraf');
var path = require('path');
var chalk = require('chalk');    //使用 chalk 模块美化命令行输出
var webpack = require('webpack');
var extend = require('extend');  //对象的合并

module.exports = function() {
    var config = extend(this.opts, require('./config/utils.tool')(this.opts));//加载编译vue2的配置
    // console.log(config);
    // process.exit();
    var webpackConfig = require('./webpack-config/webpack.config')(config);
    var spinner = ora('building for development...')
    if (config.env=='prod') {
        spinner = ora('building for production...')
    }
    spinner.start()
    rm(path.join(config[config.env]['assetsRoot'], config[config.env]['assetsSubDirectory']), err => {
        if (err) throw err
        webpack(webpackConfig, function (err, stats) {
            spinner.stop()
            if (err) throw err
            process.stdout.write(stats.toString({
                colors: true,
                modules: false,
                children: false,
                chunks: false,
                chunkModules: false
            }) + '\n\n')

            console.log(chalk.cyan('  Build complete.\n'))
        })
    })
};
```
### 环境信息配置index.config
```javascript
var path = require('path')

module.exports = function (gconfig) {
    return {
        prod: {
            assetsRoot: gconfig.dist,
            assetsSubDirectory: gconfig.assets_sub_directory,
            assetsPublicPath: gconfig.assets_public_path,
            sourceMap: false,
            // Gzip off by default as many popular static hosts such as
            // Surge or Netlify already gzip all static assets for you.
            // Before setting to `true`, make sure to:
            // npm install --save-dev compression-webpack-plugin
            productionGzip: false,
            productionGzipExtensions: ['js', 'css'],
            // Run the build command with an extra argument to
            // View the bundle analyzer report after build finishes:
            // `npm run build --report`
            // Set to `true` or `false` to always turn it on or off
            bundleAnalyzerReport: process.env.npm_config_report
        },
        dev: {
            assetsRoot: gconfig.dist,
            assetsSubDirectory: gconfig.assets_sub_directory,
            assetsPublicPath: gconfig.assets_public_path,
            sourceMap: true
        }
    }
}

```

### 生成vue2的webpack配置的工具类utils.tool
```javascript
var path = require('path')
var ExtractTextPlugin = require('extract-text-webpack-plugin')

module.exports = function (config) {
    return {
        assetsPath: function (_path) {
            return path.posix.join(config[config.env]['assetsSubDirectory'], _path);
        },
        cssLoaders: function (options) {
            options = options || {}

            var cssLoader = {
                loader: 'css-loader?minimize',
                options: {
                    minimize: config.env === 'prod',
                    sourceMap: options.sourceMap
                }
            }

            // generate loader string to be used with extract text plugin
            function generateLoaders (loader, loaderOptions) {
                var loaders = [cssLoader];
                if (loader) {
                    loaders.push({
                        loader: loader + '-loader',
                        options: Object.assign({}, loaderOptions, {
                            sourceMap: options.sourceMap
                        })
                    })
                }

                // Extract CSS when that option is specified
                // (which is the case during production build)
                if (options.extract) {
                    return ExtractTextPlugin.extract({
                        use: loaders,
                        fallback: 'vue-style-loader'
                    })
                } else {
                    return ['vue-style-loader'].concat(loaders)
                }
            }

            return {
              css: generateLoaders(),
              postcss: generateLoaders(),
              less: generateLoaders('less'),
              sass: generateLoaders('sass', { indentedSyntax: true }),
              scss: generateLoaders('sass'),
              stylus: generateLoaders('stylus'),
              styl: generateLoaders('stylus')
            }
        },
        styleLoaders: function (options) {
            var output = [];
            var loaders = this.cssLoaders(options);
            for (var extension in loaders) {
                var loader = loaders[extension]
                output.push({
                    test: new RegExp('\\.' + extension + '$'),
                    use: loader
                })
            }
            return output
        }
    }
}
```
### vue2的webpack配置vue-loader.config
```javascript

//将所有 Vue 组件中的所有已处理的 CSS 提取为单个 CSS 文件配置示例
module.exports = function (config) {
    return {
        loaders: config.cssLoaders({
            sourceMap: config[config.env]['sourceMap'],
            extract: config.env === 'prod'
        })
    }
}

```

### webpack的基本配置webpack.base.config
```javascript
var path = require('path')
var glob = require('glob')

module.exports = function (config) {
    var vueLoaderConfig = require('./vue-loader.config')(config)
    //获得js文件的路径
    const entries = {};
    glob.sync(config.source+'/app/**/app.js').forEach(p => {
        var chunk = p.split('/app/')[1].split('/app.js')[0];
        chunk = chunk.substr(0,chunk.indexOf('.js'));
        entries[chunk] = path.resolve(p)
    })
    return {
        entry: entries,//配置入口文件，有几个写几个
        output: {
            //输出目录的配置，模板、样式、脚本、图片等资源的路径配置都相对于它
            path: config[config.env]['assetsRoot'],
            //每个页面对应的主js的生成配置
            filename: '[name].js',
            //模板、样式、脚本、图片等资源对应的server上的路径
            publicPath: config[config.env]['assetsPublicPath']
        },
        resolve: {
            extensions: ['.js', '.vue', '.json'],
            alias: {
                'vue$': 'vue/dist/vue.esm.js',
                '@': config.source
            }
        },
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    loader: 'vue-loader',
                    options: vueLoaderConfig
                },
                {   //只对项目目录下src目录里的代码进行babel编译
                    test: /\.js$/,
                    loader: 'babel-loader?cacheDirectory',//开启babel-loader缓存, 复用缓存结果减少编译流程
                    // include: [config.source, config.test]
                    include: [config.source]
                },
                {
                    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                    loader: 'url-loader',
                    options: {
                        limit: 10000,
                        name: config.assetsPath('img/[name].[ext]')
                    }
                },
                {
                    test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                    loader: 'url-loader',
                    options: {
                        limit: 10000,
                        name: config.assetsPath('fonts/[name].[ext]')
                    }
                }
            ]
        }
    }
}

```

### 多页面+vue2的webpa配置webpack.config
```javascript
var path = require('path')
var webpack = require('webpack')
var merge = require('webpack-merge')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

module.exports = function (config) {
    var self = this;
    self.config = config;
    var baseWebpackConfig = require('./webpack.base.config')(config)

    var defaultConfig = {
        module: {
            rules: config.styleLoaders({
                sourceMap: self.config[self.config.env]['sourceMap'],
                extract: true
            })
        },
        devtool: self.config[self.config.env]['sourceMap'] ? 'source-map' : false,
        output: {
            path: self.config[self.config.env]['assetsRoot'],
            filename: self.config.assetsPath('js/app/[name].js'),
            chunkFilename: self.config.assetsPath('js/app/[id].js')
        },
        plugins: [
            new webpack.DefinePlugin({
                'process.env': self.config.env
            }),
            new webpack.optimize.UglifyJsPlugin({
                // 最紧凑的输出
                beautify: false,
                // 删除所有的注释
                comments: false,
                compress: {
                    // 在UglifyJs删除没有用到的代码时不输出警告
                    warnings: false,
                    // 删除所有的 `console` 语句
                    // 还可以兼容ie浏览器
                    drop_console: true,
                    // 内嵌定义了但是只用到一次的变量
                    collapse_vars: true,
                    // 提取出出现多次但是没有定义成变量去引用的静态值
                    reduce_vars: true
                },
                sourceMap: self.config[self.config.env]['sourceMap']
            }),
            // extract css into its own file
            new ExtractTextPlugin({
                filename: self.config.assetsPath('css/[name].css')
            }),
            // Compress extracted CSS. We are using this plugin so that possible
            // duplicated CSS from different components can be deduped.
            new OptimizeCSSPlugin({
                cssProcessorOptions: {
                    safe: true
                }
            }),
            // split vendor js into its own file
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor',
                minChunks: function (module, count) {
                    // any required modules inside node_modules are extracted to vendor
                    return (
                        module.resource &&
                        /\.js$/.test(module.resource) &&
                        module.resource.indexOf(
                        path.join(self.config.base, 'node_modules')) === 0
                    )
                }
            }),
            //导出webpack库
            new webpack.optimize.CommonsChunkPlugin({
                name: 'manifest',
                chunks: ['vendor']
            })
        ]
    }

    //复制静态文件
    if (self.config.assets_custom_static) {
        defaultConfig.plugins.push(new CopyWebpackPlugin([{
            from: self.config.assets_custom_static,
            to: self.config[self.config.env]['assetsSubDirectory'],
            ignore: ['.*']
        }]));
    }
    var webpackConfig = merge(baseWebpackConfig, defaultConfig)

    if (self.config[self.config.env]['productionGzip']) {
      var CompressionWebpackPlugin = require('compression-webpack-plugin')

      webpackConfig.plugins.push(
        new CompressionWebpackPlugin({
          asset: '[path].gz[query]',
          algorithm: 'gzip',
          test: new RegExp(
            '\\.(' +
            self.config[self.config.env]['productionGzipExtensions'].join('|') +
            ')$'
          ),
          threshold: 10240,
          minRatio: 0.8
        })
      )
    }

    if (self.config[self.config.env]['bundleAnalyzerReport']) {
      var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
      webpackConfig.plugins.push(new BundleAnalyzerPlugin())
    }
    return webpackConfig;
}

```