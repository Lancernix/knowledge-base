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
		- [x] 批量上传图片【上传时间】秒级时间戳，不需要处理
		- [x] 新建订单【3 个时间】秒级时间戳，不需要处理
	- 列表页
		- [x] 8 个日期筛选项（接口交互都是秒级时间戳）秒级时间戳，不需要处理
		- [x] 列表展示（有字符串、也有秒级时间戳）用 `$ToUTC` 转换
	- 详情页
		- 查看
			- [x] 基础信息【客户下单时间】秒级时间戳，不需要转换
			- [x] 运输信息
				- [x] 【预计发货、预计到货时间】秒级时间戳，不需要处理
				- [x] 可视化【4 个时间】
			- [!] 客户个性化信息 **如果存在日期相关字段，不太好处理**
			- [x] 订单路径【5 个时间，都是秒级时间戳】用 `$ToUTC` 转换
			- [/] 运输路由
				- [x] 【操作时间】用 `$ToUTC` 转换
				- [!] 【路由描述】中**存在时间信息，这个页面不好转换，如果必须转换需要后端来转**
			- [x] 异常处理
				- [x] 【生成时间、处理时间】用 `$ToUTC` 转换
				- [x] 新建工单【发生时间】秒级时间戳，不需要处理
			- [x] 操作日志【操作时间】统一在组件中处理的，秒级时间戳 用 `$ToUTC` 转换
			- [?] 温度监控【记录时间】`src/containers/OrderDetail/modules/components/TempatureMonitorTable.tsx` 转换了，但没找到数据验证
		- 编辑
			- [x] 运输信息
				- [x] 【预计发货时间、预计到货时间】（使用秒级时间戳交互），不需要处理
				- [x] 客户个性化信息中的日期相关字段（看代码也是使用秒级时间戳交互），不需要处理
- TMS- 子单管理
	- 列表页
		- [x] 3 个日期筛选项（接口交互都是秒级时间戳） 不需要处理
		- [x] 列表展示（有字符串、也有秒级时间戳） 用 `$ToUTC` 转换
	- 页面操作
		- [x] 人工排线
			- [x] 编辑【到货时间、发货时间】（交互是秒级时间戳）不需要处理
			- [x] 查看【到货时间、发货时间】 用 `$ToUTC` 转换
- TMS- 排线管理
	- 列表页
		- [x] 3 个日期筛选项（前后端交互都是秒级时间戳）不需要处理
		- [x] 列表展示【6 个字段】用 `$ToUTC` 转换
	- 详情页
		- [!] 客户个性化信息，**如果存在日期相关字段，不太好处理**
		- [x] 路线信息【8 个字段】用 `$ToUTC` 转换
		- [?] 航班信息，`src/containers/LineDetail/components/FlightInfo.tsx` 转换了，但没找到数据验证
		- [x] 路顺信息
			- [x] 展示【2 个字段】用 `$ToUTC` 转换
			- [x] 交付详情点击【实际操作时间】用 `$ToUTC` 转换
			- [x] 实际到货时间点击【2 个时间】用 `$ToUTC` 转换
			- [x] 实际发货时间点击【2 个时间】用 `$ToUTC` 转换
	- 订单操作
		- [x] 编辑【预计发货时间、预计到货时间】（交互是秒级时间戳）不需要处理
- TMS- 运单管理
	- 列表页
		- [x] 3 个日期筛选项（接口交互都是秒级时间戳）不需要处理
		- [x] 列表展示（有字符串、也有秒级时间戳）用 `$ToUTC` 处理
	- 详情页
		 - [!] 客户个性化信息，**如果存在日期相关字段，不太好处理**
		 - [x] 路线信息【8 个字段】用 `$ToUTC` 转换
		 - [x] 路顺信息
			 - [x] 展示【2 个字段】用 `$ToUTC` 转换
			 - [x] 交付详情点击【实际操作时间】用 `$ToUTC` 转换，【操作时间】提交用秒级时间戳，不需要处理
			 - [x] 实际到货时间点击【2 个时间】用 `$ToUTC` 转换
			 - [x] 实际发货时间点击【2 个时间】用 `$ToUTC` 转换
			 - [x] 上传图片点击【操作时间】提交用秒级时间戳，不需要处理
	- 页面操作
		- [x] 批量上传图片【上传时间】提交用秒级时间戳，不需要处理
		- [x] 编辑 【有日期字段】提交用秒级时间戳，不需要处理
- TMS- 收派单管理
	- 列表页
		- [x] 1 个筛选项（接口交互是字符串）直接使用 `momentToString`、`stringToMoment` 进行转换
		- [x] 列表展示（有字符串、也有秒级时间戳）用 `$ToUTC` 转换
	- 详情页（其实就是运单详情）
		- [x] 异常处理
		- [x] 日志记录
		- [x] 路线信息
		- [x] 路顺信息
- 运力管理 - 人员管理
	- 列表页
		- 列表展示
			- [!] 【驾驶证领证日期、驾驶证有效期至、有效期至】只展示日期，应该不需要进行时区转换
			- [x] 【剩余 4 个字段】用 `$ToUTC` 转换
	- 详情
		- [!] 【驾驶证领证日期、驾驶证有效期至、有效期至】只展示到日期，应该也不需要处理
		- [x] 【司机培训开始日期、司机培训有效期至】秒级时间戳，不需要处理
		- [x] 日志 用 `$ToUTC` 转换
	- 页面操作
		- 编辑/新建
			- [!] 【驾驶证领证日期、驾驶证有效期至】传的是精确到日期的字符串，应该不需要进行时区转换
			- [x] 【司机培训开始日期、司机培训有效期至、有效期至】提交用的秒级时间戳，不需要处理
- 运力管理 - 车辆管理
	- 列表页
		- 列表展示
			- [!] 【车辆注册日期、检验有效期】只展示日期，应该不需要进行时区转换
			- [x] 【剩余两个字段】用 `$ToUTC` 转换
	- 详情
		- [!] 【车辆注册日期、检验有效期、车挂注册日期】传的是精确到日期的字符串，应该不需要进行时区转换
	- 日志
		- [x] 操作时间 直接使用 `momentToString` 进行转换
	- 页面操作
		- [!] 编辑/新建 【车辆注册日期、检验有效期、车挂注册日期】传的是精确到日期的字符串，应该不需要进行时区转换
- 运力管理 - 车型管理
	- 列表页
		- [x] 列表展示【2 个字段】用 `$ToUTC` 转换
		- [x] 日志
- 运力管理 - 承运商管理
	- 列表页
		- 列表展示【2 个字段】
		- [x] 日志
	- 详情
		- 有效期相关日期
		- [x] 日志
	- 页面操作
		- 编辑/新建
			- [!] 【有效期】传的是精确到日期的字符串，应该不需要进行时区转换
- 可配置能力管理 - 可配置能力模版
	- 列表页
		- [x] 列表展示【2 个字段】
- 路由中心 - 路由管理
	- 列表展示
		- [x] 【操作时间】用 `$ToUTC` 转换
		- [!] 【操作描述、路由详细描述】内容有时间，需要后端来转
	- [x] 详情【两个字段】用 `$ToUTC` 转换
- 预处理规则管理 - 订单拆分规则
	- 编辑/新建
		- [!] 路线配置 -> 时刻模式【出发、到达时间】只有时间信息，要转换吗
	- 列表
		- [x] 日志
- 预处理规则管理 - 订单自动化规则
	- 编辑/新建
		- [!] 订单筛选条件【日期精确到天、时间只有时间信息】要转换吗
		- [!] 规则触发条件【触发时间】只有时间信息，要转换吗
		- [!] 订单自动操作【预计发货时间】只有时间信息，要转换吗
	- [x] 日志
- 预处理规则管理 - 固定排线规则
	- [x] 列表【更新时间】用 `$ToUTC` 转换
	- 编辑/新建
		- [!] 路线配置 时刻模式【到达时间、出发时间】只有时间信息，要转换吗
- 策略中心 - 承运商分配规则
	- 列表页
		- [x] 展示【2 个字段】用 `$ToUTC` 转换
	- 编辑/新建
		- [x] 【生效开始、结束时间】接口用秒级时间戳，不需要处理
		- [!] 筛选订单条件【日期精确到天、时间只有时间信息】要转换吗
	- 详情
		- [x] 【生效开始、结束时间】秒级时间戳，不需要处理
		- 日志
			- [x] 【操作时间】用 `$ToUTC` 转换
			- [!] 【操作描述】中可能存在时间信息，需要后端来转
- 网络规划管理 - 站点管理
	- 列表页
		- [x] 展示【2 个字段】用 `$ToUTC` 转换
	- 新建/编辑
		- [!] 【交接时间、送货时间】只有时间信息，要转换吗
	- 详情
		- [x] 日志
