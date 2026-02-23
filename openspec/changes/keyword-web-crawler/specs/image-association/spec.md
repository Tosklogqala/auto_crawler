# Spec: Image Association

## ADDED Requirements

### Requirement: Download images
系统 SHALL 从搜索结果中下载相关图片。

#### Scenario: Download image from URL
- **WHEN** 搜索结果页面包含图片
- **THEN** 系统下载图片并保存到本地 storage/images/ 目录

#### Scenario: Record image metadata
- **WHEN** 图片下载完成
- **THEN** 系统记录 original_url、local_path、alt_text

### Requirement: Associate image with keyword
系统 SHALL 从上下文分析图片与关键词的关联。

#### Scenario: Associate by proximity
- **WHEN** 图片周围的文本包含当前搜索关键词
- **THEN** 系统建立图片与该关键词的关联

#### Scenario: Store context text
- **WHEN** 图片被下载
- **THEN** 系统保存图片周围的文本到 context_text 字段

### Requirement: Support AI-assisted image analysis
系统 SHALL 可选地使用视觉模型辅助识别图片内容。

#### Scenario: Identify image content
- **WHEN** 启用视觉模型辅助
- **THEN** 系统使用 AI 分析图片内容并与关键词匹配度评分

### Requirement: Handle image download failures
系统 SHALL 处理图片下载失败的情况。

#### Scenario: Handle inaccessible image
- **WHEN** 图片 URL 无法访问
- **THEN** 系统记录错误但不中断搜索流程
