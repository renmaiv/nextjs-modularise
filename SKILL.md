---
name: nextjs-app-router-modularization
description: >
  Apply this rule when working in a Next.js App Router project and you detect (or are asked to fix)
  a file that mixes Server and Client concerns — for example, `export const metadata` alongside
  `"use client"`, hooks in a Server Component, or a page file that has grown unwieldy (>200–300 LOC)
  with interactive logic. Also triggers when creating new routes that need metadata AND interactivity.
---

## Confirmation Required

Before making any structural changes, explain:

1. What issue was detected
2. Why it is a problem
3. What you plan to change — for example:

   > This page mixes Server and Client concerns (`metadata` + `"use client"`).
   > I propose extracting a Client Component, converting the page to a Server Component,
   > and moving all interactive logic into the new file.
   > Proceed?

**Wait for explicit approval before touching any files.**

---

## Core Rules

- `export const metadata` must never appear in a `"use client"` file
- `page.tsx` defaults to a Server Component unless there is no metadata and no server-only data needs
- Hooks (`useState`, `useEffect`, `useRouter`, etc.) must live in Client Components

---

## When to Refactor

Refactor **only** if one or more of these is true:

- `metadata` exists in a Client Component
- Server and Client concerns are mixed in the same file
- The page exceeds ~200–300 LOC and uses hooks

Do **not** refactor unnecessarily. Prefer `layout.tsx` for metadata when it applies to multiple routes.

---

## Refactor Steps

### 1. Create a Client Component

Path: `app/<route>/<DescriptiveName>Client.tsx`

Use a name derived from the route or feature — not a generic placeholder like `PageClient`.

Move into this file:
- `"use client"` directive
- All hooks
- All JSX and component logic
- All event handlers

### 2. Rewrite `page.tsx` as a Server Component

```tsx
import type { Metadata } from 'next';
import DescriptiveNameClient from './<DescriptiveName>Client';

export const metadata: Metadata = {
  // add relevant metadata here
};

export default function Page() {
  return <DescriptiveNameClient />;
}
```

### 3. Cleanup (mandatory)

- Remove all old code from `page.tsx`
- No hooks in server files
- No duplicate component definitions
- No leftover JSX outside component scope

### 4. Validate

- No `"use client"` in `page.tsx`
- No hooks in Server Components
- No syntax errors
- Project builds successfully

---

## Repeated Structure Detection

While reading a file, scan for JSX blocks that share the same structural shape — same nesting, same className pattern, same prop surface — rendered multiple times inline. These are candidates for extraction into a shared component.

**Signals to look for:**

- Two or more JSX blocks with identical or near-identical structure (same wrapper element, same className, same child layout)
- Repeated patterns distinguished only by data (different `src`, `href`, text, or a mapped array index)
- Inline `.map()` calls where each item renders more than ~3 elements
- Copy-pasted blocks with minor value changes and no abstraction

**Example — what to flag:**

```tsx
// Repeated three times with only `src`, `alt`, `title`, `description` changing
<div className="section-row">
  <div className="col-left">
    <p className="type">...</p>
  </div>
  <div className="col-right">
    <video src="..." />
    <div className="section-copy">
      <p className="section-title">...</p>
      <p className="section-description">...</p>
    </div>
  </div>
</div>
```

This should become a `<ProjectSection />` component accepting typed props.

**Extraction threshold:** Extract when the same structure appears **2 or more times** and shares at least one className or layout pattern. Do not extract one-offs.

**Naming:** Derive the component name from the content domain, not position — `ProjectSection`, `WorkEntry`, `CaseRow` — not `Section1` or `ItemBlock`.

**Confirmation still applies:** Before extracting, state what you found and propose the component signature. Wait for approval.

---

## Anti-Patterns to Fix

- `"use client"` + `metadata` in the same file
- Hooks inside Server Components
- Duplicate component definitions
- JSX outside component scope
- Partial refactors that leave the file in a broken intermediate state
- Inline duplication of the same JSX structure 2+ times instead of a shared component

---

## Output Requirements

- Provide full updated file contents
- No placeholders or `// ... rest of component` shortcuts
- Code must compile

---

## Principle

Prefer clarity over architectural purity. Refactor only when it prevents bugs or enables required functionality. Avoid creating unnecessary files — sometimes a `layout.tsx` handles metadata more cleanly than splitting a page.
