# Design: Keyword Web Crawler

## Context

构建一个智能网络信息采集系统，从初始关键词出发自动发现、搜索、收集和组织相关网络信息。系统采用前后端分离架构：
- **后端**: Python + FastAPI（爬虫、数据处理、AI 集成）
- **数据库**: SQLite（轻量级持久化存储）
- **前端**: React + TypeScript（用户界面）

### 约束
- 单机运行，无需分布式部署
- 支持 Windows/macOS/Linux
- 最小化外部 API 依赖成本

## Goals / Non-Goals

**Goals:**
- 实现可扩展的爬虫框架，支持多搜索引擎
- 建立关键词与图片的上下文关联
- 支持多语言翻译和未知语言检测
- 提供任务队列管理，支持增量执行
- 构建用户友好的 React 管理界面

**Non-Goals:**
- 不支持分布式爬虫
- 不实现实时推送通知
- 不支持用户认证（单用户系统）

## Decisions

### D1: Python + FastAPI 作为后端框架
**理由**: FastAPI 提供异步支持、自动 API 文档、类型提示，适合爬虫和 API 服务。
**替代方案**: Flask（同步，性能较差）、Django（过于重量级）

### D2: SQLite 作为数据库
**理由**: 零配置、单文件存储、足够支持单用户场景，迁移到 PostgreSQL 成本低。
**替代方案**: PostgreSQL（部署复杂度高）、MongoDB（需要额外安装）

### D3: React + TypeScript 前端
**理由**: 类型安全、生态丰富、组件化开发效率高。
**替代方案**: Vue（团队不熟悉）、纯 JavaScript（类型不安全）

### D4: 分层爬虫策略 (HTTP → Playwright)
**理由**: 优先使用 HTTP 请求（快速、节省资源），失败或内容无效时再降级到 Playwright。
**替代方案**: 纯 Playwright（资源消耗大）、纯 HTTP（无法处理动态页面）

### D5: Agent 驱动的增量任务执行
**理由**: 每个 Agent 实例从数据库读取状态，执行单个原子任务，无需跨实例记忆。
**替代方案**: 长驻进程 + 内部队列（复杂度高）、用户手动触发（效率低）

### D6: SQLite 作为数据库
**理由**: 零配置、单文件存储、足够支持单用户场景，迁移到 PostgreSQL 成本低。
**替代方案**: PostgreSQL（部署复杂度高）、MongoDB（需要额外安装）

### D7: 简化的翻译服务
**理由**: 使用可配置的免费/低成本模型（如 gpt-4o-mini），失败时跳过，前端提供手动管理页面。
**替代方案**: 完整的翻译服务（复杂度高）

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Agent-Driven Architecture                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                        触发层 (Trigger Layer)                          │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │ │
│  │  │ run_agent.sh│  │  前端按钮   │  │  定时任务   │  │  CLI 命令   │   │ │
│  │  │ --count N   │  │  点击触发   │  │  cron/sched │  │  手动执行   │   │ │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘   │ │
│  │         └─────────────────┴─────────────────┴────────────────┘         │ │
│  │                                   │                                     │ │
│  │                                   ▼                                     │ │
│  │                       ┌─────────────────────┐                           │ │
│  │                       │ 启动纯净 Agent 实例 │                           │ │
│  │                       └──────────┬──────────┘                           │ │
│  └──────────────────────────────────┼──────────────────────────────────────┘ │
│                                     │                                       │
│  ┌──────────────────────────────────▼──────────────────────────────────────┐ │
│  │                     Agent 单次执行流程 (Stateless)                       │ │
│  │  ┌───────────────────────────────────────────────────────────────────┐  │ │
│  │  │ 1. 执行 init.sh 确保 FastAPI 服务运行                              │  │ │
│  │  │ 2. Python CLI 查询数据库状态 (pending URLs / searches)             │  │ │
│  │  │ 3. URL 选择循环:                                                   │  │ │
│  │  │    - 有合适 URL? → 访问                                            │  │ │
│  │  │    - 无合适 URL + 有待搜索? → 搜索并过滤 → 保存 → 继续循环          │  │ │
│  │  │    - 无任务? → 退出                                                │  │ │
│  │  │ 4. 访问 URL，处理内容，提取关键词                                  │  │ │
│  │  │ 5. 更新数据库，结束                                                │  │ │
│  │  └───────────────────────────────────────────────────────────────────┘  │ │
│  └──────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                       │
│  ┌──────────────────────────────────▼──────────────────────────────────────┐ │
│  │                        执行层 (Execution Layer)                          │ │
│  │                                                                         │ │
│  │   ┌─────────────────────┐         ┌─────────────────────────────────┐   │ │
│  │   │  简单查询 (Python)   │         │    复杂操作 (FastAPI)           │   │ │
│  │   │  - query pending    │         │    - 搜索 + URL 过滤            │   │ │
│  │   │  - query searches   │         │    - 访问 URL (分层爬虫)        │   │ │
│  │   │  - query status     │         │    - 关键词提取 (AI)            │   │ │
│  │   └─────────────────────┘         │    - 翻译服务                   │   │ │
│  │                                    └─────────────────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                       │
│  ┌──────────────────────────────────▼──────────────────────────────────────┐ │
│  │                          React Frontend                                  │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │ │
│  │  │关键词管理 │ │搜索控制  │ │结果展示  │ │翻译状态  │ │Agent控制 │      │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │ │
│  └──────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                       │
│  ┌──────────────────────────────────▼──────────────────────────────────────┐ │
│  │                        Python FastAPI Backend                            │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │ │
│  │  │   API    │ │   爬虫   │ │  内容    │ │关键词提取│ │  翻译    │      │ │
│  │  │ Router   │ │ HTTP→Play│ │ Processor│ │ (独立模块)│ │ gpt-4o  │      │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │ │
│  └──────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                       │
│  ┌──────────────────────────────────▼──────────────────────────────────────┐ │
│  │                          SQLite Database                                 │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │ │
│  │  │ keywords │ │   urls   │ │ articles │ │  images  │ │resource_ │      │ │
│  │  │          │ │          │ │          │ │          │ │ groups   │      │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │ │
│  │  │ search_  │ │keyword_  │ │keyword_  │ │keyword_  │ │search_   │      │ │
│  │  │ records  │ │url_rel   │ │article_  │ │relations │ │ portals  │      │ │
│  │  │          │ │          │ │  rel     │ │          │ │          │      │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘      │ │
│  └──────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Processing Flow

### Agent-Driven Incremental Task Flow (Agent 驱动的增量任务流程)

```
                    ┌──────────────────────────────┐
                    │        Agent 启动            │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │  执行 init.sh                │
                    │  确保 FastAPI 服务运行       │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
     ┌─────────────────────────────────────────────────────────────────┐
     │                    URL 选择循环 (Selection Loop)                │
     │  ┌───────────────────────────────────────────────────────────┐  │
     │  │                                                           │  │
     │  │   ┌────────────────────────────────────────────────────┐  │  │
     │  │   │  Step 1: 查询有效 URL (Python CLI)                  │  │  │
     │  │   │  python cli/query.py --valid-url                    │  │  │
     │  │   └────────────────────────┬───────────────────────────┘  │  │
     │  │                            │                               │  │
     │  │              ┌─────────────┴─────────────┐                │  │
     │  │              ▼                           ▼                │  │
     │  │         有合适的 URL               没有合适的 URL          │  │
     │  │              │                           │                │  │
     │  │              │                           ▼                │  │
     │  │              │              ┌────────────────────────┐    │  │
     │  │              │              │ 查询待搜索组合 (CLI)   │    │  │
     │  │              │              │ python cli/query.py    │    │  │
     │  │              │              │ --pending-searches     │    │  │
     │  │              │              └───────────┬────────────┘    │  │
     │  │              │                          │                 │  │
     │  │              │              ┌───────────┴───────────┐     │  │
     │  │              │              ▼                       ▼     │  │
     │  │              │         有待搜索              没有待搜索   │  │
     │  │              │              │                       │     │  │
     │  │              │              ▼                       ▼     │  │
     │  │              │   ┌──────────────────┐     ┌─────────────┐ │  │
     │  │              │   │ 执行搜索 (API)   │     │ 任务完成    │ │  │
     │  │              │   └────────┬─────────┘     │ 退出循环    │ │  │
     │  │              │            │               └─────────────┘ │  │
     │  │              │            ▼                                │  │
     │  │              │   ┌──────────────────┐                      │  │
     │  │              │   │ 过滤 URL (API)   │                      │  │
     │  │              │   │ - 排除广告域名   │                      │  │
     │  │              │   │ - 排除已访问     │                      │  │
     │  │              │   │ - AI 判断相关性  │                      │  │
     │  │              │   └────────┬─────────┘                      │  │
     │  │              │            │                                │  │
     │  │              │            ▼                                │  │
     │  │              │   ┌──────────────────┐                      │  │
     │  │              │   │ 保存到数据库     │                      │  │
     │  │              │   │ 更新搜索记录     │─────────────────────┐│  │
     │  │              │   └──────────────────┘                     ││  │
     │  │              │                                       继续││  │
     │  │              │                                        循环││  │
     │  │              └────────────────────────────────────────────┘│  │
     │  │                                                              │  │
     │  └──────────────────────────────────────────────────────────────┘  │
     │                                    │                               │
     └────────────────────────────────────┼───────────────────────────────┘
                                          │
                                          ▼
                            ┌──────────────────┐
                            │  访问 URL (API)  │
                            │                  │
                            │  分层策略:       │
                            │  HTTP → Playwright│
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │ 内容类型判断     │
                            │ html/pdf/other   │
                            └────────┬─────────┘
                                     │
                   ┌─────────────────┼─────────────────┐
                   ▼                 ▼                 ▼
             ┌─────────┐       ┌─────────┐       ┌─────────┐
             │  HTML   │       │   PDF   │       │  Other  │
             │ 转MD    │       │ 保存    │       │ 转MD    │
             │ 提取图片│       │ 提取图片│       │ 提取图片│
             └────┬────┘       └────┬────┘       └────┬────┘
                  │                 │                 │
                  └─────────────────┼─────────────────┘
                                    ▼
                            ┌──────────────────┐
                            │ 创建 article     │
                            │ 关联 keyword     │
                            │ 关联 images      │
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │ 关键词提取 (API) │
                            │ 使用项目描述     │
                            │ 作为 prompt 上下文│
                            │ - 提取新关键词   │
                            │ - 发现关键词关系 │
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │ 更新数据库       │
                            │ - 标记URL已访问  │
                            │ - 保存新关键词   │
                            │ - 更新搜索记录   │
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │   任务结束       │
                            └──────────────────┘
```

### Layered Crawler Strategy (分层爬虫策略)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         URL 访问分层策略                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         URL 访问请求                                │   │
│   └──────────────────────────────┬──────────────────────────────────────┘   │
│                                  │                                          │
│                                  ▼                                          │
│                     ┌────────────────────────┐                              │
│                     │  第一层: HTTP 请求     │                              │
│                     │  (aiohttp / requests)  │                              │
│                     └───────────┬────────────┘                              │
│                                 │                                           │
│                  ┌──────────────┴──────────────┐                            │
│                  ▼                             ▼                            │
│            成功获取内容                  失败或内容无效                      │
│            (检查是否有效)                      │                             │
│                  │                             │                             │
│                  │                             ▼                             │
│                  │               ┌────────────────────────┐                  │
│                  │               │  第二层: Playwright    │                  │
│                  │               │  (无头浏览器渲染)       │                  │
│                  │               └───────────┬────────────┘                  │
│                  │                           │                               │
│                  │              ┌────────────┴────────────┐                  │
│                  │              ▼                         ▼                  │
│                  │         成功获取                 失败                     │
│                  │              │                         │                  │
│                  │              │                         ▼                  │
│                  │              │               ┌──────────────────┐         │
│                  │              │               │ 标记为受限资源    │         │
│                  │              │               │ (需登录/付费)     │         │
│                  │              │               └──────────────────┘         │
│                  │              │                                        │
│                  └──────────────┴────────────────────────────────────────┘  │
│                                         │                                   │
│                                         ▼                                   │
│                               ┌──────────────────┐                           │
│                               │   内容处理       │                           │
│                               │   (HTML→MD)      │                           │
│                               └──────────────────┘                           │
│                                                                             │
│   有效性检查规则:                                                           │
│   - 空页面或 JS 框架占位符 → 需要第二层                                     │
│   - Cloudflare/反爬拦截 → 需要第二层                                        │
│   - 需要登录页面 → 标记为受限资源                                           │
│   - 内容长度 < 100 字符 → 可能是错误页                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Task Division (任务分工)

| 操作类型 | 执行方式 | 示例命令 |
|---------|---------|---------|
| 查询待访问 URL | Python CLI | `python cli/query.py --pending-urls` |
| 查询待搜索组合 | Python CLI | `python cli/query.py --pending-searches` |
| 查询项目状态 | Python CLI | `python cli/query.py --status` |
| 执行搜索 | FastAPI | `POST /api/search` |
| URL 过滤判断 | FastAPI | `POST /api/urls/filter` |
| 访问 URL | FastAPI | `POST /api/visit-url` |
| 关键词提取 | FastAPI | `POST /api/keywords/extract` |
| 翻译 | FastAPI | `POST /api/translate` |

## Data Model

### 实体关系图

```
┌──────────────┐         ┌──────────────┐
│   keywords   │◀───────▶│search_portals│
└──────┬───────┘         └──────────────┘
       │                        │
       │ search_records         │
       │ (M:N 搜索记录)          │
       │                        │
       │         ┌──────────────┴──────────────┐
       │         │                             │
       ▼         ▼                             ▼
┌──────────────┐                      ┌──────────────┐
│     urls     │◀─────────────────────│  articles    │
└──────┬───────┘  keyword_url_rel     └──────┬───────┘
       │                                      │
       │ url_resource_groups                  │ keyword_article_rel
       │                                      │
       ▼                                      ▼
┌──────────────┐                      ┌──────────────┐
│resource_groups│                     │    images    │
└──────────────┘                      └──────────────┘
       
┌──────────────┐
│keyword_relations│  (论文-作者、术语-缩写等)
│  (AI发现)      │
└──────────────┘
```

### SQL Schema

```sql
-- ============================================
-- 1. 核心实体表
-- ============================================

-- 关键词表（支持多种类型）
CREATE TABLE keywords (
    id INTEGER PRIMARY KEY,
    text TEXT NOT NULL UNIQUE,           -- 关键词文本
    keyword_type TEXT,                   -- person, place, term, paper, book, abbreviation, unknown_language
    language TEXT,                       -- zh, en, es, unknown
    description TEXT,                    -- 描述
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- 搜索入口表
CREATE TABLE search_portals (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,                  -- Google, Bing, Semantic Scholar
    base_url TEXT,
    portal_type TEXT,                    -- search_engine, academic, forum, website
    is_active BOOLEAN DEFAULT 1,
    config TEXT                          -- JSON配置
);

-- URL表（独立实体，去重存储）
CREATE TABLE urls (
    id INTEGER PRIMARY KEY,
    url TEXT NOT NULL UNIQUE,            -- URL唯一
    domain TEXT,                         -- 域名
    title TEXT,
    snippet TEXT,
    content_type TEXT,                   -- html, pdf, doc, etc.
    access_status TEXT DEFAULT 'pending',-- pending, success, failed
    failure_reason TEXT,                 -- 详细失败原因
    -- failure_reason 示例值：
    -- 'http_404', 'http_403_forbidden', 'timeout', 'network_error'
    -- 'ai_filtered: irrelevant', 'ai_filtered: paywall', 'ai_filtered: low_quality'
    created_at TIMESTAMP,
    accessed_at TIMESTAMP
);

-- 原文/文档表（存储处理后的内容）
CREATE TABLE articles (
    id INTEGER PRIMARY KEY,
    url_id INTEGER,                      -- 来源URL
    article_type TEXT,                   -- webpage_md, pdf, book, paper
    title TEXT,
    local_path TEXT,                     -- 本地文件路径 (.md 或 .pdf)
    is_full_version BOOLEAN DEFAULT 0,   -- 是否完整版（vs 摘要/部分）
    text_content TEXT,                   -- 纯文本内容（可选，用于搜索）
    created_at TIMESTAMP,
    FOREIGN KEY (url_id) REFERENCES urls(id)
);

-- 图片表
CREATE TABLE images (
    id INTEGER PRIMARY KEY,
    original_url TEXT,
    local_path TEXT,                     -- 本地存储路径
    alt_text TEXT,
    context_text TEXT,                   -- 周围文本
    article_id INTEGER,                  -- 所属原文
    created_at TIMESTAMP,
    FOREIGN KEY (article_id) REFERENCES articles(id)
);

-- 资源组表（同一资源的多个下载源，用于论文/书籍）
CREATE TABLE resource_groups (
    id INTEGER PRIMARY KEY,
    canonical_name TEXT,                 -- 资源名称（论文标题/书名）
    resource_type TEXT,                  -- paper, book
    is_downloaded BOOLEAN DEFAULT 0,     -- 是否已获得完整版
    best_article_id INTEGER,             -- 成功下载的article
    created_at TIMESTAMP
);

-- ============================================
-- 2. 关联表（M:N 关系）
-- ============================================

-- 关键词 × 搜索入口 = 搜索记录
CREATE TABLE search_records (
    id INTEGER PRIMARY KEY,
    keyword_id INTEGER NOT NULL,
    portal_id INTEGER NOT NULL,
    status TEXT DEFAULT 'pending',       -- pending, searching, completed, failed
    result_count INTEGER,
    searched_at TIMESTAMP,
    UNIQUE(keyword_id, portal_id),       -- 每个组合只记录一次
    FOREIGN KEY (keyword_id) REFERENCES keywords(id),
    FOREIGN KEY (portal_id) REFERENCES search_portals(id)
);

-- 关键词 × URL = 提及关系
CREATE TABLE keyword_url_relations (
    id INTEGER PRIMARY KEY,
    keyword_id INTEGER NOT NULL,
    url_id INTEGER NOT NULL,
    relation_source TEXT,                -- search_result, page_content, citation
    relevance_score REAL,                -- 相关度评分（可选）
    created_at TIMESTAMP,
    FOREIGN KEY (keyword_id) REFERENCES keywords(id),
    FOREIGN KEY (url_id) REFERENCES urls(id)
);

-- 关键词 × 原文 = 关联关系
CREATE TABLE keyword_article_relations (
    id INTEGER PRIMARY KEY,
    keyword_id INTEGER NOT NULL,
    article_id INTEGER NOT NULL,
    is_primary BOOLEAN DEFAULT 0,        -- 是否为主要关键词
    created_at TIMESTAMP,
    FOREIGN KEY (keyword_id) REFERENCES keywords(id),
    FOREIGN KEY (article_id) REFERENCES articles(id)
);

-- 关键词 × 关键词 = 关键词关系（AI发现 + 翻译关系）
CREATE TABLE keyword_relations (
    id INTEGER PRIMARY KEY,
    from_keyword_id INTEGER NOT NULL,
    to_keyword_id INTEGER NOT NULL,
    relation_type TEXT,                  -- author_of, abbreviation_of, related_to, translation_of
                                          -- translation_of: 所有非锚点语言指向锚点语言
    source TEXT,                         -- 关系来源（哪个页面发现的 / auto_translate）
    confidence REAL,                     -- 置信度（翻译关系为 1.0）
    created_at TIMESTAMP,
    FOREIGN KEY (from_keyword_id) REFERENCES keywords(id),
    FOREIGN KEY (to_keyword_id) REFERENCES keywords(id)
);

-- URL × 资源组（用于论文/书籍的多源聚合）
CREATE TABLE url_resource_groups (
    url_id INTEGER NOT NULL,
    group_id INTEGER NOT NULL,
    PRIMARY KEY (url_id, group_id),
    FOREIGN KEY (url_id) REFERENCES urls(id),
    FOREIGN KEY (group_id) REFERENCES resource_groups(id)
);

-- ============================================
-- 3. 索引
-- ============================================

CREATE INDEX idx_urls_status ON urls(access_status);
CREATE INDEX idx_search_records_status ON search_records(status);
CREATE INDEX idx_keyword_relations_type ON keyword_relations(relation_type);
CREATE INDEX idx_articles_type ON articles(article_type);
CREATE INDEX idx_keywords_type ON keywords(keyword_type);
```

### 核心设计决策

| 决策 | 说明 |
|------|------|
| **Agent 驱动** | 纯净 Agent 实例，从数据库读取状态，执行单次原子任务，无需记忆 |
| **分层爬虫** | 优先 HTTP，失败或无效时降级到 Playwright |
| **项目配置** | config.yaml 配置语言表、初始关键词、项目描述；第一个语言为锚点语言 |
| **项目描述** | 存于 config.yaml，空时 Agent 自动根据初始关键词生成 |
| **关键词多类型** | 支持 person/place/term/paper/book/abbreviation/unknown_language |
| **翻译规则** | paper/book/person/abbreviation 不翻译；未知语言仅从 config.yaml 翻译 |
| **翻译服务** | 简化版：使用 gpt-4o-mini，失败跳过，前端提供手动管理 |
| **翻译关系** | translation_of 关系，所有非锚点语言指向锚点语言 |
| **M:N 关系** | keyword-url、keyword-portal、keyword-article 均为多对多关系 |
| **原文独立存储** | articles 表存储处理后的 md/pdf，与 urls 分离 |
| **资源组聚合** | 同一论文/书籍的多个下载源通过 resource_groups 聚合 |
| **URL 失败记录** | 使用 failure_reason 记录详细失败原因（网络错误 + AI 判断） |
| **关键词提取** | 独立模块，使用 project_description 作为 prompt 上下文判断有效性 |
| **AI 发现关系** | keyword_relations 存储论文-作者、术语-缩写、翻译等关系 |
| **任务分工** | 简单查询用 Python CLI；复杂操作用 FastAPI |

## Project Configuration (config.yaml)

```yaml
# =============================================================================
# config.yaml - 项目配置
# =============================================================================

# 项目信息
project:
  name: "Keyword Web Crawler"
  description: ""  # Agent 第一次运行时自动生成，或手动填写
  # description: "搜索关于青蒿素 的研究资料..."

# Agent 行为配置
agent:
  default_run_count: 10       # 默认执行次数
  task_timeout: 300           # 单任务超时 (秒)
  retry_on_failure: true      # 失败时是否重试
  max_retries: 3              # 最大重试次数

# 搜索语言配置 - 第一个语言为锚点语言
search:
  languages:
    - "en"    # 锚点语言
    - "zh"
    - "es"
  portals:
    - name: "Google"
      type: "search_engine"
      enabled: true
    - name: "Semantic Scholar"
      type: "academic"
      enabled: true
    - name: "arXiv"
      type: "academic"
      enabled: true

# 爬虫配置
crawler:
  strategy: "layered"         # layered: 先HTTP再Playwright
  http_timeout: 30
  playwright_timeout: 60
  request_delay: 2            # 请求间隔 (秒)
  user_agent_rotation: true   # 是否轮换 User-Agent

# 翻译配置 (简化版)
translation:
  enabled: true
  provider: "openai"          # openai / free / google / deepl
  model: "gpt-4o-mini"        # 使用的模型
  # api_key: "${OPENAI_API_KEY}"  # 或环境变量
  cache_enabled: true
  fail_silently: true         # 失败时跳过，不报错

# AI 配置 (关键词提取)
ai:
  keyword_extraction:
    enabled: true
    model: "claude-3-haiku"   # 或 gpt-4o-mini
    confidence_threshold: 0.7 # 低于此值的关键词不保存
    # 提取时会自动使用 project.description 作为 prompt 上下文

# 初始关键词
initial_keywords:
  - text: "青蒿素"
    type: "term"
  - text: "Artemisinin"
    type: "term"
```

### Agent 自动生成项目描述

当 `project.description` 为空或不存在时，Agent 会：

1. 读取 `initial_keywords` 列表
2. 分析关键词文本和类型
3. 调用 LLM 生成描述 (50字以内)
4. 将描述写入 `config.yaml`

示例 Prompt:
```
根据以下初始关键词，生成一个项目描述 (50字以内):
关键词: 青蒿素、Artemisinin
类型: term

生成结果:
搜索关于青蒿素 的研究资料，包括其化学结构、药理作用、临床应用和相关论文。
```

## Translation Strategy (简化翻译策略)

### 翻译流程

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     简化的翻译服务                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   config.yaml 配置:                                                         │
│   translation:                                                              │
│     enabled: true                                                           │
│     provider: "openai"                                                      │
│     model: "gpt-4o-mini"                                                    │
│     fail_silently: true                                                     │
│                                                                             │
│   流程:                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  1. 检查缓存 → 命中则直接返回                                        │   │
│   │  2. 调用 gpt-4o-mini 翻译                                           │   │
│   │  3. 成功 → 缓存结果并返回                                            │   │
│   │  4. 失败 → 返回 None，标记为未翻译                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 翻译规则 (保留)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     关键词翻译决策树                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  输入关键词 "X"，来源 S，检测语言 L，类型 T                                  │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────────────────────────────────┐                           │
│  │ T in [paper, book, person, abbreviation] ?  │                           │
│  └─────────────────────┬───────────────────────┘                           │
│                    ┌───┴───┐                                                │
│                    ▼       ▼                                                │
│                   是       否 ─────────────────────┐                       │
│                    │                              │                        │
│                    ▼                              ▼                        │
│               不翻译            ┌──────────────────────────────┐           │
│                               │ 来源 S = 'config.yaml' ?      │           │
│                               └────────────────┬─────────────┘           │
│                                    ┌────────────┴──────────┐             │
│                                    ▼                       ▼             │
│                                   是                      否            │
│                                    │                       │             │
│                                    ▼                       ▼             │
│                              翻译到所有      ┌──────────────────┐        │
│                              语言列表目标    │ L 在语言列表中？ │        │
│                                             └────────┬─────────┘        │
│                                               ┌──────┴──────┐           │
│                                               ▼             ▼           │
│                                              是             否          │
│                                               │              │           │
│                                               ▼              ▼           │
│                                          翻译到        不翻译           │
│                                          其他列表    (保持原样)         │
│                                          中语言                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 翻译关系网络

```
语言表 = ["en", "es", "zh"]  ← en 是锚点（第一个）

                    ┌──────────────────┐
                    │  Hydrocotyle     │
                    │  (en, 锚点)       │
                    └────────┬─────────┘
                             │
              translation_of │ translation_of
                    ┌────────┴────────┐
                    ▼                 ▼
           ┌────────────────┐  ┌────────────────┐
           │  Hidrocotile   │  │   天胡荽        │
           │  (es)          │  │   (zh)         │
           └────────────────┘  └────────────────┘

查询规则：
- 从锚点语言查询 → 直接找到所有翻译
- 从非锚点语言查询 → 先找到锚点，再找到所有翻译
```

## Risks / Tradeoffs

| Risk | Mitigation |
|------|------------|
| 爬虫被封禁 | 使用代理池、请求间隔、User-Agent 轮换 |
| 翻译 API 费用 | 实现缓存机制，避免重复翻译 |
| 大量图片占用存储 | 支持图片压缩和清理策略 |
| AI API 响应慢 | 异步处理，显示进度条 |
| SQLite 并发限制 | 使用 WAL 模式，后端单进程处理写操作 |

## Migration Plan

1. **Phase 1**: 搭建基础框架（API + 数据库 + 爬虫引擎）
2. **Phase 2**: 实现核心功能（搜索、翻译、图片关联）
3. **Phase 3**: 构建前端界面
4. **Phase 4**: AI 集成和优化

**Rollback**: SQLite 数据库支持快照备份，可直接回滚。
