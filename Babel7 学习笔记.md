作为一名合格的前端工程师，免不了要和 Babel、Webpack 这一类的基础工具打交道。但很多时候，这些工具就像一个黑盒，我们使用脚手架创建项目之后，只要一条命令就成功跑起来了，至于它们是怎么配合的，流程是咋样的，各自负责的模块是啥，并不是那么清楚。既然不明白，那就学起来💪，先从简单一点的 Babel 开始。废话少说，开冲👻！！！

> 注：本文所用的 Babel 版本为 v7.21.0，版本跨度过大可能会导致配置项有变化，如有变动请参考官方文档。

# 初始化 Babel 项目

首先，初始化一个简单用于练习 Babel 的项目：

```shell
mkdir babel-demo && cd babel-demo
pnpm init
mkdir src dist
cd src && touch origin.js
cd ../dist && touch compiled.js

# 安装 Babel 基础包并创建 Babel 配置文件
pnpm @babel/core @babel/cli
touch babel.config.js
```

> 注：`@babel/core` 是 Babel 的核心，只要你使用 Babel 就必须安装它。而 `@babel/cli` 则是一个可以让我们通过命令行来执行各种 Babel 命令的工具包，它的作用和 `@babel-loader` 类似，只不过 `@babel-loader` 是供 Webpack 使用的。本文不涉及 `@babel-loader`，想要了解可参考 [Webpack 文档](https://webpack.js.org/loaders/babel-loader/)。

此时我们的目录结构是这样的：

```shell
.
├── babel.config.js
├── dist
│   └── compiled.js
├── package.json
├── pnpm-lock.yaml
└── src
    └── origin.js
```

之后我们将在 `src/origin.js` 中编写 ES6+ 代码，然后通过 Babel 将代码编译成 ES5 并输出在 `dist/compiled.js` 中。为了方便，可以在 `package.json` 中添加一条对应的 Babel 命令，之后就通过执行 `pnpm run babel` 完成代码编译。

```json
"scripts": {
  "babel": "babel src/origin.js -o dist/compiled.js"
}
```

# Babel 配置

Babel 的配置可以分为两种类型：

- 项目范围的配置：`babel.config.*` 文件，可以用 `.json`、`.js`、`.cjs`、`.mjs` 作为扩展名。
- 相对文件的配置（可以理解为局部范围的配置）：
   - `.babelrc.*` 文件，同样可以用 `.json`、`.js`、`.cjs`、`.mjs` 作为扩展名。
   - 不带扩展名的 `.babelrc` 文件。
   - `package.json` 文件中增加 `babel` 属性。
> 注：`json`、`cjs` 是 v7.7.0 之后才支持的，而 `mjs` 则是 v7.8.0 之后支持的。论适用性广，还是 `js` 扩展名更好一些。

在日常的项目中，两者最重要的区别在于：**如果你需要 Babel 应用于 `node_modules`，那么应该采用 `babel.config.*`，而不是 `.babelrc.*`**。Babel 官方建议项目中使用 JSON 配置文件，理由是：虽然 JS 配置文件在需要使用一些表达式或者构建时计算很方便，但 JS 配置的静态可分析性较差， 因此对可缓存性、代码检测、IDE 自动完成等有负面影响。 而 JSON 配置文件是静态 JSON 文件，它允许其他使用 Babel 的工具，如 Webpack 等打包工具安全地缓存 Babel 的结果， 这可能会带来巨大的构建性能优势。所以，如果你的项目中使用了新版本的 Babel，建议使用 `babel.config.json` 作为配置文件。如果是 monorepo 项目或者有特殊的需求，可以参考 [官网示例](https://babeljs.io/docs/config-files) 了解更加细致的配置。  
虽然 Babel 配置有多种写法，但是写起来都是差不多的。我们只需要重点关注其中两个参数即可：**`presets` 数组、`plugins` 数组**。

```javascript
// json写法
{
  "presets": [],
  "plugins": []
}

// js写法
module.exports = {
  presets: [],
  plugins: []
}

// 内嵌在package.json文件
{
  "name": "babel-demo",
  "version": "1.0.0",
  },
  "babel": {
    "presets": [],
    "plugins": []
  }
}
```

`presets`、`plugins` 的用法和区别是什么，我们接下来通过实践来说明。

# Babel 的插件和预设

## 插件（plugin）

初始化完成项目之后，我们就可以进行实战了。首先在 `origin.js` 文件中写入几行 ES6+ 代码：

```javascript
// origin.js
const a = 1;
const b = () => {}
const c = a ?? 0;
```

现在我们执行 `pnpm run babel` 编译一下，看 `compiled.js` 中是否出现了我们期待的 ES5 代码。

```javascript
// compiled.js
const a = 1;
const b = () => {};
const c = a ?? 0;
```

🤔为啥还是原来的代码？回想一下，我们只加了 Babel 配置文件，但是并没有写入具体的配置项，Babel 也就不知道怎么编译代码，只能原样输出了。所以需要明确告诉 Babel 我们要编译代码中的什么新特性才可以，这个时候就需要插件（plugin）的帮助了。`origin.js` 中，我们用了 `const`、箭头函数和空值合并运算符，所以需要安装并在 `babel.config.js` 中添加对应的插件。

```javascript
// babel.config.js
module.exports = {
  presets: [],
  plugins: [
    '@babel/plugin-transform-block-scoping',
    '@babel/plugin-transform-arrow-functions',
    '@babel/plugin-proposal-nullish-coalescing-operator',
  ],
};
```

OK，此时我们再编译一次，可以发现，现在是我们想要的结果了：

```javascript
// compiled.js
var a = 1;
var b = function () {};
var c = a !== null && a !== void 0 ? a : 0;
```

到这里，我们大概能总结出一个套路：如果你想要 Babel 编译一个代码中使用的新特性，只需要添加对应的插件（plugin）即可。很简单是吧，但随着而来的就是一个烦人的问题：我们日常的开发中已经全面拥抱 ES6+ 了，代码里到处都有新特性的使用，一个项目可能要配置几十甚至上百个插件才能保证 Babel 顺利编译。光是正确找出这些插件的名字可能就要花几个小时🤯，这换谁来都要创业未半而中道崩殂。。。  
那有没有类似插件包的东西呢？比如每年的新特性都整合成一个包，这样我们只需要添加几个插件集合就可以了。确实有这样的插件包，这就是我们的**预设（preset）。**

## 预设（preset）

预设的作用和好处上面应该已经解释清楚了。以 React + TypeScript 项目为例，我们最为常用的预设只有三个：`@babel/preset-env`、`@babel/preset-react`、`@babel/preset-typescript`，这可比一堆插件看起来爽多了，大家的头发也可以少掉一点了🤣。其中最为重量级的当属 `@babel/preset-env`。`@babel/preset-env` 是一个智能的预设，它会随着迭代不断增加新特性进去。也就意味着我们可以长期使用这个预设，不必太担心它过时、不够全面的问题。  
当前的 Babel 练习项目中只有 js 代码，所以我们用 `@babel/preset-env` 就可以了。接下来我们来改造一下之前的配置：

```javascript
// babel.config.js
module.exports = {
  presets: [['@babel/preset-env']],
  plugins: [],
};
```

然后看一下 `compiled.js` 中的编译结果，和之前的是一致的，符合预期。

```javascript
// compiled.js
"use strict";

var a = 1;
var b = function b() {};
var c = a !== null && a !== void 0 ? a : 0;
```

# polyfill

趁热打铁，我们再来加一些新的特性试一试，看看这位重量级选手到底几斤几两😏。

```javascript
// origin.js
const obj1 = {
  name: 'tim',
  age: 18,
};
let d = obj1?.name;
let e = obj1?.gender;
const f = [1, 2, 3, 4].includes(5);
```

Babel 编译结果：

```javascript
// compiled.js
"use strict";

var obj1 = {
  name: 'tim',
  age: 18
};
var d = obj1 === null || obj1 === void 0 ? void 0 : obj1.name;
var e = obj1 === null || obj1 === void 0 ? void 0 : obj1.gender;
var f = [1, 2, 3, 4].includes(5);
```

可选链确实编译成功了，但是 `Array.prototype.includes` 为啥没有变化呢😮？按照我们之前的解释，这个新特性 `@babel/preset-env` 应该是可以处理的呀。怎么肥事🤨？  
这里就需要澄清一下了：TC39 每年都会在 ES 标准中加入一些新的东西，一般我们都笼统的称为**新特性。但其实可以分成两大类，一种是语法（syntax），而一种是 API**。

- 语法包括：`const/let`、`class`、模版字符串、可选链、`async/await` 等。
- API 则包括对象的实例/静态方法，比如 `Array.from`、`Array.prototype.includes`，还有内置对象如 `Promise` 等。

有了语法和 API 这两个新的概念之后，我们需要更正一下之前的理解：**`@babel/preset-env` 可以将 ES6+ 的新语法成功编译成 ES5，但是对于新的 API，它无能为力。要编译新的 API，我们就需要另一位重量级选手 polyfill 了。polyfill，可以翻译为 " 填充物、垫片 "，对于前端来讲就是将 ES 高版本的 API 用 ES5 的 API 实现，这样就可以抹平不同环境之间的差异**，我们就可以愉快的使用新的 API 啦！  
polyfill 并不是一个包的名字，而是一类方法的统称，你可以自己用 ES5 实现一个 `Array.prototype.includes`，这个方法也是一个 polyfill。目前 JavaScript 最好用、也最流行的 polyfill 是 `core-js`。在 Babel v7.4.0 之前，Babel 实现 polyfill 的方式是通过 `@babel/polyfill` 这个包，但是在 v7.4.0 之后，这个包就被废弃了，Babel 更加推荐直接使用 `core-js`+`regenerator-runtime`（实际上废弃的 `@babel/polyfill` 就是 `core-js` 和 `regenerator-runtime` 的一个整合）。

> 注：v7.18.0 之后也不再需要 `regenerator-runtime`，具体原因下面会讲到。

OK，有了上面的铺垫之后，我们就可以着手解决我们项目中 `Array.prototype.includes` 无法编译的问题了。首先安装需要的 `core-js`，然后修改一下 Babel 配置并在 `origin.js` 文件中引入 `core-js`：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        useBuiltIns: 'entry',
        corejs: '3.29',
      },
    ],
  ],
  plugins: [],
};


// origin.js
import 'core-js';

const obj1 = {
  name: 'tim',
  age: 18,
};
let d = obj1?.name;
let e = obj1?.gender;
const f = [1, 2, 3, 4].includes(5);
```

执行编译之后，可以看到文件顶部增加了 `require("core-js/modules/es.array.includes.js")` 的语句，这意味着 Babel 帮助我们引入了 corejs 实现的 `Array.prototype.includes` 方法，现在使用这个方法就不会有任何问题了。

```javascript
// compiled.js
"use strict";

// 此处省略一些引入语句
require("core-js/modules/es.array.for-each.js");
require("core-js/modules/es.array.from.js");
require("core-js/modules/es.array.includes.js");
require("core-js/modules/es.array.index-of.js");
// 此处省略一些引入语句

var obj1 = {
  name: 'tim',
  age: 18
};
var d = obj1 === null || obj1 === void 0 ? void 0 : obj1.name;
var e = obj1 === null || obj1 === void 0 ? void 0 : obj1.gender;
var f = [1, 2, 3, 4].includes(5);
```

到这里，我们对于 Babel 是如何将 ES6+ 代码编译到 ES5 有了一个基础的理解。那么接下来，我们就深入了解一下各个关键配置项的作用以及如何配置。

# 常用配置详解

首先要说明的是：本章所指的配置项都是 `@babel/preset-env` 的配置项。实际项目中我们需要关注的肯定不仅仅这一个预设的配置项，但相比较而言，`@babel/preset-env` 的配置更需要理解和掌握。

## `targets`

- 含义：项目运行的目标环境。
- 类型：`string | Array<string> | { [string]: string }`。
- 默认值：如果没有设置此项配置，那么 Babel 会尝试查找并使用项目中的 `.browserslistrc`、`package.json` 中的 `browserslist` 属性、或者是 Babel 配置文件中和 `presets` 同层级的 `targets` 配置项。如果前面的都没有找到，则表示 no targets，Babel 就认为我们的代码是要运行在版本非常早的浏览器上，会将新特性全部编译为 ES5。

Babel 通过读取此配置来进行更加智能的代码编译。如果我们的代码运行环境是比较新的 Chrome，那么许多的 ES6+ 特性浏览器本身已经支持，Babel 就不会在编译这一部分的新特性，可以达到一个减少项目打包体积的效果。  
在之前的练习中，我们是没有使用到这个配置的，现在我们添加一下试试效果：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '56',
        },
        useBuiltIns: 'entry',
        corejs: '3.29',
      },
    ],
  ],
  plugins: [],
};


// origin.js
const a = 1;
const b = () => {};
const c = a ?? 0;
```

你会发现，Babel 只对空值合并运算符进行了降级编译，箭头函数和 `const` 是不变的，原因就是 Chrome 56 已经支持了这两个新语法。  
Babel 获取 `targets` 配置值的方式很多，那是不是随便选哪个都可以呢？如果你的项目中只有 `@babel/preset-env` 用到这个配置，那用哪一种没啥区别。但在实际项目中，往往还有其他的工具或者是 Babel 的其他插件也会需要项目运行目标环境（在 Babel 中就是 `targets`）这个配置，比如 `autoprefixer`、`postcss`、`stylelint` 等。为了能够共享配置，避免重复定义，一般更加推荐使用项目中的 `.browserslistrc` 文件或 `package.json` 中的 `browserslist` 属性。

## `useBuiltIns`

- 含义：预设以何种方式进行 polyfill。
- 类型：`"usage" | "entry" | false`。
- 默认值：`false`，即不做任何的 API 编译。

实际项目中，我们肯定是需要进行 polyfill 的，所以肯定会开启这个配置。当开启这个配置时，`@babel/preset-env` 会使用 `core-js` 来完成对 API 的编译。

### `useBuiltIns: 'entry'`

entry 表示入门，入口。此时需要我们手动安装 `core-js` 并在入口文件中增加引用 `import "core-js"`（如果使用的是 Babel v7.4.0 之前的版本，那么需要安装的是 `@babel/polyfill`，并且需要在入口文件中增加引用 `import "@babel/polyfill"`）。之后，Babel 会根据我们设置的运行目标环境添加必要的 polyfill。比如：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '80',
        },
        useBuiltIns: 'entry',
        corejs: '3.29',
      },
    ],
  ],
  plugins: [],
};


// origin.js
import 'core-js';

const a = 1;
const b = () => {};
const c = a ?? 0;
const e = [1, 2, 3, 4].at(2);
```

编译完成之后，会发现多了很多的引入，这些就是 Chrome 80 所需要的所有的 polyfill。

```javascript
// compiled.js
'use strict';

require('core-js/modules/es.error.cause.js');
require('core-js/modules/es.aggregate-error.js');
require('core-js/modules/es.aggregate-error.cause.js');
require('core-js/modules/es.array.at.js');
require('core-js/modules/es.array.find-last.js');
require('core-js/modules/es.array.find-last-index.js');
require('core-js/modules/es.array.push.js');
require('core-js/modules/es.array.reduce.js');
require('core-js/modules/es.array.reduce-right.js');
require('core-js/modules/es.array.to-reversed.js');
// 篇幅原因，中间省略一些引入语句，原文件中有接近 200 行的引入语句
require('core-js/modules/web.immediate.js');
require('core-js/modules/web.self.js');
require('core-js/modules/web.structured-clone.js');
require('core-js/modules/web.url-search-params.size.js');
const a = 1;
const b = () => {};
const c = a ?? 0;
const e = [1, 2, 3, 4].at(2);
```

#### `ASYNC/AWAIT/GENERATOR` 的特殊处理

按照我们之前对于新特性的分类，`async/await/generator` 应该是属于新语法的一类。也就是说 `@babel/preset-env` 是能够将这些语法进行一个降级编译的。但如果你用的 `@babel/plugin-transform-regenerator`（`@babel/preset-env` 的一个依赖）版本是 v7.18.0 之前的，`@babel/preset-env` 并不能完成这一任务。

> 注：官方的 [v7.18.0 发布文档](https://babeljs.io/blog/2022/05/19/7.18.0) 说明了这一个变化，但如果你装了 v7.18.0 之前的 `@babel/preset-env`，并不一定能看到预期的效果，你会发现并不需要安装 `regenerator-runtime` 也是可以的。那不是官方说的有问题吗？也不是，其实是由于官方的说法比较笼统。准确的说法是这样的：v7.18.0 是指的 `@babel/plugin-transform-regenerator` 的版本，这个包就是专门用来处理 `async/await/generator` 的，它是 `@babel/preset-env` 的依赖之一。你可能装了 v7.16.8 的 `@babel/preset-env`，但安装的 `@babel/plugin-transform-regenerator` 是 v7.18.0 之后的，就会出现上面的情况。

这里我们用版本为 v7.17.9 的 `@babel/plugin-transform-regenerator` 来试一试：

```yaml
# pnpm-lock.yaml
/@babel/preset-env/7.20.2_@babel+core@7.21.0:
    resolution: {integrity: sha512-1G0efQEWR1EHkKvKHqbG+IN/QdgwfByUpM5V5QroDzGV2t3S/WXNQd693cHiHTlCFMpr9B6FkPFXDA2lQcKoDg==}
    engines: {node: '>=6.9.0'}
    peerDependencies:
      '@babel/core': ^7.0.0-0
    dependencies:
      ...
      '@babel/plugin-transform-regenerator': 7.17.9_@babel+core@7.21.0
      ...
    transitivePeerDependencies:
      - supports-color
    dev: false
      
```

```javascript
// origin.js
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

// compiled.js
"use strict";

var _marked = /*#__PURE__*/regeneratorRuntime.mark(generator);

function generator() {
  return regeneratorRuntime.wrap(function generator$(_context) {
    while (1) switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 1;
      case 2:
        _context.next = 4;
        return 2;
      case 4:
        _context.next = 6;
        return 3;
      case 6:
      case "end":
        return _context.stop();
    }
  }, _marked);
}
```

```shell
$> node dist/compiled.js

var _marked = /*#__PURE__*/regeneratorRuntime.mark(generator);
                           ^

ReferenceError: regeneratorRuntime is not defined
    at Object.<anonymous> (E:\Projects\babel-demo\dist\compiled.js:3:28)
    at Module._compile (node:internal/modules/cjs/loader:1254:14)
    at Module._extensions..js (node:internal/modules/cjs/loader:1308:10)
    at Module.load (node:internal/modules/cjs/loader:1117:32)
    at Module._load (node:internal/modules/cjs/loader:958:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:23:47
```

用 node 执行编译后的文件可以发现，出现了一个 `ReferenceError: regeneratorRuntime is not defined` 的错误。怎么解决呢？这就需要用到一个上面提到的名为 `regenerator-runtime` 的包来帮忙了。我们需要安装 `regenerator-runtime` 并在入口文件增加一个引入语句 `import 'regenerator-runtime/runtime'`（其作用就是在全局注入一个名为 `regeneratorRuntime` 的变量）。

```javascript
// origin.js
import 'regenerator-runtime/runtime';

function* generator() {
  yield 1;
  yield 2;
  yield 3;
}


// compiled.js
"use strict";

require("regenerator-runtime/runtime");
var _marked = /*#__PURE__*/regeneratorRuntime.mark(generator);
function generator() {
  return regeneratorRuntime.wrap(function generator$(_context) {
    while (1) switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 1;
      case 2:
        _context.next = 4;
        return 2;
      case 4:
        _context.next = 6;
        return 3;
      case 6:
      case "end":
        return _context.stop();
    }
  }, _marked);
}

```

此时再用 node 执行 `compiled.js` 就没有问题了。虽然问题解决了，但这里的处理方式其实是有点怪异的：`async/await/generator` 显然不属于新的 API，但我们却需要像对待新 API 一样采用类似于 polyfill 的方式进行一个处理。Babel 官方可能也注意到了这个问题，所以在 v7.18.0（确切的来讲是 `@babel/plugin-transform-regenerator` 的版本）之后，`regeneratorRuntime` 也变成了像其他的内联 helper 方法一样，并由 `@babel/plugin-transform-regenerator` 来提供。也就是说：v7.18.0 之后，`@babel/preset-env` 就可以完成 `async/await/generator` 的降级编译，我们不再需要 `regenerator-runtime` 这个包了。在我看来，这应该算是一种拨乱反正，更加明确了上面的一个约定：`@babel/preset-env` 负责语法，而 API 则交给 polyfill。  
修改一下 `@babel/plugin-transform-regenerator` 版本尝试一下：

```yaml
# pnpm-lock.yaml
/@babel/preset-env/7.20.2_@babel+core@7.21.0:
    resolution: {integrity: sha512-1G0efQEWR1EHkKvKHqbG+IN/QdgwfByUpM5V5QroDzGV2t3S/WXNQd693cHiHTlCFMpr9B6FkPFXDA2lQcKoDg==}
    engines: {node: '>=6.9.0'}
    peerDependencies:
      '@babel/core': ^7.0.0-0
    dependencies:
      ...
      '@babel/plugin-transform-regenerator': 7.20.5_@babel+core@7.21.0
      ...
    transitivePeerDependencies:
      - supports-color
    dev: false
```

```javascript
// origin.js
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}


// compiled.js
'use strict';

function _typeof(obj) {
 // 篇幅原因，这里不展示详细的代码
}
function _regeneratorRuntime() {
  // 篇幅原因，这里不展示详细的代码
}
var _marked = /*#__PURE__*/ _regeneratorRuntime().mark(generator);
function generator() {
  return _regeneratorRuntime().wrap(function generator$(_context) {
    while (1)
      switch ((_context.prev = _context.next)) {
        case 0:
          _context.next = 2;
          return 1;
        case 2:
          _context.next = 4;
          return 2;
        case 4:
          _context.next = 6;
          return 3;
        case 6:
        case 'end':
          return _context.stop();
      }
  }, _marked);
}
```

可以看到，编译后的文件中，多了一个 `_regeneratorRuntime` 的函数，不再是之前的那个全局变量了。

### `useBuiltIns: 'usage'`

上面的例子中你可能已经发现，我们在代码里只用了 `Array.prototype.at` 这一个方法，但是 Babel 编译之后，却给我们增加了很多并没有用到的 polyfill。有没有只引入我们实际所需的 polyfill 的方式呢？肯定是有的，`usage` 就是干这个事儿的。当设置了 `useBuiltIns: 'usage'` 时，我们不再需要在入口文件中显式地引入 `core-js`（如果 `entry` 时需要引入 `regenerator-runtime`，同样也不需要了），Babel 会分析我们的代码，并只引入我们实际用到的 polyfill。

> 注：虽然不需要我们在入口文件中添加引入语句了，但是需要的 `core-js`、`regenerator-runtime`（v7.18.0 之前）包还是需要安装的。

修改一下上面练习中的配置：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '80',
        },
        useBuiltIns: 'usage',
        corejs: '3.29',
      },
    ],
  ],
  plugins: [],
};


// origin.js
const a = 1;
const b = () => {};
const c = a ?? 0;
const e = [1, 2, 3, 4].at(2);
```

查看编译结果发现，确实只引入了我们使用的 `Array.prototype.at`polyfill，清爽了不少。

```javascript
// compiled.js
"use strict";

require("core-js/modules/es.array.at.js");
const a = 1;
const b = () => {};
const c = a ?? 0;
const d = [1, 2, 3, 4].includes(5);
const e = [1, 2, 3, 4].at(2);
```

## `corejs`

- 含义：使用哪个版本的 `core-js` 进行 polyfill。
- 类型：`string | { version: string, proposals: boolean }`。
- 默认值：`"2.0"`。

这个配置项需要配合 `useBuiltIns` 来使用，如果是 `useBuiltIns: false`，那么设置 `corejs` 是没有什么用的。配置此项时，官方推荐的做法是指定次要版本，即使用 `corejs: '3.29'` 而非 `corejs: '3'`。如果只写了主版本（major）号，比如 `corejs: '3'`，Babel 会当作 `3.0` 处理。查看 npm 可以知道，`core-js@3.0.0` 已经是 4 年前的版本了，很多新的 API 是没办法提供 polyfill 的。默认情况下，Babel 只会注入进入 ES 标准的 API 对应的 polyfill，如果你想用处在提案阶段的 API，可以开启 `proposals: true` 这个选项。为了能够用到最新的 API，推荐这样写：`corejs: { version: '3.29', proposals: true }`（`core-js@2.x` 已经是过时的包了，无论我们采用何种方式的进行 polyfill，都应该使用 `core-js@3.x` 版本）。随着时间的推移，我们的项目可能会需要升级 core-js 以满足我们的开发需要，这时就必须手动的修改 `corejs` 的 `version` 字段，多少有点不太方便。如果你想 `version` 字段的值和当前项目安装的 core-js 保持一致，也可以这样写：

```javascript
// babel.config.js
const corejsVersion = require('core-js/package.json').version;

module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '80',
        },
        useBuiltIns: 'usage',
        corejs: corejsVersion,
      },
    ],
  ],
  plugins: [],
};
```

> 注：`corejs` 是在 Babel v7.4.0 之后才有的配置，v7.4.0 之前没有这个配置。

## `include`

- 含义：需要一直包含的插件。
- 类型：`Array<string|RegExp>`。
- 默认值：`[]`。

如果目标环境对于某一个新语法的实现有 bug，或者不支持的功能 + 支持的功能组合不起作用，那么就可以将你需要的插件放在这个配置数组中。

## `exclude`

- 含义：需要一直排除掉的插件。
- 类型：`Array<string|RegExp>`。
- 默认值：`[]`。

如果你的项目中确定没有用到某一个语法，需要将用于这个编译这个语法的插件去掉，就可以放在这个配置中。或者说你需要用其他插件替换掉 `@babel/preset-env` 中的某一个插件，也可以将需要替换掉的插件放在这个配置数组中。

---

以上 5 个配置项，就是我们项目配置中最常用的配置项。`@babel/preset-env` 还有其他的配置项，但一般来讲其他的配置我们采用默认的就可以满足情况。如果有特殊需求，参考 [官方文档](https://babeljs.io/docs/babel-preset-env#options) 即可。

# `@babel/plugin-transform-runtime`

上面的介绍中，我们知道当 `useBuiltIns` 设置为 `usage` 时，Babel 只会引入我们需要的 polyfill，相比 `entry` 要轻量不少。那是不是我们在项目中一律使用 `useBuiltIns: 'usage'` 就万事大吉了呢？肯定不是的，不同的配置有优点也有缺点，还是要根据具体的业务场景来选择。这个我们先按下不表，接下来我们先来聊一下 `@babel/perset-env` 编译存在的问题。  
第一个问题是：**通过 `@babel/preset-env` 进行 polyfill 时会污染全局环境，无论你用的是 `usage` 还是 `entry`**。以 `Array.prototype.includes` 为例，查看引入文件中的代码会发现，polyfill 就是在全局对象 `Array` 的原型上增加了对应的方法实现。

```javascript
// compiled.js
"use strict";

require("core-js/modules/es.array.includes.js");
var a = [1, 2, 3, 4, 5].includes(4);


// core-js/modules/es.array.includes.js
...
$({ target: 'Array', proto: true, forced: BROKEN_ON_SPARSE }, {
  includes: function includes(el /* , fromIndex = 0 */) {
    return $includes(this, el, arguments.length > 1 ? arguments[1] : undefined);
  }
});
...
```

第二个问题则是：**Babel 在编译有些语法时，需要用到一些辅助方法（如 Class 的编译时会用到 `_defineProperties`、`_createClass`、`_toPrimitive` 等），这些 helpers 在编译的时候，会被添加到每个需要它的文件中去，造成不必要的重复代码**。

> 可以亲自动手尝试一下：新建 `sub1.js` 和 `sub2.js` 并在其中各自声明一个类，之后在 `origin.js` 中引用这两个类。通过 webpack 将这三个文件打包输出到 `compiled.js` 后，就会看到有些函数有两次定义，这些重复定义的函数就是上面提到的那几个辅助函数。

对于这两个问题，可以通过使用 `@babel/plugin-transform-runtime` 来解决。在 Babel 官网上对于 `@babel/plugin-transform-runtime` 的介绍只有一句话：一个可以重用 Babel 插入的 helpers 代码的插件，可以帮助节省项目体积。但实际上不止这一个功能，具体来说它主要提供了 3 个优化点：

- 提供了一系列的 helpers 方法，需要的时候直接引入而不是在文件顶部定义，避免了不必要的重复代码。
- 提供了一个不会污染全局的 `core-js`，需要进行 polyfill 时不会直接在全局对象上增加对应方法。
- 项目如果使用了 `async/await/generator`，会自动帮我们引入 `@babel/runtime/regeneraotr`，这一点和 `@babel/preset-env` 中的 `useBuiltIns: 'usage'` 的表现是一样的。

我们需要明确一点：**`@babel/plugin-transform-runtime` 本身并不包含任何的 helpers 和 polyfill，它必须配合 `@babel/runtime`、`@babel/runtime-corejs2`、`@babel/runtime-corejs3` 三个包中任意一个一起使用**，这三个包包含了我们需要的 helpers 和不会污染全局的 `core-js`。

## 常用配置项

接下来看一下 `@babel/plugin-transform-runtime` 的几个常用配置项。

### `helpers`

- 含义：是否将内联的 helpers 方法切换为从模块中引入。
- 类型：`boolean`。
- 默认值：`true`。

我们简单对比一下，看一看插件做了什么。首先是不使用插件进行编译：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '40',
        },
        useBuiltIns: 'usage',
        corejs: { version: '3.29', proposals: true },
      },
    ],
  ],
  plugins: [],
};


// origin.js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}


// compiled.js
'use strict';

require('core-js/modules/es.symbol.to-primitive.js');
require('core-js/modules/es.date.to-primitive.js');
require('core-js/modules/es.symbol.js');
require('core-js/modules/es.symbol.description.js');
require('core-js/modules/es.object.to-string.js');
require('core-js/modules/es.number.constructor.js');
require('core-js/modules/es.error.cause.js');
function _defineProperties(target, props) {
  // 篇幅原因省略
}
function _createClass(Constructor, protoProps, staticProps) {
  // 篇幅原因省略
}
function _toPropertyKey(arg) {
  // 篇幅原因省略
}
function _toPrimitive(input, hint) {
  // 篇幅原因省略
}
function _classCallCheck(instance, Constructor) {
  // 篇幅原因省略
}
var Person = /*#__PURE__*/ _createClass(function Person(name, age) {
  _classCallCheck(this, Person);
  this.name = name;
  this.age = age;
});
```

加入 `@babel/plugin-transform-runtime` 再编译一次：

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '40',
        },
        useBuiltIns: 'usage',
        corejs: { version: '3.29', proposals: true },
      },
    ],
  ],
  plugins: [['@babel/plugin-transform-runtime']],
};


// origin.js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}


// compiled.js
"use strict";

var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");
var _createClass2 = _interopRequireDefault(require("@babel/runtime/helpers/createClass"));
var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime/helpers/classCallCheck"));
var Person = /*#__PURE__*/(0, _createClass2.default)(function Person(name, age) {
  (0, _classCallCheck2.default)(this, Person);
  this.name = name;
  this.age = age;
});
```

原来内联的 helpers 变成了从 `@babel/runtime/helpers` 中引入。查看 `@babel/runtime` 的代码你就会发现，helpers 目录中包含的就是 Babel 所需要的所有辅助函数，`@babel/plugin-transform-runtime` 在这里就是做了一个替换。  
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12433539/1679498748595-e23b559b-369d-4705-bab8-27bbdd44b478.png#averageHue=%23272729&clientId=ua79b3d14-25fe-4&from=paste&height=674&id=ud1105ad1&originHeight=1011&originWidth=621&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=105496&status=done&style=none&taskId=ud470f32a-d3b7-4a38-a491-f0854a88232&title=&width=414)

### `corejs`

- 含义：是否使用不污染全局的 `core-js` 进行 polyfill。
- 类型：`false | 2 | 3 | { version: 2 | 3, proposals: true }`。
- 默认值：`false`。

这里不同的配置需要对应安装不同的 runtime 包配合使用：

| `corejs` 配置 | 需要安装的包 |
| --- | --- |
| `false` | `@babel/runtime` |
| `2 &#124; { version: 2, proposals: true }` | `@babel/runtime-corejs2` |
| `3 &#124; { version: 3, proposals: true }` | `@babel/runtime-corejs3` |

简单来说就是：如果你需要插件帮你做 polyfill，则需要安装带有 polyfill 的 runtime 包（`@babel/runtime-corejs3` 就是 `@babel/runtime`+`core-js-pure`）。而且还有一点需要特别注意：**`@babel/plugin-transform-runtime` 和 `@babel/preset-env` 都可以进行 polyfill，但两者并不是相互配合的，项目中只能采用其中一种 polyfill 方式**。如果你使用插件进行 polyfill，则不能开启预设的 `useBuiltIns` 配置；如果你使用预设进行 polyfill，则插件的 `corejs` 必须为 `false`。

> 注：这里所使用的 corejs 包和预设 `preset-env` 中的并不是一个东西。插件用的是 `core-js-pure`，一个不会污染全局的包。而 `preset-env` 使用的是会污染全局变量的包，注意区分。

接下来我们用练习项目来看一下 `@babel/preset-env` 和 `@babel/plugin-transform-runtime` 在进行 polyfill 时的表现有什么不同。

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '60',
        },
      },
    ],
  ],
  plugins: [
    [
      '@babel/plugin-transform-runtime',
      {
        corejs: { version: 3, proposals: true },
      },
    ],
  ],
};


// origin.js
const a = [1, 2, 3, 4].includes(1);
const b = [1, 2, 3, 4].at(-1);
const c = 'lancernix'.endsWith('x');


// compiled.js
"use strict";

var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault");
var _includes = _interopRequireDefault(require("@babel/runtime-corejs3/core-js/instance/includes"));
var _at = _interopRequireDefault(require("@babel/runtime-corejs3/core-js/instance/at"));
var _endsWith = _interopRequireDefault(require("@babel/runtime-corejs3/core-js/instance/ends-with"));
var _context, _context2, _context3;
const a = (0, _includes.default)(_context = [1, 2, 3, 4]).call(_context, 1);
const b = (0, _at.default)(_context2 = [1, 2, 3, 4]).call(_context2, -1);
const c = (0, _endsWith.default)(_context3 = 'lancernix').call(_context3, 'x');
```

通过编译结果可以看到，polyfill 的引入方式有比较大的变化，这就是我们上面提到的 `@babel/plugin-transform-runtime` 在进行 polyfill 时用的是一个不会污染全局的 `core-js`，和 `preset-env` 所用到的不一样。  
细心的同学可能还会发现一个现象：在 Babel 的配置中我们设置了 `targets` 为 `chrome 60`，这个版本的 chrome 其实已经实现了 `Array.prototype.includes` 和 `String.prototype.endWith` 这两个方法了，但插件还是引入了这两个方法对应的 polyfill。但实际上 `@babel/plugin-transform-runtime` 是可以根据目标运行环境（`targets`）智能引入所需 polyfill 的，那为啥在这没有起作用呢？原因也很简单：我们将 `targets` 设置在了 `@babel/preset-env` 中，插件读不到这个配置，所以没办法进行智能引入。这就印证了我们上面提出的一个说法：推荐使用项目中的 `.browserslistrc` 文件或 `package.json` 中的 `browserslist` 属性来设置项目的目标运行环境，而不是在 `@babel/preset-env` 中配置。

> 注：`@babel/plugin-transform-runtime` 在处理 helpers 和 regenerator 时，是可以根据 `@babel/preset-env` 中的 `targets` 配置进行智能引入的，但在使用 corejs 进行 polyfill 时却不行。所以为了避免出现不必要的问题，最好还是使用项目中的 `.browserslistrc` 文件或 `package.json` 中的 `browserslist` 属性设置项目的目标运行环境。

好，简单修改一下：

```javascript
// babel.config.js
module.exports = {
	presets: [
    [
      '@babel/preset-env',
    ],
  ],
  plugins: [
    [
      '@babel/plugin-transform-runtime',
      {
        corejs: { version: 3, proposals: true },
      },
    ],
  ],
  targets: {
    chrome: '60',
  },
};


// compiled.js
"use strict";

var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault");
var _at = _interopRequireDefault(require("@babel/runtime-corejs3/core-js/instance/at"));
var _context;
const a = [1, 2, 3, 4].includes(1);
const b = (0, _at.default)(_context = [1, 2, 3, 4]).call(_context, -1);
const c = 'lancernix'.endsWith('x');
```

此时再看一下编译结果可以发现符合我们的预期，插件只帮我们引入了 `Array.prototype.at` 的 polyfill。

> 注：这里为了节省时间使用了 Babel 的 `targets` 配置，换成 `.browserslistrc` 或者 `package.json` 中的 `browserslist` 属性，得到的结果是一样的，大家可以亲自尝试一下。

### `regenerator`

- 含义：是否自动引入用于编译 `async/await/generator` 的 regenerator runtime。
- 类型：`boolean`。
- 默认值：`true`。

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          chrome: '40',
        },
        useBuiltIns: 'usage',
        corejs: { version: '3.29', proposals: true },
      },
    ],
  ],
  plugins: [['@babel/plugin-transform-runtime']],
};


// origin.js
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}


// compiled.js
"use strict";

var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");
var _regenerator = _interopRequireDefault(require("@babel/runtime/regenerator"));
var _marked = /*#__PURE__*/_regenerator.default.mark(generator);
function generator() {
  return _regenerator.default.wrap(function generator$(_context) {
    while (1) switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 1;
      case 2:
        _context.next = 4;
        return 2;
      case 4:
        _context.next = 6;
        return 3;
      case 6:
      case "end":
        return _context.stop();
    }
  }, _marked);
}
```

通过编译结果可以发现，用到的 `_regenerator` 变成了从 `@babel/runtime/regenerator` 引入，不再是内联定义，也不再是从 `regenerator-runtime` 中引入了。

### `version`

- 含义：使用 `@babel/runtime` 的版本。
- 类型：`string`。
- 默认值：默认情况下，插件认为项目已经安装了 v7.0.0 的 `@babel/runtime`（或者是 `@babel/runtime-corejs3`）。

建议设置 `version` 为项目中安装的版本，这样插件可以用到最新的特性，如：

```javascript
{
  plugins: [
    [
      "@babel/plugin-transform-runtime",
      {
        corejs: 3,
        version: "^7.21.0"
      }
    ]
  ]
}

// 也可以这样写
const runtimeVersion = require('@babel/runtime/package.json').version;
{
  plugins: [
    [
      "@babel/plugin-transform-runtime",
      {
        corejs: 3,
        version: runtimeVersion
      }
    ]
  ]
}
```

## 关于 `regeneratorRuntime` 的探讨

上面我们提到过，在 v7.18.0 之后 Babel 将 `regeneratorRuntime` 变成了像其他的内联 helper 方法一样，我们不再需要单独安装 `regenerator-runtime` 包来注入一个全局变量了。但这里还有些细节需要关注，我们可以从 `@babel/runtime` 包的变化来深入了解一下。  
v7.18.0 之后 `regeneratorRuntime` 变成了一个 helper 方法，那应该在 `@babel/runtime/helpers` 中有一个对应的文件，打开 node_modules 验证一下，果然是有的。如果换一个 v7.18.0 之前的版本，你会发现 helpers 文件夹中是缺少这个文件的，很符合我们的理解。  
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12433539/1679840985292-fd6d80d3-5df2-4e4e-886c-e622df679f96.png#averageHue=%23262628&clientId=u0a3db822-07a0-4&from=paste&height=339&id=u6820910d&originHeight=508&originWidth=877&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=90120&status=done&style=none&taskId=u6a72bd3e-1d39-428c-87b5-c3567434b55&title=&width=584.6666666666666)  
那既然已经是 helper 了，为啥在编译的时候不从 `@babel/runtime/helpers/regeneratorRuntime` 中引入，而从 `@babel/runtime/regeneratorRuntime` 中引入呢？也容易理解：v7.17.x 到 v7.18.x 的升级是兼容性升级，之前插件帮我们自动引入使用的是 `require("@babel/runtime/regenerator")`，如果直接改成 `require("@babel/runtime/helpers/regeneratorRuntime")`，会导致很多使用 v7.18.0 之前版本插件编译的代码无法正常运行。但也并非没有变化，对比一下 v7.18.0 前后 regenerator 文件夹中 `index.js` 文件的改动我们可以知道：v7.18.0 之前，这个文件就是简单的帮助我们引入 `regenerator-runtime` 这个包，但在 v7.18.0 时就改成了引入 helpers 文件夹中的 `regeneratorRuntime.js` 文件并导出。

```javascript
// @babel/runtime/regenerator/index.js(v7.18.0)
module.exports = require("../helpers/regeneratorRuntime")();


// @babel/runtime/regenerator/index.js(v7.17.9)
module.exports = require("regenerator-runtime");
```

但实际上，v7.18.0 这个版本的改动是有问题的，我们对比一下 v7.18.3 版本就会发现代码中多了一个步骤：将 `regeneratorRuntime.js` 引入的变量命名为 `regeneratorRuntime` 并挂载到全局。

```javascript
// @babel/runtime/regenerator/index.js(v7.18.3)
var runtime = require("../helpers/regeneratorRuntime")();
module.exports = runtime;

try {
  regeneratorRuntime = runtime;
} catch (accidentalStrictMode) {
  if (typeof globalThis === "object") {
    globalThis.regeneratorRuntime = runtime;
  } else {
    Function("r", "regeneratorRuntime = r")(runtime);
  }
}
```

为啥需要这样呢，不是已经不需要这个全局变量了吗？其实还是为了保证兼容性升级，v7.18.0 之前，`require("regenerator-runtime")` 是会在全局挂载一个 `regeneratorRuntime` 变量的，如果升级版本之后没有了这个全局变量，肯定会导致使用 v7.18.0 之前版本插件编译的代码有问题，出现 `ReferenceError: regeneratorRuntime is not defined` 的错误。

> 关于这个问题的细节讨论，可查看 [再谈 Babel 7.18.0 引发的问题](https://developer.aliyun.com/article/982111) 这篇文章中的讲述。

这其实是一个过渡性的措施，作者也在 `regenerator/index.js` 中有注释到：_TODO(Babel 8): Remove this file。_Babel8 中，应该就会是直接从 `@babel/runtime/helpers/regeneratorRuntime` 中引入，而且不再有 regenerator 这个文件夹了。

# 不同业务场景下的 Babel 配置

通过上面章节的我们能发现，`@babel/preset-env` 配合 `@babel/plugin-transform-runtime` 可以组合出好几种 Babel 配置，在项目中我应该用哪一种呢？接下来我们给出常用的两种配置，并探讨一下两种配置所适用的业务场景。

## 业务项目

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
       	// targets 不在这里配置
        useBuiltIns: 'usage',
        corejs: { version: '3.29', proposals: true },
      },
    ],
  ],
  plugins: [['@babel/plugin-transform-runtime']],
};
```

我们日常的业务项目开发中基本不会有需要重写全局对象方法的场景，所以 polyfill 带来的全局污染并不是一个值得关注的问题。使用 `@babel/preset-env` 进行 polyfill，并配合 `@babel/plugin-transform-runtime` 提供的内联 helpers 函数转化为引入的功能，能够尽可能的减少项目最终打包的体积，是一个比较不错的选择。

> 如果项目不在意体积大小，那其实将 `useBuiltIns` 设置为 `'entry'` 其实是一个更加省心的方式。这其实提供了一个兜底的功能，如果你用的某个第三方包编译有问题（少引入了某个 polyfill 或者没有编译类似 `async/await` 这样的语法），那么 `useBuiltIns: 'entry'` 可以保证你的项目正常使用而不出问题，但 `useBuiltIns: 'usage'` 却不一定。举个例子：之前我们开发了一个通用的 FAQ SDK，各个业务系统都可以引入使用，在一个项目中用的时候没有问题，而换到另一个项目却出现了 `ReferenceError: regeneratorRuntime is not defined` 的错误。根本原因是 SDK 没有引入 `regenerator-runtime` 这个包导致没有 `regeneratorRuntime` 这个变量，那为啥有的项目可以正常用呢？就是这个项目使用的 Babel 配置中是 `useBuiltIns: 'entry'`，已经在全局环境注入了 `regeneratorRuntime`，所以可以正常运行。

## 工具库 & 类库

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      // targets 不在这里配置
      '@babel/preset-env',
    ]
  ],
  plugins: [
    [
    	'@babel/plugin-transform-runtime',
      {
        corejs: { version: 3, proposals: true },
        version: "^7.21.0",
      }
    ]
  ],
};
```

如果你正在开发工具库或者组件库，那么全局污染其实是一个需要尽量去避免的问题。所以我们不能采用 `@babel/preset-env` 所提供的 polyfill 方案，需要通过 `@babel/plugin-transform-runtime` 来进行 polyfill，同时插件也可以将内联 helpers 函数转化为引入。这样既能达到不污染全局环境的目的，也能尽可能减少项目打包后的体积。

> 业务项目中如果有需要，也可以采用这种配置。多数情况下第一种就足够应付业务场景了。

# 总结

通过上面的学习，我们应该基本掌握了如何配置并使用 Babel，可喜可贺🥳！这篇近万字的文章花了一周的时间写完，对于我自己来说收获很大，精通 Babel 配置说不上，熟练应该没问题😛。如果你也想掌握如何使用 Babel，那么我强烈建议你动手敲一遍，看再多的文章，都比不上自己动手有用。这一类的文章通常时效性很强，可能你 2 个月后再看会发现这篇文章就已经过时了。最好的参考资料永远是官方文档，大部分问题都可以在官方文档中找到答案。如果有解决不了的疑惑或者问题，去 Github 提 issue ，和作者交流也是一种最有效率的方式。  
最后，祝大家越来越强！完结撒花🎉🎉🎉

# 参考文档

- [Babel 官方文档](https://babeljs.io/docs)
- [小白都能听懂的最新版 Babel 配置探索——保姆级教学](https://juejin.cn/post/7147582343471955999)
- [一文聊完前端项目中的 Babel 配置](https://juejin.cn/post/7151653067593613320)
- [再谈 Babel 7.18.0 引发的问题](https://developer.aliyun.com/article/982111)
