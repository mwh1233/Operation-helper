# SDD 规格驱动开发文档：独立开发者代运营 Agent

| 字段 | 内容 |
|---|---|
| 文档版本 | v1.0 |
| 状态 | 草稿（待评审） |
| 创建日期 | 2026-09-15 |
| 作者 | 产品经理 Agent |
| 技术栈 | Python 后端 + Next.js 前端 + SQLite |
| 目标用户 | 独立开发者 / 一人公司 |
| 商业模式 | 代运营外包服务 |

---

## 1. 产品愿景与范围

### 1.1 产品愿景

为独立开发者提供"代理运营"服务：开发者专注写代码，我们的 Agent 系统深入理解其产品，制定定制化运营方案，按计划生成多模态内容（文字/图文/视频），经审核后自动发布到国内外主流平台，并追踪效果持续优化。

### 1.2 核心价值主张

- **对客户（独立开发者）：** "你提供产品代码和账号，我们帮你搞定运营。"
- **对运营者（我们）：** 一套工具让一个人能同时高效服务多个客户，内容生成效率提升 10 倍。

### 1.3 本期范围（MVP）

| 模块 | 包含 | 不包含 |
|---|---|---|
| 客户管理 | ✅ 客户增删改查、产品信息录入 | ❌ 客户自助门户、付费系统 |
| 产品理解引擎 | ✅ 基于输入信息生成产品运营画像 | ❌ 自动克隆 Git 仓库分析代码 |
| 运营策略生成 | ✅ 生成平台选择、内容矩阵、发布节奏 | ❌ 竞品自动分析 |
| 运营计划排期 | ✅ 内容日历、按周/月排期、手动调整 | ❌ 自动拖拽式日历（第一版用列表） |
| 多模态内容生成 | ✅ 文字、图文（AI 配图）、视频（AI 生成） | ❌ 真人出镜视频、复杂剪辑 |
| 审核工作流 | ✅ 内容列表、通过/修改/拒绝、批量审核 | ❌ 多人协作审核、评论功能 |
| 发布组件 | ✅ X/Twitter、掘金（API 优先）；其他平台半自动 | ❌ 所有平台全自动发布 |
| 数据追踪 | ✅ 手动录入效果数据 + 简单报表 | ❌ 自动抓取所有平台数据 |

### 1.4 非目标（本期明确不做）

1. **不做 SaaS 化**：不做用户注册、付费、自助使用，工具是内部运营效率工具
2. **不做全自动黑箱运营**：所有内容必须人工审核后才能发布
3. **不做高并发后端**：本地优先，单用户（运营者）使用，不需要分布式架构
4. **不做复杂的团队协作**：单运营者模式，不需要权限管理、多人协作
5. **不做移动端**：仅 Web 端桌面使用

---

## 2. 术语表

| 术语 | 定义 |
|---|---|
| 客户 (Client) | 购买代运营服务的独立开发者 |
| 运营者 (Operator) | 使用本工具执行代运营服务的人（即我们自己） |
| 产品画像 (Product Profile) | AI 生成的产品运营画像，包含定位、目标用户、内容主题等 |
| 运营策略 (Strategy) | 基于产品画像生成的平台选择、内容矩阵、发布节奏等方案 |
| 运营计划 (Plan) | 具体的内容排期表，按时间节点规划每条内容 |
| 内容项 (Content Item) | 一条待生成/已生成/待审核/已发布的内容 |
| 平台适配器 (Platform Adapter) | 封装单个平台的内容生成规范和发布能力的模块 |
| 多模态 (Multimodal) | 内容形式包括纯文字、图文（文字+AI生成图片）、视频 |
| 审核队列 (Review Queue) | 待审核内容的列表 |
| 发布队列 (Publish Queue) | 审核通过、等待定时发布的内容列表 |

---

## 3. 用户角色

### 3.1 运营者（Operator，核心用户）

- **身份：** 代运营服务执行者，即工具的主要使用者
- **目标：** 高效管理多个客户的运营工作，快速生成高质量内容，按时发布
- **核心操作：** 管理客户、配置产品信息、生成运营策略、查看运营计划、审核内容、触发发布、查看效果数据

### 3.2 客户（Client，间接用户）

- **身份：** 购买代运营服务的独立开发者
- **目标：** 产品获得曝光和用户增长，不需要自己做运营
- **交互方式：** 不直接使用工具，通过运营者交付的报告和内容了解运营情况
- **提供：** 产品信息、代码仓库、各平台账号（API Key 或 Cookie）

---

## 4. 核心用户故事

### 4.1 客户管理

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-001 | 作为运营者，我想创建一个新客户，录入产品名称、描述、官网、Git 仓库、目标市场（国内/国外/都要），以便开始为其提供代运营服务 | P0 |
| US-002 | 作为运营者，我想查看和编辑客户的产品信息，以便在产品迭代时更新运营基础 | P0 |
| US-003 | 作为运营者，我想查看客户列表，了解每个客户的运营状态（活跃/暂停/已结束），以便管理服务进度 | P0 |

### 4.2 产品理解引擎

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-004 | 作为运营者，我想基于客户的产品信息，让 AI 自动生成产品运营画像（定位、目标用户、核心卖点、内容主题、语气风格），以便为后续运营提供基础 | P0 |
| US-005 | 作为运营者，我想手动编辑和优化 AI 生成的产品画像，以便修正不准确的部分 | P0 |
| US-006 | 作为运营者，我想在产品信息更新后重新生成产品画像，以便保持运营基础的时效性 | P1 |

### 4.3 运营策略生成

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-007 | 作为运营者，我想基于产品画像，让 AI 生成定制化运营策略（平台选择、内容矩阵配比、发布节奏、增长策略），以便有明确的运营方向 | P0 |
| US-008 | 作为运营者，我想手动调整运营策略（如增减平台、调整内容配比），以便符合客户实际情况 | P0 |
| US-009 | 作为运营者，我想为国内和国外市场分别生成运营策略，以便不同市场有针对性的方案 | P0 |

### 4.4 运营计划与排期

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-010 | 作为运营者，我想基于运营策略，自动生成未来一周/一月的内容排期计划（哪天、哪个平台、什么主题、什么内容形式），以便有明确的内容生产节奏 | P0 |
| US-011 | 作为运营者，我想手动调整排期（修改日期、平台、主题、内容形式），以便灵活安排 | P0 |
| US-012 | 作为运营者，我想查看内容日历视图，直观了解每天的发布安排，以便把控整体节奏 | P1 |
| US-013 | 作为运营者，我想标记排期项为"已生成/已跳过"，以便跟踪内容生产进度 | P0 |

### 4.5 多模态内容生成

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-014 | 作为运营者，我想根据排期项，让 AI 自动生成对应平台的文字内容，以便快速产出内容 | P0 |
| US-015 | 作为运营者，我想为内容生成 AI 配图（图文形式），以便丰富内容形式 | P0 |
| US-016 | 作为运营者，我想为内容生成 AI 视频（视频形式），以便覆盖视频平台 | P1 |
| US-017 | 作为运营者，我想基于一个核心主题，一次性生成多个平台的内容版本（一鱼多吃），以便提高效率 | P0 |
| US-018 | 作为运营者，我想重新生成某条内容（换个风格/角度），以便获得更好的内容质量 | P0 |
| US-019 | 作为运营者，我想手动编辑 AI 生成的内容，以便修正和优化 | P0 |

### 4.6 审核工作流

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-020 | 作为运营者，我想查看待审核内容队列，了解有哪些内容需要审核 | P0 |
| US-021 | 作为运营者，我想对内容执行"通过/修改后通过/拒绝/延后"操作，以便控制发布质量 | P0 |
| US-022 | 作为运营者，我想批量审核内容（全选通过），以便提高效率 | P1 |
| US-023 | 作为运营者，我想在审核时直接编辑内容，以便一次性完成修改和通过 | P0 |

### 4.7 发布组件

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-024 | 作为运营者，我想配置客户的各平台发布凭证（API Key / Cookie / 账号密码），以便能够自动发布 | P0 |
| US-025 | 作为运营者，我想让审核通过的内容按排期时间自动发布到对应平台，以便实现定时发布 | P0 |
| US-026 | 作为运营者，我想手动触发立即发布，以便紧急内容能及时发出 | P0 |
| US-027 | 作为运营者，我想查看发布结果（成功/失败、发布链接、失败原因），以便追踪发布状态 | P0 |
| US-028 | 作为运营者，我想对发布失败的内容进行重试，以便处理临时故障 | P1 |
| US-029 | 作为运营者，我想为不支持 API 的平台使用"半自动发布"（生成内容后一键复制，打开平台发布页），以便覆盖更多平台 | P1 |

### 4.8 数据追踪与优化

| ID | 用户故事 | 优先级 |
|---|---|---|
| US-030 | 作为运营者，我想手动录入每条已发布内容的效果数据（曝光、点击、点赞、评论、转发、注册转化），以便追踪运营效果 | P1 |
| US-031 | 作为运营者，我想查看客户的运营效果报表（按平台/按时间/按内容主题），以便向客户交付报告 | P1 |
| US-032 | 作为运营者，我想让 AI 分析效果数据，给出运营策略优化建议，以便持续改进 | P2 |

---

## 5. 系统架构

### 5.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    Next.js 前端 (Web Dashboard)               │
│  客户管理 / 产品画像 / 运营策略 / 内容日历 / 审核 / 发布 / 报表 │
└──────────────────────────────┬──────────────────────────────┘
                               │ REST API
┌──────────────────────────────▼──────────────────────────────┐
│                    Python 后端 (FastAPI)                       │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ 客户管理    │  │ 产品理解    │  │ 策略生成    │            │
│  └────────────┘  └────────────┘  └────────────┘            │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │ 计划排期    │  │ 内容生成    │  │ 审核工作流  │            │
│  └────────────┘  └─────┬──────┘  └────────────┘            │
│                         │                                      │
│  ┌────────────┐  ┌─────▼──────┐  ┌────────────┐            │
│  │ 发布调度    │  │ 多模态引擎  │  │ 数据追踪    │            │
│  └─────┬──────┘  └────────────┘  └────────────┘            │
└────────┼─────────────────────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────────────────────────┐
│                    发布组件 (独立 CLI / 微服务)                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ X 适配器  │ │ 掘金适配器 │ │ 小红书    │ │ Reddit   │       │
│  │ (API)    │ │ (API)    │ │ (浏览器)  │ │ (API)    │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ V2EX     │ │ IndieHack│ │ 即刻      │ │ 更多...   │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
└──────────────────────────────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────────────────────────┐
│                    外部服务                                     │
│  LLM API (Doubao/OpenAI) / 图像生成 / 视频生成 / 平台 API    │
└──────────────────────────────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────────────────────────┐
│                    SQLite 数据库                                │
│  客户 / 产品画像 / 策略 / 计划 / 内容 / 发布记录 / 效果数据    │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 架构原则

1. **本地优先**：所有服务可在本地运行，不需要云服务器
2. **模块解耦**：内容生成、审核、发布完全解耦，通过数据库状态衔接
3. **平台适配器模式**：每个平台一个独立 adapter，统一接口，扩展新平台只加 adapter
4. **发布组件独立**：发布逻辑封装为独立 CLI/微服务，可单独运行和调试
5. **单用户设计**：不需要复杂的权限系统和并发控制

---

## 6. 功能模块详细规格

### 6.1 客户管理模块

#### 6.1.1 客户数据结构

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | INTEGER | 是 | 主键，自增 |
| name | TEXT | 是 | 客户名称（公司/个人名） |
| product_name | TEXT | 是 | 产品名称 |
| product_description | TEXT | 是 | 产品描述（一段文字） |
| product_url | TEXT | 否 | 产品官网 URL |
| github_url | TEXT | 否 | Git 仓库 URL |
| target_market | TEXT | 是 | 目标市场：`domestic`（国内）/ `international`（国外）/ `both`（都要） |
| status | TEXT | 是 | 状态：`active`（活跃）/ `paused`（暂停）/ `ended`（已结束） |
| created_at | DATETIME | 是 | 创建时间 |
| updated_at | DATETIME | 是 | 更新时间 |

#### 6.1.2 客户列表页

- **展示：** 表格形式，列包括：客户名称、产品名称、目标市场、状态、最近发布时间、操作
- **筛选：** 按状态筛选（全部/活跃/暂停/已结束）
- **搜索：** 按客户名称/产品名称搜索
- **操作：** 查看详情、编辑、暂停/恢复、删除（需确认）

#### 6.1.3 客户详情页

- **顶部：** 客户基本信息卡片，可编辑
- **Tab 导航：**
  - 产品画像
  - 运营策略
  - 内容计划
  - 内容审核
  - 发布记录
  - 效果数据
- **每个 Tab 对应一个功能模块的视图**

---

### 6.2 产品理解引擎

#### 6.2.1 产品画像数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| client_id | INTEGER | 关联客户 |
| one_liner | TEXT | 一句话定位（不超过 50 字） |
| target_users | TEXT | 目标用户画像（JSON 数组，每个包含：人群描述、核心需求、使用场景） |
| core_selling_points | TEXT | 核心卖点（JSON 数组，3-5 条） |
| content_themes | TEXT | 内容主题（JSON 数组，每个包含：主题名、描述、示例角度） |
| tone_style | TEXT | 语气风格（JSON：正式/轻松/技术/故事化，以及具体描述） |
| differentiation | TEXT | 差异化优势（与竞品的区别） |
| keywords | TEXT | 核心关键词（JSON 数组，用于内容生成和社区监控） |
| generated_at | DATETIME | 生成时间 |
| version | INTEGER | 版本号，每次重新生成 +1 |

#### 6.2.2 生成流程

1. **输入：** 客户的产品名称、描述、官网、Git 仓库、目标市场
2. **Prompt 构建：** 将产品信息组装为结构化 prompt，要求 AI 输出指定格式的 JSON
3. **LLM 调用：** 调用 Doubao/OpenAI API，temperature=0.7
4. **输出解析：** 解析返回的 JSON，验证必填字段
5. **存储：** 保存产品画像到数据库，version +1
6. **展示：** 在前端以结构化卡片形式展示，每个字段可编辑

#### 6.2.3 产品画像展示页

- **顶部：** 一句话定位（大字号展示）
- **卡片网格：**
  - 目标用户卡片（列表展示）
  - 核心卖点卡片（列表展示）
  - 内容主题卡片（列表展示，每个主题可点击展开示例角度）
  - 语气风格卡片
  - 差异化优势卡片
  - 关键词卡片（标签形式）
- **操作按钮：** 重新生成、编辑、导出为 JSON

---

### 6.3 运营策略生成器

#### 6.3.1 运营策略数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| client_id | INTEGER | 关联客户 |
| market | TEXT | 市场：`domestic` / `international`（每个市场一份策略） |
| platforms | TEXT | 平台配置（JSON 数组，每个包含：platform_name、priority（primary/secondary）、content_types、posting_frequency） |
| content_matrix | TEXT | 内容矩阵（JSON：各内容类型占比，如 product_update: 40%, industry_view: 20%, tutorial: 20%, engagement: 20%） |
| posting_schedule | TEXT | 发布节奏（JSON：每个平台的每周发布天数、最佳发布时间） |
| growth_strategy | TEXT | 增长策略（JSON 数组，每条包含：策略名、描述、执行步骤） |
| milestones | TEXT | 里程碑规划（JSON 数组，每条包含：时间节点、目标、关键动作） |
| generated_at | DATETIME | 生成时间 |
| version | INTEGER | 版本号 |

#### 6.3.2 平台配置说明

**国内平台候选：** 掘金、V2EX、小红书、即刻、知乎、B站、微信公众号

**国外平台候选：** X/Twitter、IndieHackers、Reddit、Hacker News、Product Hunt、LinkedIn、Dev.to、Medium

**每个平台配置包含：**
- `platform_name`：平台名称
- `priority`：`primary`（主力平台，每周 3+ 篇）/ `secondary`（次要平台，每周 1-2 篇）
- `content_types`：适合的内容形式数组（`text` / `image_text` / `video`）
- `posting_frequency`：每周发布次数
- `best_times`：最佳发布时间数组（如 `["09:00", "12:00", "20:00"]`）

#### 6.3.3 运营策略展示页

- **市场切换：** Tab 切换国内/国外策略
- **平台配置卡片：** 展示每个平台的优先级、内容形式、发布频率，可编辑
- **内容矩阵：** 饼图展示各内容类型占比
- **发布节奏：** 周历视图展示每个平台的发布安排
- **增长策略：** 列表展示，可展开查看执行步骤
- **里程碑：** 时间线展示
- **操作：** 重新生成、编辑、导出

---

### 6.4 运营计划与排期

#### 6.4.1 排期项数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| client_id | INTEGER | 关联客户 |
| plan_id | INTEGER | 关联运营计划（每周/每月一个 plan） |
| scheduled_date | DATE | 计划发布日期 |
| scheduled_time | TIME | 计划发布时间 |
| platform | TEXT | 目标平台 |
| content_theme | TEXT | 内容主题（来自产品画像的 content_themes） |
| content_type | TEXT | 内容形式：`text` / `image_text` / `video` |
| title_hint | TEXT | 内容标题提示/角度说明 |
| status | TEXT | 状态：`pending`（待生成）/ `generated`（已生成待审核）/ `reviewed`（已审核待发布）/ `published`（已发布）/ `skipped`（已跳过） |
| content_item_id | INTEGER | 关联的内容项 ID（生成后填充） |
| created_at | DATETIME | 创建时间 |
| updated_at | DATETIME | 更新时间 |

#### 6.4.2 运营计划生成逻辑

1. **输入：** 客户 ID、市场（国内/国外）、时间范围（开始日期、结束日期）
2. **读取：** 该市场的运营策略（平台配置、内容矩阵、发布节奏）
3. **算法：**
   - 遍历时间范围内的每一天
   - 对每个平台，根据发布节奏判断当天是否需要发布
   - 如果需要，根据内容矩阵权重随机选择一个内容类型
   - 从产品画像的 content_themes 中选择一个主题（轮询或随机）
   - 生成 title_hint（AI 生成一个内容角度提示）
   - 选择最佳发布时间
4. **输出：** 排期项列表，保存到数据库
5. **展示：** 按日期分组的列表视图，或日历视图

#### 6.4.3 内容计划页

- **顶部：** 时间范围选择器（本周/下周/本月/自定义）、市场切换、生成计划按钮
- **列表视图（默认）：** 按日期分组，每天下列出该天的所有排期项，每个排期项显示：时间、平台、主题、内容形式、状态、操作
- **日历视图（可选）：** 月历格子，每个格子显示当天的排期数量和平台标签
- **排期项操作：** 编辑、立即生成内容、标记跳过、删除
- **批量操作：** 批量生成选中排期的内容

---

### 6.5 多模态内容生成引擎

#### 6.5.1 内容项数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| client_id | INTEGER | 关联客户 |
| schedule_item_id | INTEGER | 关联排期项 |
| platform | TEXT | 目标平台 |
| content_type | TEXT | 内容形式：`text` / `image_text` / `video` |
| title | TEXT | 内容标题（部分平台需要） |
| body | TEXT | 内容正文（Markdown 格式） |
| images | TEXT | 图片 URL 列表（JSON 数组，image_text 和 video 类型使用） |
| video_url | TEXT | 视频 URL（video 类型使用） |
| hashtags | TEXT | 标签列表（JSON 数组） |
| metadata | TEXT | 其他元数据（JSON，如平台特定字段） |
| generation_context | TEXT | 生成上下文（JSON：使用的 prompt、产品画像版本、策略版本、参考内容等） |
| status | TEXT | 状态：`draft`（草稿）/ `pending_review`（待审核）/ `approved`（已审核）/ `rejected`（已拒绝）/ `published`（已发布） |
| generated_at | DATETIME | 生成时间 |
| version | INTEGER | 版本号（每次重新生成 +1） |

#### 6.5.2 内容生成流程

```
排期项触发生成
    ↓
读取产品画像 + 运营策略 + 平台规范
    ↓
构建生成 Prompt（包含：产品信息、目标用户、内容主题、平台调性、字数限制、格式要求）
    ↓
调用 LLM 生成文字内容
    ↓
┌─────────────┬──────────────┬──────────────┐
│  text 类型   │ image_text   │   video      │
│  直接完成     │  类型         │   类型        │
│              │              │              │
│              │ 调用图像生成  │  生成视频脚本 │
│              │ API 生成配图  │  + 调用视频   │
│              │              │  生成 API     │
└─────────────┴──────────────┴──────────────┘
    ↓
组装完整内容项（文字 + 图片/视频 + 标签）
    ↓
保存到数据库，status = pending_review
    ↓
进入审核队列
```

#### 6.5.3 平台内容规范（每个平台适配器提供）

| 平台 | 字数限制 | 内容形式 | 格式要求 | 标签规范 |
|---|---|---|---|---|
| X/Twitter | 280 字符（长推 25000） | text, image_text, video | 短平快、钩子开头 | #标签，2-3 个 |
| 掘金 | 无严格限制 | text, image_text | 技术深度、代码块、结构化标题 | 话题标签，1-2 个 |
| V2EX | 无严格限制 | text | 技术讨论、真诚分享、避免硬广 | 节点选择 |
| 小红书 | 标题 20 字，正文 1000 字 | image_text, video | 故事化标题、emoji 分段、个人视角 | #标签，5-10 个 |
| 即刻 | 无严格限制 | text, image_text | 轻松、动态、个人化 | 话题标签 |
| IndieHackers | 无严格限制 | text, image_text | 长帖、故事化、数据透明 | 无标签 |
| Reddit | 标题 300 字符，正文无限制 | text, image_text | subreddit 规则、避免硬广 | subreddit 选择 |
| Hacker News | 标题 80 字符 | text（链接或文字） | 技术深度、有讨论价值 | 无标签 |

#### 6.5.4 图像生成规格

- **调用方式：** 调用图像生成 API（如 Doubao 图像生成、Seedream）
- **输入：** 从文字内容中提取的画面描述 + 平台风格要求
- **输出：** 1-4 张图片 URL
- **尺寸：** 根据平台要求（小红书 3:4，X 16:9 或 1:1，掘金 16:9）
- **风格：** 与产品调性一致（科技感/简约/插画/实拍风）

#### 6.5.5 视频生成规格

- **调用方式：** 调用视频生成 API（如 Doubao 视频生成、Seedance）
- **输入：** 视频脚本（从文字内容转换）+ 画面描述 + 风格要求
- **输出：** 视频 URL
- **时长：** 15-60 秒（短视频平台）
- **格式：** MP4，竖版 9:16（小红书/抖音）或横版 16:9（X/B站）

#### 6.5.6 "一鱼多吃"功能

- **输入：** 一个核心主题 + 目标平台列表
- **流程：**
  1. 生成核心内容（长文版本）
  2. 对每个目标平台，基于核心内容生成适配版本
  3. 同时生成多个内容项，关联到同一个主题
- **输出：** 多个平台的内容项，批量进入审核队列

---

### 6.6 审核工作流

#### 6.6.1 审核队列页

- **顶部：** 筛选器（客户、平台、内容形式、时间范围）、统计卡片（待审核数量、今日已审核、通过率）
- **列表：** 待审核内容卡片，每个卡片显示：
  - 客户名称、平台、内容形式、计划发布时间
  - 内容预览（标题 + 正文摘要 + 图片/视频缩略图）
  - 操作按钮：通过、编辑、拒绝、延后
- **批量操作：** 全选、批量通过、批量拒绝

#### 6.6.2 内容详情/编辑页

- **左侧：** 内容编辑区（标题、正文、图片/视频、标签），支持 Markdown 编辑
- **右侧：** 内容信息（关联客户、排期项、生成上下文、版本历史）
- **底部：** 操作按钮（保存修改、审核通过、拒绝、重新生成）

#### 6.6.3 审核状态流转

```
pending_review（待审核）
    ├── 审核通过 → approved（已审核，进入发布队列）
    ├── 编辑后通过 → approved（保存修改后进入发布队列）
    ├── 拒绝 → rejected（记录拒绝原因，可重新生成）
    └── 延后 → pending_review（保持待审核，排期延后）
```

---

### 6.7 发布组件

#### 6.7.1 发布凭证数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| client_id | INTEGER | 关联客户 |
| platform | TEXT | 平台名称 |
| auth_type | TEXT | 认证方式：`api_key` / `oauth` / `cookie` / `password` |
| credentials | TEXT | 凭证内容（加密存储的 JSON，如 api_key、api_secret、cookie 等） |
| status | TEXT | 状态：`active` / `expired` / `invalid` |
| last_verified | DATETIME | 最近验证时间 |
| created_at | DATETIME | 创建时间 |

#### 6.7.2 发布记录数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| content_item_id | INTEGER | 关联内容项 |
| client_id | INTEGER | 关联客户 |
| platform | TEXT | 发布平台 |
| scheduled_at | DATETIME | 计划发布时间 |
| published_at | DATETIME | 实际发布时间 |
| status | TEXT | 状态：`pending`（等待发布）/ `publishing`（发布中）/ `success`（成功）/ `failed`（失败） |
| result_url | TEXT | 发布成功后的内容 URL |
| error_message | TEXT | 失败原因 |
| retry_count | INTEGER | 重试次数 |
| created_at | DATETIME | 创建时间 |

#### 6.7.3 平台适配器接口规范

每个平台适配器必须实现以下接口：

```python
class PlatformAdapter(ABC):
    @abstractmethod
    def get_platform_name(self) -> str:
        """返回平台名称"""
        pass

    @abstractmethod
    def get_content_spec(self) -> ContentSpec:
        """返回平台内容规范（字数限制、支持的内容形式、格式要求等）"""
        pass

    @abstractmethod
    def validate_credentials(self, credentials: dict) -> bool:
        """验证发布凭证是否有效"""
        pass

    @abstractmethod
    def publish(self, content: ContentItem, credentials: dict) -> PublishResult:
        """发布内容，返回发布结果（成功/失败、URL、错误信息）"""
        pass

    @abstractmethod
    def fetch_stats(self, content_url: str, credentials: dict) -> ContentStats:
        """抓取内容效果数据（可选实现）"""
        pass
```

#### 6.7.4 发布调度逻辑

1. **定时任务：** 后台调度器每分钟检查一次发布队列
2. **筛选：** 找出 `status = approved` 且 `scheduled_at <= now` 的内容项
3. **发布：** 对每个内容项：
   - 查找对应客户的平台凭证
   - 加载对应平台的适配器
   - 调用 `adapter.publish(content, credentials)`
   - 记录发布结果
   - 成功：更新内容项状态为 `published`，记录 result_url
   - 失败：更新状态为 `failed`，记录 error_message，retry_count +1
4. **重试：** 失败的内容在 5 分钟后自动重试，最多重试 3 次
5. **通知：** 发布失败时在前端显示告警（第一版不做邮件/短信通知）

#### 6.7.5 半自动发布模式

对于不支持 API 的平台（如小红书、即刻）：
- 生成内容后，提供"一键复制内容"按钮
- 提供"打开发布页面"按钮（在浏览器中打开对应平台的发布页）
- 运营者手动粘贴内容并发布
- 发布后手动填写发布 URL，标记为已发布

#### 6.7.6 发布记录页

- **筛选：** 客户、平台、状态、时间范围
- **列表：** 每条发布记录显示：客户、平台、内容标题、计划时间、实际时间、状态、操作
- **状态标签：** 成功（绿色）、失败（红色）、等待中（灰色）、发布中（蓝色）
- **操作：** 查看内容、查看发布链接、重试（失败的）、查看错误详情

---

### 6.8 数据追踪与优化

#### 6.8.1 效果数据结构

| 字段 | 类型 | 说明 |
|---|---|---|
| id | INTEGER | 主键 |
| publish_record_id | INTEGER | 关联发布记录 |
| content_item_id | INTEGER | 关联内容项 |
| client_id | INTEGER | 关联客户 |
| platform | TEXT | 平台 |
| impressions | INTEGER | 曝光量 |
| clicks | INTEGER | 点击量 |
| likes | INTEGER | 点赞数 |
| comments | INTEGER | 评论数 |
| shares | INTEGER | 转发/分享数 |
| conversions | INTEGER | 转化数（注册/下载） |
| recorded_at | DATETIME | 数据记录时间 |
| data_source | TEXT | 数据来源：`manual`（手动录入）/ `auto`（自动抓取） |

#### 6.8.2 效果数据录入页

- **列表：** 已发布内容列表，每条显示：平台、内容标题、发布时间、已录入的效果数据
- **录入：** 点击"录入数据"打开表单，填写各指标数值
- **批量录入：** 支持 CSV 导入
- **趋势：** 每个指标显示最近 7 天的趋势小图

#### 6.8.3 运营报表页

- **时间范围选择：** 本周/本月/自定义
- **客户切换：** 查看单个客户或全部客户
- **核心指标卡片：** 总曝光、总点击、总互动、总转化、平均互动率
- **平台对比：** 柱状图展示各平台的效果对比
- **内容主题效果：** 表格展示各内容主题的平均效果
- **趋势图：** 折线图展示每日/每周效果趋势
- **导出：** 导出为 PDF/Excel，用于向客户交付报告

---

## 7. 数据模型总览

### 7.1 实体关系图

```
Client (客户)
  │
  ├── ProductProfile (产品画像) 1:1
  │
  ├── Strategy (运营策略) 1:N（每个市场一份）
  │
  ├── Plan (运营计划) 1:N（每周/每月一份）
  │     │
  │     └── ScheduleItem (排期项) 1:N
  │           │
  │           └── ContentItem (内容项) 1:1
  │                 │
  │                 ├── PublishRecord (发布记录) 1:1
  │                 │     │
  │                 │     └── ContentStats (效果数据) 1:1
  │                 │
  │                 └── ContentVersion (内容版本历史) 1:N
  │
  ├── PlatformCredential (平台凭证) 1:N
  │
  └── PlatformAccount (平台账号信息) 1:N
```

### 7.2 数据库表清单

| 表名 | 说明 | 核心字段 |
|---|---|---|
| clients | 客户 | id, name, product_name, product_description, product_url, github_url, target_market, status, created_at, updated_at |
| product_profiles | 产品画像 | id, client_id, one_liner, target_users, core_selling_points, content_themes, tone_style, differentiation, keywords, generated_at, version |
| strategies | 运营策略 | id, client_id, market, platforms, content_matrix, posting_schedule, growth_strategy, milestones, generated_at, version |
| plans | 运营计划 | id, client_id, market, start_date, end_date, status, created_at |
| schedule_items | 排期项 | id, client_id, plan_id, scheduled_date, scheduled_time, platform, content_theme, content_type, title_hint, status, content_item_id, created_at, updated_at |
| content_items | 内容项 | id, client_id, schedule_item_id, platform, content_type, title, body, images, video_url, hashtags, metadata, generation_context, status, generated_at, version |
| content_versions | 内容版本历史 | id, content_item_id, version, title, body, images, video_url, created_at |
| publish_records | 发布记录 | id, content_item_id, client_id, platform, scheduled_at, published_at, status, result_url, error_message, retry_count, created_at |
| platform_credentials | 平台凭证 | id, client_id, platform, auth_type, credentials, status, last_verified, created_at |
| content_stats | 效果数据 | id, publish_record_id, content_item_id, client_id, platform, impressions, clicks, likes, comments, shares, conversions, recorded_at, data_source |
| audit_logs | 操作日志 | id, operator, action, target_type, target_id, details, created_at |

---

## 8. API 设计

### 8.1 API 基础信息

- **Base URL：** `http://localhost:8000/api/v1`
- **认证：** 第一版无认证（本地单用户），后续可加 API Key
- **请求/响应格式：** JSON
- **分页：** `?page=1&page_size=20`

### 8.2 客户管理 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/clients` | 获取客户列表（支持搜索、筛选、分页） |
| GET | `/clients/{id}` | 获取客户详情 |
| POST | `/clients` | 创建客户 |
| PUT | `/clients/{id}` | 更新客户信息 |
| PATCH | `/clients/{id}/status` | 更新客户状态（active/paused/ended） |
| DELETE | `/clients/{id}` | 删除客户（需确认，级联删除关联数据） |

### 8.3 产品画像 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/clients/{id}/product-profile` | 获取客户最新产品画像 |
| POST | `/clients/{id}/product-profile/generate` | 重新生成产品画像 |
| PUT | `/clients/{id}/product-profile` | 手动更新产品画像 |
| GET | `/clients/{id}/product-profile/history` | 获取产品画像版本历史 |

### 8.4 运营策略 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/clients/{id}/strategies` | 获取客户所有策略（国内/国外） |
| GET | `/clients/{id}/strategies/{market}` | 获取指定市场的策略 |
| POST | `/clients/{id}/strategies/{market}/generate` | 生成指定市场的运营策略 |
| PUT | `/clients/{id}/strategies/{market}` | 手动更新运营策略 |

### 8.5 运营计划 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/clients/{id}/plans` | 获取运营计划列表 |
| POST | `/clients/{id}/plans/generate` | 生成运营计划（参数：market, start_date, end_date） |
| GET | `/plans/{id}` | 获取计划详情（含排期项） |
| GET | `/schedule-items` | 获取排期项列表（支持按客户、日期范围、状态筛选） |
| PUT | `/schedule-items/{id}` | 更新排期项 |
| PATCH | `/schedule-items/{id}/status` | 更新排期项状态 |
| DELETE | `/schedule-items/{id}` | 删除排期项 |

### 8.6 内容生成 API

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `/schedule-items/{id}/generate-content` | 为排期项生成内容 |
| POST | `/contents/batch-generate` | 批量生成内容（排期项 ID 列表） |
| POST | `/contents/one-fish-many` | 一鱼多吃（主题 + 平台列表） |
| GET | `/contents/{id}` | 获取内容详情 |
| PUT | `/contents/{id}` | 更新内容（手动编辑） |
| POST | `/contents/{id}/regenerate` | 重新生成内容 |
| GET | `/contents/{id}/versions` | 获取内容版本历史 |
| POST | `/contents/{id}/generate-image` | 为内容生成配图 |
| POST | `/contents/{id}/generate-video` | 为内容生成视频 |

### 8.7 审核工作流 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/contents/review-queue` | 获取待审核内容队列 |
| POST | `/contents/{id}/approve` | 审核通过 |
| POST | `/contents/{id}/reject` | 拒绝（参数：reason） |
| POST | `/contents/{id}/defer` | 延后 |
| POST | `/contents/batch-approve` | 批量通过 |
| POST | `/contents/batch-reject` | 批量拒绝 |

### 8.8 发布组件 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/clients/{id}/credentials` | 获取客户所有平台凭证 |
| POST | `/clients/{id}/credentials` | 添加平台凭证 |
| PUT | `/clients/{id}/credentials/{platform}` | 更新平台凭证 |
| DELETE | `/clients/{id}/credentials/{platform}` | 删除平台凭证 |
| POST | `/credentials/{id}/verify` | 验证凭证有效性 |
| GET | `/publish-records` | 获取发布记录列表 |
| POST | `/contents/{id}/publish-now` | 立即发布 |
| POST | `/publish-records/{id}/retry` | 重试发布 |
| GET | `/publish-records/{id}` | 获取发布记录详情 |

### 8.9 数据追踪 API

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/contents/{id}/stats` | 获取内容效果数据 |
| POST | `/publish-records/{id}/stats` | 录入效果数据 |
| PUT | `/content-stats/{id}` | 更新效果数据 |
| GET | `/clients/{id}/reports` | 获取客户运营报表（参数：start_date, end_date） |
| GET | `/clients/{id}/reports/export` | 导出报表 |

---

## 9. 多模态内容生成规格

### 9.1 内容类型定义

| 类型 | 说明 | 组成部分 | 适用平台 |
|---|---|---|---|
| `text` | 纯文字内容 | 标题（可选）、正文、标签 | X、掘金、V2EX、IndieHackers、Reddit、HN、即刻 |
| `image_text` | 图文内容 | 标题、正文、1-4 张图片、标签 | 小红书、X、掘金、即刻、LinkedIn |
| `video` | 视频内容 | 标题、视频文件、描述文字、标签 | 小红书、X、B站、抖音、YouTube |

### 9.2 文字生成 Prompt 模板

```
你是一位专业的独立开发者运营专家，正在为以下产品生成 {platform} 平台的内容。

【产品信息】
产品名称：{product_name}
一句话定位：{one_liner}
目标用户：{target_users}
核心卖点：{core_selling_points}
语气风格：{tone_style}

【内容要求】
内容主题：{content_theme}
内容角度：{title_hint}
内容形式：{content_type}
字数限制：{word_limit}
平台调性：{platform_tone}

【输出格式】
请输出 JSON 格式，包含以下字段：
- title: 内容标题（如平台需要）
- body: 内容正文（Markdown 格式）
- hashtags: 标签数组（2-5 个）
- suggested_image_prompt: 配图建议（如需要图片）
- suggested_video_script: 视频脚本（如需要视频）

请确保内容：
1. 符合 {platform} 平台的调性和规范
2. 自然融入产品信息，不生硬广告
3. 有明确的钩子开头，吸引读者继续阅读
4. 结尾有明确的行动号召（CTA）
```

### 9.3 图像生成 Prompt 模板

```
为以下内容生成一张配图：

【内容主题】{content_theme}
【内容摘要】{content_summary}
【产品调性】{tone_style}
【平台要求】
- 平台：{platform}
- 尺寸：{aspect_ratio}
- 风格：{style}

【画面描述】
{suggested_image_prompt}

请生成符合以上要求的图片，确保：
1. 视觉风格与产品调性一致
2. 文字清晰可读（如有）
3. 构图适合 {platform} 平台的展示方式
```

### 9.4 视频生成 Prompt 模板

```
为以下内容生成一个短视频：

【内容主题】{content_theme}
【视频脚本】{suggested_video_script}
【产品调性】{tone_style}
【平台要求】
- 平台：{platform}
- 时长：{duration}秒
- 画幅：{aspect_ratio}
- 风格：{style}

请生成符合以上要求的视频，确保：
1. 画面与脚本内容匹配
2. 节奏紧凑，前 3 秒有钩子
3. 视觉风格与产品调性一致
4. 适合 {platform} 平台的短视频消费场景
```

---

## 10. 非功能需求

### 10.1 性能需求

| 指标 | 要求 |
|---|---|
| 页面加载时间 | < 2 秒（本地运行） |
| API 响应时间 | < 500ms（不含 LLM 调用） |
| 内容生成时间 | 文字 < 30 秒，图文 < 60 秒，视频 < 5 分钟 |
| 并发用户数 | 1（单运营者使用） |
| 同时管理客户数 | 支持至少 20 个客户 |

### 10.2 可靠性需求

- 内容生成失败时自动重试 1 次，仍失败则记录错误并通知
- 发布失败时自动重试 3 次，间隔 5 分钟
- 数据库每日自动备份（本地文件）
- 所有操作记录 audit log，可追溯

### 10.3 安全性需求

- 平台凭证（API Key、Cookie、密码）加密存储
- 不向第三方泄露客户产品信息和运营数据
- 发布操作必须经过人工审核，不允许全自动发布
- 本地运行，不暴露公网端口

### 10.4 可维护性需求

- 代码模块化，每个功能模块独立
- 平台适配器插件化，新增平台不需要修改核心代码
- 配置文件化，LLM API Key、平台配置等可配置
- 完整的日志记录，便于排查问题

### 10.5 可扩展性需求

- 后续可支持多运营者协作（加权限系统）
- 后续可支持客户自助门户（加用户系统）
- 后续可支持 SaaS 化（加多租户、付费系统）
- 后续可支持更多平台（加适配器）

---

## 11. 里程碑与迭代计划

### 11.1 总体节奏

考虑到每天 1-2 小时、周六 8 小时的投入，以及质量优先的原则，建议按以下节奏迭代：

| 阶段 | 时间 | 目标 | 核心交付 |
|---|---|---|---|
| **M1：基础框架** | 第 1-2 周 | 搭建项目骨架，实现客户管理和产品画像 | 可运行的 Web 应用，客户 CRUD，产品画像生成 |
| **M2：策略与计划** | 第 3-4 周 | 实现运营策略生成和内容排期 | 运营策略生成，内容计划生成与管理 |
| **M3：内容生成** | 第 5-7 周 | 实现多模态内容生成引擎 | 文字/图文/视频生成，一鱼多吃，内容编辑 |
| **M4：审核与发布** | 第 8-10 周 | 实现审核工作流和发布组件 | 审核队列，X/掘金自动发布，发布记录 |
| **M5：数据与优化** | 第 11-12 周 | 实现数据追踪和运营报表 | 效果数据录入，运营报表，导出 |
| **M6：打磨与验证** | 第 13-14 周 | 整体打磨，用自己的产品验证 | Bug 修复，体验优化，真实运营验证 |

### 11.2 M1：基础框架（第 1-2 周）

**目标：** 搭建项目骨架，实现客户管理和产品画像

**任务清单：**
- [ ] 初始化 Python 后端项目（FastAPI + SQLAlchemy + SQLite）
- [ ] 初始化 Next.js 前端项目（TypeScript + Tailwind CSS）
- [ ] 设计并创建数据库表（clients, product_profiles）
- [ ] 实现客户管理 API（CRUD）
- [ ] 实现客户管理前端页面（列表、详情、创建、编辑）
- [ ] 集成 LLM API（Doubao/OpenAI）
- [ ] 实现产品画像生成 API
- [ ] 实现产品画像展示和编辑页面
- [ ] 基础布局和导航（侧边栏 + 顶部栏）

**验收标准：**
- 可以创建、编辑、删除客户
- 可以基于客户信息生成产品画像
- 可以查看和编辑产品画像
- 界面基本可用，导航清晰

### 11.3 M2：策略与计划（第 3-4 周）

**目标：** 实现运营策略生成和内容排期

**任务清单：**
- [ ] 创建 strategies, plans, schedule_items 表
- [ ] 实现运营策略生成 API（国内/国外分别生成）
- [ ] 实现运营策略展示和编辑页面
- [ ] 实现运营计划生成 API（基于策略生成排期）
- [ ] 实现内容计划页面（列表视图，按日期分组）
- [ ] 实现排期项的编辑、删除、状态更新
- [ ] 实现市场切换（国内/国外）功能

**验收标准：**
- 可以为客户生成国内/国外运营策略
- 可以查看和编辑运营策略
- 可以基于策略生成内容排期计划
- 可以查看和调整排期

### 11.4 M3：内容生成（第 5-7 周）

**目标：** 实现多模态内容生成引擎

**任务清单：**
- [ ] 创建 content_items, content_versions 表
- [ ] 实现文字内容生成 API（基于排期项）
- [ ] 实现平台内容规范配置（每个平台的字数限制、格式要求）
- [ ] 集成图像生成 API
- [ ] 实现图文内容生成（文字 + AI 配图）
- [ ] 集成视频生成 API
- [ ] 实现视频内容生成（脚本 + AI 视频）
- [ ] 实现"一鱼多吃"功能（一个主题多平台版本）
- [ ] 实现内容编辑页面（Markdown 编辑器）
- [ ] 实现内容重新生成、版本历史
- [ ] 实现内容预览（模拟各平台展示效果）

**验收标准：**
- 可以为排期项生成文字内容
- 可以生成图文内容（含 AI 配图）
- 可以生成视频内容（含 AI 视频）
- 可以一个主题生成多个平台版本
- 可以编辑和重新生成内容
- 可以查看内容版本历史

### 11.5 M4：审核与发布（第 8-10 周）

**目标：** 实现审核工作流和发布组件

**任务清单：**
- [ ] 创建 platform_credentials, publish_records 表
- [ ] 实现审核队列页面（待审核内容列表）
- [ ] 实现审核操作（通过/拒绝/延后/编辑后通过）
- [ ] 实现批量审核
- [ ] 实现平台凭证管理（添加、编辑、验证）
- [ ] 开发平台适配器框架（抽象基类 + 接口规范）
- [ ] 实现 X/Twitter 适配器（API 发布）
- [ ] 实现掘金适配器（API 发布）
- [ ] 实现发布调度器（定时检查发布队列，自动发布）
- [ ] 实现立即发布、重试发布
- [ ] 实现发布记录页面
- [ ] 实现半自动发布模式（一键复制 + 打开发布页）

**验收标准：**
- 可以审核内容（通过/拒绝/延后）
- 可以批量审核
- 可以配置平台凭证并验证
- 可以自动发布到 X 和掘金
- 可以查看发布记录和结果
- 发布失败可以重试
- 不支持 API 的平台可以半自动发布

### 11.6 M5：数据与优化（第 11-12 周）

**目标：** 实现数据追踪和运营报表

**任务清单：**
- [ ] 创建 content_stats 表
- [ ] 实现效果数据录入页面
- [ ] 实现批量数据录入（CSV 导入）
- [ ] 实现运营报表页面（核心指标、平台对比、趋势图）
- [ ] 实现内容主题效果分析
- [ ] 实现报表导出（PDF/Excel）
- [ ] 实现 AI 运营优化建议（基于数据分析）

**验收标准：**
- 可以录入每条内容的效果数据
- 可以查看客户运营报表（指标、图表、趋势）
- 可以导出报表
- AI 可以基于数据给出优化建议

### 11.7 M6：打磨与验证（第 13-14 周）

**目标：** 整体打磨，用自己的产品真实运营验证

**任务清单：**
- [ ] 全面 Bug 修复
- [ ] 用户体验优化（交互细节、加载状态、错误提示）
- [ ] 性能优化（页面加载、API 响应）
- [ ] 编写使用文档
- [ ] 用自己的产品开始真实代运营
- [ ] 收集运营过程中的问题和需求
- [ ] 迭代优化核心流程

**验收标准：**
- 系统稳定运行，无重大 Bug
- 可以顺畅完成"客户创建→产品画像→策略生成→计划排期→内容生成→审核→发布→数据追踪"全流程
- 用自己的产品运营至少 2 周，验证工具价值

---

## 12. 验收标准

### 12.1 功能验收

每个用户故事（US-001 至 US-032）都需要对应验收测试用例，核心验收点：

| 模块 | 核心验收点 |
|---|---|
| 客户管理 | 可以完整创建、编辑、删除客户；客户列表筛选搜索正常 |
| 产品理解 | 输入产品信息后可以生成结构化产品画像；画像可以手动编辑 |
| 运营策略 | 可以分别生成国内/国外运营策略；策略包含平台、内容矩阵、发布节奏 |
| 运营计划 | 可以基于策略生成排期；排期可以手动调整；状态流转正确 |
| 内容生成 | 可以生成文字、图文、视频三种形式；一鱼多吃功能正常；内容可以编辑和重新生成 |
| 审核工作流 | 待审核队列展示正确；通过/拒绝/延后操作正常；批量审核正常 |
| 发布组件 | X 和掘金可以自动发布；发布记录完整；失败可以重试；半自动发布可用 |
| 数据追踪 | 效果数据可以录入；报表展示正确；可以导出 |

### 12.2 流程验收（端到端）

完整走通以下流程，无阻断性问题：

```
1. 创建客户（录入产品信息）
2. 生成产品画像
3. 生成国内/国外运营策略
4. 生成本周内容排期
5. 为排期项生成内容（文字/图文/视频）
6. 审核内容（通过）
7. 配置平台凭证
8. 自动发布到 X/掘金
9. 查看发布记录
10. 录入效果数据
11. 查看运营报表
```

### 12.3 质量验收

- 代码有基本的单元测试覆盖（核心模块 > 60%）
- 所有 API 有接口文档（自动生成 Swagger）
- 前端页面无明显布局错乱
- 错误处理完善，不出现未捕获的异常导致页面崩溃
- 数据库操作有事务保护，不出现数据不一致

---

## 13. 风险与待决事项

### 13.1 技术风险

| 风险 | 影响 | 概率 | 应对策略 |
|---|---|---|---|
| 国内平台 API 不开放，发布自动化困难 | 高 | 高 | MVP 先做 API 成熟的平台（X、掘金）；其他平台用半自动模式；后续研究浏览器自动化方案 |
| AI 生成内容质量不稳定 | 高 | 中 | 建立内容质量评分机制；高风险内容必须人工重写；积累优质内容模板库；持续优化 Prompt |
| 视频生成 API 成本高、速度慢 | 中 | 中 | 视频内容作为可选功能，不强制；控制视频生成频率；选择性价比高的 API |
| 平台封号风险 | 高 | 中 | 内容必须人工审核；控制发布频率；模拟真人操作节奏；不发垃圾内容 |
| LLM API 调用失败/限流 | 中 | 中 | 自动重试机制；多 API 备份；失败时记录错误，允许手动触发 |

### 13.2 产品风险

| 风险 | 影响 | 概率 | 应对策略 |
|---|---|---|---|
| 代运营效果难以量化，客户不满意 | 高 | 中 | 从第一天就追踪数据；设定明确基线和目标；用案例说话；定期沟通汇报 |
| 一个人能服务的客户数有上限 | 中 | 高 | 工具的核心价值就是提升人效；先验证 1 人能服务多少客户；到上限再考虑招人或产品化 |
| 产品理解深度不够，内容泛化 | 高 | 中 | 产品画像模块持续优化；增加手动输入产品细节的入口；积累行业知识库 |
| 运营策略模板化，缺乏定制化 | 中 | 中 | 策略生成时深度结合产品画像；允许手动调整；积累不同类型产品的策略模板 |

### 13.3 待决事项

| 事项 | 说明 | 影响 | 建议决策时间 |
|---|---|---|---|
| LLM API 选择 | Doubao vs OpenAI vs 其他 | 内容质量、成本、速度 | M1 开始前 |
| 图像生成 API 选择 | Doubao 图像 vs Seedream vs Midjourney | 图片质量、成本、速度 | M3 开始前 |
| 视频生成 API 选择 | Doubao 视频 vs Seedance vs Runway | 视频质量、成本、速度 | M3 开始前 |
| 国内平台发布方案 | API vs 浏览器自动化 vs 半自动 | 发布自动化程度、开发工作量 | M4 开始前 |
| 是否需要客户自助门户 | 客户能否自己查看运营数据和审核内容 | 客户体验、开发工作量 | M5 后评估 |
| 是否需要多运营者协作 | 未来是否招人一起做 | 架构设计、权限系统 | 验证 1 人上限后 |

---

## 14. 附录

### 14.1 技术栈详细版本

| 组件 | 技术 | 版本建议 |
|---|---|---|
| 后端框架 | FastAPI | 最新稳定版 |
| ORM | SQLAlchemy | 2.0+ |
| 数据库 | SQLite | 3.x |
| 数据验证 | Pydantic | 2.x |
| 任务调度 | APScheduler | 最新稳定版 |
| HTTP 客户端 | httpx | 最新稳定版 |
| 前端框架 | Next.js | 14+（App Router） |
| 前端语言 | TypeScript | 5.x |
| UI 框架 | Tailwind CSS | 3.x |
| 组件库 | shadcn/ui 或 Ant Design | 最新稳定版 |
| 状态管理 | Zustand 或 React Query | 最新稳定版 |
| Markdown 编辑器 | CodeMirror 或 TipTap | 最新稳定版 |
| 图表 | ECharts 或 Recharts | 最新稳定版 |

### 14.2 项目目录结构建议

```
operation-agent/
├── backend/                 # Python 后端
│   ├── app/
│   │   ├── main.py          # FastAPI 入口
│   │   ├── config.py        # 配置
│   │   ├── database.py      # 数据库连接
│   │   ├── models/          # SQLAlchemy 模型
│   │   ├── schemas/         # Pydantic 模型
│   │   ├── api/             # API 路由
│   │   │   ├── clients.py
│   │   │   ├── profiles.py
│   │   │   ├── strategies.py
│   │   │   ├── plans.py
│   │   │   ├── contents.py
│   │   │   ├── review.py
│   │   │   ├── publish.py
│   │   │   └── stats.py
│   │   ├── services/        # 业务逻辑
│   │   │   ├── llm.py       # LLM 调用封装
│   │   │   ├── image_gen.py # 图像生成
│   │   │   ├── video_gen.py # 视频生成
│   │   │   ├── product_understanding.py
│   │   │   ├── strategy_generation.py
│   │   │   ├── content_generation.py
│   │   │   └── scheduler.py # 发布调度
│   │   ├── adapters/        # 平台适配器
│   │   │   ├── base.py      # 抽象基类
│   │   │   ├── twitter.py
│   │   │   ├── juejin.py
│   │   │   ├── xiaohongshu.py
│   │   │   └── ...
│   │   └── utils/           # 工具函数
│   ├── tests/               # 测试
│   ├── requirements.txt
│   └── .env
├── frontend/                # Next.js 前端
│   ├── src/
│   │   ├── app/             # App Router 页面
│   │   ├── components/      # 组件
│   │   ├── lib/             # 工具函数、API 调用
│   │   ├── types/           # TypeScript 类型
│   │   └── styles/          # 样式
│   ├── package.json
│   └── tsconfig.json
├── docs/                    # 文档
│   └── SDD-独立开发者代运营Agent.md
├── scripts/                 # 脚本
│   ├── backup.py            # 数据库备份
│   └── init_db.py           # 数据库初始化
└── README.md
```

### 14.3 配置文件示例（.env）

```env
# LLM 配置
LLM_PROVIDER=doubao  # doubao / openai
LLM_API_KEY=your_api_key
LLM_BASE_URL=https://api.doubao.com/v1
LLM_MODEL=doubao-pro

# 图像生成配置
IMAGE_GEN_PROVIDER=doubao
IMAGE_GEN_API_KEY=your_api_key

# 视频生成配置
VIDEO_GEN_PROVIDER=doubao
VIDEO_GEN_API_KEY=your_api_key

# 数据库
DATABASE_URL=sqlite:///./operation_agent.db

# 应用
APP_HOST=0.0.0.0
APP_PORT=8000
DEBUG=true

# 加密密钥（用于平台凭证加密）
ENCRYPTION_KEY=your_encryption_key
```

---

**文档结束**

> 本文档为 SDD 规格驱动开发文档，后续开发过程中如有变更，应及时更新本文档，保持文档与代码一致。
