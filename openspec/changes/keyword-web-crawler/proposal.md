# Proposal: Keyword Web Crawler

## Why

需要构建一个智能的网络信息采集系统，能够从初始关键词出发，自动发现、搜索、收集并组织相关的网络信息。该系统需要支持多语言关键词转换、图片与关键词关联、学术资源发现等功能，并通过 React 前端提供用户友好的管理界面。

这是一个自动化知识收集和管理工具，适用于研究人员、内容创作者、或需要深度信息收集的场景。

## What Changes

### 新增功能
- **关键词搜索引擎**: 支持从多种入口（Google、学术搜索引擎、论坛等）进行自动搜索
- **多语言转换系统**: 自动识别输入关键词语言，转换为多种搜索语言，并以用户指定语言显示结果
- **土著/未知语言检测**: 自动发现未识别语言的关键词，不要翻译，直接搜索
- **图片关联系统**: 从上下文自动识别图片与关键词的关系，保存并建立关联
- **关键词发现与扩展**: 从搜索结果中自动发现新关键词及其描述、相关图片
- **学术资源发现**: 识别论文、书籍、作者等信息作为学术搜索的关键词来源
- **搜索入口发现**: 自动发现相关网站、论坛等作为除 Google 外的补充搜索入口
- **状态追踪系统**: 记录已完成和待处理的搜索任务，支持增量执行
- **付费/受限资源记录**: 标记需要登录或付费的资源，等待后续处理
- **AI 辅助入口**: 集成 Claude、Copilot 等 AI 工具作为自动化入口

### 技术栈
- **后端**: Python (爬虫、数据处理、AI 集成)
- **数据库**: SQLite (关键词、URL、图片、描述等持久化存储)
- **前端**: React (用户界面)
- **API**: RESTful API 连接前后端

## Capabilities

### New Capabilities

- `keyword-management`: 关键词的创建、更新、删除、查询，包括语言识别、类型分类（人名/地名/术语/论文/书籍/缩写/未知语言）、描述管理
- `web-search`: 执行网络搜索，支持多种搜索引擎和自定义搜索入口
- `academic-search`: 学术资源搜索，包括论文、书籍、作者信息的发现和检索
- `language-translation`: 多语言翻译功能，支持输入语言识别、搜索语言转换、显示语言设置
- `image-association`: 图片下载、存储，以及与原文和关键词的上下文关联建立
- `crawl-state-management`: 搜索状态追踪，任务队列管理，增量执行支持，关键词×搜索入口组合管理
- `article-processing`: 原文处理，包括网页转MD、PDF存储、图片提取、与关键词关联
- `resource-bookmark`: 受限资源（付费/需登录）的记录和后续处理管理
- `resource-aggregation`: 论文/书籍多源聚合，资源组管理，完整版下载状态追踪
- `keyword-relation-discovery`: AI辅助发现关键词关系（论文-作者、术语-缩写等）
- `data-storage`: SQLite 数据库存储，包括 M:N 关系、去重、索引优化
- `web-ui`: React 前端界面，支持关键词输入、搜索结果查看、原文浏览、图片管理、URL状态批量修改

### Modified Capabilities

(无 - 这是新建项目)

## Impact

### 新建组件
- `backend/` - Python 后端服务
  - ` crawlers/` - 爬虫模块
  - `api/` - RESTful API
  - `services/` - 业务逻辑
  - `models/` - 数据模型
- `frontend/` - React 前端应用
- `database/` - SQLite 数据库和迁移脚本
- `storage/` - 图片和原始数据存储

### 外部依赖
- 翻译 API (如 Google Translate、DeepL)
- AI API (Claude、Copilot)
- 搜索引擎 API
- 学术搜索 API (如 Semantic Scholar、arXiv)

### 系统要求
- Python 3.10+
- Node.js 18+
- SQLite 3
