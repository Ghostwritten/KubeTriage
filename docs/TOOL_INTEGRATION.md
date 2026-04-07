# Tool Integration Guide

This document explains how to use `Kubernetes Triage Expert` with different tools and interfaces while preserving its intended boundary.

The most important rule is simple: this skill is an analysis layer, not an execution layer.

It should help structure diagnosis, rank hypotheses, and define the next checks. It should not be treated as a cluster observer, command runner, or remediation system.

## Purpose

The goal of this guide is to prevent misuse.

Without explicit usage guidance, users often do one of the following:

- assume the skill can inspect the cluster directly
- use it as a generic Kubernetes Q&A bot
- expect it to generate fixes before evidence is sufficient

This document defines the correct division of responsibility between the skill, the user, and surrounding tools.

## Role of the Skill

`Kubernetes Triage Expert` is responsible for:

- classifying the fault
- normalizing the incident description
- separating facts from guesses
- ranking the most likely hypotheses
- recommending the next highest-value checks
- stating when the issue exceeds Kubernetes triage scope

It is not responsible for:

- running commands
- collecting logs or metrics automatically
- querying Kubernetes objects
- verifying live state independently
- applying changes

## Responsibility Model

When using this skill with other tools, the responsibility split should remain stable.

### The Skill

- interprets evidence
- structures the troubleshooting process
- narrows the search space
- proposes what to inspect next

### The User or Operator

- executes commands
- retrieves logs, events, metrics, manifests, and rollout history
- pastes or summarizes evidence back into the conversation
- decides whether to apply remediation

### External Tools

- expose raw operational evidence
- do not replace reasoning
- should be used to answer the specific checks requested by the skill

## Using the Skill with Codex

Codex is the most natural environment for this skill when you want a structured multi-turn diagnosis workflow.

Recommended usage:

1. Install or load the skill definition.
2. Explicitly invoke the skill by name.
3. Provide the fault object, symptom, time context, and strongest evidence.
4. Let the skill propose the next checks.
5. Run those checks yourself and return the evidence.

Recommended prompt style:

```text
Use the kubernetes-triage-expert skill and stay within skill boundaries only.

namespace=prod
deployment=checkout-api
pods are CrashLoopBackOff
started after today's rollout
log excerpt: failed to load config: missing REDIS_URL
```

Best use cases in Codex:

- multi-turn incident triage
- structured hypothesis refinement
- evidence review
- prompt evaluation against examples and stress tests

## Using the Skill in Generic Chat Interfaces

When used in a regular chat UI, the skill should be treated as a prompt-constrained reasoning assistant.

Recommended usage:

- paste the prompt from [PROMPT.md](/Users/zongxun/github/KubeTriage/PROMPT.md)
- explicitly state that the assistant must remain inside skill boundaries
- provide evidence in a structured way

What to avoid:

- asking the assistant to "check the cluster"
- asking for direct fixes before evidence is sufficient
- giving only a status label and expecting a confirmed root cause

## Using the Skill with `kubectl`

This is the most common integration pattern.

The correct workflow is:

1. Describe the problem to the skill.
2. Let the skill request the next most useful checks.
3. Run the corresponding `kubectl` commands manually.
4. Paste the output or a precise summary back into the conversation.
5. Let the skill update hypotheses and next steps.

Good pattern:

- skill asks for scheduler event text
- operator runs `kubectl describe pod ...`
- operator returns the relevant scheduler event line
- skill narrows the fault path

Bad pattern:

- operator asks "what command should I run to fix this immediately?"

The skill may suggest what kind of evidence is needed next, but it should still behave as an analysis layer first.

## Using the Skill with Logs

Log systems are evidence providers.

Use them to retrieve:

- startup failures
- termination messages
- probe-related errors
- dependency connection failures
- rollout-time regressions

Best practice:

- provide a short, exact excerpt instead of a vague summary
- include timing context
- include whether the log comes from the current or previous container attempt

Good input:

```text
Current pod log excerpt:
failed to load config: missing REDIS_URL

Previous container exit code: 1
Started right after rollout at 10:20
```

## Using the Skill with Metrics and Dashboards

Metrics systems are useful when the skill asks questions about:

- memory pressure
- CPU saturation
- replica health
- request failure rates
- latency spikes
- rollout correlation

Metrics should be used to answer a diagnostic question, not to dump large dashboard summaries into the conversation.

Good pattern:

- skill asks whether OOM evidence exists
- operator checks memory and restart patterns
- operator returns a concise result

Bad pattern:

- pasting an entire dashboard screenshot summary without linking it to a hypothesis

## Using the Skill with Alerts and Incident Systems

Alerts are best used as the entry point to triage.

Recommended input from alerts:

- what fired
- when it started
- affected service or namespace
- blast radius
- whether a rollout or config change happened nearby

The skill should then convert the alert into:

- a primary fault classification
- initial hypotheses
- the next evidence to collect

## Using the Skill with Runbooks

Runbooks and this skill serve different roles.

- runbooks encode known operational procedures
- the skill helps determine which procedure is most relevant, or whether the situation does not yet justify jumping to a runbook

The skill should not pretend to execute runbooks.
It may suggest that the current evidence points toward a known runbook path.

## Recommended End-to-End Workflow

1. Start with the symptom, time, object, and strongest evidence.
2. Let the skill classify the primary fault.
3. Let the skill request the next 1-3 checks.
4. Use tools such as `kubectl`, logs, dashboards, or alerts to gather only that evidence.
5. Return the evidence to the skill.
6. Repeat until:
   - a root cause is confirmed
   - the issue shifts outside Kubernetes triage scope
   - remediation decision-making needs to begin

## Anti-Patterns

Do not use the skill as if it were:

- a cluster-connected agent
- an automated incident commander
- a remediation engine
- a generic Kubernetes encyclopedia
- a substitute for application debugging when the evidence already points to app-level failure

Specific anti-patterns:

- "Check my cluster and tell me the issue."
- "Just give me the fix."
- "This is CrashLoopBackOff, so what is the root cause?"
- "Here is a screenshot of a dashboard, diagnose everything."

## Example Session Pattern

### Step 1

User:

```text
Use the kubernetes-triage-expert skill.
service is unreachable
namespace=prod
deployment=checkout-api
started after rollout
```

### Step 2

Skill:

- classifies the likely fault family
- asks for the next missing evidence

### Step 3

User:

```text
service endpoints are empty
one pod is Running but not Ready
one pod is CrashLoopBackOff
```

### Step 4

Skill:

- reclassifies the primary fault if needed
- narrows hypotheses
- asks for the next highest-value check

## Summary

Use this skill to reason.
Use tools to gather evidence.
Use operators to execute commands and make changes.

If those roles stay separate, the skill remains honest and useful.
