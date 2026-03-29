# 讨论记录 002：接口与 Schema 初探

**日期**：2026-03-29  
**参与者**：用户、Kimi Code CLI

---

## 1. 背景

在 001 号讨论中，已确定项目采用**分层记忆模型**和**语言中立**的设计原则。本次讨论聚焦于系统对外暴露的接口契约和记忆的数据结构（Schema）。

## 2. 核心接口候选集

### 2.1 读写操作

| 操作 | 作用 | 示例 |
|------|------|------|
| `add` | 写入一条记忆 | 记录用户偏好 |
| `search` | 语义检索相关记忆 | 搜索"用户喜欢的编程语言" |
| `get` | 按 ID 精确读取 | 查看某条记忆的完整内容 |
| `update` | 更新已有记忆 | 偏好从 Python 改为 Rust |
| `delete` | 删除或归档记忆 | 主动遗忘过时信息 |

### 2.2 元数据与管理操作

| 操作 | 作用 |
|------|------|
| `list` | 列出记忆，支持按时间、类型、来源过滤 |
| `summarize` | 对一批记忆生成摘要（短期记忆压缩） |
| `reflect` | 主动整理记忆，发现冲突、合并重复、生成洞察 |

## 3. 接口形态

这些操作可以通过多种形态暴露：

- **MCP Tools**：`memory_add`, `memory_search`, `memory_update`...
- **REST API**：`POST /memories`, `GET /memories/search`...
- **CLI**：`memsys add "..."`, `memsys search "..."`
- **gRPC**：适合高性能、强类型的跨语言调用

**当前状态**：接口形态**尚未确定**，用户明确表示这是后续必须确定的事项。

## 4. 记忆 Schema 草案

```json
{
  "id": "uuid-v4",
  "content": "用户明确说明自己更喜欢用 Go 写后端服务，而不是 Python。",
  "type": "preference",
  "source": "user_explicit",
  "confidence": 1.0,
  "created_at": "2026-03-29T10:00:00Z",
  "updated_at": "2026-03-29T10:00:00Z",
  "expires_at": null,
  "tags": ["programming", "language", "go"],
  "relations": [
    { "target_id": "another-memory-uuid", "relation_type": "contradicts" }
  ],
  "embedding": [0.12, -0.05],
  "metadata": {
    "session_id": "abc-123",
    "agent_id": "kimi-cli"
  }
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string (UUID) | 唯一标识 |
| `content` | string | 记忆的文本内容 |
| `type` | string | `fact`, `preference`, `event`, `insight`, `task` 等 |
| `source` | string | `user_explicit`, `inferred`, `system_generated` |
| `confidence` | float [0,1] | 置信度 |
| `created_at` | ISO 8601 | 创建时间 |
| `updated_at` | ISO 8601 | 最后更新时间 |
| `expires_at` | ISO 8601 / null | 过期时间 |
| `tags` | string[] | 标签 |
| `relations` | object[] | 与其他记忆的关系（图扩展预留） |
| `embedding` | float[] / null | 向量表示 |
| `metadata` | object | 扩展字段 |

## 5. 待澄清问题

以下问题在本次讨论中提出，但尚未得出结论：

1. **接口粒度**：5 个核心操作是否足够？MVP 阶段是否需要 `summarize` 或 `reflect`？
2. **Schema 字段**：
   - 是否需要 `version` 字段追踪修改历史？
   - `relations` 在 MVP 阶段是否过度设计？
   - 是否需要 `scope` 字段区分项目/用户/命名空间？
3. **接口形态**：最终选择 MCP / REST / CLI / gRPC 中的哪一种或哪几种组合？

## 6. 下一步

等待用户就上述问题给出方向，或继续探讨其他设计层面（如存储引擎选型、记忆的生命周期策略、嵌入模型集成方式等）。
