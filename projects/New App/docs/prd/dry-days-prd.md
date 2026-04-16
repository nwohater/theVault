---
id: 053d5147-33f7-4747-b019-ca80ed4056c8
title: Dry Days — Product Requirements Document
author: ai
tags:
- prd
- swiftui
- cloudkit
created: 2026-04-15T22:17:50.620628900Z
modified: 2026-04-15T22:17:50.620628900Z
project: New App
status: draft
ai_tool: claude-code
canonical: false
protected: false
---

# Dry Days — Product Requirements Document

## Overview

**App name:** Dry Days (working title; codename: drinky poo)  
**Platform:** iOS (SwiftUI)  
**Data layer:** CloudKit (private database)  
**Status:** Pre-development  

A personal sobriety and mindful drinking tracker that lets users log alcohol-free days, build streaks, and reflect on their drinking patterns over time — with no social component, no judgment, and no account signup friction.

---

## Problem Statement

People who want to drink less (Dry January, Sober October, general mindfulness) have no lightweight, private tool that focuses purely on tracking dry days vs. drinking days. Existing apps are either AA-adjacent (stigmatizing), social (public accountability), or overly complex (journaling, mood tracking, CBT frameworks). This app does one thing: helps you see how many days you didn't drink, celebrate streaks, and notice trends.

---

## Target User

- Adults experimenting with reducing alcohol consumption
- Participants in "dry month" challenges (Dry January, Sober October, etc.)
- Anyone who wants a private, low-friction habit log with no sign-up wall

---

## Core Features (v1.0)

### 1. Day Log
- Calendar view showing the current month
- Tap a day to toggle: **Dry** | **Drink** | **Unmarked**
- Allow backdating up to 365 days; no future dates
- Days default to Unmarked until the user logs them

### 2. Streak Tracking
- Current dry streak (consecutive dry days ending today or yesterday)
- Longest dry streak (all-time)
- Displayed prominently on the home screen

### 3. Stats Summary
- This month: dry day count, drink day count, % dry
- This year: same breakdown
- All-time totals

### 4. Home Screen Widget
- Small widget: current streak count
- Medium widget: current streak + this-month dry/drink ratio

### 5. CloudKit Sync
- All data stored in the user's iCloud private database
- Automatic, silent sync across the user's devices (iPhone + iPad)
- No backend server; no user account; no email required
- Graceful offline support — local writes queue and sync when connectivity returns

---

## Data Model

### `DayRecord`
| Field | Type | Notes |
|---|---|---|
| `date` | `Date` (day precision) | Unique per user |
| `status` | `enum` (dry / drink / unmarked) | |
| `note` | `String?` | Optional freeform note, max 280 chars |
| `createdAt` | `Date` | |
| `modifiedAt` | `Date` | Used for conflict resolution (last-write-wins) |

CloudKit record type: `DayRecord`  
Zone: default private zone  

---

## UI / UX Requirements

- **Minimal, calm aesthetic** — muted palette, no red/green pass/fail framing
- **One-handed operation** — primary actions reachable with thumb
- Dark mode support (follows system)
- Haptic feedback on day toggle
- No onboarding beyond a single permissions prompt (iCloud access)
- No push notifications in v1 (optional streak reminders in v2)

---

## Technical Stack

| Layer | Choice | Rationale |
|---|---|---|
| UI | SwiftUI | Native, modern, widget support |
| Local persistence | SwiftData (or Core Data) | Offline buffer before CloudKit sync |
| Sync | CloudKit (CKSyncEngine or NSPersistentCloudKitContainer) | Zero-cost, private, no server |
| Widgets | WidgetKit | Home screen + lock screen widgets |
| Min deployment | iOS 17 | SwiftData + CKSyncEngine GA |

---

## Out of Scope (v1)

- Social features / sharing streaks
- Drink type or quantity logging
- Push notifications / reminders
- Apple Watch app
- Android / web
- Drink-related health data integration (HealthKit) — revisit in v2

---

## Success Metrics

- User can log a day in < 2 taps from app open
- Sync latency < 5 seconds on a good connection
- Zero data loss on reinstall (full restore from CloudKit)
- App size < 15 MB

---

## Open Questions

1. Should "unmarked" days count against streaks or be treated as neutral?
2. Lock screen widget — yes in v1 or defer?
3. App name: "Dry Days" is descriptive but possibly crowded on the App Store — need title check
4. Should the note field be surfaced in the calendar view or only on day detail?
