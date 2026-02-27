# 实施计划

- [ ] 1. 创建 TavilyWebSearchConfig 配置结构
   - 在 [nanobot/config/schema.py](/github.com/HKUDS/nanobot/nanobot/config/schema.py) 中添加 `TavilyWebSearchConfig` 类
   - 定义 `api_key` 字段为可选的字符串类型，默认值为 None
   - 添加字段描述说明该配置用于 Tavily 搜索引擎 API 认证
   - _需求：7.1、7.3、7.4、7.5、7.6_

- [ ] 2. 在 WebSearchConfig 中添加 tavily 字段
   - 修改 [nanobot/config/schema.py](/github.com/HKUDS/nanobot/nanobot/config/schema.py) 中的 `WebSearchConfig` 类
   - 添加 `tavily` 字段，类型为 `TavilyWebSearchConfig | None`，默认值为 None
   - 添加字段描述说明当该字段非空时使用 Tavily 搜索引擎而非 Brave Search
   - 确保向后兼容，不影响现有配置文件的解析
   - _需求：8.1、8.2、8.3、8.4、10.1、10.2_

- [ ] 3. 创建 TavilyWebSearchTool 工具类基础结构
   - 在 [nanobot/agent/tools/web.py](/github.com/HKUDS/nanobot/nanobot/agent/tools/web.py) 中创建 `TavilyWebSearchTool` 类
   - 继承自 `Tool` 基类
   - 设置 `name` 属性为 `"tavily_web_search"`
   - 设置 `description` 属性描述工具用途
   - 定义 `parameters` 参数 schema，包含 `query` 和 `count` 参数，与 `WebSearchTool` 保持一致
   - _需求：1.1、1.2、1.3、1.4、5.1、5.2、5.4_

- [ ] 4. 实现 Tavily API 调用逻辑
   - 在 `TavilyWebSearchTool` 中实现 `api_key` 属性，支持从环境变量 `TAVILY_API_KEY` 或构造函数参数读取
   - 实现 `execute` 异步方法，调用 Tavily API 进行搜索
   - 处理 Tavily API 响应，将结果格式化为统一的输出格式（包含标题、URL、描述）
   - 应用结果数量限制（1-10），与 `WebSearchTool` 保持一致
   - _需求：2.1、2.2、2.3、2.4、2.5、3.1、5.3、5.4_

- [ ] 5. 实现参数验证和错误处理
   - 在 `execute` 方法中添加参数验证：检查 `query` 是否为空或未提供
   - 验证 `count` 参数范围，小于 1 时返回错误，大于 10 时限制为 10
   - 添加 API key 未配置时的错误处理，返回清晰的配置提示信息
   - 使用具体的异常类型捕获 Tavily API 请求失败
   - 在错误消息中包含 HTTP 状态码和错误描述信息
   - _需求：4.1、4.2、4.3、4.4、4.5、4.6、3.3、3.4_

- [ ] 6. 实现配置优先级选择逻辑
   - 在 `TavilyWebSearchTool` 中添加 `__init__` 方法，支持通过 `api_key` 参数传入配置
   - 确保当配置了 `tavily` 字段时，`TavilyWebSearchTool` 使用 Tavily API
   - 当 `tavily` 字段为 None 时，`WebSearchTool` 继续使用 Brave Search
   - _需求：9.1、9.2、9.3、9.4、10.3_

- [ ] 7. 编写 TavilyWebSearchTool 单元测试
   - 创建测试文件 `tests/test_tavily_web_search.py`
   - 编写测试类 `TestTavilyWebSearchTool`
   - 测试参数验证逻辑（空 query、count 范围验证）
   - 测试 API key 配置逻辑（环境变量、未配置的情况）
   - 测试 API 调用成功和失败的场景
   - 使用 pytest 框架和 mock 对象模拟 Tavily API 响应
   - _需求：11.1、11.2、11.3、11.4、11.5、11.6_

- [ ] 8. 编写配置逻辑单元测试
   - 在测试文件中添加配置相关测试
   - 测试 `tavily` 字段为空和非空时的行为
   - 测试配置兼容性，验证旧配置文件（无 tavily 字段）仍能正常工作
   - 验证两个工具的接口一致性（参数名称、类型、结果格式）
   - _需求：11.7、11.8、10.1、10.2、10.3、10.4_
