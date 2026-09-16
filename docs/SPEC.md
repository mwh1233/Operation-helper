# SPEC.md：独立开发者代运营 Agent M1 MVP 产品规格

## 1. 文档定位

本文档从 `SDD-独立开发者代运营Agent.md` 提炼当前 M1 MVP 的产品规格，供开发、验收和范围控制使用。

优先级规则：

1. SDD 是最高产品依据。
2. 本文档用于把当前执行阶段的 M1 MVP 范围压缩成可执行规格。
3. 如本文档与 SDD 冲突，以 SDD 为准，并先更新本文档后再开发。

## 2. 产品定位

独立开发者代运营 Agent 是一个单运营者使用的内部 Web 工作台。它帮助运营者管理多个独立开发者客户，理解客户产品，生成运营策略和内容计划，生产多平台内容，经人工审核后交付给运营者发布，并在后续阶段追踪效果。

核心价值：

- 对客户：降低独立开发者做运营的时间成本。
- 对运营者：让一个人可以用标准流程服务多个客户。

## 3. 用户角色

### 运营者 Operator

唯一直接使用系统的人。

目标：

- 管理多个客户。
- 快速理解产品并形成运营策略。
- 批量生成内容。
- 审核并发布内容。
- 追踪效果并向客户汇报。

### 客户 Client

购买代运营服务的独立开发者或一人公司。

客户不直接登录系统。客户通过运营者交付的内容、报告和沟通了解运营结果。

## 4. M1 MVP 目标

M1 MVP 要验证一条最小可用运营工作流：

```text
创建客户
生成或编辑产品画像
生成或编辑运营策略
生成内容排期
生成内容
人工审核
复制或导出内容，用于人工发布
```

MVP 成功标准：

- 运营者可以完整管理至少 20 个客户。
- 一个客户可以完整走通从建档到可人工发布内容产出的流程。
- 所有内容导出或复制前都有人工审核状态。
- M1 不存储真实平台凭证，不接入真实平台发布。

## 5. M1 MVP 范围

### 5.1 包含

#### 客户管理

- 创建客户。
- 查看客户列表。
- 搜索客户名称和产品名称。
- 按客户状态筛选。
- 查看客户详情。
- 编辑客户和产品基础信息。
- 删除客户，删除前必须确认。
- 更新客户状态：`active`、`paused`、`ended`。

#### 产品理解

- 基于客户产品信息生成产品画像。
- 产品画像可人工编辑。
- 产品画像保留版本号。
- 产品画像至少包含：
  - 一句话定位。
  - 目标用户。
  - 核心卖点。
  - 内容主题。
  - 语气风格。
  - 差异化优势。
  - 关键词。

#### 运营策略

- 基于产品画像生成运营策略。
- 支持国内和海外市场策略。
- 策略可人工编辑。
- 策略至少包含：
  - 平台选择。
  - 平台优先级。
  - 内容形式。
  - 发布频率。
  - 内容矩阵。
  - 增长策略。
  - 里程碑。

#### 运营计划与排期

- 基于策略生成一周或一月排期。
- 第一版以列表视图为主。
- 排期项可编辑、删除、跳过。
- 排期项支持按客户、市场、日期、状态筛选。
- 排期项至少包含：
  - 日期和时间。
  - 平台。
  - 内容主题。
  - 内容形式。
  - 标题或角度提示。
  - 状态。

#### 内容生成

- 基于排期项生成平台适配内容。
- M1 只要求支持文字内容。
- 图文、视频和真实多模态生成放到 M3 优化。
- 支持手动编辑生成内容。
- 支持重新生成并保留版本历史。
- M1 可以预留“一鱼多吃”入口，但增强版放到 M3。
- 生成内容默认进入待审核状态。

#### 审核工作流

- 查看待审核内容队列。
- 按客户、平台、内容形式、时间范围筛选。
- 支持通过、编辑后通过、拒绝、延后。
- 支持批量审核。
- 拒绝必须记录原因。
- 审核通过后内容可以复制或导出，用于人工发布。

#### 人工发布准备

- 一键复制审核通过内容。
- 导出审核通过内容为 Markdown 或 JSON。
- M1 不要求管理平台凭证。
- M1 不要求发布记录。
- M1 不要求自动发布或半自动发布闭环。

### 5.2 不包含

- 用户注册、登录、付费和 SaaS 化。
- 客户自助门户。
- 多运营者协作和权限系统。
- 移动端。
- 自动克隆 Git 仓库并分析代码。
- 竞品自动分析。
- 所有平台全自动发布。
- 平台凭证管理。
- 真实平台发布。
- 发布记录。
- 效果数据录入和报表。
- 自动抓取所有平台数据。
- 真人出镜视频和复杂剪辑。
- 未审核内容自动发布。

## 6. 核心实体

M1 MVP 必需实体为 `Client`、`ProductProfile`、`Strategy`、`Plan`、`ScheduleItem`、`ContentItem`、`ContentVersion`。

`PlatformCredential`、`PublishRecord`、`ContentStats` 属于后续阶段实体：M4 引入凭证和发布记录，M5 引入效果数据。

### Client

客户与产品基础信息。

核心字段：

- `id`
- `name`
- `product_name`
- `product_description`
- `product_url`
- `github_url`
- `target_market`
- `status`
- `created_at`
- `updated_at`

### ProductProfile

客户产品画像。

核心字段：

- `client_id`
- `one_liner`
- `target_users`
- `core_selling_points`
- `content_themes`
- `tone_style`
- `differentiation`
- `keywords`
- `generated_at`
- `version`

### Strategy

客户运营策略。

核心字段：

- `client_id`
- `market`
- `platforms`
- `content_matrix`
- `posting_schedule`
- `growth_strategy`
- `milestones`
- `generated_at`
- `version`

### Plan 与 ScheduleItem

运营计划和排期项。

排期项核心字段：

- `client_id`
- `plan_id`
- `scheduled_date`
- `scheduled_time`
- `platform`
- `content_theme`
- `content_type`
- `title_hint`
- `status`
- `content_item_id`

### ContentItem 与 ContentVersion

生成内容和版本历史。

内容项核心字段：

- `client_id`
- `schedule_item_id`
- `platform`
- `content_type`
- `title`
- `body`
- `images`
- `video_url`
- `hashtags`
- `metadata`
- `generation_context`
- `status`
- `generated_at`
- `version`

### PlatformCredential（M4）

客户平台发布凭证，M1 不实现。

核心字段：

- `client_id`
- `platform`
- `auth_type`
- `credentials`
- `status`
- `last_verified`

### PublishRecord（M4）

发布记录，M1 不实现。

核心字段：

- `content_item_id`
- `client_id`
- `platform`
- `scheduled_at`
- `published_at`
- `status`
- `result_url`
- `error_message`
- `retry_count`

### ContentStats（M5）

内容效果数据，M1 不实现。

核心字段：

- `publish_record_id`
- `content_item_id`
- `client_id`
- `platform`
- `impressions`
- `clicks`
- `likes`
- `comments`
- `shares`
- `conversions`
- `recorded_at`
- `data_source`

## 7. 状态定义

### Client.status

- `active`：服务中。
- `paused`：暂停服务。
- `ended`：服务结束。

### ScheduleItem.status

- `pending`：待生成。
- `generated`：已生成，待审核。
- `reviewed`：已审核，可复制或导出。
- `published`：已发布，M4 发布阶段使用。
- `skipped`：已跳过。

### ContentItem.status

- `draft`：草稿。
- `pending_review`：待审核。
- `approved`：已审核。
- `rejected`：已拒绝。
- `published`：已发布，M4 发布阶段使用。

### PublishRecord.status（M4）

- `pending`：等待发布。
- `publishing`：发布中。
- `success`：发布成功。
- `failed`：发布失败。

### PlatformCredential.status（M4）

- `active`：可用。
- `expired`：过期。
- `invalid`：无效。

## 8. 页面规格

### 客户列表页

必须包含：

- 搜索框。
- 状态筛选。
- 客户表格。
- 创建客户入口。
- 查看、编辑、暂停或恢复、删除操作。

### 客户详情页

必须包含客户信息区和 Tab 导航：

- 产品画像。
- 运营策略。
- 内容计划。
- 内容审核。
- 发布记录（M4）。
- 效果数据（M5）。

### 产品画像页

必须包含：

- 一句话定位。
- 目标用户卡片。
- 核心卖点卡片。
- 内容主题卡片。
- 语气风格。
- 差异化优势。
- 关键词标签。
- 重新生成、编辑、导出 JSON 操作。

### 运营策略页

必须包含：

- 市场切换。
- 平台配置。
- 内容矩阵。
- 发布节奏。
- 增长策略。
- 里程碑。
- 重新生成、编辑、导出操作。

### 内容计划页

必须包含：

- 时间范围选择。
- 市场切换。
- 生成计划按钮。
- 按日期分组的排期列表。
- 排期项编辑、删除、跳过、立即生成内容。
- 批量生成内容。

### 审核队列页

必须包含：

- 客户、平台、内容形式、时间范围筛选。
- 待审核统计。
- 内容预览。
- 通过、编辑、拒绝、延后。
- 批量通过和批量拒绝。

### 发布记录页（M4）

必须包含：

- 客户、平台、状态、时间范围筛选。
- 发布记录列表。
- 发布链接。
- 错误详情。
- 失败重试。

### 效果数据页（M5）

必须包含：

- 已发布内容列表。
- 效果数据录入表单。
- 核心指标卡片。
- 平台对比。
- 内容主题效果。

## 9. 业务规则

- 内容生成后默认进入 `pending_review`。
- M1 中，只有 `approved` 内容可以复制或导出。
- M4 发布阶段，只有 `approved` 内容可以发布。
- 拒绝内容必须记录拒绝原因。
- M4 发布阶段，发布成功后必须记录 `result_url` 或明确说明无法获得链接。
- M4 发布阶段，发布失败必须记录 `error_message`。
- M4 发布阶段，平台凭证必须脱敏展示。
- 删除客户属于高风险操作，必须二次确认。
- M4 发布阶段，半自动发布也必须创建或更新发布记录。
- M5 数据阶段，效果数据以手动录入为准。

## 10. 非功能需求

- 本地运行页面加载应小于 2 秒。
- 非 AI API 响应应小于 500ms。
- 文字生成目标小于 30 秒。
- 图文生成目标小于 60 秒。
- 视频生成目标小于 5 分钟。
- 单运营者使用，支持至少 20 个客户。
- 数据库需要支持本地备份。
- 所有关键操作需要日志或 audit log。
- 外部 API 调用失败必须可见。

## 11. 验收口径

一个功能只有同时满足以下条件，才能标记完成：

- 与本文档和 SDD 范围一致。
- API、数据模型、前端类型同步。
- 成功路径和失败路径都有可见反馈。
- 必要测试已覆盖。
- 已执行与变更范围匹配的验证。
- 没有绕过人工审核和安全约束。
