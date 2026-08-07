# 问小白对话提取器 Skill

> 一键导出你在 wenxiaobai.com 的**全部历史对话**为本地 Markdown 文件。
> 零脚本 · 纯 Codex 内置浏览器控制 · 自动滚动加载 · 逐个提取全文

v2.1 基于真实页面全面验证，修复 URL 模式、CSS 选择器、加载更多逻辑，并新增验证码绕过处理。

---

## 快速开始

告诉 Codex：

```
帮我安装这个 skill：https://github.com/MTAlexKen/codex-wenxiaobai-extractor
```

安装完成后，Codex 会自动：
1. 连接到你已登录的 Chrome 浏览器
2. 滚动侧边栏加载全部对话
3. 逐个提取对话内容
4. 保存为 `.md` 文件到 `~/Desktop/问小白对话记录/`

---

## 前置条件

- **Codex 桌面应用**已安装 Chrome 扩展
- Chrome 浏览器中**已登录**问小白
- 当前 Codex 线程**有 `mcp__node_repl__js` 工具**

> 如果 `mcp__node_repl__js` 不可用，请新开线程、重启 Codex 或检查 Chrome 扩展。详见 [references/BROWSER_FIX.md](references/BROWSER_FIX.md)。

---

## 执行流程

| 阶段 | 说明 |
|------|------|
| 一、确认 Chrome 控制 | 检查 `mcp__node_repl__js`，不可用则自动修复 |
| 二、初始化浏览器 | 加载 browser-client.mjs，获取 Chrome 实例 |
| 三、获取对话列表 | 导航 → 处理验证码 → 滚动+点击"更早"加载全部 |
| 四、逐个提取内容 | 遍历对话 → 导航到页面 → 提取全文 → 保存 MD |
| 五、完成报告 | 报告提取数量、保存路径、失败文件 |

---

## 输出格式

| 项目 | 说明 |
|------|------|
| 保存位置 | `~/Desktop/问小白对话记录/` |
| 文件命名 | `0001_对话标题.md` ~ `9999_对话标题.md` |
| 文件编码 | UTF-8 with BOM（确保中文正常显示） |
| 内容格式 | Markdown：标题 + 对话ID + 提取时间 + 消息记录 |

---

## CSS 选择器参考

基于真实页面验证（2026-08-07）：

| 元素 | 选择器 | 说明 |
|------|--------|------|
| 侧边栏容器 | `.ConversationHistory_listContainer__WDjRs` | 对话列表滚动区域 |
| 对话项 | `[data-conversation-id]` | 需过滤空值 |
| 对话标题 | `.ConversationItem_contents__M8Tvu` | 对话标题文本 |
| 加载更多 | `div[textContent="更早"]` | class: `ConversationHistory_groupTittle__BWuv6` |
| 聊天内容 | `[class*="ChatContent"]` | 主聊天区域 |
| 验证码 iframe | `iframe[src*="captcha"]` | 需移除 |

---

## 故障排除

### `mcp__node_repl__js` 不可用

1. 新开 Codex 线程
2. 重启 Codex 应用
3. 检查 Chrome 扩展

### 提取数量少

- 技能会自动点击"更早"和滚动，大量对话需要更长时间

### 文件内容为空

- 对话可能已被删除或需要特殊权限

### 文件名乱码

- 文件使用 UTF-8 with BOM，用 VS Code/Notepad++ 打开即可

### 验证码无法绕过

- 手动在浏览器中完成滑块验证后重新执行

---

## 仓库结构

```text
wenxiaobai-extractor/
├── SKILL.md                      # 技能主指令
├── agents/
│   └── openai.yaml               # Codex UI 元数据
└── references/
    └── BROWSER_FIX.md            # Chrome 控制修复指南
```

---

## 致谢

- 修复方法论基于 [4SAPI 文章](https://www.4sapi.net/blog/codex-chrome-plugin-node-repl-4sapi)
- 真实修复案例来自 Codex 对话 `019fcc9f-2278-78a0-8d04-c0028fc16b11`

---

## 许可证

[MIT](LICENSE)
