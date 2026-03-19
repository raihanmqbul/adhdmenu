# Design Document: ADHD Dopamine Menu Tracker

## Overview

A single `index.html` Progressive Web App that provides ADHD users with a personalized "dopamine menu" — a curated list of activities to help them get started, feel rewarded, and build momentum. The app is fully offline-capable, stores all state in localStorage, and prioritizes calm, warm UX over productivity pressure.

The entire application lives in one HTML file. No build step. No server. Just open and use.

---

## Architecture

```
index.html
├── <head>
│   ├── Inline SVG favicon (data URI)
│   ├── Inline PWA manifest (blob URL)
│   ├── CDN: Tailwind CSS
│   ├── CDN: Lucide Icons
│   ├── CDN: html2pdf.js
│   ├── CDN: canvas-confetti
│   └── <style> — CSS custom properties (design tokens)
│
├── <body>
│   ├── Welcome Modal (shown on first launch only)
│   ├── App Shell
│   │   ├── Sidebar (desktop ≥768px)
│   │   ├── Content Area (5 screen panels)
│   │   └── Bottom Tab Bar (mobile <768px)
│   └── 3× hidden <audio> elements
│
└── <script>
    ├── Service Worker registration (inline blob)
    ├── State management (localStorage read/write)
    ├── Screen router (show/hide panels)
    ├── Welcome modal logic
    ├── Home screen logic
    ├── Build screen logic
    ├── Wins screen logic
    ├── History screen logic
    ├── Settings screen logic
    ├── Music controller
    ├── Confetti helper
    └── PDF export
```

### Key Architectural Decisions

- **Single file**: All JS, CSS, and HTML in one `index.html`. Music files are the only external assets.
- **No framework**: Vanilla JS with direct DOM manipulation. Keeps the file self-contained and fast.
- **Screen routing**: Each screen is a `<div>` panel. The router shows one and hides the rest. No URL changes.
- **State**: All state lives in localStorage. The in-memory JS object is always a mirror of localStorage.
- **Music**: Three `<audio>` elements, always present in DOM. The controller fades between them.

---

## Components and Interfaces

### Navigation Component
- Mobile: fixed bottom bar, 5 icon buttons, 48px height minimum
- Desktop: fixed left sidebar, 240px wide, 5 nav items with icon + label
- Active state: teal accent color + subtle background highlight
- Shared nav items: Home, Build, Wins, History, Settings

### Welcome Modal
- Full-screen overlay, z-index above everything
- Not dismissable by outside click (pointer-events on backdrop disabled)
- Three age group cards: Child (🧒 8–12), Teen (🧑 13–17), Adult (🧑‍💼 18+)
- Each card: large emoji, label, one-line description
- On selection: fade out modal (300ms), start music, navigate to Home

### Menu Item Card
- Emoji (large, 2rem)
- Name (bold, text-primary)
- Time estimate badge (text-muted, small)
- Age badge (colored pill)
- "I Did This ✓" button — full width, teal, min 48px height
- On win: scale animation (transform: scale(1.05) → scale(1), 200ms ease-out)

### Category Tab Bar
- Four tabs: Appetizers | Mains | Desserts | Sensory
- Active tab: teal underline + teal text
- Inactive: muted text
- Scrollable horizontally on small screens

### Search Bar
- Full width, surface background, rounded
- Placeholder: "Search your menu..."
- Filters items in real time (case-insensitive name match)
- Clear button (×) appears when text is present

### Heat Map Calendar
- 30 cells in a 7-column grid
- Each cell: date number + teal background with opacity proportional to win count
- 0 wins: surface color; 1 win: teal at 20%; 2: 40%; 3: 60%; 4+: 80%
- Tap → slide-up panel with that day's items

### Slide-Up Panel
- Fixed bottom sheet, slides up from bottom (translateY animation, 300ms ease-out)
- Backdrop overlay (semi-transparent)
- Close button (×) at top right
- Lists items with emoji + name + timestamp

### PDF Export Layout (in-memory div)
- Teal header bar: app name + "Generated on [date]" + streak count
- Two-column layout:
  - Left: "Your Current Menu" — all enabled items grouped by category
  - Right: "Last 30 Days" — date + win count per day
- Teal accent borders on section headers
- White background, dark text (for print readability)

---

## Data Models

### localStorage Keys

```
adhd_age_group        → "child" | "teen" | "adult"
adhd_music_enabled    → true | false
adhd_music_playing    → true | false
adhd_menu_items       → MenuItem[]
adhd_custom_items     → MenuItem[]
adhd_wins             → { [dateKey: string]: WinEntry[] }
adhd_license_key      → string
adhd_first_launch     → false  (only written, never "true")
```

### MenuItem

```js
{
  id: string,           // unique, e.g. "appetizer-water"
  name: string,         // display name
  emoji: string,        // single emoji character
  category: "appetizer" | "main" | "dessert" | "sensory",
  timeEstimate: string, // e.g. "2 min", "10 min"
  ageTags: string[],    // ["all"] | ["child","teen"] | ["teen","adult"] | ["adult"] | ["child"]
  enabled: boolean,     // toggled by user in Build screen
  isCustom: boolean     // false for pre-loaded, true for user-added
}
```

### WinEntry

```js
{
  id: string,           // menu item id
  name: string,
  emoji: string,
  category: string,
  timestamp: string     // ISO 8601, e.g. "2025-03-18T14:32:00.000Z"
}
```

### AppState (in-memory mirror)

```js
{
  ageGroup: "child" | "teen" | "adult" | null,
  musicEnabled: boolean,
  menuItems: MenuItem[],
  customItems: MenuItem[],
  wins: { [dateKey]: WinEntry[] },
  licenseKey: string,
  currentScreen: "home" | "build" | "wins" | "history" | "settings",
  activeCategory: "appetizer" | "main" | "dessert" | "sensory",
  searchQuery: string
}
```

### Date Key Format

All win entries are keyed by `YYYY-MM-DD` in local time:
```js
const dateKey = new Date().toLocaleDateString('en-CA'); // "2025-03-18"
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Age filter correctness

*For any* menu item and any age group selection, if the item's ageTags do not include the selected age group (or "all"), then the item SHALL NOT appear in the filtered menu.

**Validates: Requirements 8.5, 8.6, 8.7**

### Property 2: Win logging round trip

*For any* menu item that is logged as a win, querying today's wins from localStorage SHALL return an entry containing that item's id, name, emoji, and category.

**Validates: Requirements 2.6, 4.1**

### Property 3: Streak monotonicity

*For any* sequence of days where each day has at least one win, the streak counter SHALL equal the length of the longest consecutive suffix of that sequence ending on today.

**Validates: Requirements 4.3**

### Property 4: Search filter correctness

*For any* search query string and menu item list, the filtered result SHALL contain only items whose name includes the query string (case-insensitive), and SHALL contain ALL such items.

**Validates: Requirements 2.1**

### Property 5: Music volume invariant

*For any* age group, the active audio track's volume SHALL equal 0.4 for child, 0.45 for teen, and 0.5 for adult, regardless of how many times the age group is changed.

**Validates: Requirements 7.2, 7.3, 7.4**

### Property 6: Toggle persistence round trip

*For any* menu item whose enabled state is toggled, reloading the app (re-reading from localStorage) SHALL reflect the same enabled state that was set.

**Validates: Requirements 3.2**

### Property 7: Win count message correctness

*For any* win count N, the encouraging message displayed SHALL be the "gentle nudge" variant when N=0, the "warm praise" variant when 1≤N≤3, and the "celebration" variant when N≥4.

**Validates: Requirements 4.2**

### Property 8: Heat map intensity correctness

*For any* day with W wins, the heat map cell for that day SHALL use a teal opacity of: 0% when W=0, 20% when W=1, 40% when W=2, 60% when W=3, and 80% when W≥4.

**Validates: Requirements 5.1**

### Property 9: License key validation

*For any* non-empty string entered as a license key, THE App SHALL display a green checkmark and persist the value. *For any* empty string, THE App SHALL not display the checkmark.

**Validates: Requirements 6.6**

### Property 10: Custom item CRUD round trip

*For any* custom item added via the Build screen, the item SHALL appear in the menu and in localStorage. After deletion, the item SHALL NOT appear in the menu or in localStorage.

**Validates: Requirements 3.4, 3.6**

---

## Error Handling

| Scenario | Behavior |
|---|---|
| Audio file missing (404) | `onerror` handler on `<audio>` — silently swallows error, no UI change |
| localStorage full | Wrap all writes in try/catch — fail silently, log to console only |
| html2pdf.js CDN unavailable | PDF button shows a toast: "PDF export unavailable offline" |
| canvas-confetti CDN unavailable | Win logging still works; confetti simply doesn't fire |
| Invalid JSON in localStorage | Reset that key to its default value on parse error |
| No wins today | Wins screen shows empty state with gentle nudge message |
| Age group not set | App shows welcome modal (should not happen after first launch) |

---

## Testing Strategy

### Unit Tests (example-based)
- Age filter: verify child gets only [ALL]/[C]/[C][T] items
- Streak calculation: verify streak=0 when no wins, streak=N for N consecutive days
- Date key generation: verify format is YYYY-MM-DD in local time
- Win message: verify correct message for counts 0, 1, 3, 4, 10
- Heat map opacity: verify correct opacity for win counts 0–5

### Property-Based Tests
Each property above maps to a property-based test using fast-check (TypeScript) or hypothesis (Python). Since this is a single HTML file, property tests would be run in a separate test harness against the extracted pure functions.

**Property Test Configuration**:
- Minimum 100 iterations per property
- Tag format: `Feature: adhd-dopamine-menu-tracker, Property N: <title>`

### Manual Smoke Tests
- First launch flow: welcome modal → age selection → music starts → home screen
- Win logging: tap "I Did This" → confetti → item appears in Wins screen
- PDF export: tap export → PDF downloads with correct content
- Offline: disable network → reload → app works fully
- Music toggle: toggle off → music pauses; toggle on → music resumes
- Age group change in Settings → music fades and swaps
