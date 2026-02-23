# Spec: Academic Search

## ADDED Requirements

### Requirement: Search academic resources
系统 SHALL 能够搜索学术资源（论文、书籍、作者）。

#### Scenario: Search papers
- **WHEN** 系统对关键词进行学术搜索
- **THEN** 系统从 Semantic Scholar/arXiv 等来源返回相关论文列表

#### Scenario: Extract author information
- **WHEN** 学术搜索结果包含作者信息
- **THEN** 系统将作者名作为新关键词记录，来源标记为 "academic"

### Requirement: Discover academic keywords
系统 SHALL 从学术资源中发现新的关键词。

#### Scenario: Extract paper keywords
- **WHEN** 论文包含关键词标签
- **THEN** 系统将这些关键词添加到关键词表

#### Scenario: Extract book references
- **WHEN** 论文引用了相关书籍
- **THEN** 系统将书籍名和作者作为新关键词记录

### Requirement: Download academic resources
系统 SHALL 尝试下载可访问的学术资源。

#### Scenario: Download open access paper
- **WHEN** 论文是开放获取的
- **THEN** 系统下载 PDF 并存储本地路径

#### Scenario: Handle paywalled paper
- **WHEN** 论文需要付费访问
- **THEN** 系统标记 requires_payment=true，记录在 resource_bookmark 表
