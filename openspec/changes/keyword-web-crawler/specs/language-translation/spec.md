# Spec: Language Translation

## ADDED Requirements

### Requirement: Use simplified translation service
系统 SHALL 使用简化的翻译服务，通过可配置的低成本模型实现。

#### Scenario: Translate with gpt-4o-mini
- **WHEN** 翻译服务配置为 `provider: "openai"`，`model: "gpt-4o-mini"`
- **THEN** 系统使用 gpt-4o-mini 进行翻译

#### Scenario: Fail silently
- **WHEN** 翻译请求失败（网络错误、API 限制等）
- **THEN** 系统返回 None 并标记关键词为未翻译，不中断流程

#### Scenario: Skip translation if disabled
- **WHEN** 配置 `translation.enabled: false`
- **THEN** 系统跳过所有翻译操作

### Requirement: Configure translation provider
系统 SHALL 支持配置翻译服务提供商。

#### Scenario: Configure OpenAI provider
- **WHEN** 用户配置 `provider: "openai"` 和 `model: "gpt-4o-mini"`
- **THEN** 系统使用 OpenAI API 进行翻译

#### Scenario: Use environment variable for API key
- **WHEN** 配置 `api_key: "${OPENAI_API_KEY}"`
- **THEN** 系统从环境变量读取 API 密钥

### Requirement: Translate keyword based on rules
系统 SHALL 根据翻译规则决定是否翻译关键词。

#### Scenario: Translate keyword from config.yaml
- **WHEN** 关键词来源为 config.yaml，无论检测语言是否在列表中
- **THEN** 系统翻译为语言列表中的所有目标语言

#### Scenario: Translate discovered keyword in language list
- **WHEN** 爬取发现的关键词语言在语言列表中
- **THEN** 系统翻译为语言列表中的其他语言

#### Scenario: Do not translate discovered keyword not in language list
- **WHEN** 爬取发现的关键词 "waxaklahun" 语言不在列表中
- **THEN** 系统不翻译，直接使用原始文本

#### Scenario: Do not translate non-translatable types
- **WHEN** 关键词类型为 paper/book/person/abbreviation
- **THEN** 系统不翻译，即使语言在列表中

### Requirement: Establish translation relationships
系统 SHALL 为翻译后的关键词建立 translation_of 关系。

#### Scenario: All relationships point to anchor language
- **WHEN** 语言列表为 `["en", "es", "zh"]`，en 为锚点（第一个）
- **THEN** 所有非锚点语言的关键词通过 translation_of 指向锚点语言关键词

#### Scenario: Query all synonyms from any language
- **WHEN** 用户查询 "天胡荽"(zh) 的同义词
- **THEN** 系统通过 translation_of 关系找到锚点 "Hydrocotyle"(en)
- **AND** 通过锚点找到所有其他语言的同义词

### Requirement: Translate results for display
系统 SHALL 将搜索结果翻译为用户指定的显示语言。

#### Scenario: Translate to display language
- **WHEN** 用户设置 display-lang="zh"，搜索结果为英文
- **THEN** 系统将标题和摘要翻译为中文显示

### Requirement: Cache translations
系统 SHALL 缓存翻译结果以减少 API 调用。

#### Scenario: Use cached translation
- **WHEN** 相同文本需要翻译
- **THEN** 系统优先从缓存返回翻译结果

### Requirement: Detect keyword language
系统 SHALL 自动检测关键词的语言。

#### Scenario: Detect Chinese language
- **WHEN** 用户输入 "天胡荽"
- **THEN** 系统检测语言为 "zh"

#### Scenario: Detect English language
- **WHEN** 用户输入 "Hydrocotyle"
- **THEN** 系统检测语言为 "en"

#### Scenario: Detect unknown language
- **WHEN** 翻译 API 无法识别语言
- **THEN** 系统将关键词标记为 language="unknown"
