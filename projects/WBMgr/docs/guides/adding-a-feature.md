---
id: d99d78b0-a7a5-4f0f-84eb-c543a7401986
title: How to Add a New Feature (View)
author: ai
tags:
- guide
- development
- feature
created: 2026-04-07T15:17:32.851558700Z
modified: 2026-04-07T15:17:32.851558700Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# How to Add a New Feature (View)

This guide walks through adding a new top-level screen to WBMgr. The example adds a "Playoffs" view.

---

## Step 1 — Add the View to the Type

In `src/types/index.ts`, add the new view name to the `View` union:

```typescript
export type View = "dashboard" | "myteam" | "schedule" | "standings" | "statistics" | "leaders" | "settings" | "playoffs";
```

---

## Step 2 — Create the View Component

Create `src/views/Playoffs.tsx`:

```tsx
export default function Playoffs() {
  return (
    <div className="playoffs">
      <h2>Playoffs</h2>
      {/* your content here */}
    </div>
  );
}
```

---

## Step 3 — Register in App.tsx

Open `src/App.tsx` and make three changes:

**Import the component:**
```tsx
import Playoffs from "./views/Playoffs";
```

**Add to NAV_ITEMS** (if it should appear in the main nav):
```tsx
const NAV_ITEMS: { id: View; label: string }[] = [
  // ...existing items...
  { id: "playoffs", label: "Playoffs" },
];
```

**Add the conditional render:**
```tsx
{activeView === "playoffs" && <Playoffs />}
```

---

## Step 4 — Add Data (if needed)

If the feature needs new static data, create a file in `src/data/`, e.g. `src/data/playoffs.ts`. Add any new types to `src/types/index.ts`.

---

## Step 5 — Add Styles

Add CSS to `src/styles.css`. Follow the existing convention of using a root class matching the view name (`.playoffs { ... }`).

---

## Adding a Reusable Component

If you're building a component that will be used across multiple views (e.g. a generic bracket component), create it in `src/components/`. If it's only used by one view, co-locate it in the view file or keep it in the view directory.

---

## Making a Component Modal-Aware

If your component renders player names that should open the PlayerModal, consume `PlayerCtx`:

```tsx
import { useContext } from "react";
import { PlayerCtx } from "../context/PlayerContext";

export default function MyComponent() {
  const openModal = useContext(PlayerCtx);
  return <button onClick={() => openModal("Marcus Webb")}>Marcus Webb</button>;
}
```

Or use the pre-built `<PlayerLink name="Marcus Webb" />` component which handles this automatically.

---

## Persisting New User State

If your feature has user-editable state that should survive page reloads, use `localStorage`. Follow the pattern in `src/utils/stats.ts`:

```typescript
// Loading with fallback
const [myState, setMyState] = useState(() =>
  JSON.parse(localStorage.getItem("wbmgr_mykey") ?? "null") ?? defaultValue
);

// Saving
localStorage.setItem("wbmgr_mykey", JSON.stringify(myState));
```

Use the `wbmgr_` prefix for new keys to avoid collisions.
