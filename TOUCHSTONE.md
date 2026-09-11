# TOUCHSTONE — touchstone 仓库自用

> 本仓库吃自己的狗粮：机制条目按 PCR 格式记录于此，验证 hook/skill 全链路。

## 索引区

| 日期 | 主题 | 状态 | 一句话 |
|---|---|---|---|
| 2026-09-11 | 机制定名 touchstone | BETA | 仓库定名试金石，机制名 PCR，五态 BETA/RC/GA/SUPERSEDED/EOL |
| 2026-09-11 | 三层加载架构落地 | BETA | skill+SessionStart hook+记忆瘦身全部落地，hook 因本机无 jq 改 node 实现 |

## 已归档

（空——待用户裁决）

## 候选区（BETA，未经确认不可作为已验证教训引用）

### 2026-09-11 · 机制定名 touchstone 【BETA】
- 结论：仓库定名 touchstone（试金石），条目格式沿用 ADR 惯例称 PCR（Project Conclusion Records），五态 BETA/RC/GA/SUPERSEDED/EOL；命名原则：不出现 log（语义是流水，与"验证后才入册"相悖）
- 证据：头脑风暴四方向（台账/判例/公证/航海日志）→ 用户哲学视角纠偏（扬弃/抽丝剥茧）→ 工程师可懂性修正（ground-truth/pinned-facts/PCR）→ 用户拍板 touchstone+PCR
- 影响面：仓库命名、文档自称、TOUCHSTONE-TEMPLATE

### 2026-09-11 · 三层加载架构落地 【BETA】
- 结论：①用户级 skill（Claude Code 系为 `~/.claude/skills/touchstone/`）（description 用用户口语触发，含退役判据）②SessionStart hook（fail-open：git root 找一份 TOUCHSTONE.md，注入索引前 30 行+候选计数，非 UTF-8/无文件/非 git 一律静默 exit 0）③记忆条目瘦身为指针仅留"归档/推翻裁决权在用户"
- 证据：三路 subagent 审查（实证/质量/对抗）全部防线合入；hook 四路径压测通过；**关键实证：本机 bash 无 jq（exit 127），改用 node 实现并逐路径压测——避免了带病上线**
- 推翻：初版 jq 管道方案（不可用）；DESIGN.md 原"两周记忆试点后再 skill 化"节奏（用户批准 A 档直接三层落地）
- 影响面：所有仓库的会话启动行为；token 成本 ~800-1500/会话
