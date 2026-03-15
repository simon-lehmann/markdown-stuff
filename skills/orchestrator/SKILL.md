---
name: orchestrator
description: Two-subagent generate/validate loop for code generation tasks.
  The orchestrator controls flow — it never generates or validates code itself.
  Triggers on: generate code, build feature, implement spec, create component,
  or any multi-step code production task that benefits from iterative validation.
---

# Orchestrator Pattern

This codebase uses a two-subagent loop for all code generation tasks.
The orchestrator controls flow only — it never generates or validates code itself.

---

## Subagents

### GENERATE

Owns all code production.

- Receives: spec + active skills + violations (empty on first pass)
- Accumulates violation history across iterations so it can cross-reference previous attempts
- Never self-validates

### VALIDATE

Owns the exit condition.

- Receives: generated code only — no spec, no generator context, no intent
- Loads the validation skills defined in the active skill set for this task
- Returns: `{ pass: boolean, violations: Violation[] }`

---

## Loop

```
violations = []

loop:
  code = GENERATE(spec, skills, violations)
  result = VALIDATE(code)

  if result.pass        → return code
  if retries >= MAX     → ESCALATE(violations)
  else                  → violations = result.violations, retries++
```

MAX retries is 3 unless overridden by a skill.

---

## Skills

Skills are the only variable in this pattern. Each project defines an active skill set
that scopes what VALIDATE checks. Examples:

| Domain     | Possible validation skills                          |
|------------|-----------------------------------------------------|
| UI         | design-contract, tsc, eslint                        |
| Logic      | tsc, vitest/jest, eslint                            |
| API routes | tsc, schema-validation, integration-tests           |
| Full stack | tsc, jest, design-contract, schema-validation       |

The orchestrator loads the active skill set at the start of a task.
VALIDATE runs each skill in sequence and aggregates violations before returning.

Skill resolution order:

1. Task-level override (explicit in the prompt)
2. Project-level default (defined below)
3. Fallback: tsc only

## Project defaults

<!-- Override this section per project -->
active_skills:

- tsc

---

## Violation schema

Every validation skill must return violations in this shape:

```ts
type Violation = {
  skill: string       // which skill caught this
  location: string    // file + line if available
  found: string       // what was found
  expected: string    // what the contract requires
  severity: "error" | "warning"
}
```

Only `error` severity blocks the loop. `warning` is passed to GENERATE as context
but does not prevent exit.

---

## Escalation

When retries === MAX, the orchestrator stops the loop and surfaces to the user:

```
Could not resolve the following violations after 3 attempts:

[violation list]

Options:
  1. Clarify the spec
  2. Relax a constraint in the active skill
  3. Override and accept current output
```

The orchestrator never silently exits a loop with unresolved errors.

---

## Rules

- Orchestrator owns: loop control, retry count, state passing, escalation
- GENERATE owns: code production, violation history
- VALIDATE owns: exit condition, skill execution
- VALIDATE never receives generator context — cold isolation is mandatory
- No subagent self-validates its own output
- Violations always flow forward — GENERATE sees the full history, not just the latest
