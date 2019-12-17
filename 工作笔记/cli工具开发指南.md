### cli工具开发指南

CLI，全称是command-line interface，也就是命令行交互接口。在项目开发时可以快速生成自己需要的代码模板。提高团队的开发速度。


#### 1、启动命令行执行
首先，如何在命令行中启动脚本呢？
在这里，推荐使用package.json bin字段，来指定各个内部命令对应的可执行文件的位置。
```
	"bin": {
        "auto": "./bin/index.js"
    }
```
上面代码指定，auto命令对应可执行文件为bin子目录下的index.js。npm会寻找该文件，并且在node_modules/.bin建立符号链接，因此在运行npm时，就可以不携带路径直接运行auto调用index.js脚本。

#### 2、命令行工具解析
解决了启动命令行脚本，那么我们是如何读取到用户输入信息呢？
[**commander.js**](https://github.com/tj/commander.js#custom-option-processing)是目前很成熟的Node命令行交互接口实现工具，通过program.parse对命令行进行解析。

```
    const program = require("commander");
    program.parse(process.argv);
```
process是node中的一个模块，通过访问process.argv我们能轻松愉快的接收通过命令执行node程序时候所传入的参数。

#### 3、命令行与用户交互
能够解析用户输入后，我们或许需要去询问操作者问题，获取用户输入并检测用户回答是否合法。


* 输入 Input
```
	inquirer.prompt([
        {
            type: "input",
            message: "please input your name",
            name: "label",
            validate: function(input) {}
        },
    ]);
```

* 选择  List
```
	inquirer.prompt([
        {
            type: "list",
            message: "please select types ",
            choices: ["group", "detail"],
            name: "types"
        },
    ]);
```

更多请参考：[inquirer](https://www.npmjs.com/package/inquirer)



基于以上三点，我们就可以启动命令行，与操作者交互，并读取到输入数据，进行下一步的操作啦...




