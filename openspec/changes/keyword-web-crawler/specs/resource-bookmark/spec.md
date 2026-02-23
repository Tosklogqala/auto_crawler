# Spec: Resource Bookmark

## ADDED Requirements

### Requirement: Record inaccessible resources
系统 SHALL 记录需要登录或付费的资源。

#### Scenario: Record paywalled resource
- **WHEN** 搜索结果需要付费才能访问完整内容
- **THEN** 系统标记 requires_payment=true，记录 URL 和说明

#### Scenario: Record login-required resource
- **WHEN** 资源需要登录才能访问
- **THEN** 系统标记 requires_auth=true，记录 URL 和说明

### Requirement: List bookmarked resources
系统 SHALL 提供查看受限资源列表的功能。

#### Scenario: List all bookmarks
- **WHEN** 用户请求查看受限资源
- **THEN** 系统返回所有 requires_auth=true 或 requires_payment=true 的 URL

### Requirement: Support manual resolution
系统 SHALL 允许用户标记受限资源已处理。

#### Scenario: Mark as accessible
- **WHEN** 用户手动访问并获取了受限资源
- **THEN** 系统更新 is_accessible=true，清除受限标记

### Requirement: Retry bookmarked resources
系统 SHALL 支持重试之前受限的资源。

#### Scenario: Retry access
- **WHEN** 用户触发重试
- **THEN** 系统重新尝试访问该资源
