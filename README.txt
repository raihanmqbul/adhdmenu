================================================================================
  ADHD DOPAMINE MENU TRACKER
  A calming, offline-first PWA for ADHD brains
  Version 1.0.0
================================================================================

SETUP INSTRUCTIONS
------------------
1. Place index.html in any folder on your computer or web server.
2. Open index.html in a modern browser (Chrome, Edge, Firefox, Safari).
3. On first open, select your age group to get started.
4. That's it. No installation, no account, no internet required after first load.

To install as a PWA (add to home screen):
  - Chrome/Edge: Click the install icon in the address bar, or
    Menu → "Install ADHD Dopamine Menu Tracker"
  - iOS Safari: Tap Share → "Add to Home Screen"
  - Android Chrome: Tap Menu → "Add to Home Screen"


MUSIC FILE SETUP
----------------
The app plays calming background music based on your age group.
Music files are NOT included — you supply your own MP3 files.

Place these files in the SAME folder as index.html:

  music-child.mp3   →  For Child (8–12): bright, playful, gentle lo-fi with soft chimes
  music-teen.mp3    →  For Teen (13–17): lo-fi hip-hop, chill beats, slightly more energy
  music-adult.mp3   →  For Adult (18+):  deep focus ambient, binaural-adjacent, very calm

File naming is EXACT — lowercase, hyphenated, .mp3 extension.

Where to find suitable music:
  - Free Music Archive (freemusicarchive.org) — search "lo-fi" or "ambient"
  - YouTube Audio Library (studio.youtube.com/channel/music)
  - Pixabay Music (pixabay.com/music) — free, no attribution required
  - Lofi.cafe tracks (check individual licenses)

If a music file is missing, the app silently continues without music.
No error is shown to the user.


iOS BACKGROUND AUDIO BEHAVIOR
------------------------------
Due to iOS Safari restrictions:

1. Music will NOT autoplay until the user taps something on the page.
   The app handles this by starting music on the age group selection tap
   (first launch) or on the first tap after returning to the app.

2. When the iOS screen locks or the app goes to background, audio will pause.
   This is an iOS system restriction and cannot be overridden by web apps.

3. To keep music playing on iOS while using other apps, use a native music
   app instead and keep the Dopamine Menu open in Safari.

4. The music toggle in Settings will remember your preference across sessions.


LICENSE KEY INSTRUCTIONS
------------------------
The license key field in Settings is for future Pro features.

Current behavior:
  - Enter ANY non-empty string → green checkmark appears
  - The key is stored in localStorage on your device only
  - No server validation occurs (basic client-side check only)

If you purchased a license from Payhip or Gumroad:
  - Copy your license key from your purchase confirmation email
  - Paste it into Settings → "Activate Pro"
  - The green checkmark confirms it's saved

To remove a license key:
  - Clear the field in Settings, or
  - Use "Reset All Data" (this clears everything)


DATA & PRIVACY
--------------
All data is stored locally in your browser's localStorage.
Nothing is sent to any server. Ever.

Data stored:
  adhd_age_group      — your selected age group
  adhd_music_enabled  — music on/off preference
  adhd_music_playing  — music playing state
  adhd_menu_items     — your menu with enabled/disabled states
  adhd_custom_items   — any custom activities you've added
  adhd_wins           — your logged wins (last 365 days)
  adhd_license_key    — your license key (if entered)
  adhd_first_launch   — whether you've completed onboarding

To export your data: use the "Export to PDF" button in Today's Wins.
To delete all data: Settings → Reset All Data.


BROWSER COMPATIBILITY
---------------------
Fully supported:
  ✓ Chrome 90+
  ✓ Edge 90+
  ✓ Firefox 88+
  ✓ Safari 14+ (iOS and macOS)
  ✓ Samsung Internet 14+

Not supported:
  ✗ Internet Explorer (any version)


TROUBLESHOOTING
---------------
Music not playing?
  → Check that music-child.mp3 / music-teen.mp3 / music-adult.mp3 exist
    in the same folder as index.html
  → Try tapping anywhere on the screen (browser autoplay policy)
  → Check that the music toggle in Settings is ON

PDF export not working?
  → Requires internet connection (html2pdf.js loads from CDN)
  → Try again when connected, or use browser print (Ctrl+P / Cmd+P)

App not working offline?
  → Visit the app once while online to cache it
  → The service worker caches the app shell on first visit
  → Music files are NOT cached (they're too large)

Data disappeared?
  → Check if you're using a private/incognito window (localStorage clears on close)
  → Check if browser storage was cleared
  → Use the PDF export regularly to back up your wins

Confetti not showing?
  → Requires internet connection (canvas-confetti loads from CDN)
  → Win logging still works — confetti is purely decorative


CREDITS
-------
Built with:
  - Tailwind CSS (tailwindcss.com)
  - Lucide Icons (lucide.dev)
  - html2pdf.js (ekoopmans.github.io/html2pdf.js)
  - canvas-confetti (catdad.github.io/canvas-confetti)

Designed for ADHD brains everywhere. 🧠💙

================================================================================
