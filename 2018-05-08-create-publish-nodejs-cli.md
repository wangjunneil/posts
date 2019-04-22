---
title: 从零开始创建并发布NPM Cli的应用
name: create-publish-nodejs-cli
date: 2018-05-08
tags: [npm,cli,nodejs,进度,气温,雅虎]
categories: [Web, Javascript]
---

* 目录
{:toc}

## 1. 背景

当下 javascript 大行其道，无论从传统web、手机端还是桌面程序，javacsript 无所不能。ES6 的定义 以及 nodejs 大大推动了 javascript 的发展。而大部分的应用都离不开 **npm**，很多应用都使用 nodejs cli 的方式进行初始化等操作。

本篇文章将创建一个简单的 **nodejs cli** 的气温应用，以便熟悉如何自己创建 **cli** 命令行程序。

## 2. 前提

+ 安装好 nodejs 环境
+ 具有 javascript 编程经验，最好熟悉 ES5 或者 ES6 的规范
+ 好的编辑器，比如 visual code 后者 sublime

## 3. Hello World

### 3. 1 创建工程基本结构

和很多 javascript 工程一样，需要一个 `package.json` 文件以及一个入口的 `index.js` 文件，这里强调简单的应用，所以尽量不做过多依赖库。

**[package.json]**

```json
{
  "name": "weather-cli",
  "version": "1.0.0",
  "license": "MIT",
  "scripts": {},
  "devDependencies": {},
  "dependencies": {}
}
```

**[index.js]**

```js
module.exports = () => {
  console.log('Welcome to the weather app!')
}
```



### 3.2 创建一个Bin文件



我们需要一种方法来调用新开发的 javascript 程序并显示欢迎信息，并将其加入到系统路径中，以便在任何地方都可以调用它，bin 文件是实现此目的的好方法。



**[bin/weather]**

```shell
#!/usr/bin/env node
require('../')()
```



使用 bin 文件运行程序，使用命令 `cd bin` 进入bin目录，并给 **weather** 文件赋予执行权限 `chmod u+x ./weather`，直接执行 `./weather` 就可以显示欢迎信息。



但这不是我们想要的，我们需要在任何路径下都可以执行，也就是使用 npm 全局安装的方式 `npm install -g weather`，所以需要将 bin 文件配置到 package.json 中。



**[package.json]**

```json
{
  "name": "weather-cli",
  "version": "1.0.0",
  "license": "MIT",
  "bin": {
    "weather": "bin/weather"
  },
  "scripts": {},
  "devDependencies": {},
  "dependencies": {}
}
```



增加好之后，在工程根路径下执行 `npm link` 命令会将二进制文件符号链接到系统路径，使其可以在任何地方通过在外部运行访问。

**此时你可以在任何路径下都可以执行 `weaher` 命令了。**



到此为止的工程结构为：

```
weather-cli/
	|- package.json
	|- index.js
	|- bin
		|- weather
```



## 4. 气温应用



### 4.1 命令行参数



如何处理接受的命令行参数，这里使用 **[minimist](https://github.com/substack/minimist)** 的解析库，在根路径下使用 `npm install --save minimist` 进行安装



**[index.js]**

```js
const minimist = require('minimist')

module.exports = () => {
    // slice(2) 删除前两个参数，因为第一个参数是解释器，第二个参数是文件名
    const args = minimist(process.argv.slice(2));
    console.log(args);
}
```



现在运行 `weather today --location "NanJing, ZH"`，它将会输出 `{ _: [ 'today' ], location: 'NanJing, ZH' } `，从输出结果可以得知，这是一个清晰的 json 结构，到此我们建立了第一个解析命令行参数的 cli 程序。



### 4.2 运行命令



将每个命令代码分隔到不同文件中，需要的时候进行加载。这会防止加载不必要的模块并减少启动程序的时间，在这种设置下，每个命令文件都应该导出一个函数，并将参数传递给它。



**[index.js]**

```js
const minimist = require('minimist')

module.exports = () => {
    // slice(2) 删除前两个参数，因为第一个参数是解释器，第二个参数是文件名
    const args = minimist(process.argv.slice(2));
    // args = { _: [ 'today' ], location: 'NanJing, ZH' }
    // 取第一个命令
    const cmd = args._[0];

    switch (cmd) {
        case 'today':
            // 需要的时候加载运行并把参数进行传递
            require('./cmds/today')(args);
            break;
        default:
            console.error(`"${cmd}" is not a valid command!`)
            break;
    }
}
```



**[cmds/today.js]**

```js
module.exports = (args) => {
    console.log('today is sunny')
}
```



现在如果运行 `weather today` 则会输出 "today is sunny"，若运行 `weather hello` 则会输出错误的信息 "hello is not a valid command!"，



### 4.3 预期与容错



除了程序用命令，还需要额外处理如 `version` 和 `help` 此类的命令，当用户输入错误的参数应该会得到正确的反馈信息，以提示用户正确的进行参数的输入。



**[index.js]**

```js
const minimist = require('minimist')

module.exports = () => {
    const args = minimist(process.argv.slice(2));
    // 取第一个参数命令，若没有获取到则默认使用 help 命令
    let cmd = args._[0] || 'help';

    if (args.version || args.v) {
        cmd = 'version'
    }

    if (args.help || args.h) {
        cmd = 'help'
    }

    switch (cmd) {
        case 'today':
            require('./cmds/today')(args);
            break;
        case 'version':
            require('./cmds/version')(args);
            break;
        case 'help':
            require('./cmds/help')(args);
            break;
        default:
            console.error(`"${cmd}" is not a valid command!`)
            break;
    }
}
```



**[cmds/version]**

```js
const { version } = require('../package.json')

module.exports = (args) => {
  console.log(`v${version}`)
}
```



**[cmds/help]**

```js
const menus = {
    main : `
        weather [command] <options>

        today   ........... show weather for today
        version ........... show program version
        help    ........... show help menu for a command
    `,

    today : `
        weather today <options>

        --location, -l .... the location to use
    `,    
}

module.exports = (args) => {
    const subCmd = args._[0] === 'help' ? args._[1] : args._[0];

    console.log(menus[subCmd] || menus.main)
}
```



此时如果运行 `weather help today` 或者 `weather today -h`，将会看见 **today** 命令的帮助信息。运行 `weather` 或者 `weather -h` 将会输出主菜单帮助信息。



### 4.4 进度与请求



某些命令可能会执行很长时间，为了让用户得到正在运行的反馈状态，应该在执行时显示 **进度**，使用命令 `npm install --save axios ora` 安装第三方的进度与网络请求的库。



+ 进度 ora 的更多用法请参看 [https://www.npmjs.com/package/ora](https://www.npmjs.com/package/ora)
+ 请求 axios 的更多用法请参看 [https://www.npmjs.com/package/axios](https://www.npmjs.com/package/axios)



现在我们开始请求天气，使用的是 **雅虎** 的 API，此 API 使用的 **"YQL"** 的语法查询，关键的是它不需要 **API Key** 就可以直接进行接口的调用，非常方便。



**[utils/yahoo.js]**

```js
const axios = require('axios')

module.exports = async (location) => {
    const results = await axios({
        method : 'get',
        url : 'https://query.yahooapis.com/v1/public/yql',
        params : {
            format : 'json',
            q: `select item from weather.forecast where woeid in
            (select woeid from geo.places(1) where text="${location}")`,
        }
    });

    return results.data.query.results.channel.item
}
```



**[cmds/today.js]**

```js
const ora = require('ora')
const getWeather = require('../utils/yahoo')

module.exports = async (args) => {
    // 开启进度
    const spinner = ora().start()

    try {
        const location = args.location || args.l
        const weather = await getWeather(location)

        // 关闭进度
        spinner.stop()

        console.log(`Current conditions in ${location}:`)
        console.log(`\t${weather.condition.temp}° ${weather.condition.text}`)
    } catch (err) {
        // 关闭进度
        spinner.stop()

        console.error(err)
    }
}
```



这里在 **yahoo.js** 中封装了调用温度的接口，在命令文件 **today.js** 中进行调用，加载进度并解析返回值，当输入 `weather today --location "BeiJing, ZH" ` 时，将会输出指定城市的气温，注意这里只是个示例，只取了返回值的 temp 字段。



### 4.5 错误与退出码



至此我们的程序结束了吗？没有。如果你的 cli 程序有严重错误，应该使用 `process.exit(1)` 的方式进行退出，这可以告诉终端此进程异常终止。



**[utils/error.js]**

```js
module.exports = (message, exit) => {
    console.error(message)
    exit && process.exit(1)
}
```



**[index.js]**

```js
// ...
const error = require('./utils/error')

module.exports = () => {
  // ...
  default:
    // console.error(`"${cmd}" is not a valid command!`)
    error(`"${cmd}" is not a valid command!`, true)
    break
    // ...
}
```



## 5. 发布到NPM仓库



为了让更多人可以使用到你编写的 nodejs 库，应该上传到 npm 仓库中，当然这只是个例子，为了能够上传，需要完善 **package.json** 文件



```json
{
  "name": "weather-cli",
  "version": "1.0.0",
  "description": "A cli app that gives you the weather",
  "license": "MIT",
  "homepage": "https://github.com/wangjunneil/weather-cli",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/wangjunneil/weather-cli"
  },
  "engines": {
    "node": ">=8"
  },
  "keywords": [
    "weather",
    "vinny",
    "javascript"
  ],
  "preferGlobal": true,
  "bin": {
    "weather": "bin/weather"
  },
  "scripts": {},
  "devDependencies": {},
  "dependencies": {
    "axios": "^0.18.0",
    "minimist": "^1.2.0",
    "ora": "^2.1.0"
  }
}
```



+ engine 设置告诉安装人员使用 node 的最新版本，由于使用了 async 和 await 语法，因此需要8.0或更高的版本。
+ preferGlobal 的设置是若用户使用 `npm install --save` 而不是 `npm install --gobal` 的方式进行安装，将会发出警告。



首次上传的话你需要有 npm 的账号信息，若没有请去 [https://www.npmjs.com/](https://www.npmjs.com/) 进行注册，注册完成后，先使用 `npm adduser` 将账号信息在电脑中进行验证，最后在根路径中执行 `npm publish` 命令进行发布即可。



>  本文中所有的示例代码在 [https://github.com/wangjunneil/weather-cli](https://github.com/wangjunneil/weather-cli) 可以获取到
