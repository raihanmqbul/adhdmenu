# Implementation Plan: ADHD Dopamine Menu Tracker

## Overview

Implement a complete single-file PWA in `index.html`. All logic, styles, and markup live in one file. Music files are the only external assets. Implementation proceeds screen by screen, with the data layer and core infrastructure first.

## Tasks

- [ ] 1. Set up HTML shell, design tokens, CDN imports, and PWA infrastructure
  - Add the required comment at line 1: `<!-- ADHD Dopamine Menu Tracker – Fully offline PWA – Age-adaptive music – Built for ADHD brains -->`
  - Add `<meta>` tags, viewport, theme-color (#00b4a6), inline SVG favicon
  - Add CDN script tags: Tailwind CSS, Lucide, html2pdf.js, canvas-confetti
  - Define CSS custom properties for all design tokens (background, accent, surface, text, etc.)
  - Add global base styles: font stack, line-height, tap target minimums
  - Register Service Worker via inline blob URL (cache-first strategy)
  - Embed PWA manifest via blob URL `<link rel="manifest">`
  - Add 3 hidden `<audio>` elements for child/teen/adult tracks with correct src, volume, loop
  - _Requirements: 9.1, 9.2, 9.3, 10.1, 10.2, 10.7, 10.8, 7.1_

- [ ] 2. Implement data layer and state management
  - [ ] 2.1 Define all localStorage keys as constants
    - Define STORAGE_KEYS object with all adhd_* key names
    - _Requirements: all storage requirements_

  - [ ] 2.2 Implement loadState() and saveState() functions
    - loadState(): reads all localStorage keys, parses JSON, returns AppState object with defaults
    - saveState(key, value): JSON.stringify + localStorage.setItem wrapped in try/catch
    - Handle JSON parse errors by resetting to defaults
    - _Requirements: 9.1, 3.2, 6.2_

  - [ ] 2.3 Define complete pre-loaded menu items array
    - All 15 Appetizers, 20 Mains, 13 Desserts, 13 Sensory items
    - Each with id, name, emoji, category, timeEstimate, ageTags, enabled:true, isCustom:false
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [ ]* 2.4 Write property test for age filter function
    - **Property 1: Age filter correctness**
    - **Validates: Requirements 8.5, 8.6, 8.7**
    - For any age group, filterByAge(items, ageGroup) should return only items whose ageTags include the age group or "all"

  - [ ] 2.5 Implement filterByAge(items, ageGroup) pure function
    - child: include items with ageTags containing "all", "child"
    - teen: include items with ageTags containing "all", "teen"
    - adult: include items with ageTags containing "all", "adult"
    - _Requirements: 8.5, 8.6, 8.7_

  - [ ]* 2.6 Write property test for search filter function
    - **Property 4: Search filter correctness**
    - **Validates: Requirements 2.1**
    - For any query and item list, filterBySearch(items, query) returns exactly items whose names include query (case-insensitive)

  - [ ] 2.7 Implement filterBySearch(items, query) pure function
    - Case-insensitive substring match on item name
    - Returns all items if query is empty
    - _Requirements: 2.1_

  - [ ]* 2.8 Write property test for win message function
    - **Property 7: Win count message correctness**
    - **Validates: Requirements 4.2**
    - For N=0: gentle nudge; 1≤N≤3: warm praise; N≥4: celebration

  - [ ] 2.9 Implement getWinMessage(count) pure function
    - Returns appropriate encouraging message string based on count
    - _Requirements: 4.2_

  - [ ]* 2.10 Write property test for streak calculation
    - **Property 3: Streak monotonicity**
    - **Validates: Requirements 4.3**
    - For any wins object, calculateStreak(wins) returns the length of the longest consecutive suffix of days with wins ending on today

  - [ ] 2.11 Implement calculateStreak(wins) pure function
    - Iterates backwards from today, counts consecutive days with ≥1 win
    - Returns 0 if today has no wins
    - _Requirements: 4.3_

  - [ ]* 2.12 Write property test for heat map opacity
    - **Property 8: Heat map intensity correctness**
    - **Validates: Requirements 5.1**
    - For W=0: 0%; W=1: 20%; W=2: 40%; W=3: 60%; W≥4: 80%

  - [ ] 2.13 Implement getHeatOpacity(winCount) pure function
    - Returns opacity value (0–0.8) based on win count
    - _Requirements: 5.1_

- [ ] 3. Checkpoint — Ensure pure functions are correct
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Implement app shell layout (navigation + screen routing)
  - [ ] 4.1 Build app shell HTML structure
    - Outer flex container: sidebar (hidden on mobile) + main content area
    - Bottom tab bar (visible on mobile only, hidden on desktop)
    - 5 screen panel divs: home, build, wins, history, settings (all hidden by default)
    - Use Tailwind classes for responsive layout (md:flex, md:hidden, etc.)
    - _Requirements: 10.5, 10.6_

  - [ ] 4.2 Implement screen router
    - showScreen(screenName): hides all panels, shows target panel, updates nav active state
    - Wire all nav buttons (bottom bar + sidebar) to showScreen()
    - _Requirements: 10.5, 10.6_

  - [ ] 4.3 Build welcome modal HTML
    - Full-screen overlay with z-50
    - App name, tagline, three age group cards
    - No outside-click dismiss (pointer-events: none on backdrop)
    - _Requirements: 1.1, 1.2_

  - [ ] 4.4 Implement welcome modal logic
    - On app load: check adhd_first_launch; if not false, show modal
    - On age card click: save age group, filter and save menu items, fade modal, start music, navigate to home
    - _Requirements: 1.3, 1.4, 1.5, 1.6, 1.7, 1.8_

- [ ] 5. Implement Home screen
  - [ ] 5.1 Build Home screen HTML
    - Search bar with clear button
    - Category tab bar (Appetizers, Mains, Desserts, Sensory)
    - Item cards grid/list
    - "Feeling stuck?" button
    - _Requirements: 2.1, 2.2, 2.4, 2.5, 2.9_

  - [ ] 5.2 Implement Home screen rendering
    - renderHome(): reads state, applies search + category filters, renders item cards
    - Each card: emoji, name, time estimate, age badge, "I Did This ✓" button
    - _Requirements: 2.3, 2.4_

  - [ ] 5.3 Implement win logging
    - logWin(item): adds WinEntry to today's wins in localStorage
    - Triggers confetti with exact specified config
    - Applies scale animation to card (200ms ease-out)
    - Re-renders wins count if wins screen is visible
    - _Requirements: 2.6, 2.7, 2.8_

  - [ ]* 5.4 Write property test for win logging round trip
    - **Property 2: Win logging round trip**
    - **Validates: Requirements 2.6, 4.1**
    - For any menu item logged as a win, today's wins in localStorage contains that item's id, name, emoji, category

  - [ ] 5.5 Implement "Feeling stuck?" logic
    - Randomly selects one enabled Appetizer item
    - Scrolls to it and applies a highlight animation (teal glow, 2s)
    - _Requirements: 2.9, 2.10_

  - [ ] 5.6 Wire search bar and category tabs
    - Search input: oninput → update searchQuery state → re-render
    - Category tabs: onclick → update activeCategory state → re-render
    - _Requirements: 2.1, 2.2, 2.3_

- [ ] 6. Implement Build / Edit Menu screen
  - [ ] 6.1 Build Build screen HTML
    - List of all menu items with toggle switches
    - "Add Custom Item" form (name, category, emoji, time estimate)
    - Delete buttons on custom items only
    - Drag handles on items
    - _Requirements: 3.1, 3.3, 3.5_

  - [ ] 6.2 Implement item toggle logic
    - Toggle switch: updates item.enabled in state + localStorage immediately
    - _Requirements: 3.2_

  - [ ]* 6.3 Write property test for toggle persistence
    - **Property 6: Toggle persistence round trip**
    - **Validates: Requirements 3.2**
    - For any item and toggle state, re-reading from localStorage returns the same enabled state

  - [ ] 6.4 Implement custom item add logic
    - Validate form (name required, emoji required)
    - Generate unique id, set isCustom:true
    - Save to adhd_custom_items in localStorage
    - Re-render Build screen and Home screen
    - _Requirements: 3.3, 3.4_

  - [ ]* 6.5 Write property test for custom item CRUD round trip
    - **Property 10: Custom item CRUD round trip**
    - **Validates: Requirements 3.4, 3.6**
    - For any custom item added, it appears in localStorage; after deletion, it does not

  - [ ] 6.6 Implement custom item delete logic
    - Remove from adhd_custom_items in localStorage
    - Re-render Build screen
    - _Requirements: 3.5, 3.6_

  - [ ] 6.7 Implement drag-to-reorder (if supported)
    - Use HTML5 drag-and-drop API on item rows
    - On drop: reorder array in state + persist to localStorage
    - _Requirements: 3.7_

- [ ] 7. Checkpoint — Ensure Home and Build screens work end-to-end
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 8. Implement Today's Wins screen
  - [ ] 8.1 Build Wins screen HTML
    - Win count + encouraging message
    - Streak counter
    - List of today's wins with timestamps
    - "Export to Beautiful PDF" button
    - _Requirements: 4.1, 4.2, 4.3, 4.4_

  - [ ] 8.2 Implement Wins screen rendering
    - renderWins(): reads today's wins from state, renders list with timestamps
    - Calls getWinMessage(count) for the message
    - Calls calculateStreak(wins) for the streak
    - _Requirements: 4.1, 4.2, 4.3_

  - [ ] 8.3 Implement PDF export
    - Build in-memory styled div: teal header, two-column layout
    - Left column: enabled menu items grouped by category
    - Right column: last 30 days wins summary (date + count)
    - Include streak count and generation date
    - Call html2pdf().set({margin:10, filename:'dopamine-menu.pdf', html2canvas:{scale:2}, jsPDF:{unit:'mm',format:'a4'}}).from(element).save()
    - If html2pdf unavailable: show toast "PDF export unavailable offline"
    - _Requirements: 4.4, 4.5, 4.6_

- [ ] 9. Implement History screen
  - [ ] 9.1 Build History screen HTML
    - 30-day heat map calendar grid
    - Monthly summary section
    - Slide-up day detail panel (hidden by default)
    - _Requirements: 5.1, 5.2, 5.3_

  - [ ] 9.2 Implement heat map rendering
    - renderHistory(): generates 30 cells for last 30 days
    - Each cell: date number + teal background with opacity from getHeatOpacity()
    - _Requirements: 5.1_

  - [ ] 9.3 Implement day detail slide-up panel
    - On cell tap: populate panel with that day's wins, slide up (translateY 0, 300ms ease-out)
    - Backdrop click or close button: slide down and hide
    - _Requirements: 5.2_

  - [ ] 9.4 Implement monthly summary
    - Total wins for last 30 days
    - Most-used category (count wins per category, return max)
    - Longest streak in the period
    - _Requirements: 5.3_

- [ ] 10. Implement Settings screen
  - [ ] 10.1 Build Settings screen HTML
    - Music toggle (default ON)
    - Age group selector (3 radio/button options)
    - License key input with checkmark indicator
    - Reset all data button
    - App version display
    - _Requirements: 6.1, 6.3, 6.5, 6.7, 6.10_

  - [ ] 10.2 Implement music toggle
    - Toggle: pause/resume active track, save adhd_music_enabled to localStorage
    - _Requirements: 6.1, 6.2_

  - [ ] 10.3 Implement age group change in Settings
    - On selection: fade out current track (300ms), swap to new track, fade in
    - Update adhd_age_group in localStorage
    - Re-filter menu items for new age group
    - _Requirements: 6.3, 6.4, 7.7_

  - [ ]* 10.4 Write property test for license key validation
    - **Property 9: License key validation**
    - **Validates: Requirements 6.6**
    - For any non-empty string: checkmark shown, value persisted. For empty string: no checkmark.

  - [ ] 10.5 Implement license key input logic
    - oninput: if value.trim() !== '': show green checkmark, save to localStorage
    - else: hide checkmark
    - _Requirements: 6.5, 6.6_

  - [ ] 10.6 Implement reset all data
    - Show confirm dialog: "This will delete all your wins, settings, and custom items. Are you sure?"
    - On confirm: remove all adhd_* keys from localStorage, reload page
    - _Requirements: 6.7, 6.8, 6.9_

- [ ] 11. Implement music controller
  - [ ] 11.1 Build music controller module
    - References to 3 audio elements
    - currentTrack variable tracking active age group
    - playTrack(ageGroup): sets volume, plays track, updates currentTrack
    - stopTrack(): pauses current track
    - fadeOut(track, duration): gradually reduces volume to 0 over duration ms
    - fadeIn(track, targetVolume, duration): gradually increases volume from 0 to target
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

  - [ ]* 11.2 Write property test for music volume invariant
    - **Property 5: Music volume invariant**
    - **Validates: Requirements 7.2, 7.3, 7.4**
    - For any age group, after playTrack(ageGroup), the track's volume equals the specified constant

  - [ ] 11.3 Implement silent error handling for missing audio files
    - Add onerror handler to each audio element: e.preventDefault(), return false
    - _Requirements: 7.6_

- [ ] 12. Final wiring and polish
  - [ ] 12.1 Wire all screens to shared state
    - Ensure renderHome(), renderBuild(), renderWins(), renderHistory() all read from shared state
    - Ensure all mutations go through saveState() before re-rendering
    - _Requirements: all_

  - [ ] 12.2 Add confetti configuration
    - Implement fireConfetti() using exact specified config:
      particleCount:80, spread:70, origin:{y:0.7}, colors:['#00b4a6','#22c55e','#f59e0b','#818cf8']
    - _Requirements: 2.7_

  - [ ] 12.3 Initialize app on DOMContentLoaded
    - Load state from localStorage
    - Check first launch flag
    - Show welcome modal or navigate to home
    - Initialize music controller state
    - Call lucide.createIcons() to render Lucide icons
    - _Requirements: 1.1, 1.8_

  - [ ] 12.4 Add README.txt to workspace root
    - Setup instructions
    - Music file naming guide
    - iOS background audio behavior note
    - License key instructions

- [ ] 13. Final checkpoint — Full smoke test
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties
- Unit tests validate specific examples and edge cases
