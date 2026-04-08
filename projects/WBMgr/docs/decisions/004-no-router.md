---
id: 0aba29d4-b4ee-4a2d-bdcf-5ad2c292bdb7
title: 004 — No Router, Manual View Switching
author: ai
tags:
- decision
- routing
- architecture
created: 2026-04-07T15:16:41.131393100Z
modified: 2026-04-07T15:16:41.131393100Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# 004 — No Router, Manual View Switching

**Status:** Accepted  
**Inferred from:** `src/App.tsx`, `src/types/index.ts` (`View` union type)

---

## Decision

Navigation between screens is implemented as a `useState<View>` in `App.tsx` with conditional rendering (`{activeView === "x" && <X />}`), rather than using React Router or any routing library.

---

## Rationale

A desktop app with a fixed sidebar navigation and 7 screens does not benefit meaningfully from URL-based routing. There are no deep links, no browser history requirements, and no need to share URLs. A simple state variable is the minimal-complexity solution.

---

## Implementation

```typescript
// types/index.ts
export type View = "dashboard" | "myteam" | "schedule" | "standings" | "statistics" | "leaders" | "settings";

// App.tsx
const [activeView, setActiveView] = useState<View>("dashboard");
```

The `View` union type acts as an exhaustive contract for valid screen names. Adding a new screen requires adding it to the union, the `NAV_ITEMS` array, and the conditional render block in `App.tsx`.

---

## Consequences

- **No browser history**: Back/forward browser buttons do nothing (irrelevant in Tauri).
- **No deep linking**: Cannot launch directly to a specific view.
- **Simple to reason about**: The entire navigation state is a single string.
- **All views mount fresh on first visit**: No route-level code splitting (all JS is loaded upfront, which is fine for an app this size).
