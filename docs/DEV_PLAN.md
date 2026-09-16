# DEV_PLAN.md：开发计划

## 1. 当前状态

当前阶段：M0 文档与技术方案确认。

已存在：

- `docs/SDD-独立开发者代运营Agent.md`
- `docs/AGENTS.md`
- `docs/SPEC.md`
- `docs/ARCHITECTURE.md`
- `docs/DEV_PLAN.md`
- `docs/CHECKLIST.md`

当前产品节奏调整为：

- **M1 交付可用 MVP 闭环**：先让工具能真实辅助运营者完成一次从客户建档到内容审核导出的最小流程。
- **M2、M3 聚焦内容优化**：在 M1 可用的基础上优化内容质量、平台适配、多模态能力和生产效率。
- **M4 以后再加强发布、数据和真实运营验证**。

进入代码开发前，必须先确认：

- M1 MVP 闭环范围已确认。
- 技术方案已确认。
- M1 第一切片已拆清楚。
- 需要安装的依赖已获批准。

## 2. 开发原则

- 每次只做一个垂直切片。
- 每个切片都要有可运行、可验证的结果。
- M1 优先交付能用的闭环，不追求内容质量和自动化发布一步到位。
- M2、M3 再系统优化内容质量、平台风格、多模态和生产效率。
- 不绕过人工审核和安全规则。
- 任何数据模型、API 契约、第三方服务选型变化，都必须先说明并等待批准。

## 3. 里程碑总览

| 阶段 | 目标 | 核心交付 |
|---|---|---|
| M0 | 文档和技术方案 | SPEC、ARCHITECTURE、DEV_PLAN、CHECKLIST |
| M1 | MVP 闭环 | 客户建档、产品画像、策略方向、一周排期、文字内容生成、审核、复制/导出 |
| M2 | 内容质量优化 | Prompt 优化、平台风格、内容模板、改写重生成、版本质量对比 |
| M3 | 内容形态与效率优化 | 一鱼多吃增强、图文/视频脚本、多模态接口、批量生产、内容预览 |
| M4 | 发布与凭证 | 平台凭证、发布记录、半自动发布、X/掘金自动发布 |
| M5 | 数据与报表 | 效果数据录入、基础报表、导出、策略复盘 |
| M6 | 打磨验证 | 全流程稳定性、真实运营验证、体验优化 |

## 4. M0：文档和技术方案

目标：

- 把 SDD 转化成后续开发可直接执行的文档。
- 明确产品范围、架构边界、技术选型和验收门禁。

交付：

- [x] `AGENTS.md`
- [x] `SPEC.md`
- [x] `ARCHITECTURE.md`
- [x] `DEV_PLAN.md`
- [x] `CHECKLIST.md`

验收：

- 文档之间没有明显范围冲突。
- 开发 Agent 能从文档中判断下一步任务。
- M1 MVP 闭环可以被明确规划。

## 5. M1：可用 MVP 闭环

M1 目标不是做完 SDD 里的所有自动化能力，而是先产出一个能用的内部 MVP：

```text
客户建档
→ 产品画像生成/编辑
→ 运营策略方向生成/编辑
→ 一周内容排期生成/编辑
→ 生成文字内容
→ 人工审核
→ 复制或导出内容，用于人工发布
```

M1 不接入真实平台发布，不做效果数据报表，不做图像/视频真实生成。

### M1.1 项目骨架

目标：

- 初始化后端和前端项目。
- 建立基础目录、配置、运行脚本和开发约定。

变更范围：

- `backend/`
- `frontend/`
- `scripts/`
- `README.md`
- `.gitignore`

交付：

- FastAPI 应用可启动。
- Next.js 应用可启动。
- SQLite 配置可读取。
- Alembic 初始化完成。
- 基础健康检查 API。
- 前端基础工作台布局。

验收：

- 后端 `/health` 返回正常。
- 前端首页可打开。
- `.env` 不被提交。
- README 写明本地启动命令。

### M1.2 客户建档

目标：

- 实现客户和产品基础信息管理。

变更范围：

- `backend/app/models/`
- `backend/app/schemas/`
- `backend/app/api/clients.py`
- `backend/app/services/client_service.py`
- `backend/tests/`
- `frontend/src/app/clients/`
- `frontend/src/components/clients/`
- `frontend/src/lib/api/clients.ts`
- `frontend/src/types/client.ts`

交付：

- `clients` 表。
- 客户创建、列表、详情、更新、状态更新、删除。
- 搜索、状态筛选、分页。
- 客户列表页、创建页、编辑页、详情入口。

验收：

- 前端可以真实调用后端完成客户 CRUD。
- 表单校验和 API 失败提示可见。
- 客户状态非法时返回明确错误。

### M1.3 产品画像

目标：

- 让系统能基于客户信息生成和编辑产品画像。

变更范围：

- `backend/app/models/product_profile.py`
- `backend/app/schemas/product_profile.py`
- `backend/app/api/profiles.py`
- `backend/app/services/product_profile_service.py`
- `backend/app/services/llm_service.py`
- `backend/app/services/prompts/`
- `backend/tests/`
- `frontend/src/app/clients/[clientId]/profile/`
- `frontend/src/components/profiles/`
- `frontend/src/lib/api/profiles.ts`
- `frontend/src/types/product-profile.ts`

交付：

- `product_profiles` 表。
- 获取最新画像。
- mock LLM 生成画像。
- 手动更新画像。
- 查看版本历史。
- 产品画像展示和编辑页面。

验收：

- 没有真实 API Key 时也可以用 mock 走通流程。
- AI 输出保存前经过 schema 校验。
- 重新生成会更新版本。
- 前端完整展示画像字段。

### M1.4 运营策略方向

目标：

- 生成一版足够指导内容排期的策略方向，不追求精细运营优化。

变更范围：

- `backend/app/models/strategy.py`
- `backend/app/schemas/strategy.py`
- `backend/app/api/strategies.py`
- `backend/app/services/strategy_service.py`
- `backend/app/services/prompts/`
- `backend/tests/`
- `frontend/src/app/clients/[clientId]/strategies/`
- `frontend/src/components/strategies/`
- `frontend/src/lib/api/strategies.ts`
- `frontend/src/types/strategy.ts`

交付：

- `strategies` 表。
- 基于产品画像生成国内/海外策略方向。
- 获取和更新策略。
- 策略页面展示平台、内容矩阵、发布节奏和增长建议。

验收：

- 能为客户生成至少一个市场策略。
- 策略可编辑。
- 策略字段足够支撑一周排期生成。

### M1.5 一周内容排期

目标：

- 基于策略生成一周内容计划，并允许人工调整。

变更范围：

- `backend/app/models/plan.py`
- `backend/app/models/schedule_item.py`
- `backend/app/schemas/plan.py`
- `backend/app/api/plans.py`
- `backend/app/services/plan_service.py`
- `backend/tests/`
- `frontend/src/app/clients/[clientId]/plans/`
- `frontend/src/components/plans/`
- `frontend/src/lib/api/plans.ts`
- `frontend/src/types/plan.ts`

交付：

- `plans` 和 `schedule_items` 表。
- 生成一周排期。
- 排期查询、编辑、删除、跳过。
- 按日期分组的内容计划页。

验收：

- 能从策略生成未来 7 天内容排期。
- 排期项包含日期、时间、平台、主题、内容形式、角度提示和状态。
- 排期可以人工调整。

### M1.6 文字内容生成

目标：

- 基于排期项生成可人工审核的文字内容。

变更范围：

- `backend/app/models/content_item.py`
- `backend/app/models/content_version.py`
- `backend/app/schemas/content.py`
- `backend/app/api/contents.py`
- `backend/app/services/content_service.py`
- `backend/app/services/prompts/`
- `backend/tests/`
- `frontend/src/app/contents/`
- `frontend/src/components/contents/`
- `frontend/src/lib/api/contents.ts`
- `frontend/src/types/content.ts`

交付：

- `content_items` 和 `content_versions` 表。
- 为排期项生成文字内容。
- 手动编辑内容。
- 重新生成内容。
- 版本历史。

验收：

- 内容生成后进入 `pending_review`。
- 生成内容包含标题、正文、标签。
- 内容可编辑、可重新生成、可查看版本历史。

### M1.7 审核与复制导出

目标：

- 完成最小人工审核和人工发布准备流程。

变更范围：

- `backend/app/api/review.py`
- `backend/app/services/review_service.py`
- `backend/tests/`
- `frontend/src/app/review/`
- `frontend/src/components/review/`
- `frontend/src/components/contents/`

交付：

- 审核队列。
- 通过、编辑后通过、拒绝、延后。
- 复制内容。
- 导出 Markdown 或 JSON。

验收：

- 只有 `pending_review` 内容可以审核。
- 审核通过后内容状态为 `approved`。
- 拒绝必须记录原因。
- 审核通过内容可以复制或导出，用于人工发布。

## 6. M2：内容质量优化

M2 在 M1 MVP 可用后进行，目标是让生成内容更像可交付作品。

### M2.1 Prompt 体系优化

交付：

- 产品画像、策略、排期、内容生成 Prompt 分文件管理。
- Prompt 版本记录。
- 输出 JSON schema 更严格。
- 失败重试和解析错误提示。

### M2.2 平台风格适配

交付：

- 平台内容规范配置。
- X/Twitter、掘金、V2EX、小红书、IndieHackers、Reddit 等平台风格模板。
- 字数、标题、标签、CTA 规则。

### M2.3 内容模板库

交付：

- 教程型、复盘型、更新公告型、故事型、对比型、互动型模板。
- 支持按客户和产品类型选择模板。
- 支持模板效果备注。

### M2.4 改写与质量对比

交付：

- 改写内容。
- 改变语气。
- 改变角度。
- 生成多个候选版本。
- 版本质量对比和人工选择。

## 7. M3：内容形态与生产效率优化

M3 继续优化内容生产能力，重点不是先做真实发布，而是提升一批内容从主题到多平台版本的产能。

### M3.1 一鱼多吃增强

交付：

- 从一个主题生成多平台版本。
- 多平台版本关联同一主题。
- 批量进入审核队列。
- 支持批量复制或导出。

### M3.2 图文内容

交付：

- 图像生成 service 接口。
- mock 图像 provider。
- 配图 Prompt 生成。
- 图片生成状态和失败原因记录。
- 图文内容预览。

### M3.3 视频脚本与视频接口

交付：

- 视频脚本生成。
- 视频生成 service 接口。
- mock 视频 provider。
- 视频生成状态和失败原因记录。

### M3.4 内容预览与批量生产

交付：

- 平台预览。
- 批量生成。
- 批量重新生成。
- 批量导出。
- 审核队列效率优化。

## 8. M4：发布与凭证

### M4.1 平台凭证

交付：

- `platform_credentials` 表。
- 凭证加密存储。
- 脱敏展示。
- 凭证验证接口。

### M4.2 发布记录与 dry-run 发布

交付：

- `publish_records` 表。
- dry-run adapter。
- 立即发布。
- 发布失败记录。

### M4.3 半自动发布

交付：

- 一键复制内容。
- 打开平台发布页。
- 手动填写发布 URL。
- 标记发布成功。

### M4.4 真实平台发布

交付：

- X/Twitter adapter。
- 掘金 adapter。
- 发布调度器。
- 失败重试。

注意：

- 真实平台接入前必须先确认 API、凭证方式、风控风险和测试账号。

## 9. M5：数据与报表

### M5.1 效果数据录入

交付：

- `content_stats` 表。
- 单条录入。
- 批量 CSV 导入。

### M5.2 报表

交付：

- 客户维度报表。
- 平台对比。
- 内容主题效果。
- 趋势图。

### M5.3 导出与复盘建议

交付：

- 报表导出。
- 基于数据的运营复盘建议。
- 下一轮内容策略建议。

## 10. M6：打磨与真实验证

交付：

- Bug 修复。
- 体验优化。
- 性能优化。
- 使用文档。
- 使用自有产品真实运营至少 2 周。
- 根据真实运营问题调整流程。

## 11. 每个切片的固定流程

1. 读取 SDD、SPEC、ARCHITECTURE、DEV_PLAN、CHECKLIST。
2. 输出任务计划。
3. 等待人工确认。
4. 实现最小切片。
5. 补充或更新测试。
6. 执行匹配范围的验证。
7. 检查 diff。
8. 汇报结果和风险。

## 12. 下一步建议

当前下一步应进入：

```text
M1.1 项目骨架
```

在动手前应先输出 M1.1 的正式执行计划，至少包含：

- 目标。
- 变更文件列表。
- 后端 scaffold 方案。
- 前端 scaffold 方案。
- 依赖清单。
- 风险点。
- 验证方式。
- 是否需要安装依赖和联网。
