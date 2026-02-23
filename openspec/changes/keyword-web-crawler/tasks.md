# Tasks: Keyword Web Crawler

## 1. Project Setup

- [ ] 1.1 创建项目目录结构 (backend/, frontend/, database/, storage/articles/, storage/images/, scripts/, logs/)
- [ ] 1.2 初始化 Python 后端项目 (requirements.txt, pyproject.toml)
- [ ] 1.3 初始化 React 前端项目 (package.json, vite.config.ts)
- [ ] 1.4 创建 config.yaml 配置文件模板
- [ ] 1.5 配置 SQLite 数据库连接和 WAL 模式
- [ ] 1.6 创建数据库迁移脚本和完整的表结构

## 2. Database Layer

- [ ] 2.1 实现实体表 CRUD: keywords (含类型分类)
- [ ] 2.2 实现实体表 CRUD: urls (独立去重存储)
- [ ] 2.3 实现实体表 CRUD: articles (原文存储)
- [ ] 2.4 实现实体表 CRUD: images
- [ ] 2.5 实现实体表 CRUD: search_portals
- [ ] 2.6 实现实体表 CRUD: resource_groups
- [ ] 2.7 实现关联表: search_records (keyword×portal)
- [ ] 2.8 实现关联表: keyword_url_relations
- [ ] 2.9 实现关联表: keyword_article_relations
- [ ] 2.10 实现关联表: keyword_relations
- [ ] 2.11 实现关联表: url_resource_groups
- [ ] 2.12 创建数据库索引优化查询性能

## 3. Backend API & CLI

- [ ] 3.1 创建 FastAPI 应用基础结构
- [ ] 3.2 Python CLI 查询工具 (backend/cli/query.py)
  - [ ] 3.2.1 --pending-urls 查询待访问 URL
  - [ ] 3.2.2 --pending-searches 查询待搜索组合
  - [ ] 3.2.3 --status 查询项目状态概览
- [ ] 3.3 实现 init.sh 服务启动脚本
- [ ] 3.4 实现 health check endpoint
- [ ] 3.5 实现 keyword-management API (含类型分类)
- [ ] 3.6 实现 web-search API (含 URL 过滤和 AI 判断)
- [ ] 3.7 实现 academic-search API
- [ ] 3.8 实现 article-processing API
- [ ] 3.9 实现 crawl-state-management API
- [ ] 3.10 实现 resource-bookmark API
- [ ] 3.11 实现 resource-aggregation API
- [ ] 3.12 实现 translation API

## 4. Crawler Engine (分层策略)

- [ ] 4.1 实现 HTTP 请求层 (aiohttp/requests)
- [ ] 4.2 实现 Playwright 浏览器自动化层
- [ ] 4.3 实现内容有效性检查逻辑
- [ ] 4.4 实现 URL 访问的自动降级策略 (HTTP → Playwright)
- [ ] 4.5 实现 Google 搜索爬虫
- [ ] 4.6 实现学术搜索爬虫 (Semantic Scholar, arXiv)
- [ ] 4.7 实现自定义搜索入口爬虫
- [ ] 4.8 实现请求限流和代理支持
- [ ] 4.9 实现错误处理和重试机制
- [ ] 4.10 实现 URL 去重和存储

## 5. Translation Service (简化版)

- [ ] 5.1 实现翻译服务抽象接口 (TranslationProvider)
- [ ] 5.2 实现 OpenAI gpt-4o-mini 翻译提供商
- [ ] 5.3 实现翻译缓存机制
- [ ] 5.4 实现 fail-silently 失败处理
- [ ] 5.5 实现语言检测功能 (langdetect)

## 6. Content Processing

- [ ] 6.1 实现 HTML 转 Markdown (提取主体内容)
- [ ] 6.2 实现 PDF 存储和图片提取
- [ ] 6.3 实现其他格式转换 (DOCX 等)
- [ ] 6.4 实现图片下载、存储和与原文关联
- [ ] 6.5 实现有效内容过滤 (广告页检测)
- [ ] 6.6 实现受限资源识别和标记

## 7. AI Integration (关键词提取模块)

- [ ] 7.1 创建独立的关键词提取模块 (keyword_extractor/)
- [ ] 7.2 实现翻译服务抽象基类和接口
- [ ] 7.3 集成 LLM API (Claude/GPT-4o-mini)
- [ ] 7.4 实现 project_description 作为 prompt 上下文
- [ ] 7.5 实现关键词类型自动分类
- [ ] 7.6 实现关键词关系发现 (论文-作者、缩写等)
- [ ] 7.7 实现内容分析和新关键词提取
- [ ] 7.8 实现置信度评分和过滤
- [ ] 7.9 Agent 自动生成 project_description 功能

## 8. Resource Aggregation

- [ ] 8.1 实现资源组创建和管理
- [ ] 8.2 实现 DOI/标题匹配去重
- [ ] 8.3 实现下载源优先级选择
- [ ] 8.4 实现完整版下载状态同步

## 9. Agent Support

- [ ] 9.1 实现 run_agent.sh 脚本 (支持 --count 和 --loop 参数)
- [ ] 9.2 实现 Agent 任务选择循环逻辑
- [ ] 9.3 实现 Agent prompt 模板
- [ ] 9.4 实现任务执行结果反馈

## 10. React Frontend

- [ ] 10.1 创建 React 项目结构和路由
- [ ] 10.2 实现关键词输入组件 (含类型选择)
- [ ] 10.3 实现搜索结果列表组件
- [ ] 10.4 实现原文浏览组件 (MD 渲染/PDF 查看)
- [ ] 10.5 实现图片展示组件
- [ ] 10.6 实现爬取进度矩阵组件 (keyword×portal)
- [ ] 10.7 实现受限资源管理页面
- [ ] 10.8 实现 URL 批量状态修改功能
- [ ] 10.9 实现关键词关系网络展示
- [ ] 10.10 实现翻译状态管理页面
- [ ] 10.11 实现 Agent 控制页面 (触发、进度、配置)
- [ ] 10.12 实现 API 服务层和状态管理

## 11. Integration & Testing

- [ ] 11.1 编写后端单元测试
- [ ] 11.2 编写 API 集成测试
- [ ] 11.3 编写前端组件测试
- [ ] 11.4 实现端到端测试
- [ ] 11.5 测试 Agent 完整流程
- [ ] 11.6 配置 CI/CD 流程

## 12. Documentation & Polish

- [ ] 12.1 编写 API 文档 (OpenAPI/Swagger)
- [ ] 12.2 编写用户使用文档
- [ ] 12.3 编写 Agent 使用指南
- [ ] 12.4 实现数据库备份恢复功能
- [ ] 12.5 性能优化和错误处理完善
