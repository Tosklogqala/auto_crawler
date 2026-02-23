# Spec: Keyword Relation Discovery

## ADDED Requirements

### Requirement: Use project description as context
系统 SHALL 在提取关键词时使用项目描述作为判断上下文。

#### Scenario: Filter irrelevant keywords
- **WHEN** AI 从内容中提取关键词，项目描述为 "青蒿素研究..."
- **THEN** AI 排除广告、导航链接等与描述无关的关键词

#### Scenario: Extract relevant keywords with confidence
- **WHEN** AI 提取关键词
- **THEN** AI 同时给出置信度分数（0-1），低于阈值的不保存

### Requirement: Modular keyword extraction
系统 SHALL 将关键词提取设计为独立、可扩展的模块。

#### Scenario: Call extraction module independently
- **WHEN** 后端需要提取关键词
- **THEN** 系统调用独立的 keyword_extractor 模块

#### Scenario: Support future knowledge graph extension
- **WHEN** 需要构建知识图谱
- **THEN** keyword_extractor 模块可独立扩展为知识图谱构建器

### Requirement: Discover author-paper relations
系统 SHALL 使用 AI 发现论文与作者的关系。

#### Scenario: Extract authors from paper
- **WHEN** AI 分析论文元数据或全文
- **THEN** 系统提取作者名作为新关键词，创建 author_of 关系

#### Scenario: Store confidence score
- **WHEN** 创建关键词关系
- **THEN** 系统记录 AI 置信度分数（0-1）

### Requirement: Discover abbreviation relations
系统 SHALL 发现术语与其缩写的关系。

#### Scenario: Match abbreviation
- **WHEN** AI 识别文本中 "A (B)" 或 "A, also known as B" 模式
- **THEN** 系统创建 abbreviation_of 关系

### Requirement: Discover related terms
系统 SHALL 发现相关术语关系。

#### Scenario: Identify related concepts
- **WHEN** AI 分析内容发现经常共现的概念
- **THEN** 系统创建 related_to 关系

### Requirement: Record relation source
系统 SHALL 记录关系的发现来源。

#### Scenario: Store source URL
- **WHEN** 创建关键词关系
- **THEN** 系统记录发现该关系的 URL 或 article_id

### Requirement: Support relation types
系统 SHALL 支持多种关系类型。

#### Scenario: Author relation
- **WHEN** 发现人物创作了作品
- **THEN** relation_type = author_of

#### Scenario: Abbreviation relation
- **WHEN** 发现术语的缩写
- **THEN** relation_type = abbreviation_of

#### Scenario: Related relation
- **WHEN** 发现相关的概念
- **THEN** relation_type = related_to
