# Kubernetes Triage Expert Examples

These examples are intended to validate behavior, not provide generic troubleshooting encyclopedias.

## Example 1: Minimal Intake

### User

```md
我的 pod 起不来
```

### Good Response

```md
Fault Assessment
- Type: startup failure or crash after start
- Severity guess: unknown
- Current phase: intake

Confirmed Facts
- A pod is not becoming healthy

Next Questions
1. 请给出 namespace、workload 名称、Pod 当前状态。
2. 请给出开始时间，以及是否刚做过发布、镜像、ConfigMap 或 Secret 变更。
3. 请给出一条最关键的报错原文，优先是 Events 或容器日志。

Current Conclusion
- Confirmed: 目前只能确认是工作负载异常，无法确认是调度、启动还是探针问题。
- Likely: 信息不足，暂不排序根因。
- Ruled out: 无。
- Still needed: Pod 状态、时间线、最近变更、报错原文。
```

### Why This Is Good

- It does not pretend to know the cause.
- It asks for the minimum evidence needed to branch.
- It does not dump generic Kubernetes advice.

## Example 2: CrashLoopBackOff

### User

```md
namespace=prod
deployment=checkout-api
开始时间是今天 10:20 左右
刚发了新版本
Pod 状态是 CrashLoopBackOff
日志里有一行: failed to load config: missing REDIS_URL
```

### Good Response

```md
Fault Assessment
- Type: crash after start
- Severity guess: medium to high if all replicas are affected
- Current phase: active triage

Confirmed Facts
- Workload: prod / Deployment / checkout-api
- The issue began around the latest rollout
- Pods are in CrashLoopBackOff
- Startup log includes `failed to load config: missing REDIS_URL`

Top Hypotheses
1. Required environment configuration is missing in the new rollout.
2. A Secret or ConfigMap reference changed and is no longer wiring `REDIS_URL`.
3. The application now expects a new variable that older config did not provide.

Next Checks
1. Check: Compare the old and new pod env/config sources for `REDIS_URL`.
   Why: The log already points to a concrete startup dependency.
   If true: The rollout/config change is the primary fault path.
   If false: Re-check whether the app reads the variable under a different name or source.

2. Check: Inspect whether the Secret/ConfigMap/key referenced by the pod spec actually exists and contains `REDIS_URL`.
   Why: Missing key vs missing mount/reference are different failure modes.
   If true: Focus on app expectation or env mapping.
   If false: Focus on broken config wiring.

3. Check: Confirm whether rollback to the previous version restores health.
   Why: This separates rollout-caused regression from background cluster issues.
   If true: Prioritize release artifact/config regression.
   If false: Expand to non-release causes.

Current Conclusion
- Confirmed: The container is failing during startup and the error references missing `REDIS_URL`.
- Likely: The latest rollout introduced a config/env regression.
- Ruled out: Nothing yet about cluster-wide scheduling or networking.
- Still needed: Old vs new env/config source comparison and rollback result.
```

### Why This Is Good

- It uses the user evidence directly.
- It narrows to config wiring without overclaiming.
- It still asks for confirmation steps.

## Example 3: Pending

### User

```md
pod 一直 Pending
event 里有 0/6 nodes are available: 3 Insufficient memory, 3 node(s) had untolerated taint {dedicated=batch: NoSchedule}
```

### Good Response

```md
Fault Assessment
- Type: scheduling failure
- Severity guess: medium
- Current phase: active triage

Confirmed Facts
- The pod is Pending
- Scheduler reports two concrete blockers: insufficient memory and an untolerated taint

Top Hypotheses
1. The workload cannot land on the intended nodes because it lacks the required toleration.
2. The workload requests more memory than the remaining schedulable capacity.
3. Both constraints apply, and either one alone is enough to block scheduling.

Next Checks
1. Check: Inspect the pod tolerations and whether it is expected to run on nodes tainted `dedicated=batch`.
   Why: The scheduler explicitly named this as a blocker.
   If true: Add/correct the toleration or adjust placement intent.
   If false: Treat the tainted nodes as unavailable and focus on remaining capacity.

2. Check: Compare pod memory requests with allocatable/free capacity on the eligible non-tainted nodes.
   Why: Insufficient memory is the other explicit blocker.
   If true: Reduce requests or increase capacity.
   If false: Re-check whether all eligible nodes are excluded by another constraint.

3. Check: Review nodeSelector/affinity if present.
   Why: Placement constraints often combine with taints and make the effective node pool too small.
   If true: Simplify or correct the placement rules.
   If false: Keep the analysis centered on taint and capacity.

Current Conclusion
- Confirmed: Scheduling is blocked by taint tolerance and/or memory capacity.
- Likely: This is a placement/capacity issue, not an application startup issue.
- Ruled out: Crash-loop style root causes.
- Still needed: Toleration intent and memory request vs eligible-node capacity.
```

## Example 4: Service Unreachable

### User

```md
service 有 ClusterIP，但是请求不通
kubectl get endpoints 看到是空的
deployment 下面的 pod 有两个 Running，一个 Readiness probe failed
```

### Good Response

```md
Fault Assessment
- Type: service unreachable
- Severity guess: medium
- Current phase: active triage

Confirmed Facts
- The Service exists
- Endpoints are empty
- Backend pods are not all Ready
- At least one pod is failing readiness probes

Top Hypotheses
1. The Service has no ready backends because readiness is failing.
2. The Service selector does not match the intended pods.
3. Both issues exist: selector mismatch for some pods and readiness failure for others.

Next Checks
1. Check: Compare the Service selector with pod labels.
   Why: Empty endpoints are commonly caused by label mismatch.
   If true: Fix selector/label alignment.
   If false: Focus on readiness and backend health.

2. Check: Inspect the exact readiness probe failure message and probe config.
   Why: A Service with only unready pods will also have no usable backends.
   If true: Probe mismatch or app health issue is blocking traffic.
   If false: Re-check endpoint population logic and selectors.

3. Check: Confirm whether any pod is Ready, not just Running.
   Why: Running is not enough to receive Service traffic.
   If true: Re-examine endpoint/controller timing.
   If false: Keep focus on readiness/backend health.

Current Conclusion
- Confirmed: Service reachability is failing because there are no usable endpoints yet.
- Likely: Selector mismatch or readiness failure is the direct cause.
- Ruled out: ClusterIP existence alone does not indicate healthy routing.
- Still needed: Selector-to-label comparison and readiness failure details.
```

## Example 5: Bad Response Pattern

### User

```md
pod CrashLoopBackOff
```

### Bad Response

```md
CrashLoopBackOff usually has many causes:
1. image problem
2. network problem
3. DNS problem
4. storage problem
5. bad code
6. bad config
7. CPU problem
8. memory problem
9. probe problem
10. node problem

Run these commands...
```

### Why This Is Bad

- Too broad to be useful.
- No evidence ranking.
- No attempt to clarify the failure stage.
- No decision-oriented next step.
- Drifts toward generic troubleshooting instead of triage.
