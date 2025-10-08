# Edit & Delete Feature - Visual Flow Diagram

## State Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      HOME SCREEN                            │
│                   (Normal Mode)                             │
│  ┌────────────────────────────────────────────────┐        │
│  │  Top Bar: "My Habits" | Menu | Profile        │        │
│  │  ┌─────────────────────────────────────────┐  │        │
│  │  │       Habit Card 1                      │  │        │
│  │  │  [Avatar] Title                         │  │        │
│  │  │           Description                   │  │        │
│  │  │  [Alarm] Reminder    [Toggle]          │  │        │
│  │  │  [Complete Button]  [Details Button]   │  │        │
│  │  └─────────────────────────────────────────┘  │        │
│  │  ┌─────────────────────────────────────────┐  │        │
│  │  │       Habit Card 2                      │  │        │
│  │  └─────────────────────────────────────────┘  │        │
│  └────────────────────────────────────────────────┘        │
│                                                             │
│  [+] Add Habit Button                                      │
└─────────────────────────────────────────────────────────────┘
                           │
                  LONG PRESS on Habit Card 1
                           │
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                      HOME SCREEN                            │
│                  (Selection Mode)                           │
│  ┌────────────────────────────────────────────────┐        │
│  │  Top Bar: "1 selected" | [X] | [Edit] | [Del] │        │
│  │  ┌─────────────────────────────────────────┐  │        │
│  │  │ ✅ Habit Card 1  (Selected)            │  │ ← Blue Border
│  │  │  [Avatar] Title              [✓]       │  │ ← Checkmark
│  │  │           Description                   │  │
│  │  │  [Alarm] Reminder    [Toggle]          │  │
│  │  │  [Complete Button]  [Details Button]   │  │
│  │  └─────────────────────────────────────────┘  │
│  │  ┌─────────────────────────────────────────┐  │
│  │  │ ⭕ Habit Card 2  (Not Selected)         │  │ ← Empty Circle
│  │  └─────────────────────────────────────────┘  │
│  └────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
        │                    │                    │
        │                    │                    │
    Tap [X]            Tap [Edit]            Tap [Del]
        │              (1 selected)               │
        ↓                    │                    ↓
   Exit Mode                 ↓          ┌──────────────────┐
                   ┌──────────────────┐ │ Confirmation     │
                   │  EDIT SCREEN     │ │ Dialog           │
                   │  (Pre-filled)    │ │ "Delete habit?"  │
                   │                  │ │ [Cancel][Delete] │
                   │ Title: [____]    │ └──────────────────┘
                   │ Desc:  [____]    │          │
                   │ Time:  [____]    │      Confirm
                   │ Freq:  [____]    │          │
                   │ Avatar:[____]    │          ↓
                   │                  │   Delete & Sync
                   │ [Update Habit]   │   to Firestore
                   └──────────────────┘          │
                            │                    │
                        Save & Update            │
                            │                    │
                            ↓                    ↓
                     Back to Home        Back to Home
                   (Selection Exited)  (Selection Exited)
```

## Multi-Selection Flow

```
┌─────────────────────────────────────────────────────────────┐
│           SELECTION MODE - Multiple Habits                   │
│  ┌────────────────────────────────────────────────┐        │
│  │  Top Bar: "3 selected" | [X] | [Edit✗] | [Del]│        │
│  │                              ↑ Disabled         │        │
│  │  ┌─────────────────────────────────────────┐  │        │
│  │  │ ✅ Habit 1  (Selected)     [✓]         │  │
│  │  └─────────────────────────────────────────┘  │
│  │  ┌─────────────────────────────────────────┐  │
│  │  │ ✅ Habit 2  (Selected)     [✓]         │  │
│  │  └─────────────────────────────────────────┘  │
│  │  ┌─────────────────────────────────────────┐  │
│  │  │ ✅ Habit 3  (Selected)     [✓]         │  │
│  │  └─────────────────────────────────────────┘  │
│  │  ┌─────────────────────────────────────────┐  │
│  │  │ ⭕ Habit 4  (Not Selected)  [○]        │  │
│  │  └─────────────────────────────────────────┘  │
│  └────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
                           │
                      Tap [Del]
                           │
                           ↓
              ┌────────────────────────┐
              │  Confirmation Dialog   │
              │                        │
              │  "Delete 3 habits?"    │
              │                        │
              │  "Are you sure you     │
              │  want to delete 3      │
              │  habits? They will be  │
              │  moved to trash..."    │
              │                        │
              │  [Cancel]  [Delete]    │
              └────────────────────────┘
                           │
                      Confirm
                           │
                           ↓
              Move 3 habits to trash
              Sync to Firestore
              Exit selection mode
                           │
                           ↓
                    Back to Normal Mode
```

## Selection Toggle Flow

```
Selection Mode Active:

Tap Habit → Toggle Selection
    │
    ├─── If not selected → Add to selection
    │                      Show checkmark
    │                      Add blue border
    │
    └─── If selected → Remove from selection
                      Hide checkmark
                      Remove blue border

If all deselected → Auto-exit selection mode
```

## Edit Button State Logic

```
Selection Mode Active:

Count Selected = 0  → Edit Button: Disabled (gray)
Count Selected = 1  → Edit Button: Enabled (blue)
Count Selected > 1  → Edit Button: Disabled (gray)
```

## User Interaction Map

```
Normal Mode:
├── Tap Card → No action (maybe add details view?)
├── Long Press → Enter Selection Mode
├── Tap Complete → Mark habit complete
├── Tap Details → Go to details screen
└── Toggle Switch → Enable/disable reminder

Selection Mode:
├── Tap Card → Toggle selection
├── Tap [X] → Exit selection mode
├── Tap [Edit] → Navigate to edit (if 1 selected)
├── Tap [Delete] → Show confirmation dialog
└── Press Back → Exit selection mode
```

## Component Hierarchy

```
HabitHomeScreen
│
├── TopBar (Dynamic)
│   ├── Normal Mode: Menu, Title, Profile
│   └── Selection Mode: Close, Count, Edit, Delete
│
├── LazyColumn
│   ├── NotificationPermissionCard (conditional)
│   ├── EmptyState (conditional)
│   └── HabitCards (List)
│       ├── Normal: Click disabled
│       └── Selection: Click toggles selection
│
├── FAB (Add Habit)
│
└── Dialogs
    ├── DeleteConfirmationDialog
    └── LoadingOverlay
```

## Data Flow Architecture

```
┌──────────┐         ┌──────────────┐         ┌──────────┐
│   UI     │ Events  │  ViewModel   │  Data   │Repository│
│ (Home    │────────→│  (Selection  │←───────→│  (Room + │
│  Screen) │         │   & Edit)    │         │ Firestore)│
└──────────┘         └──────────────┘         └──────────┘
     ↑                       │                      │
     │                       │                      │
     │    State Flow         │                      │
     └───────────────────────┘                      │
                                                    │
                                           Automatic Sync
                                                    │
                                                    ↓
                                            ┌──────────────┐
                                            │  Firestore   │
                                            │  (Cloud DB)  │
                                            └──────────────┘
```

## State Management

```
HabitScreenState {
    habits: List<HabitCardUi>
    isSelectionMode: Boolean ────┐
    selectedHabitIds: Set<Long> ─┤─→ Selection State
    addHabitState {              │
        isEditMode: Boolean ─────┤─→ Edit State
        editingHabitId: Long? ───┘
    }
}
```

## Color Coding

- 🟦 **Blue Border** = Selected habit card
- ✅ **Checkmark** = Selected in multi-select
- ⭕ **Empty Circle** = Not selected (in selection mode)
- 🔴 **Delete Icon** = Red color for destructive action
- 🔵 **Edit Icon** = Primary color when enabled
- ⚪ **Edit Icon** = Gray when disabled

---

This visual flow should help understand the complete user journey and implementation architecture!
