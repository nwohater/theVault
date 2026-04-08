---
id: 347e9e9f-eb24-46e3-aa06-cc3db6863198
title: 005 — React Context for Player Modal Trigger
author: ai
tags:
- decision
- react
- context
- modal
created: 2026-04-07T15:16:51.383929Z
modified: 2026-04-07T15:16:51.383929Z
project: WBMgr
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# 005 — React Context for Player Modal Trigger

**Status:** Accepted  
**Inferred from:** `src/context/PlayerContext.tsx`, `src/App.tsx`, `src/components/PlayerLink.tsx`, `src/views/MyTeam.tsx`, `src/views/Statistics.tsx`

---

## Decision

The `PlayerModal` is rendered at the root (`App.tsx`) and triggered via a React Context (`PlayerCtx`) that carries the setter function `(name: string) => void`, rather than passing `onOpenModal` as a prop through the component tree.

---

## Rationale

Player names appear in multiple views (Statistics, Leaders, My Team, batting order) and inside nested components (LeaderTable, roster rows). Prop-drilling `onOpenModal` through all of these intermediaries would add noise to every component's interface. React Context avoids this without introducing a third-party state library.

---

## Implementation

```typescript
// PlayerContext.tsx
export const PlayerCtx = createContext<(name: string) => void>(() => {});

// App.tsx — provides the setter
<PlayerCtx.Provider value={setModalPlayer}>
  {modalPlayer && <PlayerModal name={modalPlayer} onClose={() => setModalPlayer(null)} />}
  ...
</PlayerCtx.Provider>

// Any component — consumes it
const openModal = useContext(PlayerCtx);
<button onClick={() => openModal(playerName)}>{playerName}</button>
```

`PlayerLink` is a convenience wrapper component that encapsulates this pattern for inline use.

---

## Consequences

- **Single modal instance**: Only one player can be viewed at a time (the last name set).
- **Context is narrow and focused**: The context carries only a single function, not a global store. This keeps it lightweight.
- **Easy to extend**: Any new component that needs to open the modal just calls `useContext(PlayerCtx)` — no prop changes required elsewhere.
