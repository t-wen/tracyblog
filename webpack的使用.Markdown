# Webpack的使用

## 需要的信息

* [Webpack官网](https://webpack.js.org/guides/getting-started/)

## 开始编码
### 基本配置

* 首先让我们创建一个目录，初始化npm，在本地安装webpack，然后安装webpack-cli（用于在命令行上运行webpack的工具）：


> mkdir webpack-demo && cd webpack-demo
> npm init -y
> npm install webpack webpack-cli --save-dev


* 现在我们将创建以下目录结构，文件及其内容：

项目

``` txt
webpack-demo
  |- package.json
+ |- index.html
+ |- /src
+   |- index.js
```

所以，接下来创建src文件夹

> mkdir src

SRC / index.js

``` js
function component() {
  let element = document.createElement('div');

  // Lodash, currently included via a script, is required for this line to work
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());

```

index.html

``` html
<!doctype html>
<html>
  <head>
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

* 我们还需要调整我们的package.json文件，以确保我们将包标记为private，以及删除main条目。这是为了防止意外发布您的代码。

package.json

``` json
{
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
    "private": true,
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.0.1",
      "webpack-cli": "^2.0.9"
    },
    "dependencies": {}
  }

```

* 在此示例中，<script>标记之间存在隐式依赖关系。我们的index.js文件取决于在lodash运行之前包含在页面中。这是因为index.js从未明确声明需要lodash; 它只是假设全局变量_存在。

* 以这种方式管理JavaScript项目存在问题：

目前并不明显该脚本依赖于外部库。
如果缺少依赖项或包含在错误的顺序中，则应用程序将无法正常运行。
如果包含但未使用依赖项，则将强制浏览器下载不必要的代码。

* 让我们使用webpack来管理这些脚本。

### 创建一个包

* 首先，我们将略微调整我们的目录结构，将“源”代码（/src）与我们的“分发”代码（/dist）分开。“源”代码是我们编写和编辑的代码。“分发”代码是output我们的构建过程的最小化和优化，最终将在浏览器中加载：

项目

```txt
webpack-demo
  |- package.json
+ |- /dist
+   |- index.html
- |- index.html
  |- /src
    |- index.js
```

* 因此，这边我们还需要创建一个文件夹dist：

> mkdir dist

* 要捆绑lodash依赖项index.js，我们需要在本地安装库：

>npm install --save lodash

* 现在，将lodash导入脚本中：

SRC / index.js

``` js
import _ from 'lodash';

function component() {
  let element = document.createElement('div');

  
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

* 现在，由于我们将捆绑我们的脚本，我们必须更新我们的index.html文件。让我们<script>像现在一样删除lodash ，import并修改另一个<script>标签来加载包，而不是原始/src文件：

DIST / index.html

``` html
<!doctype html>
<html>
  <head>
    <title>Getting Started</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

* 在此设置中，index.js明确要求lodash存在，并将其绑定为_（无全局范围污染）。通过声明模块需要什么依赖项，webpack可以使用此信息来构建依赖关系图。然后，它使用该图生成一个优化的包，其中脚本将以正确的顺序执行。

* 因此，让我们运行npx webpack，它将我们的脚本src/index.js作为[入口点](https://webpack.js.org/concepts/entry-points/)，并将生成dist/main.js作为[输出](https://webpack.js.org/concepts/output/)。该npx命令随Node 8.2 / npm 5.2.0或更高版本一起运行，它运行./node_modules/.bin/webpack我们在开头安装的webpack包的webpack binary（）：

> npx webpack
```txt

Hash: dabab1bac2b940c1462b
Version: webpack 4.12.0
Time: 287ms
Built at: 13/06/2018 11:52:07
  Asset      Size  Chunks             Chunk Names
main.js  70.4 KiB       0  [emitted]  main
[1] (webpack)/buildin/module.js 497 bytes {0} [built]
[2] (webpack)/buildin/global.js 489 bytes {0} [built]
[3] ./src/index.js 216 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```

* index.html在浏览器中打开，如果一切正常，您应该看到以下文本：'Hello webpack'。

### 模块

在import和export报表已标准化，ES2015和在大多数浏览器都支持。一些旧的浏览器仍然落后，但webpack支持开箱即用的模块。

在幕后，webpack实际上“转换”了代码，以便旧版浏览器也可以运行它。如果您检查一下dist/main.js，您可能会看到webpack如何做到这一点，它非常巧妙！除了import和export，的WebPack支持各种其他模块语法以及，见模块API的详细信息。

注意的WebPack不会改变比其他任何代码import和export语句。如果你使用的是其他ES2015功能，确保使用transpiler如Babel或 Bublé 通过的WebPack的装载系统。

### 使用配置

从版本4开始，webpack不需要任何配置，但大多数项目需要更复杂的设置，这就是webpack支持配置文件的原因。这比在终端中手动输入大量命令要高效得多，所以让我们创建一个替换上面使用的CLI行选项：

项目

``` txt
webpack-demo
  |- package.json
+ |- webpack.config.js
  |- /dist
    |- index.html
  |- /src
    |- index.js
```

webpack.config.js

``` js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

* 现在，让我们再次运行构建，而不是使用我们的新配置文件：

> npx webpack --config webpack.config.js
```txt
Hash: dabab1bac2b940c1462b
Version: webpack 4.12.0
Time: 283ms
Built at: 13/06/2018 11:53:51
  Asset      Size  Chunks             Chunk Names
main.js  70.4 KiB       0  [emitted]  main
[1] (webpack)/buildin/module.js 497 bytes {0} [built]
[2] (webpack)/buildin/global.js 489 bytes {0} [built]
[3] ./src/index.js 216 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/

```

* 配置文件比简单的CLI使用具有更大的灵活性。我们可以通过这种方式指定加载器规则，插件，解析选项和许多其他增强功能.

### NPM脚本

* 鉴于从CLI运行webpack的本地副本并不是特别有趣，我们可以设置一个小捷径。让我们通过添加一个npm脚本来调整我们的package.json：

package.json

``` json
{
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
     "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.0.1",
      "webpack-cli": "^2.0.9"
    },
    "dependencies": {
      "lodash": "^4.17.5"
    }
  }
```

* 现在npm run build可以使用该npx命令代替我们之前使用的命令。请注意，scripts我们可以按照与之相同的方式引用本地安装的npm软件包npx。这个约定是大多数基于npm的项目的标准，因为它允许所有贡献者使用相同的一组公共脚本（每个脚本都有--config必要的标记）。

* 现在运行以下命令，看看你的脚本别名是否有效：

> npm run build

``` txt
Hash: dabab1bac2b940c1462b
Version: webpack 4.12.0
Time: 278ms
Built at: 13/06/2018 11:54:54
  Asset      Size  Chunks             Chunk Names
main.js  70.4 KiB       0  [emitted]  main
[1] (webpack)/buildin/module.js 497 bytes {0} [built]
[2] (webpack)/buildin/global.js 489 bytes {0} [built]
[3] ./src/index.js 216 bytes {0} [built]
    + 1 hidden module

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/.
```
## 总结

* 既然您已经有了基本的构建，那么您应该继续Asset Management学习下一个指南，以了解如何使用webpack管理图像和字体等资产。此时，您的项目应如下所示：

项目

```txt
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
  |- main.js
  |- index.html
|- /src
  |- index.js
|- /node_modules
```

* 总的来说一个流程就是：当我们运行npx webpack,脚本src/index.js是作为一个入口点，进入webpack.config.js处理后将生成的dist/main.js 输出。

## 补充：innerHTML解析
* innerHTML，[根据W3C](http://www.w3school.com.cn/htmldom/prop_tablerow_innerhtml.asp)

* innerHTML: innerHTML 属性设置或返回表格行的开始和结束标签之间的 HTML。

* 案例解析：

下面的例子返回了表格行的 inner HTML：

``` html
<html>
<head>
<script type="text/javascript">
function getInnerHTML()
  {
  alert(document.getElementById("tr1").innerHTML);
  }
</script>
</head>
<body>

<table border="1">
<tr id="tr1">
<th>Firstname</th>
<th>Lastname</th>
</tr>
<tr id="tr2">
<td>Peter</td>
<td>Griffin</td>
</tr>
</table>
<br />
<input type="button" onclick="getInnerHTML()" 
value="Alert innerHTML of table row" />

</body>
</html>
```

alert(document.getElementById("tr1").innerHTML);这一句的意思就是读取HTML内的id="tr1”内的内容。所以结果表格内会显示Firstname和Lastname相关参数。

 

