# 销售作战指南 — 分析规则

本文件供主 agent 参考，不启动 sub-agent。按 M1-M6 模块逐一分析，直接输出 JSON。

不做搜索，基于已读取的数据分析。信息不足时基于行业经验做合理推断。

---

## 分析模块

### M1：时机判断

根据 profile 中的信号判断时机阶段（刚融资/扩张期/转型期/稳定期/危机期）。

输出：时机阶段 + 切入策略 + 紧急程度（高/中/低）

### M2：竞对作战卡

识别 ≤3 个竞对（含"现状竞对"如 Excel/人工），每个竞对：
- 竞对优势（2-3 条）
- 竞对弱点（2-3 条）
- 应对策略（一句话）

### M3：决策链

- 决策者 ≤3 人：姓名/部门 + 原因
- 影响者 ≤3 人：姓名/部门 + 原因
- 阻碍者 ≤2 人：姓名/部门 + 原因

### M4：禁区话题

从 profile 中识别敏感区域（司法风险、负面新闻、人事变动等），≤3 条。

### M5：访谈提纲（从 Requirements 同步）

**不再独立生成问题。** 所有问题统一由 `/requirements` 的 `pendingQuestions` 管理。

从 `requirements.json` 的 `current.pendingQuestions[]` 中读取，按 stage 分组筛选 status=pending 或 partially-answered 的问题：

- **screening**：首次接触可问的问题
- **deep-dive**：深入沟通时的需求验证问题
- **closing**：收尾确认和推进问题

直接引用问题的 id、question、priority、status，写入 `interviewGuide.fromRequirements`。

如果 `requirements.json` 不存在，`interviewGuide.fromRequirements` 输出空数组。

**tracking 计算**：
- `totalCount`：requirements 中所有 pendingQuestions 的数量
- `coveredCount`：status 为 answered 或 resolved 的数量
- `coverageRate`：coveredCount / totalCount（totalCount 为 0 时，coverageRate = 0）
- `pendingCritical`：status=pending 且 priority=必问 的问题 id 列表
- requirements.json 不存在时：全部归零（coverageRate=0, coveredCount=0, totalCount=0, pendingCritical=[]）

### M6：当前阶段建议

基于 M1 判断的时机阶段，给出当前阶段的销售建议：
- focus：当前阶段最应该关注什么（一句话）
- approach：推荐的打法/接触方式（一句话）
- risks：当前阶段最大的风险或容易犯的错（一句话）

---

## 迭代规则

当已有 sales-guide.json 时（模式 B）：
- 只更新新输入影响的模块，其余保留原文
- 不重新分析 profile（信息已融入首次生成）
- 生成 changeSummary[] 记录本次变更

---

## 输出 JSON 结构

```json
{
  "timing": {
    "timingStage": "",
    "entryStrategy": "",
    "urgency": ""
  },

  "stageAdvice": {
    "focus": "",
    "approach": "",
    "risks": ""
  },

  "competitors": [
    {
      "name": "",
      "threat": "高|中|低",
      "strengths": [],
      "weaknesses": [],
      "counterStrategy": ""
    }
  ],

  "decisionChain": {
    "decisionMakers": [
      { "name": "", "department": "", "reason": "" }
    ],
    "influencers": [
      { "name": "", "department": "", "reason": "" }
    ],
    "blockers": [
      { "name": "", "department": "", "reason": "" }
    ]
  },

  "avoidTopics": [],

  "interviewGuide": {
    "fromRequirements": [
      {
        "questionId": "PQ-001",
        "stage": "screening|deep-dive|closing",
        "question": "",
        "priority": "必问|选问",
        "status": "pending|partially-answered"
      }
    ]
  },

  "tracking": {
    "coverageRate": 0,
    "coveredCount": 0,
    "totalCount": 0,
    "pendingCritical": []
  },

  "metadata": {
    "generatedAt": null,
    "updatedAt": null,
    "version": "1.0"
  }
}
```

### 输出约束

- competitors ≤ 3
- decisionMakers ≤ 3，influencers ≤ 3，blockers ≤ 2
- interviewGuide.fromRequirements：直接从 requirements.json 筛选，不自行生成
- avoidTopics ≤ 3
- 所有文本中文，具体可执行
