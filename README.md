# ApiPost MCP

基于 MCP 协议和 [ApiPost 官方 OpenAPI](https://docs.apipost.net/docs/detail/2a37986cbc64000?target_id=23796913b176e1) 实现的 API 管理工具。作为 AI 编码助手与 ApiPost 平台之间的桥梁，让开发者可以在 AI 对话中直接管理和维护 API 文档。

**当前版本**：1.2.0 | **开源协议**：MIT | **作者**：jlcodes (jlcodes@163.com)

## 功能

- **连接测试** - 一键验证MCP服务器状态和配置
- **工作空间管理** - 查看、切换团队和项目工作空间
- **目录管理** - 创建API文档目录，支持层级结构
- **API接口管理** - 创建、查看、修改、删除接口文档
- **增量更新** - 支持字段级别的精确更新和删除
- **层级搜索** - 强化的目录层级搜索和父子关系定位
- **递归浏览** - 递归搜索子目录，支持深度限制
- **多维筛选** - 多维度搜索和批量操作
- **结构化显示** - 树形结构和分组显示
- **路径导航** - 完整路径显示，快速定位
- **权限管理** - 多种安全模式，灵活的操作权限控制

## 技术架构

```
┌─────────────────┐     MCP Protocol     ┌──────────────────┐     HTTPS/REST      ┌─────────────────┐
│   AI 助手        │ ◄──────────────────► │  apipost-mcp     │ ◄─────────────────► │  ApiPost 服务端   │
│  (Claude/Cursor) │     (stdio/JSON-RPC) │  (Node.js 进程)   │     (OpenAPI)       │  (open.apipost.net)│
└─────────────────┘                      └──────────────────┘                     └─────────────────┘
```

运行时流程：

1. MCP 服务启动时，自动调用 ApiPost API 获取团队和项目列表，预初始化工作空间
2. AI 助手通过 MCP 协议发现 8 个可用工具
3. 用户通过对话触发工具调用，服务根据安全模式校验操作权限
4. 构造 ApiPost API 请求并发送，将返回结果格式化为结构化文本

### 技术栈

| 类别 | 技术 | 说明 |
|------|------|------|
| 运行时 | Node.js >= 18.0.0 | MCP SDK 官方最低要求 |
| 语言 | TypeScript 5.9 | 编译为 ES2022 目标 |
| 模块系统 | ESM | `"type": "module"` |
| MCP SDK | @modelcontextprotocol/sdk ^0.4.0 | MCP 协议核心实现 |
| HTTP 客户端 | axios ^1.6.0 | 调用 ApiPost OpenAPI |
| 校验库 | zod ^3.22.0 | 运行时类型校验（已声明） |
| 开发工具 | tsx ^4.0.0 | TypeScript 执行/监听 |

| 命令 | 用途 |
|------|------|
| `npm run build` | TypeScript 编译到 dist/ |
| `npm start` | 运行编译产物 |
| `npm run dev` | tsx 直接运行源码（自动读取 .env） |
| `npm run watch` | tsx 监听模式热重载 |

### 项目结构

```
apipost-mcp/
├── src/
│   └── index.ts           # 唯一源文件，包含全部业务逻辑
├── dist/
│   ├── index.js            # 编译产物
│   └── index.d.ts          # 类型声明
├── package.json
├── tsconfig.json
├── .env.example            # 环境变量模板
├── README.md               # 项目说明（本文件）
├── CHANGELOG.md            # 版本更新日志
├── DEVELOPMENT_GUIDE.md    # 开发指南
├── PACKAGE_GUIDE.md        # 打包分发指南
├── CONTRIBUTING.md         # 贡献指南
└── LICENSE                 # MIT 许可证
```

源码为单文件架构，`src/index.ts` 包含环境变量读取、API 客户端、安全校验、字段列表转换、JSON 构建、显示格式化、MCP 工具定义与处理等全部功能。

## 安装

### 环境要求

在开始安装之前，请确保您的系统已安装以下环境：

| 环境 | 版本要求 | 说明 |
|------|---------|------|
| **Node.js** | >= 18.0.0 | JavaScript 运行环境（MCP SDK 官方最低要求） |
| **npm** | >= 8.0.0 | Node.js 包管理器（通常随 Node.js 一起安装） |

#### 环境安装指南

**Node.js 安装：**
- 访问 [Node.js 官网](https://nodejs.org/) 下载 LTS 版本
- 或使用包管理器：
  ```bash
  # macOS (使用 Homebrew)
  sudo brew install node
  
  # Ubuntu/Debian
  sudo apt update && sudo apt install nodejs npm
  
  # CentOS/RHEL
  sudo yum install nodejs npm
  ```

**验证安装：**
```bash
node --version   # 应显示 v18.0.0 或更高版本
npm --version    # 应显示 8.0.0 或更高版本
```

### 方式二：手动安装（用于开发或离线环境）

```bash
git clone https://github.com/jlcodes99/apipost-mcp.git
cd apipost-mcp
npm install && npm run build
```

## 配置

在 MCP 配置文件中添加：

**使用本地路径：**
```json
{
  "mcpServers": {
    "apipost": {
      "command": "node",
      "args": ["/absolute/path/to/apipost-mcp/dist/index.js"],
      "env": {
        "APIPOST_TOKEN": "your_access_token_here",
        "APIPOST_HOST": "https://open.apipost.net",
        "APIPOST_SECURITY_MODE": "limited",
        "APIPOST_DEFAULT_TEAM_NAME": "你的团队名称",
        "APIPOST_DEFAULT_PROJECT_NAME": "你的项目名称",
        "APIPOST_URL_PREFIX": "接口前缀可定义常量比如{{host}}"
      }
    }
  }
}
```

### 环境变量

| 变量名 | 是否必需 | 默认值 | 说明 |
|--------|----------|--------|------|
| `APIPOST_TOKEN` | **是** | — | ApiPost 用户 API Token，从客户端「项目设置 > 对外能力 > Open API」获取 |
| `APIPOST_HOST` | 否 | `https://open.apipost.net` | ApiPost OpenAPI 地址 |
| `APIPOST_SECURITY_MODE` | 否 | `limited` | 安全模式：`readonly` / `limited` / `full` |
| `APIPOST_DEFAULT_TEAM_NAME` | 否 | — | 默认团队名称，自动选择指定团队 |
| `APIPOST_DEFAULT_PROJECT_NAME` | 否 | — | 默认项目名称，自动选择指定项目 |
| `APIPOST_URL_PREFIX` | 否 | `""` | 接口 URL 前缀常量（如 `{{host}}`），自动拼接到所有新建/修改接口 |
| `APIPOST_INLINE_COMMENTS` | 否 | `false` | 设为 `true` 时，raw 按字段 `desc` 生成行内注释 |

### 安全模式说明

| 模式 | 查看 | 创建 | 修改 | 删除 | 适用场景 |
|------|------|------|------|------|----------|
| `readonly` | 允许 | 禁止 | 禁止 | 禁止 | 只读参考，防止误操作 |
| `limited` | 允许 | 允许 | 允许 | 禁止 | 日常开发，保护已有接口不被误删 |
| `full` | 允许 | 允许 | 允许 | 允许 | 完全管理权限 |

## 可用工具

| 工具 | 功能 | 主要参数 |
|------|------|---------|
| `apipost_test_connection` | 连接测试 | `random_string` |
| `apipost_workspace` | 工作空间管理 | `action` (必需) |
| `apipost_create_folder` | 创建目录 | `name`, `parent_id` |
| `apipost_smart_create` | 创建接口 | `method`, `url`, `name` |
| `apipost_list` | 强化列表搜索 | `search`, `parent_id`, `target_type`, `show_structure`, `recursive`, `group_by_folder` |
| `apipost_detail` | 查看详情 | `target_id` |
| `apipost_update` | 修改接口 | `target_id`, 其他可选 |
| `apipost_delete` | 删除接口 | `api_ids` |

### API 接口映射

| MCP 工具 | ApiPost OpenAPI 端点 | HTTP 方法 |
|----------|---------------------|-----------|
| 工作空间初始化 | `/open/team/list` | GET |
| 工作空间初始化 | `/open/project/list` | GET |
| apipost_create_folder | `/open/apis/create` | POST |
| apipost_smart_create | `/open/apis/create` | POST |
| apipost_list | `/open/apis/list` | GET |
| apipost_detail | `/open/apis/details` | POST |
| apipost_update | `/open/apis/update` | POST |
| apipost_delete | `/open/apis/delete` | POST |

### apipost_test_connection 说明

**快速诊断工具**，适合首次使用或故障排查：
- 验证MCP服务器连接状态
- 检查环境变量配置
- 显示当前工作空间信息  
- 检查操作权限和安全模式
- 提供系统环境详情

### apipost_workspace 说明

**统一的工作空间管理工具**，支持以下操作：

| Action | 功能 | 主要参数 | 说明 |
|--------|------|---------|------|
| `current` | 查看当前工作空间 | `show_all` | 显示当前团队、项目信息，可选显示所有可用选项 |
| `list_teams` | 列出团队 | `show_details` | 显示所有可用团队，标识当前团队 |
| `list_projects` | 列出项目 | `team_id`, `show_details` | 显示指定团队的项目列表 |
| `switch` | 切换工作空间 | `team_id`, `project_id` 或 `team_name`, `project_name` | 切换到指定的团队和项目 |

**使用示例：**
```
# 查看当前工作空间
apipost_workspace action: "current"

# 列出所有团队
apipost_workspace action: "list_teams" show_details: true

# 列出项目
apipost_workspace action: "list_projects" team_id: "your_team_id"

# 切换工作空间（支持按名称或ID）
apipost_workspace action: "switch" team_name: "团队名" project_name: "项目名"
```

### apipost_create_folder 说明

**API文档目录创建工具**，支持在指定父目录下创建新的文件夹：

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 目录名称 |
| `parent_id` | string | 否 | 父目录ID，使用"0"表示根目录，默认为"0" |
| `description` | string | 否 | 目录描述（可选） |

**使用示例：**
```
# 在根目录创建目录
apipost_create_folder name: "用户管理" description: "用户相关接口"

# 在指定目录下创建子目录
apipost_create_folder name: "认证接口" parent_id: "folder_123" description: "用户认证相关接口"
```

### apipost_list 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `search` | string | 搜索关键词（接口名称、URL、方法、ID、描述） |
| `parent_id` | string | 父目录ID，精确查找子项目。"0"为根目录 |
| `target_type` | string | 类型筛选：`api`(仅接口)、`folder`(仅目录)、`all`(全部) |
| `show_structure` | boolean | 显示树形结构，默认false为列表模式 |
| `show_path` | boolean | 显示完整路径，默认false |
| `recursive` | boolean | 递归搜索子目录，默认false |
| `depth` | number | 深度限制（配合recursive），默认无限制 |
| `group_by_folder` | boolean | 按目录分组显示，默认false |
| `limit` | number | 显示数量限制（默认50，最大200） |
| `show_all` | boolean | 显示全部（忽略limit限制） |

### apipost_smart_create 说明（字段列表驱动）

规则（强制）：
- `responses` 只传 `fields`，不要传 `data`；所有字段（含父级）必须带 `desc`。
- headers/query/body/cookies 用字段列表字符串，嵌套用 `.`，数组用 `[]`（如 `meta.flags.debug`、`items[].id`），example 填真实值，不要放 JSON 字符串。
- 父级需显式声明并写 `desc`，示例：`{"key":"data","type":"object","desc":"返回体"},{"key":"data.user","type":"object","desc":"用户"},{"key":"data.user.id","type":"integer","example":1,"desc":"用户ID"}`。
- 可选 `APIPOST_INLINE_COMMENTS=true` 时，raw 会按 `desc` 生成行内注释（mock 始终为纯 JSON）。
- 可选 `APIPOST_URL_PREFIX={{host}}` 时，创建或更新接口时会将前缀自动拼接到 URL（避免手动重复填写路由常量）。

必填：`method`、`url`、`name`。其他字段（均为字符串化 JSON 数组/对象）：
- headers/query/body/cookies：`[{"key":"X-Request-ID","type":"string","required":true,"example":"req-1","desc":"说明"}]`
- responses：`[{"name":"成功","status":200,"fields":[{"key":"code","type":"integer","example":0,"desc":"状态码"},{"key":"data.items[].id","type":"string","example":"1","desc":"商品ID"}]}]`
- auth：`{"type":"bearer","bearer":{"key":"your_token"}}`
  
字段类型：`string`/`integer`/`number`/`boolean`/`object`/`array`/`null`

示例（嵌套）：
```json
"body": "[{\"key\":\"user.id\",\"type\":\"integer\",\"required\":true,\"example\":9001,\"desc\":\"用户ID\"},{\"key\":\"user.profile.tags[]\",\"type\":\"string\",\"example\":\"vip\",\"desc\":\"标签\"}]",
"responses": "[{\"name\":\"成功\",\"status\":200,\"fields\":[{\"key\":\"code\",\"type\":\"integer\",\"example\":0,\"desc\":\"状态码\"},{\"key\":\"data.user.profile.tags[]\",\"type\":\"string\",\"example\":\"vip\",\"desc\":\"标签\"}]}]"
```

## 核心实现要点

### 字段列表驱动机制（v1.2.0 重构）

所有参数配置（headers、query、body、cookies、responses）统一使用扁平字段列表格式：

```json
[
  {"key": "user.id", "type": "integer", "required": true, "example": 9001, "desc": "用户ID"},
  {"key": "user.profile.tags[]", "type": "string", "example": "vip", "desc": "标签"}
]
```

- `.` 表示嵌套对象路径，`[]` 表示数组元素
- 父级节点必须显式声明（如 `{"key": "user", "type": "object", "desc": "用户信息"}`）
- 未显式声明的父级会自动补位，仅用于参数表展示，不参与值生成

### 数据转换管线

```
用户输入 (JSON 字符串)
  → parseConfigParam()        解析为数组
  → ensureFieldsHaveDesc()    校验 desc 完整
  → expandFieldListWithParents()  补全父级节点
  → convertParams()           字段列表 → ApiPost 参数格式
  → buildJsonFromFieldList()  构建示例 JSON
  → normalizeResponses()      响应格式标准化
  → ApiPost API 请求
```

### 工作空间管理

- 启动时自动调用 `/open/team/list` 和 `/open/project/list` 初始化工作空间
- 支持按名称或 ID 选择团队和项目
- 可通过 `apipost_workspace` 工具动态切换

### 显示模式

- `buildPathMap()` — 递归构建从根目录到每个项目的完整路径（含循环引用检测）
- `getChildrenRecursively()` — 递归获取所有子项目，支持深度限制
- 三种显示模式：平铺列表、树形结构、按目录分组

## 获取 Token

1. [ApiPost OpenApi官方文档查看](https://docs.apipost.net/docs/detail/2a37986cbc64000?target_id=0)
2. 用户api_token。获取方式：Apipost客户端>工作台>项目设置>对外能力>open API

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| v1.2.0 | 2025-11-27 | 字段列表全面驱动重构，responses 仅用 fields，父级自动补位，行内注释支持 |
| v1.1.0 | 2025-08-14 | 新增 `apipost_create_folder` 和 `parent_id` 参数支持，目录管理层级结构 |
| v1.0.0 | 2025-08-06 | 首次发布，5 个核心工具，三种安全模式，增量更新和字段删除 |

详见 [CHANGELOG.md](CHANGELOG.md)。

## 开发

开发指南详见 [DEVELOPMENT_GUIDE.md](DEVELOPMENT_GUIDE.md)。打包分发指南详见 [PACKAGE_GUIDE.md](PACKAGE_GUIDE.md)。

```bash
git clone https://github.com/jlcodes99/apipost-mcp.git
cd apipost-mcp
npm install
# 复制 .env.example 为 .env 并填入真实 Token
cp .env.example .env
npm run watch
```

提交规范：`feat:` 新功能、`fix:` 修复、`docs:` 文档、`refactor:` 重构。

## 联系方式

- 邮箱: jlcodes@163.com
- 问题反馈: [GitHub Issues](https://github.com/jlcodes99/apipost-mcp/issues)
- 项目主页: [GitHub Repository](https://github.com/jlcodes99/apipost-mcp)

## 相关链接

- [ApiPost OpenAPI 文档](https://docs.apipost.net/docs/detail/2a37986cbc64000?target_id=23796913b176e1)
- [MCP 协议说明](https://github.com/modelcontextprotocol/specification)

---

如果这个项目对你有帮助，请给我们一个星标！
