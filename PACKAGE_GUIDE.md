# ApiPost MCP 本地打包与分发指南

当不便发布到 npm registry 时，可以通过 `.tgz` 压缩包方式将项目分发给他人使用。

---

## 一、开发者侧：打包

### 1.1 编译并打包

在项目根目录执行：

```bash
cd /path/to/apipost-mcp

# 确保依赖已安装且编译通过
npm install
npm run build

# 打包为 .tgz 文件
npm pack
```

执行后会生成 `apipost-mcp-1.0.0.tgz`（版本号对应 package.json 中的 version 字段）。

### 1.2 打包产物说明

`.tgz` 文件包含了 `files` 字段中声明的 `dist/` 目录和 `package.json`，对方解包后即可使用，无需再次编译。

### 1.3 交付文件

只需将 `apipost-mcp-1.0.0.tgz` 这一个文件发给对方即可（微信、邮件、内网共享目录等任意方式）。

---

## 二、使用者侧：安装与配置

### 2.1 环境要求

| 环境 | 版本要求 |
|------|---------|
| Node.js | >= 18.0.0 |
| npm | >= 8.0.0 |

验证方式：

```bash
node --version
npm --version
```

### 2.2 安装

将收到的 `.tgz` 文件放在任意目录，执行全局安装：

```bash
npm install -g ./apipost-mcp-1.0.0.tgz
```

安装完成后验证：

```bash
apipost-mcp
# 应输出类似内容：
# 🚀 ApiPost MCP 启动中...
# 🔗 连接到: https://open.apipost.net
# ❌ 错误: 请设置 APIPOST_TOKEN 环境变量（这是预期的，因为还没配置 Token）
```

### 2.3 配置 MCP 客户端

在 MCP 配置文件中添加（不同客户端的配置文件路径略有不同）：

```json
{
  "mcpServers": {
    "apipost": {
      "command": "apipost-mcp",
      "args": [],
      "env": {
        "APIPOST_TOKEN": "你的ApiPost访问令牌",
        "APIPOST_HOST": "https://open.apipost.net",
        "APIPOST_SECURITY_MODE": "安全模式：`readonly` / `limited` / `full` ",
        "APIPOST_DEFAULT_TEAM_NAME": "你的团队名称[可以不配置]",
        "APIPOST_DEFAULT_PROJECT_NAME": "你的项目名称[可以不配置]",
      }
    }
  }
}
```

- `command` 填 `apipost-mcp`（全局安装后可直接作为命令使用）
- `args` 为空数组即可
- 环境变量中 `APIPOST_TOKEN` 为必填项，其余按需填写

### 2.4 获取 ApiPost Token

1. 打开 ApiPost 客户端
2. 进入「工作台」→ 选择目标项目 →「项目设置」→「对外能力」→「Open API」
3. 复制用户 API Token

---

## 三、常见问题

### 3.1 安装时报权限错误

**macOS / Linux**：

```bash
# 方法一：使用 sudo
sudo npm install -g ./apipost-mcp-1.0.0.tgz

# 方法二：配置 npm 全局目录到用户目录（推荐，一劳永逸）
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g ./apipost-mcp-1.0.0.tgz
```

**Windows**：以管理员身份运行命令提示符或 PowerShell。

### 3.2 提示 "apipost-mcp: command not found"

说明 npm 全局 bin 目录不在系统 PATH 中。查找并添加：

```bash
# 查看 npm 全局 bin 目录位置
npm bin -g

# 将该路径添加到 PATH（示例）
echo 'export PATH=/usr/local/lib/node_modules/.bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### 3.3 运行报错 "APIPOST_TOKEN 未设置"

这是预期行为 — Token 应通过 MCP 客户端的 `env` 配置传入，而非直接在终端设置。在 MCP 配置文件中补全 `APIPOST_TOKEN` 即可。

---

## 四、更新流程

当开发者发布了新版本，使用者需更新：

```bash
# 1. 卸载旧版本
npm uninstall -g apipost-mcp

# 2. 安装新版本
npm install -g ./apipost-mcp-0.0.0.tgz

# 3. 重启 MCP 客户端即可生效
```

无需修改 MCP 配置文件。

---

## 五、卸载

不再需要时：

```bash
npm uninstall -g apipost-mcp
```

然后在 MCP 配置文件中移除 `apipost` 相关配置块即可。

---

## 六、开发者侧：版本号管理

发布新版本前，记得更新 `package.json` 中的 `version` 字段，遵循语义化版本：

| 变更类型 | 版本号变化 | 示例 |
|---------|-----------|------|
| Bug 修复 | patch +1 | 1.2.0 → 1.2.1 |
| 新功能（向下兼容） | minor +1 | 1.2.0 → 1.3.0 |
| 破坏性变更 | major +1 | 1.2.0 → 2.0.0 |

打包前建议创建 git tag 以便追溯：

```bash
npm version patch   # 自动更新 package.json 并创建 git tag，如 1.2.0 → 1.2.1
npm run build
npm pack
```

---

## 七、完整操作速查

**开发者每次发版：**

```bash
cd /path/to/apipost-mcp
npm version patch        # 更新版本号
npm run build            # 编译
npm pack                 # 打包 → 得到 .tgz 文件
# 将 .tgz 发送给使用者
```

**使用者每次接收新版：**

```bash
npm uninstall -g apipost-mcp
npm install -g ./apipost-mcp-x.x.x.tgz
# 重启 MCP 客户端
```
