# Kubernetes Triage Expert

Kubernetes Triage Expert is a prompt-first troubleshooting skill specification for Kubernetes incidents. It is built to help engineers reason about faults using only the evidence already present in the conversation, without pretending to be an agent, a tool runner, or a live cluster observer.

This repository defines a narrow and deliberate operating model: structured Kubernetes triage with hard boundaries.

## Overview

General-purpose conversational AI often underperforms in Kubernetes troubleshooting because it tends to:

- guess too early
- confuse symptoms with root causes
- produce long generic cause lists
- speak as if it has already inspected the environment

This project exists to correct that behavior. It specifies a disciplined skill that:

- classifies fault types
- normalizes incident reports
- separates facts from hypotheses
- ranks the most likely causes
- recommends the next highest-value checks
- remains honest about what a skill can and cannot do

## Goals

- Provide a professional specification for Kubernetes fault triage.
- Preserve evidence discipline in every response.
- Improve the quality of the next troubleshooting step, not just the explanation.
- Prevent common prompt failures such as overclaiming, generic advice dumping, and fake environment awareness.
- Make behavior testable through examples and stress cases.

## Non-Goals

- Running `kubectl` or any cluster command
- Integrating with Kubernetes, cloud providers, CI/CD, or observability backends
- Acting as an agent, MCP server, or automation workflow
- Applying remediation or generating unaudited fixes
- Replacing deep investigation of application code, runtime internals, or cloud-provider infrastructure

## Intended Audience

This repository is aimed at:

- SRE engineers
- DevOps engineers
- platform engineers
- prompt designers working on operational AI behavior
- teams evaluating whether a Kubernetes troubleshooting skill can stay useful without tool access

## Repository Contents

- [SKILL.md](/Users/zongxun/github/KubeTriage/SKILL.md): full behavioral contract and troubleshooting specification
- [PROMPT.md](/Users/zongxun/github/KubeTriage/PROMPT.md): compact runtime prompt suitable for direct deployment
- [EXAMPLES.md](/Users/zongxun/github/KubeTriage/EXAMPLES.md): positive and negative examples
- [STRESS_TESTS.md](/Users/zongxun/github/KubeTriage/STRESS_TESTS.md): adversarial evaluation cases
- [docs/README.md](/Users/zongxun/github/KubeTriage/docs/README.md): Chinese documentation

## Design Principles

- Evidence first. No root-cause claim without user-supplied evidence.
- Earliest failure first. Triage starts at the first real break in the failure chain.
- Minimal next step. Recommend only the checks that materially change diagnosis.
- Boundary honesty. Never imply environment access or execution capability.
- Auditable reasoning. Keep confirmed facts, likely hypotheses, ruled-out paths, and missing evidence separate.
- Triage over encyclopedias. The skill should narrow the search space, not recite everything it knows.

## Operating Model

The skill is intentionally narrow.

It may:

- analyze user-provided statuses, logs, events, and change context
- classify incident types
- rank likely hypotheses
- recommend the next 1-3 checks
- state when the problem likely exceeds Kubernetes triage scope

It may not:

- inspect the cluster
- run commands
- fetch telemetry
- verify live state independently
- fabricate certainty where evidence is missing

## Supported Fault Families

The skill is optimized for high-frequency Kubernetes fault classes, including:

- `CrashLoopBackOff`
- `Pending`
- `ImagePullBackOff` and `ErrImagePull`
- service reachability failures
- rollout regressions
- probe-related instability
- storage-related scheduling blockers

Its responsibility is not to solve every incident end to end. Its responsibility is to produce the next correct troubleshooting move.

## Recommended Reading Order

1. Read [SKILL.md](/Users/zongxun/github/KubeTriage/SKILL.md) for the full design contract.
2. Use [PROMPT.md](/Users/zongxun/github/KubeTriage/PROMPT.md) as the deployment-ready prompt.
3. Review [EXAMPLES.md](/Users/zongxun/github/KubeTriage/EXAMPLES.md) to understand expected good and bad behavior.
4. Use [STRESS_TESTS.md](/Users/zongxun/github/KubeTriage/STRESS_TESTS.md) to evaluate robustness and boundary discipline.

## Example Output Shape

```md
Fault Assessment
- Type:
- Severity guess:
- Current phase:

Confirmed Facts
- ...

Top Hypotheses
1. ...
2. ...
3. ...

Next Checks
1. Check:
   Why:
   If true:
   If false:

Current Conclusion
- Confirmed:
- Likely:
- Ruled out:
- Still needed:
```

## Evaluation Criteria

A strong response should:

- classify the primary fault correctly
- reduce uncertainty
- avoid overclaiming
- keep the next checks limited and high-value
- distinguish Kubernetes-level problems from application, runtime, or cloud-provider problems

A weak response usually:

- lists too many causes
- guesses without evidence
- confuses downstream symptoms with the primary failure
- speaks as if it already inspected the environment

## Roadmap

Current scope:

- skill specification
- runtime prompt
- example set
- stress-test set

Possible next steps:

- tighter response rubrics for evaluation
- fault-family-specific prompt variants
- benchmark conversations for side-by-side prompt comparison
- review checklists for prompt regressions

## Status

This repository currently contains the design package for a Kubernetes troubleshooting skill. It is suitable for prompt review, evaluation, iteration, and controlled testing.
