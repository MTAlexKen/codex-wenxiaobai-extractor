# Chrome 控制功能失效修复指南

> 参考来源：
> - 4SAPI 文章：https://www.4sapi.net/blog/codex-chrome-plugin-node-repl-4sapi
> - 真实修复案例：Codex 对话 `019fcc9f-2278-78a0-8d04-c0028fc16b11`

## 问题症状

- 当前线程没有 `mcp__node_repl__js` 工具
- 报错：`当前线程没有 mcp__node_repl__js`
- `setupBrowserRuntime` 报错：`privileged native pipe bridge is not available`
- Control Chrome skill 无法启动

## 核心原理

Codex 控制 Chrome 的完整链路：

```
Codex 当前线程
  -> 工具发现 / MCP 工具注册
  -> Node REPL 的 js 执行工具（mcp__node_repl__js）
  -> Control Chrome skill
  -> browser-client.mjs
  -> Chrome 扩展
  -> 用户自己的 Chrome 标签页
```

**关键点**：Chrome 插件本身可能完全正常，问题通常出在 Codex 线程没有拿到 `mcp__node_repl__js` 工具。这不是 Chrome 坏了，而是线程缺少启动 Chrome 控制链路的 JS 执行工具。

## 修复步骤（按顺序尝试）

### 步骤 1：工具发现搜索

告诉 Codex 以下提示词：

```
当前任务需要控制用户自己的 Chrome 插件。
请先通过工具发现搜索 node_repl js，找到 Node REPL 的 js 执行工具。
它通常叫 mcp__node_repl__js，有些界面会显示成 mcp__node_repl.js。
找到以后再按 Control Chrome skill 初始化 browser-client.mjs。
不要用 js_reset 或 js_add_node_module_dir 代替 js 工具。
```

### 步骤 2：新开线程

如果工具搜索无效，**新开一个 Codex 线程**重试。新的线程上下文可能会正确加载 Node REPL 工具。

### 步骤 3：重启 Codex

退出并重新启动 Codex 桌面应用。重启后 MCP 工具注册会刷新。

### 步骤 4：检查 Chrome 扩展

确保 Chrome 扩展已安装并启用：
1. 打开 Chrome，进入扩展管理页面（chrome://extensions/）
2. 确认 Codex 相关扩展已启用
3. 如果已禁用，启用后重启 Chrome

### 步骤 5：Windows 专用修复

#### 5a. 检查 config.toml

编辑 Codex 配置文件（通常位于 `%USERPROFILE%\.codex\config.toml`），确保包含：

```toml
[mcp_servers.node_repl]
command = "node_repl.exe"
args = []
```

#### 5b. 确保 node_repl.exe 可访问

确认 `node_repl.exe` 存在于以下路径之一：
- `%LOCALAPPDATA%\OpenAI\Codex\bin\node_repl.exe`
- Codex 安装目录的 `bin` 子目录

#### 5c. 修改 sandbox 设置

```toml
[windows]
sandbox = "unelevated"
```

这允许 Computer Use 功能在不过度提升权限的情况下运行。

#### 5d. 完全访问权限

确保 Codex 拥有完整的文件系统访问权限。

### 步骤 6：Mac 专用修复

Mac 端不要照搬 Windows 修复（不需要处理 node_repl.exe 或 sandbox 设置）。

## 验证清单

修复后，确认以下每项：

1. `mcp__node_repl__js` 工具存在于当前线程
2. 不是 `js_reset`（它是重置工具，不是 JS 执行工具）
3. 使用的是 Control Chrome，不是 in-app browser
4. 能成功初始化 browser-client.mjs
5. 能列出或接管一个 Chrome 标签页

## 常见错误速查

| 错误 | 原因 | 修复 |
|------|------|------|
| current thread does not have mcp__node_repl__js | 线程缺少 JS 工具 | 新开线程 / 重启 Codex |
| privileged native pipe bridge is not available | browser-client 需要 Codex 内部特权 | 正常现象，尝试重新初始化 |
| Chrome 扩展未连接 | 扩展未安装/未启用 | 检查 chrome://extensions |
| node_repl.exe 缺失 | Windows 上可执行文件不在 PATH | 复制到 config.toml 指定路径 |
| sandbox = elevated 导致 os error 740 | 权限过高 | 改为 sandbox = unelevated |

## TEMP_MARKER_FIX

```
问题名称：Codex Chrome 插件控制失败，提示当前线程没有 mcp__node_repl__js
适用场景：需要控制用户自己的 Chrome，而不是普通网页自动化
第一动作：让模型通过工具发现搜索 node_repl js
关键判断：mcp__node_repl__js 是 Node REPL 的 js 执行工具
不要先做：不要重装 Chrome / 不要切到 Playwright / 不要用 js_reset 代替 js
升级处理：新开线程 -> 重启 Codex -> 检查扩展 -> 确认 browser-client.mjs 存在
```

