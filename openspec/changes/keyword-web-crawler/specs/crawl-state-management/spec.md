# Spec: Crawl State Management

## ADDED Requirements

### Requirement: Query state via Python CLI
系统 SHALL 提供简单的 Python CLI 用于查询数据库状态。

#### Scenario: Query pending URLs
- **WHEN** 执行 `python cli/query.py --pending-urls`
- **THEN** 系统返回 access_status="pending" 的 URL 列表

#### Scenario: Query pending searches
- **WHEN** 执行 `python cli/query.py --pending-searches`
- **THEN** 系统返回 status="pending" 的 keyword×portal 组合列表

#### Scenario: Query project status
- **WHEN** 执行 `python cli/query.py --status`
- **THEN** 系统返回项目整体状态概览（URL 数量、关键词数量、搜索进度等）

### Requirement: Track keyword-portal search status
系统 SHALL 追踪每个关键词在每个搜索入口的搜索状态。

#### Scenario: Create search record
- **WHEN** 新关键词创建后
- **THEN** 系统为每个活跃的搜索入口创建 search_record，状态为 "pending"

#### Scenario: Update search record status
- **WHEN** 搜索开始执行
- **THEN** 系统更新 search_record 状态为 "searching"

#### Scenario: Complete search record
- **WHEN** 搜索成功完成
- **THEN** 系统更新 search_record 状态为 "completed"，记录 result_count

### Requirement: Support incremental execution
系统 SHALL 支持增量执行任务，优先处理未访问的 URL。

#### Scenario: Process pending URLs first
- **WHEN** 存在 access_status="pending" 的 URL
- **THEN** 系统优先访问这些 URL

#### Scenario: Find next search combination
- **WHEN** 所有 URL 已访问
- **THEN** 系统查找 keyword×portal 组合中 status="pending" 的记录执行搜索

### Requirement: Select independent tasks
系统 SHALL 选择尽可能少但独立的任务执行。

#### Scenario: Select minimal task set
- **WHEN** 系统选择下一批任务
- **THEN** 系统选择不互相依赖的任务（不同关键词、不同入口）

#### Scenario: Resume interrupted searches
- **WHEN** 系统重启
- **THEN** 系统恢复 status="pending" 或 status="searching" 的搜索记录

### Requirement: Track URL access status
系统 SHALL 追踪 URL 的访问状态。

#### Scenario: Mark URL as pending
- **WHEN** 从搜索结果中发现新 URL
- **THEN** 系统设置 access_status="pending"

#### Scenario: Mark URL as success
- **WHEN** URL 成功访问并处理
- **THEN** 系统更新 access_status="success"，记录 accessed_at

#### Scenario: Mark URL as failed
- **WHEN** URL 访问失败
- **THEN** 系统更新 access_status="failed"，记录 failure_reason

### Requirement: Support manual status modification
系统 SHALL 支持用户手动修改 URL 状态以触发重新抓取。

#### Scenario: Batch reset failed URLs
- **WHEN** 用户批量将失败 URL 状态改为 pending
- **THEN** 系统将在下次任务周期重新访问这些 URL

### Requirement: Record search statistics
系统 SHALL 记录搜索统计信息。

#### Scenario: Store result count
- **WHEN** 搜索完成
- **THEN** 系统记录发现的 URL 数量到 result_count 字段
