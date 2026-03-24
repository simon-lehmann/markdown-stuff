---
name: ui-ux
description: >-
  UI generation and validation skill for React + Tailwind projects.
  Use whenever building screens, layouts, components, or any UI task.
  Enforces layered composition - screens must be built from layout patterns
  and assembly components, never directly from atoms. Triggers on: build a screen,
  create a page, add a layout, build a component, or any UI generation task.
---

# UI/UX Skill

Screens are always built top-down through three layers. Never skip layers.

```
LAYOUT     page skeletons — structure and spatial zones
ASSEMBLY   compositions — groups of components with purpose  
ATOMS      base components — buttons, inputs, cards
```

**New screens must start at LAYOUT, not ATOM.**

---

## Project Config

```
layouts:  src/components/layouts/
assembly: src/components/assembly/
atoms:    src/components/atoms/
tokens:   tailwind.config.ts
```

Override these paths in your project's CLAUDE.md if your structure differs.

---

## GENERATE Instructions

### Before writing any screen

1. Read the `layouts/` directory — scan all files and their `@layer` descriptions
2. Read the `assembly/` directory — scan all files and their `@assembly` descriptions
3. State out loud:
   - Which layout you are using and why
   - Which assembly components fill each zone
   - What needs to be created that doesn't exist yet

Create missing layouts or assembly components before building the screen.

### Component descriptions (required on every component)

Every generated component must open with a description block.
This is the registry — there are no separate reference files.

**Layout component:**

```tsx
/**
 * @layer layout
 * @name DashboardShell
 * @use Authenticated screens with persistent sidebar nav
 * @zones sidebar | header | main | aside?
 * @responsive sidebar collapses to bottom nav at sm
 * @never marketing pages, onboarding, auth screens
 */
```

**Assembly component:**

```tsx
/**
 * @layer assembly
 * @name PageHeader
 * @use Top of any content zone — title, subtitle, primary action
 * @contains Heading, Text, Button atoms
 * @context First child of a layout header or main zone
 * @never Contains navigation, more than one primary action
 */
```

**Atom:**

```tsx
/**
 * @layer atom
 * @name Button
 * @variants primary | secondary | ghost | destructive
 * @sizes sm | md | lg
 * @never Hardcoded width, internal submit handling
 */
```

The `@use` field is the most important — it must be specific enough that
a future generation can decide whether to use this component or create a new one
without reading the implementation.

### Composition rules

- Screens import from `layouts/` and `assembly/` only — never from `atoms/` directly
- Assembly components import from `atoms/` only — never from other assembly components
- Atoms have no layout assumptions and no hardcoded content
- All token usage must match `tailwind.config.ts` — no arbitrary values

---

## VALIDATE Instructions

Run in order — stop and return violations at any failed layer.

**Layer 1 — Deterministic**

```bash
tsc --noEmit
eslint --max-warnings 0
```

**Layer 2 — Structure (scan the file)**

- [ ] Screen imports only from `layouts/` and `assembly/`
- [ ] Every component has a description block with all required `@layer` tags
- [ ] `@use` and `@never` fields are present and specific — not generic
- [ ] No arbitrary Tailwind values (`w-[372px]`, `text-[#3a3a3a]` etc.)
- [ ] No inline styles

**Layer 3 — LLM critique (cold context)**
Pass: the generated file + the `layouts/` and `assembly/` directories
Ask:

- Does the layout match the screen's purpose given the available options?
- Is visual hierarchy clear — one dominant action, supporting content subordinate?
- Is any assembly component used outside its documented `@context`?
- Are any `@never` rules violated?

Return all violations as:

```ts
{ skill: "ui-ux", location: string, found: string, expected: string, severity: "error" | "warning" }
```

Layer 1 + 2 are always `error`. Layer 3 defaults to `warning` unless
hierarchy is completely absent or a `@never` rule is violated.

---

## First use on a new project

**Greenfield** — directories are empty. The first component you build sets the pattern.
Lock its description carefully — it becomes the reference for everything after it.

**Existing project** — run this scan before generating anything:

```
Read all files in layouts/, assembly/, atoms/
For each file missing a description block, generate and insert one
based on the component's actual implementation.
Report any components that appear to duplicate each other.
```

This backfills the registry without creating any separate files.
The codebase is the registry.
