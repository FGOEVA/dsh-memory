# dsh-memory

dsh 持久记忆与梦境整理插件。MCP server，零外部依赖。

## 安装

```bash
cd ~/.dsh
dsh plugin --profile desktop add github:FGOEVA/dsh-memory
```

⚠️ **Windows 用户注意：** `github:` 协议需要 Git 可用。
如果你的 Windows 没装 Git，pnpm 会报错 `git ls-remote failed`。

**备选方案（不装 Git）：**
1. 手动下载 zip：https://github.com/FGOEVA/dsh-memory/archive/refs/heads/main.zip
2. 解压后把整个文件夹放到 `~/.dsh/profiles/desktop/dsh-memory/`
3. 然后在 dsh 客户端用本地路径注册（如果有支持的话），或直接复制 cordis.patch.yml 配置

## 用法

启动 dsh 后模型自动获得以下工具：

| 工具 | 说明 |
|------|------|
| `memory_search(query, limit?, corpus?)` | FTS5 检索，中文短语匹配 |
| `memory_get(path, from?, lines?)` | 按行号读原文 |
| `memory_save(text, target?, section?)` | 写入当日笔记或 MEMORY.md |
| `memory_list()` | 列出所有记忆文件 |
| `memory_stats()` | 统计信息 |

## 配置

将下面的内容追加到 `~/.dsh/web/cordis.patch.yml`（或对应 profile 的 `cordis.patch.yml`）。

该插件使用 **stdio MCP transport**，不需要单独起 HTTP 服务，由 dsh 通过 mcp-client 拉起。

```yaml
# cordis.patch.yml — dsh-memory stdio MCP entry
# ⚠️ 顶层必须是数组格式，不要写成对象！
- insert:
    - id: dsh-memory-mcp
      name: '@deepseek-ai/dsh-mcp-client'
      config:
        serverName: memory
        transport: stdio
        command: node
        args: [./server.mjs]
        cwd: !!js process.cwd()
        env:
          # 显式传 MEMORY_DIR——dsh 会清洗掉 DSH_* 前缀的环境变量
          MEMORY_DIR: !!js process.env.HOME + '/.dsh/memory'
          MEMORY_INSTRUCTIONS_MAX: 12000
```

### Windows 路径适配

如果你在 Windows 上运行，env 里的路径需要改：

```yaml
        env:
          MEMORY_DIR: "C:/Users/WJH/.dsh/memory"
          MEMORY_INSTRUCTIONS_MAX: 12000
```

## 数据目录结构

```
$DSH_HOME/memory/
├── MEMORY.md              # 长期记忆
├── memory/YYYY-MM-DD.md   # 每日笔记
└── dreams/                # 梦境日志 + 备份
```

跟 OpenClaw 共用同一套格式。

## 运行测试

```bash
node test/probe.mjs
```

26 项自测断言全部通过。

## 常见问题

**Q: Windows 上提示 `git ls-remote failed: 'git' 不是内部命令`？**

A: 你的 Windows 没装 Git。解决方案（三选一）：

1. **装 Git** → https://git-scm.com/download/win → 装完重启 CMD/Powershell
2. **等 npm 化** → 我把包发布到 npm 后就不用 git 了
3. **手动放文件** → 见上方「备选方案」

**Q: dsh 启动后报 `Unknown method` 或 MCP 连接失败？**

A: 检查 cordis.patch.yml 是否以 `- insert:` 开头（顶层必须是数组），以及 `server.mjs` 路径是否正确。
