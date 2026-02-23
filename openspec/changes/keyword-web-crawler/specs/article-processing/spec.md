# Spec: Article Processing

## ADDED Requirements

### Requirement: Use layered crawler strategy
系统 SHALL 使用分层爬虫策略访问 URL，优先 HTTP，失败时降级到 Playwright。

#### Scenario: Try HTTP first
- **WHEN** 访问 URL
- **THEN** 系统首先尝试 HTTP 请求

#### Scenario: Fallback to Playwright
- **WHEN** HTTP 请求失败或返回的内容无效（动态页面、反爬拦截等）
- **THEN** 系统降级到 Playwright 浏览器渲染

#### Scenario: Check content validity
- **WHEN** HTTP 请求成功
- **THEN** 系统检查内容是否有效（非空页面、非 JS 占位符、非反爬页面）

#### Scenario: Mark as restricted resource
- **WHEN** Playwright 也无法访问（需要登录/付费）
- **THEN** 系统标记为受限资源，不继续处理

### Requirement: Convert HTML to Markdown
系统 SHALL 将有效的网页内容转换为 Markdown 格式存储。

#### Scenario: Extract main content
- **WHEN** 访问网页成功
- **THEN** 系统提取主体内容（排除导航、广告、页脚等），转换为 MD 格式

#### Scenario: Save to local file
- **WHEN** 转换完成
- **THEN** 系统保存 MD 文件到 storage/articles/ 目录

#### Scenario: Link local images
- **WHEN** 网页中有相关图片
- **THEN** 系统下载图片到 storage/images/，在 MD 中使用相对路径链接

### Requirement: Store PDF files
系统 SHALL 存储下载的 PDF 文件。

#### Scenario: Save PDF directly
- **WHEN** 下载的是 PDF 文件
- **THEN** 系统直接保存到 storage/articles/ 目录

#### Scenario: Extract images from PDF
- **WHEN** PDF 中包含图片
- **THEN** 系统提取图片保存到 storage/images/，记录与 article 的关联

### Requirement: Convert other formats
系统 SHALL 将其他格式（DOC、DOCX 等）转换为 Markdown。

#### Scenario: Convert DOCX to MD
- **WHEN** 下载的是 DOCX 文件
- **THEN** 系统转换为 Markdown 格式存储

### Requirement: Associate article with keyword
系统 SHALL 建立原文与关键词的关联关系。

#### Scenario: Create keyword-article relation
- **WHEN** 原文与某个关键词相关
- **THEN** 系统创建 keyword_article_relations 记录

#### Scenario: Mark primary keyword
- **WHEN** 原文主要讨论某个关键词
- **THEN** 系统设置 is_primary=true

### Requirement: Identify full version
系统 SHALL 识别论文/书籍是否为完整版。

#### Scenario: Detect full paper
- **WHEN** PDF 页数超过阈值且包含完整章节
- **THEN** 系统设置 is_full_version=true

#### Scenario: Detect partial content
- **WHEN** 内容仅为摘要或部分页面
- **THEN** 系统设置 is_full_version=false

### Requirement: Filter invalid content
系统 SHALL 过滤无效内容（广告页、登录页等）。

#### Scenario: Detect advertisement page
- **WHEN** 页面主要内容为广告
- **THEN** 系统标记 URL 为无效，不创建 article

#### Scenario: Detect login required page
- **WHEN** 页面需要登录才能查看内容
- **THEN** 系统标记 requires_auth=true，不创建 article
