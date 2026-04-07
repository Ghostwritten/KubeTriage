# Kubernetes Triage Expert Stress Tests

These cases are designed to test whether the skill stays inside its boundary, preserves evidence discipline, and produces decision-oriented triage instead of generic troubleshooting.

## Test 1: Extremely Thin Input

### User Input

```md
线上挂了，k8s 有问题
```

### What Good Behavior Looks Like

- Classify as `unknown / insufficient evidence`
- Ask only the minimum questions needed to branch
- Do not guess a root cause
- Do not output a giant list of causes

### Failure Pattern

- "This is usually caused by network, DNS, storage, or deployment issues..."

## Test 2: Symptom Chain Misclassification

### User Input

```md
外部访问 503，ingress 没流量，后端 pod 都是 Pending
```

### What Good Behavior Looks Like

- Classify as `scheduling failure`, not `ingress issue`
- Explain that service exposure is downstream of unscheduled backends
- Focus next checks on scheduler events and placement constraints

### Failure Pattern

- Starting with ingress annotations, TLS, or path routing before addressing `Pending`

## Test 3: Status Name Overreach

### User Input

```md
pod CrashLoopBackOff
```

### What Good Behavior Looks Like

- Treat `CrashLoopBackOff` as a symptom, not a root cause
- Ask for exit code, logs, probe state, recent changes
- Avoid broad or confident diagnosis

### Failure Pattern

- "The app definitely has a config problem"

## Test 4: Rollout Correlation

### User Input

```md
11:05 发布后新 pod 陆续不健康，老 pod 还正常
```

### What Good Behavior Looks Like

- Classify as `rollout regression` or `crash after start` depending on additional evidence
- Heavily prioritize change-related hypotheses
- Ask for image/config/probe/resource differences and rollback result

### Failure Pattern

- Ignoring the rollout timing and starting with cluster-wide speculation

## Test 5: App vs Kubernetes Boundary

### User Input

```md
Pod Running，Readiness 也正常，但接口返回 500，日志里是 SQL syntax error
```

### What Good Behavior Looks Like

- State clearly that Kubernetes health is not the main unknown
- Identify this as likely application or dependency behavior surfacing in Kubernetes
- Stay useful, but note that further root cause is outside pure Kubernetes triage

### Failure Pattern

- Continuing as if this were primarily a Kubernetes scheduler/network issue

## Test 6: Service Object False Confidence

### User Input

```md
Service 有 ClusterIP，所以网络应该没问题吧？
```

### What Good Behavior Looks Like

- Reject the inference
- Explain that Service existence does not prove backend readiness or endpoint health
- Ask for endpoints and pod readiness

### Failure Pattern

- Accepting the Service object as proof of healthy service routing

## Test 7: Multi-Cause Scheduler Event

### User Input

```md
0/8 nodes are available: 2 Insufficient cpu, 3 node(s) had untolerated taint, 3 node(s) didn't match Pod's node affinity
```

### What Good Behavior Looks Like

- Preserve multiple blockers without collapsing them incorrectly
- Rank the most actionable constraints first
- Recommend checks that disambiguate effective eligible node pool and placement intent

### Failure Pattern

- Focusing on only one scheduler reason and ignoring the rest

## Test 8: Probe Mismatch vs App Crash

### User Input

```md
容器能启动十几秒，随后被重启，event 里一直是 Liveness probe failed
```

### What Good Behavior Looks Like

- Prioritize probe mismatch or unhealthy startup path
- Not jump straight to "application exits immediately"
- Ask for probe config, thresholds, endpoint behavior, and startup expectations

### Failure Pattern

- Treating this as a generic crash-loop without using the probe evidence

## Test 9: Missing Evidence Boundary

### User Input

```md
怀疑是 CNI 问题，但是我现在没有 node 日志，也没有网络策略信息
```

### What Good Behavior Looks Like

- State that the current evidence is insufficient to confirm a CNI issue
- Explicitly note the boundary and what evidence is required next
- Avoid pretending to validate node/network internals

### Failure Pattern

- "Yes, this is likely the CNI plugin"

## Test 10: Image Pull Auth Signal

### User Input

```md
event: Failed to pull image, unauthorized: authentication required
```

### What Good Behavior Looks Like

- Prioritize registry auth / imagePullSecrets / permissions
- Ask for exact image reference and recent secret changes
- Keep network timeout causes lower priority

### Failure Pattern

- Treating this mainly as registry downtime or generic network failure

## Test 11: Contradictory Evidence

### User Input

```md
我觉得是内存不够，但 describe 里没有 OOMKilled，日志是 failed to parse config
```

### What Good Behavior Looks Like

- Challenge the user's conclusion respectfully
- Re-rank hypotheses toward config/startup issues
- Explain why the current evidence weakens the memory hypothesis

### Failure Pattern

- Agreeing with the user without evidence review

## Test 12: PVC as Hidden Root of Pending

### User Input

```md
pod Pending，event 里有 pod has unbound immediate PersistentVolumeClaims
```

### What Good Behavior Looks Like

- Classify as `scheduling failure` with storage as the key sub-path
- Prioritize PVC binding state over CPU/memory speculation
- Ask for PVC/PV/storageClass state

### Failure Pattern

- Defaulting to generic capacity or node shortage checks

## Test 13: Endpoint Empty With Ready Pods

### User Input

```md
service endpoints 为空，但 pod 都是 Ready
```

### What Good Behavior Looks Like

- Prioritize selector mismatch or namespace/object mismatch
- Ask for service selector and pod labels
- Not blame readiness first because the user already provided contrary evidence

### Failure Pattern

- Ignoring the Ready evidence and repeating backend health theory

## Test 14: False Rollout Correlation

### User Input

```md
刚发版后报警了，但 scheduler event 是 node had untolerated taint
```

### What Good Behavior Looks Like

- Do not overfit to rollout timing
- Keep rollout as context, but rank the explicit scheduler blocker first
- Explain why concrete scheduling evidence outranks temporal correlation

### Failure Pattern

- Assuming the rollout artifact is the primary cause despite stronger contrary evidence

## Test 15: Boundary Against Remediation

### User Input

```md
你直接告诉我怎么改 yaml 修好它
```

### What Good Behavior Looks Like

- Keep the role as triage-first
- State that the fix depends on confirmed evidence
- If enough evidence exists, suggest likely fix direction conditionally, not as fabricated certainty

### Failure Pattern

- Inventing a patch without enough supporting evidence

## Evaluation Checklist

Use these questions to score responses:

1. Did it classify the primary failure correctly?
2. Did it keep facts separate from hypotheses?
3. Did it avoid pretending to inspect the environment?
4. Did it avoid broad, generic troubleshooting dumps?
5. Did it choose the next checks based on decision value?
6. Did it respect Kubernetes vs app/cloud/runtime boundaries?
7. Did it avoid confirming a root cause prematurely?
8. Did it use recent-change timing appropriately without overfitting?
9. Did it follow the default Chinese-output rule, keeping English only for standard technical terms when clearer?
