# Spec: Web Search

## ADDED Requirements

### Requirement: Execute web search
系统 SHALL 能够使用多个搜索引擎执行网络搜索。

#### Scenario: Search with Google
- **WHEN** 系统对关键词 "天胡荽" 执行 Google 搜索
- **THEN** 系统返回搜索结果列表，包含 URL、标题、摘要

#### Scenario: Search with custom portal
- **WHEN** 用户配置了自定义搜索入口（如特定论坛）
- **THEN** 系统使用该入口执行搜索并返回结果

### Requirement: Filter and judge URLs
系统 SHALL 在搜索后过滤和判断 URL 的有效性，使用项目描述作为上下文。

#### Scenario: Filter ad domains
- **WHEN** 搜索结果包含广告域名（如 doubleclick.net）
- **THEN** 系统排除这些 URL

#### Scenario: Filter already visited URLs
- **WHEN** 搜索结果包含已访问的 URL
- **THEN** 系统排除这些 URL，不重复添加

#### Scenario: AI judge relevance
- **WHEN** 搜索结果包含多个 URL
- **THEN** 系统使用 project_description 作为 prompt 上下文，让 AI 判断 URL 是否与项目相关

#### Scenario: Save valid URLs only
- **WHEN** 过滤完成后
- **THEN** 系统只保存有效且相关的 URL 到数据库

### Requirement: Discover search portals
系统 SHALL 从搜索结果中自动发现新的搜索入口（论坛、网站等）。

#### Scenario: Discover forum portal
- **WHEN** 搜索结果中出现特定领域的论坛链接
- **THEN** 系统将该论坛记录为新的 search_portal，类型为 "forum"

### Requirement: Handle search errors
系统 SHALL 正确处理搜索过程中的错误。

#### Scenario: Handle blocked request
- **WHEN** 搜索引擎返回 429 或封禁响应
- **THEN** 系统将任务标记为 "failed"，记录错误信息，并在延迟后重试

#### Scenario: Handle network timeout
- **WHEN** 搜索请求超时
- **THEN** 系统记录错误并标记任务需要重试

### Requirement: Save raw search results
系统 SHALL 保存原始搜索结果（HTML）以便后续分析。

#### Scenario: Save raw HTML
- **WHEN** 搜索完成
- **THEN** 系统将原始 HTML 存储在 urls 表的 raw_html 字段
