# @dsh-plugins/dsh-memory

dsh 持久记忆与梦境整理插件。MCP server，零外部依赖。

## 安装

```bash
cd ~/.dsh
dsh plugin --profile web add github:FGOEVA/dsh-memory
```

然后在 `~/.dsh/web/cordis.patch.yml` 注入 MCP server entry（见下方配置说明）。

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

编辑 `~/.dsh/web/cordis.patch.yml` 添加：

```yaml
- insert:
    - id: dsh-memory
      name: '@deepseek-ai/dsh-mcp-client'
      config:
        serverName: dsh_memory
        transport: stdio
        command: npx
        args: ['dsh-memory']
        cwd: !!js process.cwd()
        env:
          MEMORY_DIR: !!js process.env.DSH_HOME + '/memory'
          MEMORY_INSTRUCTIONS_MAX: 12000
```

⚠️ `config.env` 里显式传 `MEMORY_DIR`——dsh 会清洗掉 `DSH_*` 前缀的环境变量。

## 数据格式

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
