# Spec: Keyword Management

## ADDED Requirements

### Requirement: Create keyword with translation
系统 SHALL 在创建关键词时根据翻译规则自动创建多语言同义词。

#### Scenario: Create keyword with translation
- **WHEN** 从 config.yaml 创建关键词 "天胡荽"，语言列表为 `["en", "es", "zh"]`
- **THEN** 系统创建 "天胡荽"(zh)、"Hydrocotyle"(en)、"Hidrocotile"(es) 三个关键词
- **AND** 建立 translation_of 关系：天胡荽 → Hydrocotyle，Hidrocotile → Hydrocotyle

#### Scenario: Create discovered keyword with translation
- **WHEN** 爬取中发现关键词 "Hydrocotyle"（英语，在语言列表中）
- **THEN** 系统翻译为 "Hidrocotile"(es)、"天胡荽"(zh)
- **AND** 建立 translation_of 关系指向锚点语言

#### Scenario: Create keyword without translation for unknown language
- **WHEN** 爬取中发现关键词 "waxaklahun"（语言不在列表中）
- **THEN** 系统不翻译，只创建原词记录

#### Scenario: Create keyword with unknown language from config
- **WHEN** 从 config.yaml 创建关键词 "玛雅象形文字"，语言列表为 `["en", "es"]`
- **THEN** 系统翻译为英语和西班牙语，因为来源是 config.yaml

#### Scenario: Create keyword with unknown language discovered
- **WHEN** 爬取中发现关键词 "waxaklahun"（语言不在列表中）
- **THEN** 系统不翻译，只创建原词记录

### Requirement: Handle non-translatable keyword types
系统 SHALL 对特定类型的关键词不进行翻译。

#### Scenario: Do not translate paper title
- **WHEN** 创建论文标题关键词 "On the Origin of Species"
- **THEN** 系统设置 keyword_type="paper"，不进行翻译

#### Scenario: Do not translate book title
- **WHEN** 创建书籍名称关键词 "红楼梦"
- **THEN** 系统设置 keyword_type="book"，不进行翻译

#### Scenario: Do not translate person name
- **WHEN** 创建人名关键词 "Darwin"
- **THEN** 系统设置 keyword_type="person"，不进行翻译

#### Scenario: Do not translate abbreviation
- **WHEN** 创建缩写关键词 "DNA"
- **THEN** 系统设置 keyword_type="abbreviation"，不进行翻译

### Requirement: Establish translation relationships
系统 SHALL 为翻译后的关键词建立 translation_of 关系。

#### Scenario: Translation relationship points to anchor
- **WHEN** 语言列表为 `["en", "es", "zh"]`，创建 "天胡荽"(zh) 的翻译
- **THEN** 所有非锚点语言的关键词都通过 translation_of 指向锚点语言关键词

#### Scenario: Query translation synonyms
- **WHEN** 用户查询关键词 "天胡荽" 的同义词
- **THEN** 系统返回所有通过 translation_of 关系关联的其他语言关键词

### Requirement: Classify keyword type
系统 SHALL 支持多种关键词类型分类。

#### Scenario: Classify as person
- **WHEN** 关键词被识别为人名（如 "达尔文"）
- **THEN** 系统设置 keyword_type="person"

#### Scenario: Classify as paper
- **WHEN** 关键词被识别为论文标题
- **THEN** 系统设置 keyword_type="paper"

#### Scenario: Classify as book
- **WHEN** 关键词被识别为书籍名称
- **THEN** 系统设置 keyword_type="book"

#### Scenario: Classify as term
- **WHEN** 关键词被识别为专业术语
- **THEN** 系统设置 keyword_type="term"

#### Scenario: Classify as abbreviation
- **WHEN** 关键词被识别为缩写
- **THEN** 系统设置 keyword_type="abbreviation"

#### Scenario: Classify as unknown language
- **WHEN** 关键词为土著语言或无法识别的语言
- **THEN** 系统设置 keyword_type="unknown_language"

### Requirement: Detect keyword language
系统 SHALL 自动检测关键词的语言。

#### Scenario: Detect Chinese language
- **WHEN** 用户输入 "天胡荽"
- **THEN** 系统检测语言为 "zh"

#### Scenario: Detect English language
- **WHEN** 用户输入 "Hydrocotyle"
- **THEN** 系统检测语言为 "en"

#### Scenario: Detect unknown language
- **WHEN** 用户输入无法识别的语言文本
- **THEN** 系统设置语言为 "unknown"，不进行翻译

### Requirement: Ensure unique keyword text
系统 SHALL 确保关键词文本唯一。

#### Scenario: Reject duplicate keyword
- **WHEN** 用户尝试创建已存在的关键词
- **THEN** 系统返回已存在的记录，不创建重复

### Requirement: Update keyword description
系统 SHALL 允许更新关键词的描述信息。

#### Scenario: Update description
- **WHEN** 用户为关键词 "天胡荽" 添加描述 "多年生草本植物"
- **THEN** 系统更新该关键词的 description 字段

### Requirement: List keywords
系统 SHALL 支持按类型和语言筛选关键词。

#### Scenario: List keywords by type
- **WHEN** 用户筛选类型为 "paper" 的关键词
- **THEN** 系统返回所有 keyword_type="paper" 的关键词

#### Scenario: List keywords by language
- **WHEN** 用户筛选语言为 "zh" 的关键词
- **THEN** 系统返回所有 language="zh" 的关键词

### Requirement: Delete keyword
系统 SHALL 允许删除关键词及其关联数据。

#### Scenario: Delete keyword with cascading
- **WHEN** 用户删除关键词
- **THEN** 系统删除该关键词以及关联的 URLs、Images、Tasks
