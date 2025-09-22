## 项目总览

这是一个包含前端 SaaS 与后端自动化服务的全栈单仓库（monorepo）。

- 前端：Next.js（React + TypeScript），位于 `saas-starter/`，独立的 repo，部署在 Vercel。
- 后端：Python 微服务（Aachen Termin Bot），位于 `aachen-termin-bot/`，独立的 repo，用于自动预约 SuperC，部署在长期在线的服务器上并以定时任务形式运行。
- 数据库：PostgreSQL，通过 saas-starter 的 Drizzle ORM 进行设置，部署在 Supabase 上。

## 子项目与目录结构

```
supac/
├─ aachen-termin-bot/        # Python 自动预约微服务
│  ├─ superc.py / infostelle.py
│  ├─ superc/                # 预约核心模块（流程、表单、验证码识别等）
│  ├─ db/                    # SQLAlchemy 模型与工具（PostgreSQL）
│  ├─ data/                  # 调试页面、日志与样例数据
│  └─ tests/                 # 单元与集成测试
│
└─ saas-starter/             # Next.js SaaS 前端（EmailX 与 SupaC 页面）
	 ├─ app/                   # 应用路由与 API routes
	 ├─ app/lib/               # 数据库（Drizzle）/ LLM / 支付等库
	 ├─ tests/                 # Vitest 测试（单元/集成）
	 └─ e2e/                   # 端到端测试脚手架
```

更多细节请参考：
- 前端说明：`saas-starter/README.md`
- 自动预约机器人：`aachen-termin-bot/README.md`


## 功能概览

- EmailX：智能邮件管理与 AI 助手（IMAP/SMTP + Azure OpenAI）。
- SupaC：面向 RWTH 学生的 SuperC/Infostelle 预约入口与展示页面。
- Aachen Termin Bot：完整 6 步预约流程自动化，包含验证码识别（Azure OpenAI）与数据库状态追踪。


## 技术栈

- 前端：Next.js 14/15、React 19、TypeScript、Tailwind CSS、shadcn/ui
- 后端：Python 3.10+、httpx、BeautifulSoup、SQLAlchemy、psycopg2-binary
- 数据库：PostgreSQL（Supabase 托管），ORM（前端：Drizzle；后端：SQLAlchemy）
- AI 服务：Azure OpenAI（前端与机器人均有使用）
- 邮件：node-imap / nodemailer（前端）


## 快速开始

下面分别介绍前端与后端的本地开发启动方式（macOS，zsh）。

### 1) 前端（`saas-starter/`）

依赖：Node.js（建议 18+/20+）、pnpm 或 npm。

常用环境变量（从代码中已使用到的变量汇总）：
- 数据库：`POSTGRES_URL`（Drizzle 连接字符串）
- Azure OpenAI：
	- `AZURE_OPENAI_API_KEY`
	- `AZURE_OPENAI_ENDPOINT`
	- `AZURE_OPENAI_MODEL`（默认 `gpt-4.1`）
	- `AZURE_OPENAI_API_VERSION`（默认 `2024-12-01-preview`）
- Stripe：
	- `STRIPE_SECRET_KEY`
	- `STRIPE_WEBHOOK_SECRET`
- Matrix（示例可选）：
	- `NEXT_PUBLIC_MATRIX_HS_URL`（默认 `https://matrix.dbis.rwth-aachen.de`）
- 邮件 SMTP（在 hooks/tests 中用到的可选项）：
	- `NEXT_PUBLIC_RWTH_MAIL_SERVER_SMTP_HOST`
	- `NEXT_PUBLIC_RWTH_MAIL_SERVER_SMTP_PORT`
	- `NEXT_PUBLIC_RWTH_MAIL_SERVER_SMTP_SECURE`
	- 集成测试变量：`RWTH_MAIL_SERVER`、`RWTH_MAIL_SERVER_IMAP_PORT`、`TEST_EMAIL_USERNAME`、`TEST_EMAIL_PASSWORD`

本地开发步骤：
1. 在 `saas-starter/` 目录创建 `.env` 并填入上面的必要变量（至少 `POSTGRES_URL`）。
2. 安装依赖并启动开发服务器：

```bash
cd saas-starter
pnpm install        # 或 npm install
pnpm dev            # 或 npm run dev
```

数据库管理（Drizzle）：
```bash
pnpm db:generate    # 生成迁移
pnpm db:migrate     # 执行迁移
pnpm db:studio      # 可视化查看（可选）
```

构建与生产启动：
```bash
pnpm build
pnpm start
```


### 2) 后端微服务（`aachen-termin-bot/`）

依赖：Python 3.10+。推荐使用 uv 或 venv 管理环境。

安装依赖：
```bash
cd aachen-termin-bot
uv venv --python 3.10
source .venv/bin/activate
uv pip install -r requirements.txt
```

环境变量：
- Azure OpenAI（验证码识别）
	- `AZURE_OPENAI_ENDPOINT`
	- `AZURE_OPENAI_KEY`
	- `AZURE_OPENAI_DEPLOYMENT_NAME`
	- `AZURE_OPENAI_API_VERSION`（示例 `2024-12-01-preview`）
- 数据库（与 Supabase/Postgres 对接）
	- `DB_USER`、`DB_PASSWORD`、`DB_HOST`、`DB_PORT`、`DB_NAME`

运行（SuperC/Infostelle）：
```bash
# SuperC
uv run python superc.py

# Infostelle
uv run python infostelle.py
```

后台运行（示例）：
```bash
nohup uv run python superc.py 2>&1 | tee superc.log &
```

更多使用说明与流程图，请见 `aachen-termin-bot/README.md`。


## 部署参考

- 前端：
	- Vercel 部署 Next.js，确保环境变量在 Vercel 项目中配置（尤其 `POSTGRES_URL`、Azure OpenAI、Stripe 相关）。
- 后端：
	- 任意可长期运行的 Linux/macOS 服务器（或容器）上执行 Python 脚本，结合 systemd/cron/PM2 等实现守护与定时任务。
	- 确保服务器可访问数据库与 Azure OpenAI 服务，并正确配置 `.env`。

## Supabase 部署与连接（简要）

本仓库使用 Supabase（PostgreSQL）作为数据库：
- 前端（Next.js + Drizzle）使用环境变量 `POSTGRES_URL`
- 后端（Python + SQLAlchemy）使用环境变量 `DB_USER`、`DB_PASSWORD`、`DB_HOST`、`DB_PORT`、`DB_NAME`

建议：Vercel 上使用 Supabase 的 Connection Pooler（通常 6543 端口），并统一开启 `sslmode=require`。

详细指南请见：
- 前端（Drizzle + Supabase）：`saas-starter/app/lib/db/README.md`（包含如何设置 POSTGRES_URL、直连/Pooler、迁移与 Studio）
- 后端（Python + Supabase）：`aachen-termin-bot/README.md`（包含 DB_* 变量、直连/Pooler 与示例查询）


## 测试

- 前端（Vitest）：
```bash
cd saas-starter
pnpm test           # 全量
pnpm test:unit      # 单元
pnpm test:integration
```

- 后端（pytest）：
```bash
cd aachen-termin-bot
pytest
```


## 相关文档

- 前端：`saas-starter/README.md`、`saas-starter/CLAUDE.md`
- 预约机器人：`aachen-termin-bot/README.md`、`aachen-termin-bot/doc/`
- 规格说明：`aachen-termin-bot/spec/`


## 常见问题（FAQ）

- 数据库连接失败？
	- 前端检查 `POSTGRES_URL`，后端检查 `DB_USER/DB_PASSWORD/DB_HOST/DB_PORT/DB_NAME`。
	- 若使用 Supabase，注意开启 SSL：后端连接串中已通过 `sslmode=require`。
