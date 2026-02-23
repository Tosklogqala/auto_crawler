# Spec: Project Configuration

## ADDED Requirements

### Requirement: Load project configuration
系统 SHALL 从 `config.yaml` 文件加载项目配置。

#### Scenario: Load configuration on startup
- **WHEN** 系统启动
- **THEN** 系统读取 `config.yaml` 文件并解析配置项

#### Scenario: Use default configuration
- **WHEN** `config.yaml` 文件不存在
- **THEN** 系统使用默认配置并创建默认配置文件

### Requirement: Auto-generate project description
系统 SHALL 支持自动生成项目描述，用于关键词提取的上下文判断。

#### Scenario: Generate description when empty
- **WHEN** `project.description` 字段为空或不存在
- **THEN** Agent 根据初始关键词自动生成描述（50字以内）

#### Scenario: Generate description using LLM
- **WHEN** 初始关键词为 ["青蒿素", "Artemisinin"]，类型为 term
- **THEN** Agent 调用 LLM 生成描述，如 "搜索关于青蒿素的研究资料..."

#### Scenario: Save generated description
- **WHEN** 描述生成完成
- **THEN** 系统将描述写入 `config.yaml` 的 `project.description` 字段

#### Scenario: Use existing description
- **WHEN** `project.description` 已有内容
- **THEN** 系统直接使用，不重新生成

### Requirement: Use description in keyword extraction
系统 SHALL 在关键词提取时使用项目描述作为 prompt 上下文。

#### Scenario: Include description in extraction prompt
- **WHEN** AI 提取关键词
- **THEN** 系统将 `project.description` 包含在 prompt 中，帮助 AI 判断关键词是否相关

### Requirement: Configure search languages
系统 SHALL 支持配置搜索语言列表。

#### Scenario: Define search languages
- **WHEN** 用户在 `config.yaml` 中设置 `search_languages: ["en", "es", "zh"]`
- **THEN** 系统将这三个语言作为翻译目标语言

#### Scenario: First language as anchor
- **WHEN** 语言列表为 `["en", "es", "zh"]`
- **THEN** 系统将 "en" 作为锚点语言，其他语言的关键词翻译关系都指向锚点语言

### Requirement: Configure initial keywords
系统 SHALL 支持配置初始关键词列表。

#### Scenario: Define initial keywords
- **WHEN** 用户在 `config.yaml` 中设置 `initial_keywords: ["天胡荽", "Hydrocotyle"]`
- **THEN** 系统在初始化时创建这些关键词

#### Scenario: Create initial keywords from config
- **WHEN** 系统初始化且配置包含初始关键词
- **THEN** 系统为每个初始关键词创建记录，并根据翻译规则自动翻译

### Requirement: Initialize keywords with translation
系统 SHALL 在初始化时对 config.yaml 中提供的关键词进行翻译处理。

#### Scenario: Translate initial keyword with language not in list
- **WHEN** 初始关键词为 "玛雅象形文字"（中文），但语言列表为 `["en", "es"]`
- **THEN** 系统翻译为英语和西班牙语，创建 "Maya hieroglyphs" 和 "Jeroglíficos mayas"

#### Scenario: Do not translate initial keyword of non-translatable type
- **WHEN** 初始关键词为 "On the Origin of Species"（论文标题）
- **THEN** 系统不翻译，只创建原词记录

### Requirement: Validate configuration
系统 SHALL 验证配置文件的有效性。

#### Scenario: Validate language codes
- **WHEN** 语言列表包含无效的语言代码
- **THEN** 系统报告配置错误并拒绝启动

#### Scenario: Validate empty language list
- **WHEN** 语言列表为空
- **THEN** 系统使用默认语言列表 `["en"]`
