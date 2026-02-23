# Spec: Resource Aggregation

## ADDED Requirements

### Requirement: Create resource group
系统 SHALL 为同一资源的多个下载源创建资源组。

#### Scenario: Group same paper URLs
- **WHEN** 多个 URL 指向同一篇论文（通过标题/DOI匹配）
- **THEN** 系统创建 resource_group，关联所有 URL

#### Scenario: Group same book URLs
- **WHEN** 多个 URL 指向同一本书
- **THEN** 系统创建 resource_group，关联所有 URL

### Requirement: Track download status
系统 SHALL 追踪资源组的下载状态。

#### Scenario: Mark as downloaded
- **WHEN** 任一关联 URL 成功下载完整版
- **THEN** 系统设置 is_downloaded=true，记录 best_article_id

#### Scenario: Mark other URLs as success
- **WHEN** 资源组标记为已下载
- **THEN** 系统将该组所有 URL 的 access_status 更新为 success

### Requirement: Identify resource duplicates
系统 SHALL 识别重复资源。

#### Scenario: Match by DOI
- **WHEN** URL 包含 DOI 信息
- **THEN** 系统通过 DOI 匹配已有资源组

#### Scenario: Match by title
- **WHEN** 无 DOI 但标题高度相似
- **THEN** 系统通过标题相似度匹配已有资源组

### Requirement: Prioritize download sources
系统 SHALL 为资源组选择最佳下载源。

#### Scenario: Select open access source
- **WHEN** 存在开放获取的 URL
- **THEN** 系统优先选择该 URL 作为首选下载源

#### Scenario: Avoid paywalled sources
- **WHEN** 所有源都需要付费
- **THEN** 系统记录并等待用户处理
