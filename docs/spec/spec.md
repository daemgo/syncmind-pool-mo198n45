> **版本**：1.0.0 | **状态**：draft | **更新时间**：2026-04-18T00:00:00.000Z
>
> **来源方案**：会议纪要智能解析与销售跟进系统 场景 v1.0.0 版本

---

#### 约定说明

本文档使用以下标准值：
- **布局类型**: `list` / `detail` / `form` / `dashboard` / `steps` / `custom`
- **区块类型**: `table` / `form` / `card` / `cards` / `chart` / `tabs` / `steps` / `timeline` / `description` / `statistic` / `custom`
- **字段类型**: `text` / `textarea` / `number` / `money` / `percent` / `date` / `datetime` / `daterange` / `time` / `select` / `multiselect` / `radio` / `checkbox` / `switch` / `upload` / `image` / `richtext` / `cascader` / `treeselect` / `user` / `department` / `address` / `phone` / `email` / `idcard` / `url` / `color` / `rate` / `slider` / `custom`
- **列类型**: `text` / `number` / `money` / `date` / `datetime` / `tag` / `status` / `avatar` / `image` / `link` / `progress` / `action`
- **操作位置**: `toolbar` / `toolbar-left` / `toolbar-right` / `row` / `row-more` / `form-footer` / `card-header` / `card-footer`
- **操作行为**: `navigate` / `modal` / `drawer` / `action` / `download` / `print`
- fieldKey 使用 camelCase
- 字典 ID 使用 kebab-case
- 图标使用 lucide-react 图标名

---

## 一、产品概述

### 1.1 项目背景

销售每天通过钉钉闪击、线上会议跟客户聊，产生大量文字记录。这些内容散落在聊天窗口里，销售想找上次客户说了什么需求，得翻半天记录。团队需要一个工具，把这些非结构化内容自动提取成可查看、可跟进的客户档案。

### 1.2 产品目标

- 将非结构化会议内容转化为结构化客户档案，消除信息孤岛
- 通过 AI 自动解析，节省人工整理时间，提升跟进效率
- 统一汇总面板让销售团队随时掌握所有客户的跟进状态和待办事项
- 支持手动补充和修正解析内容，确保数据准确性
- Action 项支持勾选完成，帮助团队跟踪销售承诺的落地

### 1.3 目标用户

| 角色 | 描述 | 核心诉求 |
|------|------|----------|
| 销售人员 | 一线销售代表，负责客户拜访和跟进 | 快速记录会议内容，提取客户需求，不遗漏 Action |
| 售前工程师 | 配合销售提供技术支持 | 查看客户需求和跟进历史，准备技术交流材料 |
| 销售经理 | 团队管理者，关注整体销售进度 | 汇总视图，掌握团队所有客户的跟进状态和待办 |

### 1.4 范围定义

**本期包含**：
- 钉钉闪击/会议文字粘贴输入
- AI 自动解析：客户名称、需求、进度、Action、决策人、竞品
- 解析结果预览与确认写入
- followups.json 数据写入（自动）
- 首页汇总面板：客户卡片、Action 汇总、快捷入口
- 客户详情页：基本信息、需求清单、跟进历史（时间线）、Action 列表
- Action 完成状态勾选

**本期不含**：
- 多人协作编辑（单人使用场景）
- 数据导入/对接其他系统（仅支持粘贴输入）
- 权限管理和用户认证
- 数据分析和图表可视化
- 移动端适配
- 微信、企业微信等其他平台内容解析

---

## 二、信息架构

### 2.1 站点地图

```
首页 / 汇总面板 (/)
├── 解析新会议 (/parse)
└── 客户详情 (/customer/:id)
```

### 2.2 导航结构

| 一级菜单 | 二级菜单 | 路由 | 说明 |
|----------|----------|------|------|
| 首页 | - | / | 汇总面板，客户列表、Action 汇总、快捷入口 |
| 解析 | 解析新会议 | /parse | 粘贴会议内容并解析 |
| 客户 | 客户详情（动态） | /customer/:id | 查看单个客户的完整档案 |

---

## 三、功能模块

### 3.1 首页 / 汇总面板

> 销售人员打开应用后的默认视图，快速了解所有客户状态和待办事项。

**路由**：`/`
**布局**：`dashboard`
**描述**：展示所有客户的跟进状态概览、待办 Action 汇总，以及快速解析新会议的入口。

#### 客户列表（cards）

| 字段 | fieldKey | 列类型 | 可排序 | 说明 |
|------|----------|--------|--------|------|
| 公司名称 | companyName | link | 否 | 点击进入客户详情页 |
| 跟进阶段 | salesStage | tag | 是 | 当前所处销售阶段 |
| 需求数量 | needsCount | number | 是 | 该客户已提取的需求条数 |
| 最近跟进 | lastFollowupDate | date | 是 | 最近一次跟进的日期 |
| 待办 Action | pendingActions | number | 是 | 未完成的 Action 数量 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 解析新会议 | button | toolbar-right | navigate to /parse |
| 查看详情 | link | row | navigate to /customer/:id |

##### 业务规则

- 无客户数据时显示空状态引导："还没有客户记录，点击上方「解析新会议」开始"
- 卡片按最近跟进日期倒序排列
- pendingActions > 0 时显示红色角标

#### Action 汇总（table）

| 字段 | fieldKey | 列类型 | 可排序 | 说明 |
|------|----------|--------|--------|------|
| 序号 | id | number | 否 | 自动编号 |
| 事项 | action | text | 否 | Action 描述 |
| 客户 | companyName | link | 否 | 关联的客户名称，点击跳转详情 |
| 负责人 | assignee | text | 否 | 负责人姓名 |
| 截止日期 | deadline | date | 是 | 到期日，过期显示红色 |
| 状态 | done | status | 否 | 待完成 / 已完成 |
| 操作 | - | action | 否 | 勾选完成按钮 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 标记完成 | switch | row | action: 更新 followups.json 中对应 action 的 done 字段为 true 并写入 completedAt |

##### 业务规则

- 默认按截止日期升序排列（最紧急在前）
- 截止日期已过且未完成的 Action 高亮显示
- 已完成的 Action 显示删除线

#### 快捷入口（card）

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 解析新会议 | button | card-header | navigate to /parse |

---

### 3.2 解析页面

> 粘贴钉钉会议内容，AI 自动解析结构化数据，预览确认后写入跟进记录。

**路由**：`/parse`
**布局**：`custom`
**描述**：左右分栏布局，左侧粘贴原始会议内容，右侧展示 AI 解析结果，确认后一键写入。

#### 内容输入区（form）

| 字段 | fieldKey | 类型 | 必填 | 说明 |
|------|----------|------|------|------|
| 会议内容 | rawContent | textarea | 是 | 粘贴钉钉闪击或会议记录原文 |
| 跟进日期 | followupDate | date | 否 | 默认为当天，用户可手动修改 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 开始解析 | button | form-footer | action: 调用 AI 解析接口，右侧刷新预览区 |
| 清空 | button | form-footer | action: 清空输入区和预览区 |

##### 业务规则

- rawContent 少于 20 字符时禁用解析按钮
- 解析中显示 loading 状态，禁止重复提交

#### 解析结果预览区（card）

| 字段 | fieldKey | 类型 | 必填 | 说明 |
|------|----------|------|------|------|
| 客户公司 | companyName | text | 否 | AI 识别的客户公司名称，支持用户修改 |
| 客户需求 | needs | textarea | 否 | AI 提取的客户需求描述列表 |
| 跟进阶段 | salesStage | select | 否 | 选项来源：dict-sales-stage |
| 决策人 | decisionMaker | text | 否 | 拍板人姓名和职位 |
| 竞品 | competitor | text | 否 | 提及的竞品名称 |
| Action 列表 | actions | textarea | 否 | AI 提取的待办事项，格式为一行一条 |
| 沟通摘要 | content | textarea | 否 | 一句话提炼本次沟通要点 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 确认写入 | button | card-footer | action: 写入 followups.json，写入成功后 navigate to /customer/:newId |
| 取消 | button | card-footer | action: navigate to / |

##### 业务规则

- 解析完成前预览区显示 skeleton loading
- 用户可在预览区直接编辑各字段后再确认写入
- companyName 若为空，引导用户手动填写
- 写入成功后页面跳转到新创建的客户详情页

---

### 3.3 客户详情页

> 查看单个客户的完整档案，包括基本信息、需求清单、跟进历史和待办 Action。

**路由**：`/customer/:id`
**布局**：`detail`
**描述**：标签页切换展示客户信息、需求、跟进历史和 Action 列表。

#### 基本信息（description）

| 字段 | fieldKey | 类型 | 必填 | 说明 |
|------|----------|------|------|------|
| 公司名称 | companyName | text | 是 | 客户公司全称 |
| 跟进阶段 | salesStage | tag | 否 | 当前销售阶段 |
| 决策人 | decisionMaker | text | 否 | 拍板人姓名和职位 |
| 竞品 | competitor | text | 否 | 提及的竞品 |
| 创建时间 | createdAt | datetime | 否 | 首条跟进记录创建时间 |
| 最近跟进 | lastFollowupDate | date | 否 | 最近一次跟进日期 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 编辑档案 | button | card-header | drawer: 打开编辑表单 |
| 返回首页 | button | toolbar-left | navigate to / |

##### 业务规则

- 编辑表单预填充当前值，保存后覆盖原数据

#### 需求清单（table）

| 字段 | fieldKey | 列类型 | 可排序 | 说明 |
|------|----------|--------|--------|------|
| 序号 | id | number | 否 | 自动编号 |
| 需求描述 | need | text | 否 | 从历次会议解析提取的需求描述 |
| 来源 | sourceDate | date | 是 | 提取自哪次跟进记录 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 新增需求 | button | toolbar-right | modal: 新增需求表单 |
| 删除 | button | row-more | action: 删除该条需求 |

##### 业务规则

- 需求从所有关联的 followup 记录中聚合提取
- 删除仅从当前视图移除，不影响原始跟进记录

#### 跟进历史（timeline）

##### 业务规则

- 按时间倒序展示所有跟进记录
- 每条记录显示日期、类型、摘要和 Action 列表
- 点击单条可展开查看详情

#### Action 列表（table）

| 字段 | fieldKey | 列类型 | 可排序 | 说明 |
|------|----------|--------|--------|------|
| 序号 | id | number | 否 | 自动编号 |
| 事项 | action | text | 否 | Action 描述 |
| 负责人 | assignee | text | 否 | 负责人 |
| 截止日期 | deadline | date | 是 | 到期日 |
| 状态 | done | status | 否 | 待完成 / 已完成 |
| 操作 | - | action | 否 | 勾选完成开关 |

##### 操作

| 按钮 | 类型 | 位置 | 行为 |
|------|------|------|------|
| 标记完成 | switch | row | action: 更新 followups.json 中对应 action 的 done 和 completedAt |
| 新增 Action | button | toolbar-right | modal: 新增 Action 表单 |
| 删除 | button | row-more | action: 删除该条 Action |

---

## 四、全局规则

### 4.1 角色权限

| 角色 | 描述 | 模块权限 |
|------|------|----------|
| 销售人员 | 一线销售代表 | 首页（读写）、解析页面（读写）、客户详情（读写） |
| 售前工程师 | 配合技术支持 | 首页（读）、客户详情（读） |
| 销售经理 | 团队管理者 | 首页（读写）、解析页面（读写）、客户详情（读写） |

> 本期不做权限控制，假设所有用户为同一角色。

### 4.2 数据字典

#### dict-sales-stage

| 值 | 显示 | 颜色 |
|----|------|------|
| initial | 初访 | #6b7280 |
| needs-confirmed | 需求确认 | #3b82f6 |
| proposal | 方案 | #f59e0b |
| business | 商务 | #8b5cf6 |
| closed | 成交 | #22c55e |

#### dict-action-owner

| 值 | 显示 | 颜色 |
|----|------|------|
| sales | 销售 | #3b82f6 |
| presales | 售前 | #f59e0b |
| solution | 方案 | #8b5cf6 |
| customer | 客户 | #22c55e |

#### dict-followup-type

| 值 | 显示 | 颜色 |
|----|------|------|
| 拜访 | 拜访 | #3b82f6 |
| 电话 | 电话 | #f59e0b |
| 会议 | 会议 | #8b5cf6 |
| 线上沟通 | 线上沟通 | #06b6d4 |
| 其他 | 其他 | #6b7280 |

### 4.3 状态流转

#### Action 完成状态

| 当前状态 | 操作 | 目标状态 | 条件 |
|----------|------|----------|------|
| 待完成 | 标记完成 | 已完成 | 用户点击完成开关 |
| 已完成 | 取消完成 | 待完成 | 用户取消完成标记 |

---

## 附录

### A. 变更记录

| 版本 | 日期 | 变更内容 |
|------|------|----------|
| 1.0.0 | 2026-04-18 | 初始版本，从会议纪要智能解析与销售跟进系统方案导出 |
