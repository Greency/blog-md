- #### 初始化环境
1. 新建example-cli目录（当然，这里的名称可以随意，没有强制要求）
2. 在example-cli目录下新建package.json，并增加如下内容（具体的字段名，我这里就不详细介绍了）：
```
{
    "name": "@codercy1995/example-cli",
    "version": "1.0.0",
    "description": "Command line interface for Vue.js",
    "license": "MIT",
    "author": {
        "name": "coderCy"
    },
    "repository": {
        "type": "git",
        "url": "git@github.com:Greency/example-cli.git"
    }
}
```
3. 初始化npm，执行如下命令：
> 在初始化的时候，会让你填写一些信息，直接默认即可
```
npm init
```
> 最终效果如下：
```
{
  "name": "@codercy1995/example-cli",
  "version": "1.0.0",
  "description": "Command line interface for Vue.js",
  "license": "MIT",
  "author": "coderCy",
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/Greency/example-cli.git"
  },
  "bugs": {
    "url": "https://github.com/Greency/example-cli/issues"
  },
  "homepage": "https://github.com/Greency/example-cli#readme",
  "main": "index.js",  //这个可以去掉
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "vue"
  ]
}
```

- #### 安装依赖
> 使用npm install --save xxx
1. 为了实现命令行的交互，我们需要安装==commander==
2. 为了是实现在js文件中调用命令，比如：npm install, node xxx.js。我们需要安装==execa==。
3. 为了能够移动文件，我们需要安装==fs-extra==。
4. 这个cli，我们是基于webpack的，所以需要安装==webpack==来进行打包。还需要安装==webpack-dev-server==来进行本地启动服务。
4. 在解析.vue单文件中，我们需要安装==vue-loader==。再由于vue-loader的原因，我们需要==vue-template-compiler==
5. 我们需要webpack的一个插件来生成html以及对应的资源引用。所以我们要安装==html-webpack-plugin==
6. 我们需要==mini-css-extract-plugin==来提取css到单独的.css文件中。

> 最终，安装完后，package.json文件中的依赖信息如下：
```
"dependencies": {
    "commander": "^2.20.0",
    "execa": "^2.0.3",
    "fs-extra": "^8.1.0",
    "html-webpack-plugin": "^3.2.0",
    "mini-css-extract-plugin": "^0.7.0",
    "vue-loader": "^15.7.0",
    "vue-template-compiler": "^2.6.10",
    "webpack": "^4.35.3",
    "webpack-dev-server": "^3.7.2"
}
```

- #### 第一个命令：example-cli create xxx
> 作用：通过这个命令可以新建一个vue的开发项目

1. 新建bin目录，并新建index.js文件，内容如下：
> commader的具体用法，我这里不介绍，直接去github上看文档即可
```
#! /usr/bin/env node

const program = require('commander');

program
    .command('create <projectName>')
    .action((projectName) => {
        require('../lib/command/create.js')(projectName);
    });

program.parse(process.argv);
```
2. 新建lib目录，并在lib下新建command目录，并新建create.js文件，内容如下：
> 默认后续的操作都在lib目录下

> 具体的代码逻辑还未填充，只是写好了要做的流程
```
module.exports = function(projectName){
    //以<projectName>为名称创建目录，并将template目录下的src全部复制到<projectName>目录下

    //在<projectName>目录下，新建package.json文件，并在此文件中添加基本的信息以及项目所需要的依赖

    //执行npmm install，安装项目的依赖
}
```
3. 新建template目录，并在此目录下新建如下的目录结构：
```
template
    src
        assets
        components
        pages
        App.vue
        main.js
```
> 然后在App.vue中新增如下内容：
```
<template>
	<div>
		<h1>hello world!</h1>
		<h1>欢迎使用example-cli!</h1>
	</div>
</template>

<script>
export default {
	name: 'App'
}
</script>

<style>
h1 {
	text-align: center;
}
</style>
```

> 再在main.js文件添加如下内容：
```
import Vue from 'vue';
import App from './App';

new Vue({
    render: (h) => h(App)
}).$mount('#app');
```

4. 新建utils目录，并新建createDevProject.js文件，内容如下:
```
const fs = require('fs-extra'),
    path = require('path');

module.exports = function (projectName) {
    let targetDir = path.resolve(process.cwd(), projectName),  //获取当前命令执行的目录
        templateDir = path.resolve(__dirname, '../template/src');

    //先判断<projectName>目录是否已存在
    if (fs.existsSync(targetDir)) {
        console.log(`<${projectName}> 目录已存在，请输入一个新的项目名称！`);
    } else {
        try {
            //新建<projectName>目录
            fs.mkdirSync(targetDir);
            //将template下的src整个复制到指定的目录下
            fs.copySync(templateDir, path.resolve(targetDir, 'src'));
        } catch (err) {
            console.log(err);
        }
    }
}
```
5. 新建config目录，并在此目录下新建packageJson.js文件，新增如下内容：
> 此文件将被生成package.json，并且是属于用户创建的项目。
```
module.exports = {
    name: '',
    version: '0.0.1',
    description: '',
    scripts: {
        "build": "example-cli build",
        "serve": "example-cli serve"
    },
    dependencies: {
        "css-loader": "^3.0.0",
        "vue": "^2.6.10",
        "vue-loader": "^15.7.0",
        "vue-template-compiler": "^2.6.10",
        "webpack": "^4.35.3"
    }
};
```
6. 在utils下新建createPackage.js文件，内容如下：
```
const fs = require('fs-extra'),
    path = require('path');

module.exports = function (projectName) {
    let packageJsonConfig = require('../config/packageJson'),
        targetDir = path.resolve(process.cwd(), `${projectName}/package.json`);
    //修改name名称
    packageJsonConfig.name = projectName;
    
    try {
        fs.writeFileSync(targetDir, JSON.stringify(packageJsonConfig));
    } catch (err) {
        console.log(err);
    }
}
```
7. 在command目录下新建installDependencies.js文件，内容如下：
```
const execa = require('execa'),
    path = require('path');

module.exports = function (projectName) {
    /**
     * 当前的命令的目录实在<projectName>的上一级目录，
     * 而我们的npm install的执行应该在<projectName>目录下，
     * 所以cwd需要调整
     */
    const childProcess = execa.command('npm install', {
        cwd: path.resolve(process.cwd(), projectName)
    });

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
}
```
8. 最终lib/command/create.js文件的内容更新如下：
```
const createDevProject = require('../utils/createDevProject'),
    createPackageJson = require('../utils/createPackageJson'),
    installDependencies = require('../command/installDependencies');


module.exports = function(projectName){
    //以<projectName>为名称创建目录，并将template目录下的src全部复制到<projectName>目录下
    createDevProject(projectName);
    //在<projectName>目录下，新建package.json文件，并在此文件中添加基本的信息以及项目所需要的依赖
    createPackageJson(projectName);
    //执行npmm install，安装项目的依赖
    installDependencies(projectName);
}
```

9. 最后，注册example-cli命令
> 在example-cli目录下的package.json中新增如下内容：
```
{
    "bin": {
        "example-cli": "./bin/index.js"
    }
}
```
> 再在example-cli目录下，输入如下命令进行全局挂载，这样你可以在任何地方使用example-cli这个命令
```
npm link
```
> 好了，现在你就可以使用example-cli create demo，创建一个vue项目的基础环境了。

- #### 第二个命令：npm run serve
1. 注册npm run serve脚本命令
> 我们已经在“第一个命令：example-cli create xxx”章节下的第5步中的packageJson.js文件中添加了如下内容：
```
"scripts": {
    "serve": "example-cli serve"
}
```
> 从如上的配置中可以看到，当我们执行npm run serve的时候，实际执行的命令是example-cli serve，所以我们还需要对应的命令行。

2. 在bin/index.js文件中，新增如下内容：
```
program
    .command('serve')
    .action(() => {
        require('../lib/command/serve.js')();
    });
```
3. 在command目录下，新建serve.js文件，内容如下：
```
const execa = require('execa'),
    path = require('path');

module.exports = function () {
    let execaDir = path.resolve(__dirname, '../scripts/serve.js');
    const childProcess = execa('node', [execaDir]);

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
}
```
4. 在config目录下，新建server.js文件，内容如下：
> 此文件就是webpack需要的配置文件
```
const path = require('path'),
    VueLoaderPlugin = require('vue-loader/lib/plugin'),
    HtmlWebpackPlugin = require('html-webpack-plugin'),
    MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    mode: 'development',
    devtool: 'cheap-module-eval-source-map',
    context: process.cwd(),
    //需要resolve一下，不然会报错
    entry: path.resolve('src/main.js'),
    output: {
        path: path.resolve('public'),
        filename: 'main.js'
    },
    devServer: {
        overlay: true,
        stats: "none",
        writeToDisk: true,
        contentBase: path.resolve(process.cwd(), 'public')
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
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            /**
             * 我们可以把常用的css预编译器都先在这里声明，
             * 这样做的话，当用户使用了其他的预编译时，
             * 只需要安装就行，不需要再配置rules。
             * 可以大大的方便用户。
             */
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
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
        }),
        new HtmlWebpackPlugin({
            //这里由于命令的目录和模板的目录不在一起，所以需要采用绝对路径
            template: path.resolve(__dirname, '../template/index.html')
        })
    ],
    resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']  //这样配置后，再项目中引入某文件时，可以省略文件后缀
    }
};
```
5. 在template目录下，新建index.html文件，内容如下：
> 这就是html-webpack-plugin需要的模板文件
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>example-cli</title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```
6. 新建scripts目录，并在此目录下新建serve.js文件，内容如下：
```
const webpack = require('webpack'),
    WebpackDevServer = require('webpack-dev-server');

let serverConfig = require('../config/server');
const compiler = webpack(serverConfig);

new WebpackDevServer(compiler, serverConfig.devServer).listen('8080', '127.0.0.1', () => {
    console.log('Listening at http://127.0.0.1:8080');
});
```
7. 完毕
> 现在你可以再<projectName>目录下，输入npm run serve命令来启动项目啦。

- #### 第三个命令：npm run build
1. 注册npm run build脚本命令
> 我们已经在“第一个命令：example-cli create xxx”章节下的第5步中的packageJson.js文件中添加了如下内容：
```
"scripts": {
    "build": "example-cli build"
}
```
> 从如上的配置中可以看到，当我们执行npm run build的时候，实际执行的命令是example-cli build，所以我们还需要对应的命令行。

2. 再bin/index.js文件中，新增如下内容：
```
program
    .command('build')
    .action(()=>{
        require('../lib/command/build.js')();
    });
```

3. 在command目录下，新建build.js文件，内容如下：
```
const execa = require('execa'),
    path = require('path');

module.exports = function () {
    let execaDir = path.resolve(__dirname, '../scripts/build.js');
    const childProcess = execa('node', [execaDir]);

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
}
```

4. 在config目录下，新建build.js文件，内容如下：
```
const path = require('path'),
    VueLoaderPlugin = require('vue-loader/lib/plugin'),
    HtmlWebpackPlugin = require('html-webpack-plugin'),
    MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    mode: 'production',
    context: process.cwd(),
    //需要resolve一下，不然会报错
    entry: path.resolve('src/main.js'),
    output: {
        path: path.resolve('dist'),
        filename: 'js/main.js'
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
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            /**
             * 我们可以把常用的css预编译器都先在这里声明，
             * 这样做的话，当用户使用了其他的预编译时，
             * 只需要安装就行，不需要再配置rules。
             * 可以大大的方便用户。
             */
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
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
        }),
        new HtmlWebpackPlugin({
            //这里由于命令的目录和模板的目录不在一起，所以需要采用绝对路径
            template: path.resolve(__dirname, '../template/index.html')
        })
    ],
    resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']  //这样配置后，再项目中引入某文件时，可以省略文件后缀
    }
};
```

5. 在scripts目录下，新建build.js文件，内容如下：
```
const webpack = require('webpack');

let buildConfig = require('../config/build');
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

6. 完毕

> 现在你可以在<projectName>目录下，运行命令npm run build，进行打包啦！