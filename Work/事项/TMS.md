# 图片优化记录

## Common-base

- 图标基本上都是 svg 格式的，尺寸不大，基本都在 5kb 以下
- 有些 png 的图片，使用 TinyPng 压缩了一下，-30%
- `otms-login-bg.png` 这个图片最大，有 580kb，已经做了优化，使用的是 cdn，如果支持 webp，则加载 webp 格式

## Common-pass

- `otms-login-bg.png` 也做了优化，使用的是 cdn，如果支持 webp，则加载 webp 格式
- 都是用的是 cdn
- `images` 这文件夹下只有一张没有用到的图片

## Common-ros

- `images` 中的 png 压缩了一下，-70%
- `favicon.ico` 比较大，没找到用的地方
- `static/images` 中的也没啥用

## 主项目

- `components/monitor/icons` 中的图标压缩了一下，-55%
- `handover_bg.png` 比较大，3.5M，压缩之后也有 1.1M
- `images` 压缩了一下，-60%，里面应该有一些图片没用到的

# Selector 替换

```js
export default function transformer(file, api) {
  const j = api.jscodeshift;
  const root = j(file.source);

  const ns = root
    .find(j.VariableDeclarator, {
      id: { type: "Identifier", name: "selectNamespace" }
    })
    .at(0);
  if (!ns.length) {
	// 没找到selectNamespace，则直接不处理
    return root.toSource();
  } else {
    // 处理
    const path = ns.get();
    const nsId = j(path.node.id).toSource();
    const nsExp = j(path.node.init).toSource();

    console.log(nsId);
    console.log(nsExp);
  }

  const selectors = root
    .find(j.ExportNamedDeclaration)
    .filter((path) => {
      const { declaration } = path.node;
      if (declaration.type !== "VariableDeclaration") return false;
      return declaration.declarations.some(
        (dec) =>
          dec.init.type === "CallExpression" &&
          dec.init.callee.type === "Identifier" &&
          dec.init.callee.name === "createSelector"
      );
    })

  return root.toSource();
}
```

 [**src/containers/RequirementOrderManage/selectors.tsx**](https://gitlab.sftcwl.com/oms-tms/apodidae/-/compare/master...online_20240815_optimize#509cfe157fad9aae85ecaebb7bcd9440fb4c7f83)

# P0 错误统计

| 错误                                                                                                                                                      | 页面                                                                                                                                 | 进度          | 备注              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------- | --------------- |
| TypeError: Cannot read properties of undefined (reading 'map')                                                                                          | /onOmsSystem/requirementOrderManage/requirementOrderDetail?pre_order_id=117258725733200003&customer_order_code=ERP 和实 202400909008 | #NotStarted | 没 sourceMap，不好排查 |
| TypeError: t[n.label].indexOf is not a function                                                                                                         | /rule/orderAutoRule                                                                                                                | #NotStarted |                 |
| TypeError: o.flat is not a function                                                                                                                     | /centerConsole/TemHumMonitor                                                                                                       | #NotStarted |                 |
| TypeError: Cannot read properties of undefined (reading 'key')                                                                                          | /onTmsSystem/splitManage                                                                                                           | #NotStarted |                 |
| TypeError: Invalid attempt to destructure non-iterable instance.<br/>In order to be iterable, non-array objects must have a [Symbol.iterator]() method. | /waybillCenter/orderManage/orderCreate                                                                                             | #NotStarted |                 |
| TypeError: Cannot read properties of undefined (reading '59')                                                                                           | /waybillCenter/orderManage                                                                                                         | #NotStarted |                 |
| TypeError: Cannot read properties of undefined (reading 'name')                                                                                         | /centerConsole/alarmStatistics                                                                                                     | #NotStarted |                 |

# 时区改造记录

- 运单中心 - 运输订单管理
	- 页面操作
		- 批量上传图片【上传时间】
		- 新建订单【3 个时间】
	- 列表页
		- 8 个日期筛选项（前后端交互都是秒级时间戳）
		- 列表展示（有字符串、也有秒级时间戳）
	- 详情页
		- 查看
			- 操作日志【操作时间】
			- 订单路径【5 个时间】
			- 异常处理【生成时间、处理时间】**是否需要处理**
		- 编辑
			- 【预计发货时间、预计到货时间】，客户个性化信息中的日期相关字段 **是否需要处理**
- TMS- 子单管理
	- 列表页
		- 3 个日期筛选项（前后端交互都是秒级时间戳）
		- 列表展示（有字符串、也有秒级时间戳）
	- 页面操作
		- 人工排线【到货时间、发货时间】
- TMS- 排线管理
	- 列表页
		- 3 个日期筛选项（前后端交互都是秒级时间戳）
		- 列表展示（有字符串、也有秒级时间戳）
	- 详情页
		- 所有模块的时间信息，**客户个性化信息中如果有日期相关字段要处理吗**
	- 订单操作
		- 编辑
			- 【预计发货时间、预计到货时间】**是否需要处理**
- TMS- 运单管理
	- 列表页
		- 3 个日期筛选项（前后端交互都是秒级时间戳）
		- 列表展示（有字符串、也有秒级时间戳）
	- 详情页
		- 所有模块的时间信息，**客户个性化信息中如果有日期相关字段要处理吗**
	- 页面操作
		- 批量上传图片【上传时间】
		- 编辑
			- 客户个性化信息中有日期相关字段 **是否要处理**

