# Requirements Document

## Introduction

ADHD Dopamine Menu Tracker is a single-file Progressive Web App designed for people with ADHD. It provides a calming, personalized "dopamine menu" — a curated list of activities that help ADHD brains get started, feel rewarded, and build momentum. The app runs 100% offline, stores all data in localStorage, and prioritizes warmth and calm over productivity pressure.

## Glossary

- **Dopamine_Menu**: A curated list of activities categorized by effort level, used to help ADHD brains choose what to do when stuck
- **Win**: A logged completion of a menu item
- **Streak**: Consecutive days with at least one logged win
- **Age_Group**: One of three user profiles — Child (8–12), Teen (13–17), Adult (18+) — that filters menu content and music
- **Menu_Item**: A single activity card with emoji, name, time estimate, and category
- **Category**: One of four groupings — Appetizers (≤5 min), Mains (5–15 min), Desserts (rewards), Sensory (nervous system resets)
- **Service_Worker**: Browser background script enabling offline caching
- **PWA**: Progressive Web App — installable, offline-capable web application
- **Heat_Map**: A 30-day calendar visualization where teal intensity represents win count per day
- **License_Key**: A string entered by the user to activate Pro features, stored in localStorage

---

## Requirements

### Requirement 1: First Launch Welcome Flow

**User Story:** As a new user, I want to be greeted with a welcoming onboarding modal so that I can set my age group and get a personalized experience immediately.

#### Acceptance Criteria

1. WHEN the app is opened for the first time (adhd_first_launch is not false in localStorage), THE App SHALL display a full-screen welcome modal that cannot be dismissed by clicking outside it
2. THE Welcome_Modal SHALL display the app name, the tagline "Your brain's personalized dopamine menu", and three large selectable age group cards (Child / Teen / Adult) with emoji and brief descriptions
3. WHEN a user selects an age group card, THE App SHALL save the age group to localStorage under adhd_age_group
4. WHEN a user selects an age group card, THE App SHALL pre-load all age-appropriate menu items into localStorage under adhd_menu_items
5. WHEN a user selects an age group card, THE App SHALL dismiss the welcome modal with a fade animation
6. WHEN a user selects an age group card, THE App SHALL immediately begin playing the age-appropriate background music track
7. WHEN a user selects an age group card, THE App SHALL navigate to the Home screen
8. WHEN the app is opened and adhd_first_launch is false in localStorage, THE App SHALL skip the welcome modal and go directly to the Home screen

---

### Requirement 2: Home / Menu Screen

**User Story:** As a user, I want to browse and interact with my dopamine menu so that I can quickly find and log activities that help me get started.

#### Acceptance Criteria

1. THE Home_Screen SHALL display a search bar at the top that filters menu items in real time as the user types
2. THE Home_Screen SHALL display category tabs for Appetizers, Mains, Desserts, and Sensory that filter the visible items
3. WHEN a category tab is selected, THE Home_Screen SHALL show only items belonging to that category
4. THE Home_Screen SHALL display each enabled menu item as a card showing: emoji, name, time estimate, and age badge
5. THE Home_Screen SHALL display a teal "I Did This ✓" button on each item card with a minimum tap target of 48×48px
6. WHEN the "I Did This ✓" button is tapped, THE App SHALL log the item to today's wins in localStorage under adhd_wins with the current timestamp
7. WHEN the "I Did This ✓" button is tapped, THE App SHALL trigger a confetti burst using the specified confetti configuration
8. WHEN the "I Did This ✓" button is tapped, THE App SHALL apply a subtle scale animation (200–300ms ease-out) to the card
9. THE Home_Screen SHALL display a "Feeling stuck?" shortcut button
10. WHEN the "Feeling stuck?" button is tapped, THE App SHALL randomly select one Appetizer item and visually highlight it

---

### Requirement 3: Build / Edit Menu Screen

**User Story:** As a user, I want to customize my dopamine menu so that it reflects activities that actually work for my brain.

#### Acceptance Criteria

1. THE Build_Screen SHALL display all pre-loaded menu items with a toggle to enable or disable each one
2. WHEN a menu item toggle is changed, THE App SHALL persist the updated enabled state to localStorage under adhd_menu_items immediately
3. THE Build_Screen SHALL provide a form to add custom menu items with fields for: name, category, emoji, and time estimate
4. WHEN a valid custom item is submitted, THE App SHALL add it to localStorage under adhd_custom_items and display it in the menu
5. THE Build_Screen SHALL display a delete button on custom items only (not pre-loaded items)
6. WHEN a custom item delete button is tapped, THE App SHALL remove that item from localStorage and the displayed list
7. WHERE drag-and-drop reordering is supported by the browser, THE Build_Screen SHALL provide drag handles to reorder items

---

### Requirement 4: Today's Wins Screen

**User Story:** As a user, I want to see what I've accomplished today so that I feel encouraged and can track my momentum.

#### Acceptance Criteria

1. THE Wins_Screen SHALL display a list of all items logged today with their timestamps
2. THE Wins_Screen SHALL display a total win count with an encouraging message that varies by count: 0 wins = gentle nudge, 1–3 wins = warm praise, 4+ wins = celebration message
3. THE Wins_Screen SHALL display the current streak counter (consecutive days with at least one win)
4. THE Wins_Screen SHALL display an "Export to Beautiful PDF" button
5. WHEN the "Export to Beautiful PDF" button is tapped, THE App SHALL generate a styled PDF using html2pdf.js containing: current menu, last 30 days wins summary, streak count, and generation date
6. THE PDF SHALL use teal accents and include the app name and generation date in the header

---

### Requirement 5: History Screen

**User Story:** As a user, I want to see my activity history so that I can recognize patterns and celebrate my progress over time.

#### Acceptance Criteria

1. THE History_Screen SHALL display a 30-day heat-map calendar where teal color intensity represents the number of wins logged on each day
2. WHEN a day cell on the heat-map is tapped, THE App SHALL display that day's logged items in a slide-up panel
3. THE History_Screen SHALL display a monthly summary including: total wins, most-used category, and longest streak for the displayed period

---

### Requirement 6: Settings Screen

**User Story:** As a user, I want to configure the app to match my preferences so that it feels right for me.

#### Acceptance Criteria

1. THE Settings_Screen SHALL display a toggle for "Calming Background Music" that is ON by default
2. WHEN the music toggle is changed, THE App SHALL pause or resume the active audio track and persist the state to localStorage under adhd_music_enabled
3. THE Settings_Screen SHALL display an age group selector that allows the user to change their age group
4. WHEN the age group is changed in Settings, THE App SHALL fade out the current music track over 300ms, swap to the new age-appropriate track, and fade it in
5. THE Settings_Screen SHALL display a license key input field labeled "Activate Pro — Enter your Payhip/Gumroad key"
6. WHEN a non-empty string is entered in the license key field, THE App SHALL display a green checkmark and persist the value to localStorage under adhd_license_key
7. THE Settings_Screen SHALL display a "Reset all data" button
8. WHEN the "Reset all data" button is tapped, THE App SHALL display a confirmation dialog before executing the reset
9. WHEN the reset is confirmed, THE App SHALL clear all adhd_* keys from localStorage and reload the app
10. THE Settings_Screen SHALL display the app version as v1.0.0

---

### Requirement 7: Age-Adaptive Background Music

**User Story:** As a user, I want calming background music suited to my age group so that the app helps regulate my nervous system.

#### Acceptance Criteria

1. THE App SHALL use three separate hidden <audio> elements, one per age group, referencing music-child.mp3, music-teen.mp3, and music-adult.mp3
2. THE Child_Track SHALL play at volume 0.4 with the loop attribute enabled
3. THE Teen_Track SHALL play at volume 0.45 with the loop attribute enabled
4. THE Adult_Track SHALL play at volume 0.5 with the loop attribute enabled
5. WHEN the age group is selected on the welcome modal (first user gesture), THE App SHALL begin playing the corresponding audio track
6. IF an audio file returns a 404 or load error, THE App SHALL fail silently without displaying any error to the user
7. WHEN the user changes age group in Settings, THE App SHALL fade out the current track over 300ms and fade in the new track
8. WHEN the app is loaded and adhd_music_enabled is true and adhd_music_playing is true, THE App SHALL resume the active track after a user gesture

---

### Requirement 8: Pre-loaded Menu Items

**User Story:** As a user, I want a rich set of pre-loaded activities appropriate for my age so that I have a useful menu from day one.

#### Acceptance Criteria

1. THE App SHALL include all 15 Appetizer items as specified, filtered by age group tags [ALL], [C], [T], [A]
2. THE App SHALL include all 20 Main items as specified, filtered by age group tags
3. THE App SHALL include all 13 Dessert items as specified, filtered by age group tags
4. THE App SHALL include all 13 Sensory Reset items as specified, filtered by age group tags
5. WHEN age group is "child", THE App SHALL include only items tagged [ALL] or [C] or [C][T]
6. WHEN age group is "teen", THE App SHALL include only items tagged [ALL] or [T] or [C][T] or [T][A]
7. WHEN age group is "adult", THE App SHALL include only items tagged [ALL] or [A] or [T][A]
8. THE App SHALL display an age badge on each item card indicating which age groups the item applies to

---

### Requirement 9: PWA and Offline Support

**User Story:** As a user, I want the app to work offline and be installable so that I can use it anywhere without an internet connection.

#### Acceptance Criteria

1. THE App SHALL register a Service_Worker that uses a cache-first strategy for offline use
2. THE App SHALL include a Web App Manifest with theme color #00b4a6, app name, and appropriate icons
3. THE App SHALL include an inline SVG favicon
4. WHEN the app is loaded offline after first visit, THE Service_Worker SHALL serve the cached app shell

---

### Requirement 10: Design System and Accessibility

**User Story:** As a user with ADHD, I want the app to feel calm, warm, and easy to use so that it doesn't add cognitive load.

#### Acceptance Criteria

1. THE App SHALL use the specified design tokens: background #0f172a, primary accent #00b4a6, surface #1e293b, surface raised #334155, text primary #f1f5f9, text muted #94a3b8, success #22c55e, warning #f59e0b
2. THE App SHALL use a system UI font stack with large sizes and generous line-height
3. ALL interactive tap targets SHALL be a minimum of 48×48px
4. ALL animations SHALL be subtle, between 200–300ms, using ease-out timing, with no jarring motion
5. THE App SHALL use a mobile-first layout with a bottom tab bar on screens narrower than 768px
6. THE App SHALL use a left sidebar (240px wide) with navigation items on screens 768px and wider
7. THE App SHALL use Tailwind CSS via CDN for layout utilities
8. THE App SHALL use Lucide icons via CDN or emoji fallbacks where Lucide is unavailable
