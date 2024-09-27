# 一些重要的改动

## 打包优化（2024 年 07 月 19 日）

相关代码：

```bash
internals
└── webpack
    ├── babelConfig.js # 项目的babel配置
    ├── babelDepConfig.js # 用于node_modules的babel配置
    ├── terserOptions.js # terser压缩配置
    ├── webpack.base.js # webpack基础配置
    ├── webpack.dev.js # webpack开发配置
    └── webpack.prod.js # webpack打包配置
```

之前做过一次打包优化，不过在做 dayjs 替换 moment 时发现当时写的拆 chunk 规则有点问题，顺便优化了一下，目前应该是比较合理的一个打包配置。经过两次优化，产物体积从 13.5M 降低到了 5.3M，效果明显。

**后面也许还能优化的点：**

- 「拦截调用规则」中的条件设置组件用了一个叫 ruleconfig 的三方包（这个包的问题可以咨询阳哥），这个包体积比较大。现在配置了打包时拆成单独的 chunk 并且支持按需引入。后面如果有需要，可以考虑下掉这个包（如果打算下掉，先和后端确认一下会不会影响后端现有的逻辑），用其他组件替换一下，能节省 1.3M+ 的体积。
- 将 webpack 切换成 rspack，提升开发、打包时的速度。

## 时区改造（2024 年 07 月 19 日）

整体方案是前后端交互采用东八区日期时间字符串（最简单的方式当然是使用时间戳来交互，但是这样后端也有比较大的改动量，所以就还是按照原来的接口交互来做了），这样前端就要做大量的转换工作。

**主要有以下几个场景：**

- 表单筛选项（比如订单创建时间、发货时间等），通常也可能是一个范围，需要在提交接口的时候转成东八区的时间。
- 列表、详情展示，需要将接口给的东八区时间转换成用户所在的时区，并且在后面展示当前时区对应的 UTC 信息（比如 `2024-09-09 09:09:09 (UTC+09:00)`）。
- 表单提交，这里需要先将接口传回的东八区时间转换成用户所在的时区（查看、编辑的时候需要），然后提交的时候再转成东八区的时间。
- 导入、导出的操作，文件的解析和生成都是由后端来完成的。这需要在导入、导出相关的接口 header 中增加时区信息让后端能感知，简单做就是直接在所有请求中都做这个逻辑。

**有几个注意点：**

- 有些展示只精确到日期的字段，可能前端做时间补充，比如页面展示的是 `2024-09-09 ~ 2024-09-10`，但实际上接口交互的时候用的是 `2024-09-09 00:00:00 ~ 2024-09-10 23:59:59`。这种情况，也需要进行转换。
- 获取时区使用的 `Intl.DateTimeFormat().resolvedOptions().timeZone` 这个方法，得到的是时区字符串（比如：`Asia/shanghai`）。需要注意浏览器兼容性，必要的话需要进行 polyfill。

**其他的一些记录：**

- 入库、出库、库存三个重要的模块都做了时区改造。
- 规则目前还没做，其实应该需要做的。
- 剩下的改动页面可以参考 [需求文档](https://confluence.sf-express.com/pages/viewpage.action?pageId=405147971)。
- 首页可以跳转其他 4 个页面，其中 2 个做了时区转换、2 个没做，所以这里通过 `history.location` 传参有额外的处理。
- 所有的日期、时间、范围选择组件都切换成了组件库中使用 dayjs 封装的组件，不建议再使用 antd 原生的对应组件。
- moment 在项目中没用到，所以打包的时候去掉了，降低一点打包体积。

## 国际化

### 中文直接作为值

 -  src/components/DistrictsComponent/index.tsx: "Line [128]: 其它区", "Line [200]: 中国"
- "src/pages/InboundDetailManage/modules/FormTable.tsx": ["Line [107]: 质量状态 "], 中文做匹配
- "src/pages/LackSku/modules/FormTable.tsx": ["Line [74]: 质量状态 "], 中文做匹配

**国际化目前存在的问题：**

- [ ] 存在使用中文做匹配的逻辑（不到 10 个）
- [ ] 使用 `i8nMessageToMap` 定义的词条提取脚本提取不出来（大概 40 个文件），需要改造成 `defineMessages`，脚本改造好像看起来有点难度
- [ ] 在非组件逻辑中使用 `react-intl`，目前程序中提供了一个直接从 json 文件中提取指定 id 的词条来实现，可以考虑切换官方推荐的方式
- [ ] 提取脚本是直接覆盖还是增量更新

## 列表筛选项初始值设置&重置（2024 年 07 月 29 日）

本次改动将项目的列表筛选项初始值设置都统一了（**这里说的只是列表页，不涉及弹窗等页面**），具体的场景和开发规范是这样的：

- 没有条件导出的页面：
	- 没有条件导出的页面，绝大多数情况是不需要用 `searchCondition` 缓存查询条件的，所以就没必要在 redux store 中增加这个多余的东西。
	- 如果筛选项需要设置初始值的（如默认的时间范围），直接在 `Form.Item` 的 `initialValue` 属性中添加对应的初始值即可。
	- 筛选项不需要设置初始值的，就不要给 `Form.Item` 设置 `initialValue`，没意义。
	- 重置的时候，直接调用 `form.resetFields()` 即可。
- 有条件导出的页面：
	- 条件导出依赖于缓存的查询条件，所以 redux store 中都有 `searchCondition` 。这个时候为了便于维护，初始值统一在 `initialState.searchCondition` 中维护是最好的。
	- `initialState.searchCondition` 中只设置必要的筛选项初始值，不需要设置初始值的就不写。
	- 有初始值的 `Form.Item` 的 `initialValue`，直接从 `initialState.searchCondition` 取。
	- 重置的时候，除了要调用 `form.resetFields()`，还需要同步重置 `searchCondition`，所以要多加一个操作：`dispatch(actions.updateSearchCondition(initialState.searchCondition))`。

> 上面列出的两个规范大多数页面都是遵循的，一些页面可能有些特殊的操作，但也只是一些很小的差别。可以看一下出库单处理页面（OrderManage）和货主管理页面（CompanyManage），理解一下这些差别。
