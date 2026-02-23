# Spec: Web UI

## ADDED Requirements

### Requirement: Display keyword input form
系统 SHALL 提供关键词输入界面。

#### Scenario: Input keyword with type and language
- **WHEN** 用户在前端输入关键词并选择类型和语言设置
- **THEN** 系统将数据发送到后端 API 创建关键词

#### Scenario: Auto-detect language
- **WHEN** 用户输入关键词
- **THEN** 前端调用语言检测 API 并自动填充语言字段

### Requirement: Display search results
系统 SHALL 展示搜索结果列表。

#### Scenario: Show result list
- **WHEN** 搜索任务完成
- **THEN** 前端显示 URL 列表、标题、摘要，支持点击跳转

#### Scenario: Show translation toggle
- **WHEN** 显示非显示语言的结果
- **THEN** 用户可切换查看原文/译文

### Requirement: Display article viewer
系统 SHALL 提供原文浏览界面。

#### Scenario: View markdown article
- **WHEN** 用户点击已处理的文章
- **THEN** 前端渲染 Markdown 内容，显示关联图片

#### Scenario: View PDF article
- **WHEN** 用户点击 PDF 类型的文章
- **THEN** 前端打开 PDF 查看器

### Requirement: Manage saved information
系统 SHALL 提供已保存信息的管理界面。

#### Scenario: View saved images
- **WHEN** 用户查看关键词详情
- **THEN** 显示关联的图片缩略图，支持点击查看大图

#### Scenario: Edit keyword description
- **WHEN** 用户编辑描述并保存
- **THEN** 系统更新关键词描述

### Requirement: Display crawl progress
系统 SHALL 显示爬取进度。

#### Scenario: Show keyword-portal matrix
- **WHEN** 用户进入进度页面
- **THEN** 显示关键词×搜索入口矩阵，标记已搜索/待搜索状态

#### Scenario: Show URL access status
- **WHEN** 用户查看 URL 列表
- **THEN** 按颜色标记 pending/success/failed 状态

### Requirement: Manage resource bookmarks
系统 SHALL 提供受限资源管理界面。

#### Scenario: View bookmarked resources
- **WHEN** 用户进入受限资源页面
- **THEN** 显示所有待处理的受限资源列表

### Requirement: Batch modify URL status
系统 SHALL 支持批量修改 URL 访问状态。

#### Scenario: Select multiple URLs
- **WHEN** 用户在 URL 列表中选择多条记录
- **THEN** 前端高亮显示已选记录

#### Scenario: Batch reset to pending
- **WHEN** 用户选择批量将失败 URL 重置为 pending
- **THEN** 系统更新所有选中 URL 的 access_status="pending"

### Requirement: Display keyword relations
系统 SHALL 显示关键词关系网络。

#### Scenario: Show related keywords
- **WHEN** 用户查看关键词详情
- **THEN** 显示相关关键词（作者、缩写、相关术语等）

#### Scenario: View relation type
- **WHEN** 用户悬停关系连线
- **THEN** 显示关系类型和置信度

### Requirement: Display translation status
系统 SHALL 提供翻译状态管理页面。

#### Scenario: Show translation coverage
- **WHEN** 用户进入翻译状态页面
- **THEN** 显示每种语言的翻译完成度

#### Scenario: Show missing translations
- **WHEN** 某些关键词缺少特定语言的翻译
- **THEN** 显示缺失列表，用户可手动触发翻译

#### Scenario: Manually add translation
- **WHEN** 用户为关键词手动添加翻译
- **THEN** 系统保存翻译并建立 translation_of 关系

#### Scenario: Batch retry failed translations
- **WHEN** 用户点击"批量重试翻译"
- **THEN** 系统重新尝试翻译所有失败的记录

### Requirement: Control Agent execution
系统 SHALL 提供 Agent 执行控制界面。

#### Scenario: Trigger Agent from UI
- **WHEN** 用户点击"开始抓取"按钮
- **THEN** 系统触发 Agent 执行任务

#### Scenario: Configure run count
- **WHEN** 用户设置执行次数 N
- **THEN** Agent 执行 N 次任务后停止

#### Scenario: View Agent status
- **WHEN** Agent 正在执行
- **THEN** 前端显示当前任务进度和状态

#### Scenario: Start scheduled execution
- **WHEN** 用户配置定时任务
- **THEN** 系统按配置自动触发 Agent
