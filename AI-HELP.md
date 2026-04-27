# Ghost Lens AI Help System

## Overview

Ghost Lens now features **AI-powered help generation**. When a user sits idle for 6 seconds, Ghost Lens:

1. **Shows an Invitation Trigger**: A subtle ghost icon appears in the corner.
2. **User "Invites" Help**: Clicking the trigger begins the full experience.
3. **Screenshots the page** (html2canvas or Canvas API)
4. **Sends to vision AI** (Ollama llava:34b or Claude API)
5. **Gets structured help data** (JSON with element descriptions)
6. **Renders beautifully** with:
   - SVG arrows pointing to elements
   - Animated help cards
   - Sequential reveal (top to bottom)
   - Heartbeat pulse every 8 seconds
   - Auto-fade after 60 seconds

## Quick Start

### Basic Usage

```javascript
import { GhostLensEngine } from './core.js';

const engine = new GhostLensEngine(document.body, {
  useAIHelp: true,  // Enable AI help
  idleDelay: 6000,  // 6 seconds before activation
});
```

### With Claude Fallback

```javascript
const engine = new GhostLensEngine(document.body, {
  useAIHelp: true,
  claudeApiKey: 'sk-ant-...',  // Optional fallback
});
```

### Hardcoded Help (Fallback)

```javascript
const engine = new GhostLensEngine(document.body, {
  useAIHelp: false,  // Use data-ghost attributes instead
});
```

## Configuration

### Options

```javascript
{
  // Core timing
  idleDelay: 6000,                // Wait 6s before activation
  resetDelay: 300,                // Reset timer after activity
  curtainDuration: 600,           // Screen wipe animation (ms)
  
  // Heartbeat & duration
  heartbeatInterval: 8000,        // Pulse every 8 seconds
  maxActiveDuration: 60000,       // Auto-fade after 60 seconds
  
  // AI Help options
  useAIHelp: true,                // Enable AI help generation
  requireInvitation: true,        // Show trigger first (default: true)
  claudeApiKey: null,             // Claude fallback (optional)
  useHardcodedFirst: false,       // Try data-ghost first
  
  // Styling
  curtainColor: 'rgba(0, 200, 255, 0.06)',
  tooltipBg: 'rgba(0, 18, 30, 0.92)',
  tooltipBorder: 'rgba(0, 200, 255, 0.4)',
  tooltipColor: '#a8f0ff',
  accentColor: '#00c8ff',
  zIndex: 9999,
  
  // Callbacks
  onActivate: null,    // Called when help appears
  onDeactivate: null,  // Called when help closes
}
```

## Architecture

### Files

- **core.js** — Main Ghost Lens engine (updated with AI integration)
- **ai-help.js** — AI help generation (screenshot → vision AI → JSON)
- **animated-overlay.js** — Beautiful SVG + animated card rendering
- **demo-ai.html** — Test page with working examples

### Flow

```
User idle 6s
    ↓
Show subtle Ghost Trigger
    ↓
User clicks trigger ("Invitation")
    ↓
Screenshot page
    ↓
Vision AI analyzes (Ollama → Claude fallback)
    ↓
Returns structured help JSON
    ↓
Enrich with element positions
    ↓
Create SVG arrows + animated cards
    ↓
Sequential reveal (top to bottom)
    ↓
Heartbeat pulse every 8s
    ↓
Auto-fade after 60s or user dismisses
```

## How It Works

### 1. Screenshot Capture

Ghost Lens uses **html2canvas** (if loaded) or Canvas API to capture the current page state.

```javascript
// Automatically detected:
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
```

### 2. Vision AI Analysis

The screenshot is sent to **Ollama's llava:34b** (local) or **Claude 3.5 Sonnet** (fallback).

**Prompt:** "Analyze this UI. List each interactive element, what it does, why it matters."

**Response:** JSON with sections, elements, descriptions.

```json
{
  "sections": [
    {
      "title": "Upload Panel",
      "description": "Main file input area",
      "elements": [
        {
          "selector": "input[type=file]",
          "label": "File Input",
          "help": "Click to select a video or audio file",
          "type": "input"
        }
      ]
    }
  ],
  "overview": "This is a transcription tool..."
}
```

### 3. Dynamic Rendering

Ghost Lens:
- Finds elements on page by CSS selector
- Gets their bounding boxes
- Positions help cards nearby
- Draws SVG arrows from cards to elements
- Animates sequentially (top to bottom)

### 4. Animations

- **Screen wipe:** Cyan curtain sweeps during activation
- **Help cards:** Fade in + slide up, staggered
- **Arrows:** Draw themselves with stroke-dasharray animation
- **Heartbeat:** Badge pulses with expanding ring every 5s
- **Fade out:** Smooth opacity fade after 60s

## Fallback Strategy

Ghost Lens tries in order:

1. **Ollama (llava:13b)** — Local, instant, no API key (using 13b to avoid OOM)
2. **Claude API** — Requires `claudeApiKey` in options
3. **Hardcoded `data-ghost` attributes** — Last resort

If all fail, it logs a warning and uses the traditional hardcoded approach.

## Environment Variables

Optional:

```bash
# For Claude fallback (not recommended in browser)
# Instead, pass claudeApiKey via options
```

**Security note:** Don't embed API keys in browser code. Use a backend relay if needed.

## Testing

### Demo Page

```bash
# Open demo-ai.html in browser
# It has sample UI elements for Ghost Lens to analyze
open demo-ai.html
```

### Local Testing

1. Ensure **Ollama is running** with `llava:34b`
2. Load the demo page
3. Click "Start Demo" or "Force Activate"
4. Wait/watch Ghost Lens analyze and reveal help

## Performance Notes

- **Screenshot:** 200-400ms (depends on page size)
- **Vision AI analysis:** 3-8 seconds (Ollama varies)
- **Rendering:** <100ms (SVG + DOM ops)
- **Total activation:** ~4-10 seconds from idle

For better UX, Ghost Lens shows the screen wipe immediately and generates help in background.

## Browser Compatibility

- **Chrome/Brave:** ✅ Full support
- **Firefox:** ✅ Full support
- **Safari:** ⚠️ Canvas screenshot works, vision API depends on fetch
- **Mobile:** ⚠️ Limited (small screens, no Ollama access)

## Customization

### Change AI Model

Edit **ai-help.js**, `analyzeWithOllama()`:

```javascript
model: 'llava:13b',  // Change to other vision models
```

**Note:** llava:34b requires significant VRAM. Use 13b for most systems.

### Change Animation Speed

```javascript
const engine = new GhostLensEngine(document.body, {
  heartbeatInterval: 3000,   // Faster pulse
  maxActiveDuration: 120000, // 2 minutes instead of 60
});
```

### Add Custom Help Prompt

Edit **ai-help.js**, modify the prompt in `analyzeWithOllama()`.

## Troubleshooting

### "Ollama API error"

- Ensure Ollama is running: `ollama serve`
- Check model exists: `ollama list | grep llava`
- Test endpoint: `curl http://localhost:11434/api/tags`

### "No JSON in response"

- Vision model might be failing
- Check Ollama logs
- Try Claude API with `claudeApiKey` option

### Screenshot is blank

- `html2canvas` might not be loaded
- Fallback uses Canvas API (simpler)
- Check browser console for errors

### Help cards not appearing

- Check element selectors in AI response
- Make sure elements exist on page
- View browser console for warnings

## Future Ideas

- [ ] Cache help responses by page URL
- [ ] Let users edit/improve AI help
- [ ] Track which help is most useful
- [ ] Adaptive help based on user behavior
- [ ] Synthesized video walkthroughs
- [ ] Multi-language support
- [ ] Integration with doc systems (Notion, Confluence)

## License

Same as Ghost Lens (MIT)
