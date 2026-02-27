# 需求文档

## 引言
本需求旨在创建一个新的 `TavilyWebSearchTool` 工具类，专门用于集成 Tavily 搜索引擎 API。现有的 `WebSearchTool` 将继续保持为 Brave Search API 专用，不做任何修改。这种设计提供了清晰的关注点分离，每个搜索引擎都有独立的工具类，便于维护和扩展。系统需要确保两个工具类具有一致的接口和使用体验。

## 需求

### 需求 1：创建 TavilyWebSearchTool 工具类

**用户故事：** 作为一名开发者，我需要一个独立的 `TavilyWebSearchTool` 工具类，以便使用 Tavily 搜索引擎进行网络搜索。

#### 验收标准

1. WHEN 创建 `TavilyWebSearchTool` 类时 THEN 系统 SHALL 继承自 `Tool` 基类
2. WHEN 定义 `TavilyWebSearchTool` 类时 THEN 系统 SHALL 包含 `name` 属性为 `"tavily_web_search"`
3. WHEN 定义 `TavilyWebSearchTool` 类时 THEN 系统 SHALL 包含 `description` 属性描述工具用途
4. WHEN 定义 `TavilyWebSearchTool` 类时 THEN 系统 SHALL 包含 `parameters` 参数定义

### 需求 2：实现 Tavily API 集成

**用户故事：** 作为一名开发者，我希望 TavilyWebSearchTool 能够调用 Tavily API 并返回搜索结果。

#### 验收标准

1. WHEN TavilyWebSearchTool 调用 execute() 方法时 THEN 系统 SHALL 使用 Tavily API 进行搜索
2. WHEN 构造 Tavily API 请求时 THEN 系统 SHALL 使用提供的查询字符串作为搜索参数
3. WHEN 构造 Tavily API 请求时 THEN 系统 SHALL 支持通过参数控制返回结果数量
4. WHEN Tavily API 返回成功响应时 THEN 系统 SHALL 将结果格式化为统一的输出格式（标题、URL、描述）
5. WHEN 限制返回结果数量时 THEN 系统 SHALL 应用与 WebSearchTool 相同的限制（1-10）

### 需求 3：支持 Tavily API key 配置

**用户故事：** 作为一名部署者，我可以通过环境变量或配置文件配置 Tavily API key，以便灵活部署到不同环境。

#### 验收标准

1. WHEN 系统初始化 `TavilyWebSearchTool` 时 THEN 系统 SHALL 优先从环境变量 `TAVILY_API_KEY` 读取 API key
2. WHEN 环境变量 `TAVILY_API_KEY` 未设置时 THEN 系统 SHALL 从配置文件读取 API key
3. WHEN 环境变量和配置文件都没有配置 API key 时 THEN 系统 SHALL 返回清晰的错误信息，提示用户配置 API key
4. WHEN API key 未配置时 THEN 系统 SHALL 在错误消息中说明如何配置（环境变量或配置文件路径）
5. WHEN 实现配置读取逻辑时 THEN 系统 SHALL 遵循安全规范，不在代码中硬编码敏感信息

### 需求 4：实现参数验证和错误处理

**用户故事：** 作为一名用户，我希望系统能够对输入参数进行验证，并在出现错误时提供清晰的错误信息。

#### 验收标准

1. WHEN 用户提供的查询字符串为空或未提供时 THEN 系统 SHALL 返回参数验证错误
2. WHEN 用户提供的 `count` 参数小于 1 时 THEN 系统 SHALL 返回参数验证错误
3. WHEN 用户提供的 `count` 参数大于 10 时 THEN 系统 SHALL 将其限制为 10
4. WHEN `Tavily` API 请求失败时 THEN 系统 SHALL 捕获异常并返回用户友好的错误信息
5. WHEN `Tavily` API 返回 HTTP 错误状态码时 THEN 系统 SHALL 包含状态码和错误描述信息
6. WHEN 实现异常处理时 THEN 系统 SHALL 遵循错误处理原则，使用具体的异常类型，避免使用裸 `except:`

### 需求 5：确保接口一致性

**用户故事：** 作为一名开发者，我希望 `TavilyWebSearchTool` 和 `WebSearchTool` 具有一致的接口和使用体验，以便在需要时可以互换使用。

#### 验收标准

1. WHEN 定义 `TavilyWebSearchTool` 的参数时 THEN 系统 SHALL 使用与 `WebSearchTool` 相同的参数名称（`query`、`count`）
2. WHEN 定义 `TavilyWebSearchTool` 的参数类型时 THEN 系统 SHALL 使用与 `WebSearchTool` 相同的参数类型（`query` 为字符串，`count` 为整数）
3. WHEN 返回搜索结果时 THEN 系统 SHALL 使用与 `WebSearchTool` 相同的结果格式（包含标题、URL、描述的字段）
4. WHEN 参数为可选时 THEN 系统 SHALL 设置与 `WebSearchTool` 相同的默认值（如 `count` 默认值为 10）

### 需求 6：保持 WebSearchTool 不变

**用户故事：** 作为一名现有用户，我希望现有的 `WebSearchTool` 保持不变，继续专门用于 Brave Search API。

#### 验收标准

1. WHEN 创建 `TavilyWebSearchTool` 时 THEN 系统 SHALL 不修改现有的 `WebSearchTool` 类
2. WHEN `WebSearchTool` 执行时 THEN 系统 SHALL 继续使用 Brave Search API
3. WHEN `WebSearchTool` 配置时 THEN 系统 SHALL 继续使用 `BRAVE_API_KEY` 环境变量或配置
4. WHEN 用户使用现有代码调用 `WebSearchTool` 时 THEN 系统 SHALL 保持与之前完全相同的行为
5. WHEN 项目中同时存在两个工具时 THEN 系统 SHALL 确保两者可以独立运行，互不干扰

### 需求 7：创建 TavilyWebSearchConfig 配置结构

**用户故事：** 作为一名配置管理员，我希望有独立的 `TavilyWebSearchConfig` 配置结构，以便单独管理 Tavily 搜索引擎的配置参数。

#### 验收标准

1. WHEN 定义配置 schema 时 THEN 系统 SHALL 创建 `TavilyWebSearchConfig` 类
2. WHEN 定义 `TavilyWebSearchConfig` 类时 THEN 系统 SHALL 将其与 `WebSearchConfig` 设置为平级结构
3. WHEN 定义 `TavilyWebSearchConfig` 类时 THEN 系统 SHALL 包含 `api_key` 字段用于存储 Tavily API 密钥
4. WHEN 定义 `TavilyWebSearchConfig` 类时 THEN 系统 SHALL 设置 `api_key` 字段类型为 `str | None`
5. WHEN 定义 `TavilyWebSearchConfig` 类时 THEN 系统 SHALL 设置 `api_key` 字段默认值为 None
6. WHEN 定义 `TavilyWebSearchConfig` 类时 THEN 系统 SHALL 包含描述信息说明该配置结构的用途

### 需求 8：在 WebSearchConfig 中添加 tavily 字段

**用户故事：** 作为一名配置管理员，我希望 `WebSearchConfig` 包含可选的 tavily 字段，以便在需要时启用 Tavily 搜索引擎。

#### 验收标准

1. WHEN 定义 `WebSearchConfig` 类时 THEN 系统 SHALL 添加 `tavily` 字段
2. WHEN 添加 `tavily` 字段时 THEN 系统 SHALL 设置其类型为 `TavilyWebSearchConfig | None`
3. WHEN 添加 `tavily` 字段时 THEN 系统 SHALL 设置该字段为可选，默认值为 None
4. WHEN 添加 `tavily` 字段时 THEN 系统 SHALL 添加描述信息说明当该字段非空时将使用 Tavily 而非 Brave Search

### 需求 9：实现搜索引擎优先级选择逻辑

**用户故事：** 作为一名系统，我希望在 tavily 配置非空时优先使用 Tavily 搜索引擎，在 tavily 配置为空时使用原有的 Brave Search。

#### 验收标准

1. WHEN `WebSearchConfig` 中 `tavily` 字段非空时 THEN 系统 SHALL 使用 Tavily 搜索引擎进行搜索
2. WHEN `WebSearchConfig` 中 `tavily` 字段非空时 THEN 系统 SHALL 忽略 `WebSearchConfig` 中的 Brave Search 相关配置
3. WHEN `WebSearchConfig` 中 `tavily` 字段为 None 时 THEN 系统 SHALL 使用 Brave Search 搜索引擎
4. WHEN `WebSearchConfig` 中 `tavily` 字段为 None 时 THEN 系统 SHALL 使用 `WebSearchConfig` 中的 Brave Search 配置
5. WHEN 初始化 `WebSearchTool` 时 THEN 系统 SHALL 根据配置自动判断使用哪个搜索引擎

### 需求 10：确保配置兼容性

**用户故事：** 作为一名现有用户，我希望在未配置 tavily 字段时系统仍能正常使用 Brave Search，确保向后兼容。

#### 验收标准

1. WHEN 配置文件不包含 `tavily` 字段时 THEN 系统 SHALL 将 `tavily` 字段解析为 None
2. WHEN `tavily` 字段为 None 时 THEN 系统 SHALL 使用原有的 Brave Search 配置和行为
3. WHEN 用户升级版本后未修改配置文件时 THEN 系统 SHALL 保持与之前完全相同的搜索功能
4. WHEN 使用旧的配置格式时 THEN 系统 SHALL 不报错，正常解析并使用 Brave Search

### 需求 11：编写单元测试

**用户故事：** 作为一名开发者，我希望为 `TavilyWebSearchTool` 和配置逻辑编写单元测试，确保代码质量和功能正确性。

#### 验收标准

1. WHEN 编写测试文件时 THEN 系统 SHALL 使用 `pytest` 框架
2. WHEN 命名测试文件时 THEN 系统 SHALL 使用 `test_*.py` 格式
3. WHEN 编写测试类时 THEN 系统 SHALL 使用 `Test*` 命名格式
4. WHEN 编写测试函数时 THEN 系统 SHALL 使用 `test_*` 命名格式
5. WHEN 测试 `TavilyWebSearchTool` 时 THEN 系统 SHALL 测试参数验证逻辑
6. WHEN 测试 `TavilyWebSearchTool` 时 THEN 系统 SHALL 测试 API 调用成功和失败的场景
7. WHEN 测试配置逻辑时 THEN 系统 SHALL 测试 `tavily` 字段为空和非空时的行为
8. WHEN 测试接口一致性时 THEN 系统 SHALL 验证返回结果格式与 `WebSearchTool` 一致
