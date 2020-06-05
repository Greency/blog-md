- #### 声明
> 这一篇是紧接着上一篇"《简单实现CLI》（一）之实现基于vue的组件库开发环境"。在上一篇中所安装的依赖，在这一篇中也是相同的。所以直接略过“初始化环境”的篇章。

- #### 安装依赖
1. 为了实现将每个.vue文件中的css抽取出来打包成一个.css文件，所以我们需要安装==extract-text-webpack-plugin==。请记住，一定要安装它的4.0beta版本。
> 最终，安装完成后，package.json文件内容如下：
```
"dependencies": {
    "commander": "^2.20.0",
    "execa": "^2.0.3",
    "extract-text-webpack-plugin": "^4.0.0-beta.0",  //新增
    "fs-extra": "^8.1.0",
    "html-webpack-plugin": "^3.2.0",
    "mini-css-extract-plugin": "^0.7.0",
    "vue-loader": "^15.7.0",
    "vue-template-compiler": "^2.6.10",
    "webpack": "^4.35.3",
    "webpack-dev-server": "^3.7.2"
  }
```
- #### 构建基础环境：example-cli create-lib xxx
1. 在bin/index.js文件中新增如下内容：
```
program
    .command('create-lib <projectName>')
    .action((projectName) => {
        require('../lib/command/createLib.js')(projectName);
    });
```
2. 在lib/command目录下，新增createLib.js文件，并添加如下内容：
```
const createLibProject = require('../utils/createLibProject'),
    createPackageJson = require('../utils/createPackageJson'),
    installDependencies = require('../command/installDependencies');


module.exports = function(projectName){
    //以<projectName>为名称创建目录，并将template目录下的library全部复制到<projectName>目录下
    createLibProject(projectName);
    //在<projectName>目录下，新建package.json文件，并在此文件中添加基本的信息以及项目所需要的依赖
    createPackageJson(projectName);
    //执行npmm install，安装项目的依赖
    installDependencies(projectName);
}
```

- #### 打包命令：npm run lib
> 组件库的打包需要遵守babel-plugin-component的规范，所以我们最终要的目录结构如下：
```
lib
    theme-chalk
        base.css  //所以css的集合
        button.css  //单个组件的css
    
    index.js  //所以js的集合
    button.js  //单个组件的js
```
> 安装上面的目录结构，我们需要分为两步走：第一步是先进行整个组件库的打包，也就是base.css，index.js文件的生成。第二步是每个组件的css，js单独打包。

1. 注册脚本命令，在config/packageJson.js文件中如下内容：
```
scripts: {
    "lib": "example-cli lib-entirety && example-cli lib-each"
}
```
> 可以看到，运行npm run lib其实是执行的example-cli lib-entirety和example-cli lib-each这两个命令。所以，我们就开始这两个命令的编写吧。

- #### 打包整体：example-cli lib-entirety
1. 在bin/index.js下，新增如下内容：
```
program
    .command('lib-entirety')
    .action(()=>{
        require('../lib/command/libEntirety')();
    });
```
2. 在lib/command目录下新建libEntirety.js文件，并添加如下内容：
```
const execa = require('execa'),
    path = require('path');

module.exports = function () {
    let execaDir = path.resolve(__dirname, '../scripts/libEntirety.js');
    const childProcess = execa('node', [execaDir]);

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
};
```
3. 在lib/config目录下，新建library目录，并entirety.js文件，并添加如下内容：
```
const VueLoaderPlugin = require('vue-loader/lib/plugin'),
    ExtractTextPlugin = require("extract-text-webpack-plugin"),
    path = require('path');

module.exports = {
    mode: 'production',
    entry: path.resolve('packages/index.js'),
    output: {
        path: path.resolve('lib'),
        filename: 'index.js',
        libraryTarget: 'commonjs2'
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                use: ExtractTextPlugin.extract({  //将每个组件中的css全部打包成一个css文件
                    fallback: 'style-loader',
                    use: ['css-loader']
                })
            },
            {
                test: /\.styl(us)?$/,
                use: ExtractTextPlugin.extract({
                    fallback: 'style-loader',
                    use: ['css-loader', 'stylus-loader']
                })
            },
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new ExtractTextPlugin("theme-chalk/base.css")  //将每个组件中的css全部打包成一个css文件
    ],
    resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']
    },
    externals: /^(vue|\$)$/i  //防止将vue也打包进来
};
```
3. 在lib/scripts目录下，新增libEntirety.js文件，并添加如下内容：
```
const webpack = require('webpack');

let buildConfig = require('../config/library/entirety');
const compiler = webpack(buildConfig);

compiler.run((err, stats) => {
    if (err || stats.hasErrors()) {
        if (err)
            console.log(err);

        if (stats.hasErrors())
            console.log(stats.toJson('minimal').errors);

        if (stats.hasWarnings())
            console.log(stats.toJson('minimal').warnings);
    } else {
        console.log('build successful!');
    }
});
```

- #### 打包单个组件：example-cli lib-each
1. 在bin/index.js文件中新增如下内容：
```
program
    .command('lib-each')
    .action(()=>{
        require('../lib/command/libEach.js')();
    });
```
2. 在lib/command目录下，新增libEach.js文件，并添加如下内容：
```
const execa = require('execa'),
    path = require('path');

module.exports = function () {
    let execaDir = path.resolve(__dirname, '../scripts/libEach.js');
    const childProcess = execa('node', [execaDir]);

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
};
```
3. 在lib/config/library目录下，新增each.js文件，并添加如下内容：
```
const VueLoaderPlugin = require('vue-loader/lib/plugin'),
    MiniCssExtractPlugin = require('mini-css-extract-plugin'),
    path = require('path'),
    fs = require('fs');

/**
 * 约定一下：
 * 组件库打包的时候，需要在项目的根目录下新建一个components.json文件，
 * 此文件就是每个组件的相对路径，
 * 格式未： "component name": "component url"
 */
let componentsJsonDir = path.resolve(process.cwd(), 'components.json'),
    components = {};

if (fs.existsSync(componentsJsonDir)) {
    components = require(componentsJsonDir);
} else {
    console.log('未在当前目录下找到 components.json 文件，打包失败！');
}

module.exports = {
    mode: 'production',
    entry: components,
    output: {
        path: path.resolve('lib'),
        filename: '[name].js',
        libraryTarget: 'commonjs2'  //决定打包后的模式
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,  //用于将每个组件的css文件独立出来
                    'css-loader'
                ]
            },
            {
                test: /\.styl(us)?$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader',
                    'stylus-loader'
                ]
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new MiniCssExtractPlugin({  //用于将每个组件的css文件独立出来
            filename: 'theme-chalk/[name].css'
        })
    ],
    resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']  //在引入文件或模块时，可以不用写文件后缀
    },
    externals: /^(vue|\$)$/i  //防止将vue也打包
};
```
4. 在lib/scripts目录下，新增libEach.js文件，并添加如下内容：
```
const webpack = require('webpack');

let buildConfig = require('../config/library/each');
const compiler = webpack(buildConfig);

compiler.run((err, stats) => {
    if (err || stats.hasErrors()) {
        if (err)
            console.log(err);

        if (stats.hasErrors())
            console.log(stats.toJson('minimal').errors);

        if (stats.hasWarnings())
            console.log(stats.toJson('minimal').warnings);
    } else {
        console.log('build successful!');
    }
});
```

- #### 完毕
> 好啦，现在你可以使用example-cli create-lib创建一个组件库基础项目，然后通过npm run lib进行打包。

> 从上面的代码可以看出，有很多重复的地方，还有很大的优化空间，所以后续我将会继续完善。