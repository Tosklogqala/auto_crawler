# Spec: Data Storage

## ADDED Requirements

### Requirement: Initialize database
系统 SHALL 在首次运行时自动创建 SQLite 数据库和表结构。

#### Scenario: Create entity tables on first run
- **WHEN** 数据库文件不存在
- **THEN** 系统创建实体表：keywords, search_portals, urls, articles, images, resource_groups

#### Scenario: Create relation tables on first run
- **WHEN** 数据库文件不存在
- **THEN** 系统创建关联表：search_records, keyword_url_relations, keyword_article_relations, keyword_relations, url_resource_groups

### Requirement: Persist keyword with type
系统 SHALL 支持多种类型的关键词存储。

#### Scenario: Store keyword with type
- **WHEN** 创建关键词时指定类型
- **THEN** 系统将 keyword_type 存储为 person/place/term/paper/book/abbreviation/unknown_language

#### Scenario: Unique keyword text
- **WHEN** 尝试创建已存在的关键词
- **THEN** 系统拒绝创建，返回已存在的记录

### Requirement: Persist URL independently
系统 SHALL 独立存储 URL，支持去重和失败原因记录。

#### Scenario: Deduplicate URLs
- **WHEN** 多个来源发现相同 URL
- **THEN** 系统只存储一次，但创建多个关联记录

#### Scenario: Track URL access status
- **WHEN** URL 被访问后
- **THEN** 系统更新 access_status 为 pending/success/failed

#### Scenario: Record AI-filtered URL failure
- **WHEN** AI 在搜索阶段判断 URL 无效
- **THEN** 系统记录 access_status="failed"，failure_reason="ai_filtered: <原因>"

#### Scenario: Record content evaluation failure
- **WHEN** AI 判断 URL 内容无效（无有效信息、付费墙、低质量等）
- **THEN** 系统记录 failure_reason 为具体原因

#### Scenario: Record network failure
- **WHEN** URL 访问遇到网络错误
- **THEN** 系统记录 failure_reason 为 "http_404" / "timeout" / "network_error" 等

### Requirement: Persist articles
系统 SHALL 存储处理后的原文内容。

#### Scenario: Store article with local file
- **WHEN** 网页内容转换为 MD 或下载 PDF
- **THEN** 系统创建 article 记录，存储 local_path

#### Scenario: Mark full version
- **WHEN** 下载完整版论文/书籍
- **THEN** 系统设置 is_full_version=true

### Requirement: Persist keyword-portal search records
系统 SHALL 记录每个关键词在每个搜索入口的搜索状态。

#### Scenario: Create search record
- **WHEN** 对关键词 A 在搜索入口 B 进行搜索
- **THEN** 系统创建 search_records 记录，关联 keyword_id 和 portal_id

#### Scenario: Prevent duplicate search records
- **WHEN** 重复搜索同一组合
- **THEN** 系统更新现有记录而非创建新记录（UNIQUE约束）

### Requirement: Persist M:N relations
系统 SHALL 支持多对多关系的存储。

#### Scenario: Store keyword-url relation
- **WHEN** URL 内容中提及关键词
- **THEN** 系统创建 keyword_url_relations 记录

#### Scenario: Store keyword-article relation
- **WHEN** article 与关键词相关
- **THEN** 系统创建 keyword_article_relations 记录

#### Scenario: Store keyword-keyword relation
- **WHEN** AI 发现两个关键词存在关系
- **THEN** 系统创建 keyword_relations 记录，包含 relation_type 和 confidence

#### Scenario: Store translation relation
- **WHEN** 系统自动翻译关键词创建同义词
- **THEN** 系统创建 keyword_relations 记录，relation_type="translation_of"，confidence=1.0

#### Scenario: Translation relation points to anchor
- **WHEN** 语言列表锚点为 "en"，翻译创建 "天胡荽"(zh)
- **THEN** 系统创建关系：天胡荽 → Hydrocotyle，relation_type="translation_of"

### Requirement: Support resource group aggregation
系统 SHALL 支持论文/书籍的多源聚合。

#### Scenario: Aggregate same resource URLs
- **WHEN** 多个 URL 指向同一论文/书籍
- **THEN** 系统创建 resource_group 并关联所有 URL

#### Scenario: Mark group as downloaded
- **WHEN** 任一 URL 成功下载完整版
- **THEN** 系统设置 resource_group.is_downloaded=true，更新 best_article_id

### Requirement: Support CRUD operations
系统 SHALL 支持所有表的创建、读取、更新、删除操作。

#### Scenario: Query with joins
- **WHEN** 查询关键词及其关联数据
- **THEN** 系统支持 JOIN 查询返回完整数据

### Requirement: Backup and restore
系统 SHALL 支持数据库备份和恢复。

#### Scenario: Create backup
- **WHEN** 用户请求备份
- **THEN** 系统创建数据库快照文件

#### Scenario: Restore from backup
- **WHEN** 用户选择恢复备份
- **THEN** 系统从快照文件恢复数据库

### Requirement: Query performance
系统 SHALL 对常用查询建立索引以保证性能。

#### Scenario: Index on access_status
- **WHEN** 按 access_status 查询未访问 URL
- **THEN** 查询使用 idx_urls_status 高效执行

#### Scenario: Index on search_records status
- **WHEN** 查询待处理的搜索记录
- **THEN** 查询使用 idx_search_records_status 高效执行

