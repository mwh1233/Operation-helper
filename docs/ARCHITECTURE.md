# ARCHITECTURE.md：技术方案与系统架构

## 1. 架构目标

本项目采用本地优先、单运营者、前后端分离的架构。目标是在保持开发成本可控的前提下，快速验证代运营完整流程。

架构必须支持：

- 客户、产品画像、策略、排期、内容、审核、发布、数据追踪的完整链路。
- 本地 SQLite 存储。
- 外部 AI 服务和平台 API 的可替换封装。
- 平台发布适配器插件化。
- 后续扩展到多运营者、客户门户或 SaaS 化时不推翻核心领域模型。

## 2. 技术选型

### 2.1 后端

- 语言：Python 3.11+
- Web 框架：FastAPI
- 数据验证：Pydantic v2
- ORM：SQLAlchemy 2.x
- 数据库：SQLite 3.x
- 数据库迁移：Alembic
- HTTP 客户端：httpx
- 定时任务：APScheduler，在 M4 发布调度阶段引入
- 测试：pytest
- 代码质量：ruff，必要时再增加 mypy

说明：

- FastAPI 自动生成 OpenAPI，适合当前 REST API 契约。
- SQLite 满足单运营者本地优先需求。
- SQLAlchemy + Alembic 避免手写建表脚本造成模型漂移。
- 外部服务调用统一通过 service 层，不允许路由直接调用 provider。

### 2.2 前端

- 框架：Next.js App Router
- 语言：TypeScript
- 样式：Tailwind CSS
- 组件：shadcn/ui 优先
- 服务端状态：TanStack Query
- 本地 UI 状态：React state；跨页面状态确有需要时使用 Zustand
- 表单：React Hook Form
- 校验：Zod
- 图表：Recharts
- Markdown 编辑：M3 阶段选择 CodeMirror 或 TipTap

说明：

- 前端只负责界面、交互和 API 调用。
- 业务规则优先落在后端 service 层，避免前后端规则不一致。
- UI 风格应是内部工作台，强调密度、清晰状态和批量处理效率。

### 2.3 依赖版本策略

- SDD 中的版本建议作为下限参考。
- 实际 scaffold 时以稳定版本和 lockfile 为准。
- 新增依赖必须在对应任务计划中说明用途和替代方案。

## 3. 仓库目录结构

```text
operation-agent/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── api/
│   │   ├── services/
│   │   ├── adapters/
│   │   └── utils/
│   ├── alembic/
│   ├── tests/
│   ├── requirements.txt
│   └── requirements-dev.txt
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── lib/
│   │   ├── types/
│   │   └── styles/
│   ├── package.json
│   └── tsconfig.json
├── scripts/
├── docs/
└── README.md
```

## 4. 后端分层

### 4.1 API 层

位置：`backend/app/api/`

职责：

- 定义路由。
- 接收请求参数。
- 调用 service。
- 返回 schema。
- 将已知异常转换为统一错误响应。

禁止：

- 在路由中写复杂业务逻辑。
- 在路由中直接调用 LLM、图像生成、视频生成或平台 API。
- 在路由中拼接 SQL。

### 4.2 Schema 层

位置：`backend/app/schemas/`

职责：

- 定义请求体。
- 定义响应体。
- 定义枚举和嵌套 JSON 结构。
- 校验 AI 生成结果。

原则：

- 外部 API 返回前必须经过 schema。
- 前端 TypeScript 类型应与 schema 保持一致。

### 4.3 Model 层

位置：`backend/app/models/`

职责：

- 定义 SQLAlchemy 表模型。
- 定义关系。
- 定义索引和唯一约束。
- 保持与 Alembic migration 一致。

原则：

- SQLite JSON 字段可用 SQLAlchemy JSON 或 Text 存储，但进入数据库前必须用 Pydantic 校验。
- 时间字段统一存储 ISO 8601 或 UTC datetime，前端展示再转换。

### 4.4 Service 层

位置：`backend/app/services/`

职责：

- 实现业务用例。
- 管理事务边界。
- 组合多个 repository/model 操作。
- 组装 Prompt。
- 调用外部 provider。
- 更新状态机。

核心服务：

- `client_service.py`
- `product_profile_service.py`
- `strategy_service.py`
- `plan_service.py`
- `content_service.py`
- `review_service.py`
- `publish_service.py`
- `stats_service.py`
- `llm_service.py`
- `image_generation_service.py`
- `video_generation_service.py`
- `scheduler_service.py`

### 4.5 Adapter 层

位置：`backend/app/adapters/`

职责：

- 封装平台内容规范。
- 验证平台凭证。
- 发布内容。
- 获取发布结果。
- 可选获取效果数据。

适配器接口：

```python
class PlatformAdapter:
    def get_platform_name(self) -> str:
        ...

    def get_content_spec(self) -> "ContentSpec":
        ...

    def validate_credentials(self, credentials: dict) -> bool:
        ...

    def publish(self, content: "ContentItem", credentials: dict) -> "PublishResult":
        ...

    def fetch_stats(self, content_url: str, credentials: dict) -> "ContentStats | None":
        ...
```

M1 不实现发布 adapter，只提供审核通过内容的复制和导出能力。M4 先实现 dry-run adapter，再接入真实 X/Twitter 和掘金。

## 5. 前端分层

### 5.1 App 页面

位置：`frontend/src/app/`

建议路由：

```text
/
/clients
/clients/new
/clients/[clientId]
/clients/[clientId]/profile
/clients/[clientId]/strategies
/clients/[clientId]/plans
/clients/[clientId]/review
/clients/[clientId]/publish-records
/clients/[clientId]/stats
/review
/publish-records
/reports
```

### 5.2 组件

位置：`frontend/src/components/`

建议分类：

- `layout/`：侧边栏、顶部栏、页面容器。
- `clients/`：客户表格、客户表单、状态标签。
- `profiles/`：产品画像卡片和编辑表单。
- `strategies/`：平台配置、内容矩阵、策略编辑。
- `plans/`：排期列表、排期项表单。
- `contents/`：内容编辑器、内容预览、版本历史。
- `review/`：审核卡片、审核操作。
- `publish/`：凭证表单、发布记录表格。
- `stats/`：指标卡、图表。

### 5.3 API 调用

位置：`frontend/src/lib/api/`

原则：

- 每个领域一个 API client 文件。
- 所有请求通过统一 fetch wrapper。
- 统一处理错误响应。
- 不在组件里散写 URL 和 fetch 配置。

### 5.4 类型

位置：`frontend/src/types/`

原则：

- 与后端响应 schema 保持一致。
- 复杂枚举用 TypeScript union type。
- 若后续引入自动生成类型，以 OpenAPI 为来源。

## 6. API 规范

Base URL：

```text
http://localhost:8000/api/v1
```

响应格式：

```json
{
  "data": {},
  "meta": {}
}
```

列表响应：

```json
{
  "data": [],
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 0
  }
}
```

错误响应：

```json
{
  "error": {
    "code": "CLIENT_NOT_FOUND",
    "message": "客户不存在",
    "details": {}
  }
}
```

分页参数：

- `page`
- `page_size`

通用筛选参数按资源定义，例如：

- `status`
- `client_id`
- `platform`
- `start_date`
- `end_date`
- `query`

## 7. 数据库设计原则

- 所有表使用整数自增主键。
- 所有核心表包含 `created_at`。
- 需要编辑的核心表包含 `updated_at`。
- 删除客户是高风险操作，MVP 可先物理删除，但 API 必须二次确认；后续可改软删除。
- 关键状态字段使用枚举约束，由 schema 和 service 双层校验。
- 平台凭证字段必须加密后存储。
- audit log 记录关键操作，不记录明文密钥。

## 8. 状态机

### 内容生成与审核

```text
draft
  -> pending_review
  -> approved
  -> published

pending_review
  -> rejected
  -> pending_review
```

规则：

- AI 生成内容后进入 `pending_review`。
- 人工编辑不自动改变审核状态，除非选择“编辑后通过”。
- `approved` 内容才允许发布。
- `rejected` 内容可以重新生成，重新生成后回到 `pending_review`。

### 发布

```text
pending
  -> publishing
  -> success
  -> failed
  -> pending
```

规则：

- 发布失败最多自动重试 3 次。
- 每次失败记录错误原因。
- 手动重试必须新增或更新发布记录的重试信息。

## 9. AI 服务设计

### LLMService

职责：

- 统一读取 LLM 配置。
- 统一超时、重试和错误处理。
- 记录 provider、model、prompt 版本和调用状态。
- 返回结构化结果。

M1 可先支持 mock provider，用于在没有真实 API Key 时完成开发。

### Prompt 管理

位置建议：

```text
backend/app/services/prompts/
```

原则：

- 产品画像、策略、排期、内容生成 Prompt 分文件维护。
- Prompt 输出必须要求 JSON。
- 保存生成结果时记录 Prompt 版本。

### 图像和视频生成

M3 阶段接入。

原则：

- 与 LLM 一样走 service 抽象。
- 生成状态和失败原因必须入库。
- 不允许前端直接调用 provider。

## 10. 发布组件设计

MVP 先在后端进程内实现发布服务和调度器。若后续复杂度上升，再拆为独立 CLI 或微服务。

发布流程：

1. 查询审核通过且到达发布时间的内容。
2. 查询客户对应平台凭证。
3. 加载平台 adapter。
4. 调用 `publish`。
5. 写入 `publish_records`。
6. 更新内容状态。

阶段策略：

- M1：不实现发布记录，只支持审核通过内容的复制和导出。
- M2/M3：继续优化内容质量、平台风格和多模态生产效率。
- M4：先实现 dry-run 发布记录，再接入真实 X/Twitter 和掘金。
- 其他平台：先半自动发布。

## 11. 安全方案

- `.env` 必须加入 `.gitignore`。
- 平台凭证使用 `ENCRYPTION_KEY` 加密。
- 后端日志必须脱敏。
- 前端展示凭证只显示脱敏摘要。
- 本地开发不暴露公网端口。
- 不记录完整 Prompt 中的敏感凭证。
- 真实发布操作必须要求内容已审核。

## 12. 配置方案

后端 `.env` 示例：

```env
DATABASE_URL=sqlite:///./operation_agent.db
APP_HOST=127.0.0.1
APP_PORT=8000
DEBUG=true

LLM_PROVIDER=mock
LLM_API_KEY=
LLM_BASE_URL=
LLM_MODEL=

IMAGE_GEN_PROVIDER=mock
IMAGE_GEN_API_KEY=

VIDEO_GEN_PROVIDER=mock
VIDEO_GEN_API_KEY=

ENCRYPTION_KEY=
PUBLISH_DRY_RUN=true
```

开发阶段默认：

- `LLM_PROVIDER=mock`
- `PUBLISH_DRY_RUN=true`

这样可以先验证主流程，不依赖真实外部服务。

## 13. 测试策略

### 后端

- service 单元测试。
- API 集成测试。
- 状态机测试。
- adapter contract test。
- 凭证加密和脱敏测试。

### 前端

- 类型检查。
- 关键组件交互测试。
- API 错误展示测试。
- 表单校验测试。

### 端到端

优先手动验证核心切片：

```text
M1：客户创建 -> 产品画像 -> 策略 -> 排期 -> 内容 -> 审核 -> 复制/导出
M4/M5：发布记录 -> 数据追踪 -> 报表
```

## 14. 本地运行目标

后端：

```bash
cd backend
python -m venv .venv
pip install -r requirements.txt -r requirements-dev.txt
uvicorn app.main:app --reload
```

前端：

```bash
cd frontend
npm install
npm run dev
```

验证：

```bash
cd backend
pytest

cd frontend
npm run typecheck
npm run lint
npm run build
```

实际命令以项目创建后的 `README.md` 和 `package.json` 为准。
