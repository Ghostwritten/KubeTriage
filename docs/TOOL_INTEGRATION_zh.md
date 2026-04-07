# 工具协作使用指南

本文档说明如何在不同工具和界面中使用 `Kubernetes Triage Expert`，同时保持它原本的边界不失真。

最重要的一条原则是：这个 skill 是分析层，不是执行层。

它负责组织诊断过程、排序假设、决定下一步检查项，但不应被当成能看集群、能跑命令、能自动修复问题的系统。

## 目的

这份文档的核心目的，是避免使用方式把 skill 用坏。

如果没有明确的使用说明，用户通常会出现以下误用：

- 默认它能直接检查集群
- 把它当成通用 Kubernetes 百科问答
- 在证据不足时就要求它直接给修复方案

所以这份文档要明确 skill、操作者和外部工具之间的责任分工。

## Skill 的角色

`Kubernetes Triage Expert` 负责：

- 故障归类
- 规范化故障描述
- 区分事实与推测
- 排序最可能的假设
- 推荐下一步最高价值的检查项
- 明确指出何时已经超出 Kubernetes 初筛边界

它不负责：

- 执行命令
- 自动收集日志或指标
- 查询 Kubernetes 对象
- 独立验证实时环境状态
- 实施修复

## 责任模型

当这个 skill 和其他工具一起使用时，责任分工应该保持稳定。

### Skill

- 解释证据
- 组织排障流程
- 收敛搜索空间
- 提出下一步该检查什么

### 用户或值班工程师

- 执行命令
- 获取日志、Events、Metrics、Manifest、发布记录
- 把关键证据贴回对话
- 决定是否实施修复

### 外部工具

- 提供原始运维证据
- 不替代推理
- 应当用来回答 skill 指定的具体检查项

## 如何与 Codex 配合使用

如果你想要一个结构化、多轮推进的排障过程，Codex 是最自然的使用环境。

推荐方式：

1. 安装或加载这个 skill。
2. 在对话里显式点名 skill。
3. 提供故障对象、症状、时间线和最强证据。
4. 让 skill 给出下一步检查项。
5. 你自己执行检查，再把结果返回给 skill。

推荐提示方式：

```text
Use the kubernetes-triage-expert skill and stay within skill boundaries only.

namespace=prod
deployment=checkout-api
pods are CrashLoopBackOff
started after today's rollout
log excerpt: failed to load config: missing REDIS_URL
```

在 Codex 中最适合的场景：

- 多轮 incident triage
- 假设逐步收敛
- 已有结论的证据复核
- 用 examples 和 stress tests 做 prompt 评估

## 如何在普通聊天界面中使用

在普通聊天 UI 中，这个 skill 应当被视为“带强约束的推理助手”。

推荐方式：

- 把 [PROMPT.md](/Users/zongxun/github/KubeTriage/PROMPT.md) 的内容贴进去
- 明确要求 assistant 严格遵守 skill 边界
- 结构化提供证据

不要这样用：

- 让它“直接检查集群”
- 在证据不足时要求直接给修复方案
- 只给一个状态名就让它确认根因

## 如何与 `kubectl` 配合

这是最常见的协作模式。

正确流程：

1. 先把问题描述给 skill。
2. 让 skill 指定下一步最值得检查的内容。
3. 你手动执行相应的 `kubectl` 命令。
4. 把输出结果或精确摘要贴回对话。
5. 让 skill 更新假设和下一步动作。

好的模式：

- skill 要 scheduler event
- 操作者执行 `kubectl describe pod ...`
- 返回关键 event 行
- skill 收敛到调度约束路径

坏的模式：

- 一上来问“你直接告诉我执行什么命令可以修好”

这个 skill 可以说明“下一类证据应该去哪里拿”，但它的首要角色仍然是分析，而不是执行。

## 如何与日志系统配合

日志系统的角色是提供证据。

适合提供的内容包括：

- 启动失败日志
- 终止原因
- 探针相关错误
- 外部依赖连接失败
- 与发布时间相关的异常日志

最佳实践：

- 给短而精确的日志片段，不要给模糊总结
- 带上时间信息
- 标明是当前容器还是 previous container 的日志

好的输入：

```text
Current pod log excerpt:
failed to load config: missing REDIS_URL

Previous container exit code: 1
Started right after rollout at 10:20
```

## 如何与指标和仪表盘配合

Metrics 系统主要用于回答 skill 提出的诊断问题，例如：

- 是否存在内存压力
- 是否有 CPU 饱和
- 副本健康是否下降
- 错误率是否抬升
- 延迟是否尖峰
- 是否和 rollout 时间强相关

指标应该用来回答问题，而不是把整页 dashboard 内容塞进对话。

好的模式：

- skill 问是否存在 OOM 证据
- 操作者去看重启和内存曲线
- 把简明结果返回

坏的模式：

- 直接贴一大段 dashboard 截图总结，但不对应任何假设

## 如何与告警系统配合

告警系统最适合作为 triage 的入口。

推荐输入信息：

- 什么告警触发了
- 什么时候开始
- 影响了哪个服务或 namespace
- 影响面有多大
- 附近是否有发布或配置变更

然后由 skill 把告警转化为：

- 一个主要故障类型
- 初始假设
- 下一步要收集的证据

## 如何与 Runbook 配合

Runbook 和这个 skill 的职责不同。

- runbook 负责沉淀已知处理流程
- skill 负责判断当前证据更接近哪条 runbook，或者目前是否还没到直接套 runbook 的阶段

这个 skill 不应假装执行 runbook。
它最多只能指出：当前证据更像是某类 runbook 路径。

## 推荐的端到端工作流

1. 从症状、时间、对象和最强证据开始。
2. 让 skill 归类主要故障类型。
3. 让 skill 要求接下来 1-3 个最关键的检查项。
4. 使用 `kubectl`、日志、仪表盘、告警等工具去收集这些证据。
5. 把证据返回给 skill。
6. 重复这个过程，直到：
   - 根因被确认
   - 问题明显超出 Kubernetes 初筛范围
   - 需要进入修复决策阶段

## 反模式

不要把这个 skill 当成：

- 已接入集群的 agent
- 自动 incident commander
- 自动修复引擎
- Kubernetes 百科机器人
- 应用代码排错替代品

典型错误用法：

- “你检查下我的集群哪里有问题。”
- “你直接告诉我怎么修。”
- “现在是 CrashLoopBackOff，所以根因是什么？”
- “我贴一张 dashboard 截图，你把所有问题都诊断出来。”

## 示例会话模式

### 第一步

用户：

```text
Use the kubernetes-triage-expert skill.
service is unreachable
namespace=prod
deployment=checkout-api
started after rollout
```

### 第二步

skill：

- 先归类可能的故障族群
- 要求补充下一步缺失证据

### 第三步

用户：

```text
service endpoints are empty
one pod is Running but not Ready
one pod is CrashLoopBackOff
```

### 第四步

skill：

- 如果需要，重新定位主要故障
- 收敛假设
- 给出下一步最高价值检查项

## 总结

用这个 skill 来推理。
用外部工具来取证。
用操作者来执行命令和做变更。

只要这三者边界不混，这个 skill 才会持续保持诚实且有价值。
