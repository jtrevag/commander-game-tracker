# Commander Game Tracker - Project Context

## Project Overview
A web-based application for tracking Magic: The Gathering Commander games. Users can log game data including commanders played, brackets, winners, and turn counts. The app features Scryfall autocomplete for commander names and saves data to Google Sheets.

## Architecture

### Frontend
- Single HTML file (`index.html`) with embedded CSS and JavaScript
- No external dependencies except CDN-hosted libraries
- Runs entirely in the browser
- Can be hosted on GitHub Pages or run locally

### Backend/Data Storage
- **Commander List Service**: Google Apps Script that fetches and caches all legal commanders from Scryfall API
  - Hosted separately by the project maintainer
  - Returns JSON list of ~2,910 commander names
  - Updates weekly via time-based trigger
  - URL: `https://script.google.com/macros/s/AKfycbz_7VCIH3Pc3CKpIXYaIfcZbwu1qrnR4d-54T8MLesjmbiIwFzn63u-KTuVA7MEIiyIdA/exec`

- **Game Data Storage**: Each user has their own Google Apps Script
  - Receives POST requests with game data
  - Writes to a Google Sheet tab named "Commander Game Log"
  - Auto-creates the tab with proper headers if it doesn't exist

## Key Features

### 1. Commander Autocomplete
- Fetches complete commander list on page load from shared service
- Client-side filtering as user types (2+ characters)
- Shows up to 8 matches
- Works for all 4 commander input fields

### 2. Game Tracking Form
Fields tracked per game:
- Date (defaults to today)
- Bracket (1-5 dropdown: Casual, Focused, Optimized, High Power, Competitive)
- Your Commander (with autocomplete)
- Opponent 1-3 Commanders (with autocomplete)
- Which Commander Went First (dropdown populated from entered commanders)
- Winner (dropdown populated from entered commanders)
- Turn Game Ended (number input)

### 3. Settings/Setup System
- Modal with complete step-by-step instructions
- Users paste their Google Apps Script deployment URL
- URL saved to localStorage for persistence
- Form disabled until setup complete
- Copy button for easy Apps Script code copying

## Data Flow

### Commander List Loading
```
Page Load → Fetch from Commander List URL → Cache in memory → Enable autocomplete
```

### Game Submission
```
User fills form → Click "Record Game" → POST to user's Google Script URL → 
Append row to "Commander Game Log" sheet → Show success message → Reset form
```

## Google Apps Script Components

### Commander List Script (Maintained by Project Owner)
```javascript
// Two main functions:
doGet() - Returns JSON list of commanders
updateCommanderList() - Fetches from Scryfall API using is:commander filter
```

**Key Details:**
- Uses Scryfall search API with pagination
- Query: `is:commander&unique=cards`
- ~17 pages of results at 175 cards per page
- Takes 2-3 minutes to complete
- Batch writes to sheet for performance
- Should be run weekly via time-based trigger

### Game Tracking Script (Each User's Own)
```javascript
doPost(e) - Receives game data and appends to sheet
```

**Key Details:**
- Creates "Commander Game Log" tab if missing
- Headers: Date, Bracket, Your Commander, Opponent 1-3, Went First, Winner, Turn Ended
- Auto-formats header row (bold, colored background)
- Uses `mode: 'no-cors'` for compatibility

## Technical Considerations

### Browser Storage
- Uses localStorage to persist user's script URL
- Commander list cached in memory (not persisted)
- Commander list reloads on each page load

### CORS & CSP
- Works fine when hosted/downloaded
- Does NOT work in Claude artifacts due to CSP restrictions
- Both Scryfall API and Google Apps Script URLs are blocked in artifacts

### Performance
- Commander list: ~2,910 names, loads once per session
- Autocomplete: Client-side filtering (no API calls during typing)
- Form submission: Single POST request per game

## Setup Instructions for End Users

1. **One-time setup per user (~5 minutes)**
   - Create/open Google Sheet
   - Go to Extensions → Apps Script
   - Paste the game tracking script
   - Deploy as web app (Execute as: Me, Access: Anyone)
   - Copy deployment URL
   - Open Commander Tracker
   - Click gear icon, paste URL, save

2. **Using the app**
   - Open tracker (bookmark it)
   - Fill in game details
   - Commander names autocomplete as you type
   - Click "Record Game"
   - Data saves to personal Google Sheet

## Maintenance

### Adding New Commanders
- Automatic if weekly trigger is set up
- Manual: Run `updateCommanderList()` in Commander List Apps Script
- Takes 2-3 minutes
- All users get updates automatically (no action needed)

### Updating the Tracker
- Edit `index.html` in GitHub repo
- Changes go live immediately after commit
- Users may need to hard refresh (Ctrl+F5)

## Known Limitations
1. Cannot run in Claude artifacts (CSP restrictions)
2. Requires Google account for each user
3. Commander list must be maintained separately
4. No offline mode (requires internet for autocomplete and saving)
5. No built-in analytics/statistics (data is in Google Sheets for custom analysis)

## Future Enhancement Ideas
- Export game history as CSV
- Basic statistics dashboard (win rates, favorite commanders)
- Multi-deck tracking per user
- Game notes field
- Pod tracking (same 4 players over time)
- Integration with deck lists
- Color identity indicators on commanders