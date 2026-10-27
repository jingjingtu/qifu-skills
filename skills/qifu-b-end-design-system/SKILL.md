---
name: qifu-b-end-design-system
description: 公司级 B 端页面编排与缺口审计入口。用于把自然语言需求抽象为页面类型、Pattern、组件和平台适配器；列表、详情和抽屉路由到已有页面 Skill，工作台、树表、向导、权限和数据看板仅做规格与缺口分析，不宣称已具备自动落图能力。
---

# Qifu B 端 Design System

## 目标

把“某个平台的一张页面”提升为可复用的公司级设计能力：

```text
业务描述 → 页面原型 Archetype → 组合模式 Pattern → 组件 Component → 平台适配 Adapter → Figma 真实实例
```

本 Skill 不是某个平台的页面模板，也不是截图复刻器。它负责：

1. 识别业务对象、任务和页面类型；
2. 选择平台无关的页面骨架与组合模式；
3. 从共享组件注册表中调用真实组件；
4. 用平台适配器注入品牌、导航、权限、业务术语和平台特有组件；
5. 输出可审计、可迁移、可维护的 Figma 页面。

## 何时使用

- 用户要求“用奇富组件库做一个 B 端后台页面”；
- 用户希望把毓数、风控、运营、客服、财务等平台的页面模式统一；
- 用户要求把页面能力抽象成组件、Pattern、Page Archetype 或 Skill；
- 用户要求基于 Figma 组件库生成页面，而不是画图片或手工仿制；
- 需要评估组件缺口、建立新平台适配器或迁移已有页面。

若需求只是改一处视觉样式，直接编辑页面即可。发现重复出现的页面能力时先在交付中提出沉淀建议；只有用户明确要求维护本仓库时，才修改 Skill 或 references。

## 核心分层

| 层 | 名称 | AI 需要知道什么 | 本仓库位置 |
|---|---|---|---|
| 横切 | Governance | 命名、版本、发布、缺口和验证规则 | 本 Skill + audit |
| L1 | Token | 颜色、字体、间距、圆角、阴影、尺寸、状态 | `../qifu-shared/references/component-map.md` 与组件库变量 |
| L2 | Component | 组件集、节点 ID、发布 Key、属性、状态、可组合关系 | `../qifu-shared/references/component-map.md` |
| L3 | Pattern | 筛选栏、操作栏、详情块、表单分区、指标卡、时间线 | 各页面 Skill 的 references |
| L4 | Page Archetype | 列表、详情、抽屉、工作台、树表、向导、看板等 | `skills/` 下的页面 Skill |
| L5 | Platform Adapter | 导航、品牌、术语、权限、业务槽位、平台特有组件 | `../qifu-shared/references/platform-*.md` |
| L6 | Business Page | 某个平台某菜单下的一张具体页面 | 业务交付页面，不反向污染共享层 |

**约束：**业务差异通过 Adapter 和 Slot 注入；共享组件契约不写平台差异，平台差异只进入 `platform-*` 适配器，页面交付不复制组件库规则。

## 执行流程

### 1. 解析需求

先读取 `../qifu-shared/references/page-context.md`，形成公共 `PageContext`，再提取编排层 `PageSpec`：

```yaml
platform: <已注册平台名或 generic>
productArea: <产品域>
menuPath: [<一级>, <二级>, <三级>]
object: <业务对象>
tasks: [create, edit, view, search, approve, export]
archetype: <由下方决策树选择>
state: data | loading | empty | error | disabled
permissions: [view, create, edit, delete, export]
contentDensity: compact | normal | spacious
```

缺失信息用合理默认值，不要停在追问：默认 `platform=generic`、`state=data`、`contentDensity=normal`；将假设列入交付说明。

### 2. 选择 Page Archetype

- 多条记录、搜索/筛选、分页 → **List**；
- 单个对象查看、字段分组或时间线 → **Detail**；
- 新建/编辑且不离开当前上下文 → **Drawer Form**；
- 多个指标、趋势、分布、排行 → **Dashboard**；
- 需要高频入口、待办、快捷操作 → **Workspace**；
- 层级数据、展开收起、父子汇总 → **Tree Table**；
- 多步骤提交、校验、回退 → **Wizard**；
- 角色、资源、操作授权 → **Permission**；
- 以上组合 → 选择一个主 Archetype，其他作为 Pattern，不新造页面类型。

List、Detail、Drawer Form 路由到现有 Skill。其他 Archetype 当前状态为 `planned`：只输出 PageSpec、Pattern、组件缺口和建议，不创建页面，并返回 `UNSUPPORTED_ARCHETYPE`。先读对应 Archetype 的 `SKILL.md` 和 references，再读平台适配器；不要直接从截图推断组件名称。

### 3. 选择 Pattern 与组件

使用 `../qifu-shared/references/component-map.md` 做组件发现：

- 先匹配语义（例如“筛选单选”），再匹配视觉；
- 优先使用已发布组件 Key 创建实例；同文件生成时优先使用节点 ID；
- 组件不存在时先用合法组合；仅当共享调用基线确认 `COMPONENT_MISSING` 且用户明确允许降级时，才使用 `Fallback / <Capability>`，不得伪称为正式组件；
- 组合区块必须记录 `patternName`、输入槽位、默认状态和允许的变体；
- 业务文案通过文本覆盖或可配置属性注入，不修改母版。

### 4. 应用 Platform Adapter

平台适配器只负责平台差异：

- Header、SideMenu、Breadcrumb、平台 Logo 和主题；
- 菜单树、当前页、祖先路径高亮、平台术语；
- 平台专属业务组件（例如圈选、地图、富文本、指标选择器）；
- 权限、角色、数据范围和空态文案；
- 页面容器宽度、导航密度、品牌色等有限的布局参数。

当 `platform=generic` 时，使用 `platform-generic.md` 的中性导航和占位槽位，并在交付信息中标记待适配项。用户点名但仓库未注册的平台返回 `UNSUPPORTED_PLATFORM`，不借用其他平台规则。

### 5. 生成与验证

生成后必须检查：

- 设计系统元素是 `INSTANCE`，没有把截图、SVG 或手绘矩形冒充组件；
- 页面树符合 `Page Archetype → Pattern → Component` 层级；
- 所有实例都在正确的容器内，没有 Page 根级孤儿；
- 没有与实例重叠的重复节点；
- 组件属性、状态、尺寸、间距来自注册表或对应 blueprint；
- 页面状态只有一种；权限不允许的动作不显示或明确 disabled；
- 发现缺口时生成 `Audit / Missing Components`；未获降级授权时只说明建议方案，不创建 Fallback；
- 页面不溢出、不重叠、不截断，且能通过 Adapter 替换平台导航后继续成立。

## Figma 来源与调用基线

组件库：`奇富科技中后台组件库 新`

- File key：`gTV3VdC6a5e9vpkRHIZSXA`
- Figma URL：见用户提供的组件库链接
- 组件资产索引：`../qifu-shared/references/component-map.md`
- 毓数平台适配器：`../qifu-shared/references/platform-yushu.md`
- 通用平台适配器：`../qifu-shared/references/platform-generic.md`

节点 ID 只保证在该组件库文件内有效；发布 Key 用于其他文件导入。组件被重建后，以精确组件集名称重新发现并更新注册表，不凭旧节点 ID 猜测。

## 输出格式

完成页面或设计能力后，返回：

1. `PageSpec`：平台、菜单、对象、任务、Archetype、状态、权限；
2. `compositionName` 与 `patternName`；
3. 使用的主要组件、变体、节点 ID / 发布 Key；
4. Figma 节点 ID、页面树和平台适配器；
5. 假设、缺口、Fallback 与建议补齐的组件属性；
6. 验证结果：实例关系、层级、重复节点、布局、状态、可迁移性。

## 相关参考

- [公司级架构与 TDesign 对照](references/architecture.md)
- [组件注册表契约](references/component-contract.md)
- [页面类型与迁移矩阵](references/page-archetypes.md)
- [平台适配器契约](references/platform-adapter.md)
- [通用平台适配器](../qifu-shared/references/platform-generic.md)
- [共享页面上下文](../qifu-shared/references/page-context.md)
- [共享组件调用基线](../qifu-shared/references/component-invocation-baseline.md)

## Figma 真实变量调用

视觉 Token 的权威数据源为 `../qifu-shared/theme/tokens.json` 与 `../qifu-shared/theme/source/*.json`。先读取 `../qifu-shared/theme/component-token-contract.json`，已存在的组件变量优先绑定；未具备组件变量的插槽只能使用契约指定的 `基础` 语义变量，不得直接使用 `色彩` collection 或 Hex。调用时必须使用记录中的 `figmaPath` / `name`；根据 `基础` collection 的 `毓数/Light`、`智能运营/Light` 与 `色彩` collection 的 `mode/green`、`mode/blue` 选择真实模式。`qifu-shared/theme/*.less` 中的标识符是代码导出映射，不是 Figma 原变量名。
