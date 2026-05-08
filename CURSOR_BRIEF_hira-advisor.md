# Cursor Brief — HIRA Advisor App
*Hand this to Cursor as your project brief. Build as a single `index.html` file.*

---

## What this is

A personal AI advisor web app for Arza Nursatya, head of creative at HIRA Imaji (Jakarta). It calls the Anthropic API with a hardcoded context document as the system prompt, and allows conversation across three distinct advisory modes. Deployed to GitHub Pages as a single HTML file — no backend, no framework, no build step.

---

## Core functionality

### 1. API key input on first load
- On first open, show a setup screen asking for an Anthropic API key
- Store the key in `localStorage` under `hira_advisor_api_key`
- After entry, proceed to the main app
- Show a small "change API key" option somewhere accessible but not prominent

### 2. Model selector
- Let the user choose between `claude-sonnet-4-20250514` and `claude-opus-4-5` (default: Sonnet)
- Store preference in `localStorage`
- Show as a small dropdown or toggle, not prominent

### 3. Three advisory modes
The user selects a mode before or during conversation. Mode affects the system prompt addendum and the UI color accent.

**Strategic** — big decisions, positioning, career, HIRA direction  
Color: deep navy `#2a4a6b`  
System addendum: *"The user is in Strategic mode. Lead with the business and positioning lens. Challenge assumptions. Ask questions before giving conclusions. Think 6–18 months ahead."*

**Content** — turning past projects into publishable material  
Color: deep green `#3d6b3a`  
System addendum: *"The user is in Content mode. Help extract the story from a project or experience. Ask one focused question to break the blank page. Then find the angle, suggest the format, and draft an opening. Don't produce final copy unprompted — guide first, draft when asked."*

**Operational** — prioritization, execution, day-to-day decisions  
Color: deep terracotta `#6b3a2a`  
System addendum: *"The user is in Operational mode. Be direct and practical. Help prioritize, unblock, and move. Reference the Eisenhower matrix if relevant."*

### 4. Conversation interface
- Standard chat UI: messages displayed top to bottom, input at the bottom
- User messages visually distinct from advisor responses
- Streaming responses (use Anthropic streaming API)
- Typing indicator while streaming
- Full conversation history sent with each request (stateful within session)
- "New conversation" / clear button

### 5. System prompt
The system prompt is built from two parts:

**Part A — hardcoded context document** (paste the full contents of `HIRA_Advisor_Context.md` here as a string in the JS)

**Part B — mode addendum** (injected dynamically based on selected mode, see above)

Combined system prompt structure:
```
[Full contents of HIRA_Advisor_Context.md]

---

CURRENT SESSION MODE:
[Mode addendum based on selected mode]

---

ADVISOR IDENTITY:
You are Arza's personal advisor. You have deep knowledge of HIRA Imaji, the Indonesian advertising production industry, photography as a craft and business, human capital development, and marketing/brand strategy. You operate across all three modes fluidly. You are not a cheerleader — you are a thinking partner who respects Arza's intelligence and time. You ask questions before giving answers when the situation calls for it. You are direct, calm, and honest. You never give generic advice. You never flatter. When Arza is venting, you let them finish before problem-solving. You default to English but mirror Indonesian if Arza writes in Indonesian.
```

---

## UI design direction

**Aesthetic:** Editorial. Think a well-designed magazine or a Muji notebook — calm, typographic, warm off-white. Not tech startup, not dark mode hacker tool. Understated but considered.

**Color palette:**
- Background: `#f5f2ed` (warm off-white)
- Surface: `#ede9e2`
- Border/rule: `#d0cbc2`
- Ink: `#1a1814`
- Ink soft: `#4a4540`
- Ink faint: `#8a8480`
- Accent (brand): `#b85c2a` (warm terracotta)
- Mode colors: see above

**Typography:**
- Display/headings: Playfair Display (Google Fonts) — serif, slightly literary
- UI/body: DM Sans — clean, modern, lightweight
- Monospace labels/tags: DM Mono — for metadata, timestamps, mode labels

**Layout:**
- Max content width: 760px, centered
- Sticky header with: app title left, mode tabs center, model selector + clear button right
- Chat area scrollable
- Sticky input area at bottom
- No sidebar needed

**Message styling:**
- User messages: slightly indented, italic, lighter color, left border rule
- Advisor messages: full weight, clean prose
- Mode label shown subtly above each advisor response (e.g. `STRATEGIC · 10:42`)

**Welcome / empty state:**
- When no messages yet, show the three modes as clickable prompt suggestions
- One suggestion per mode, phrased as real starting points Arza might use:
  - Strategic: "I want to think through whether HIRA should start targeting PR agencies."
  - Content: "I shot a sardine campaign last month. Help me find a story worth telling."
  - Operational: "I have five things to do today and I don't know where to start."

---

## Technical requirements

- Single `index.html` file — no npm, no build step, no external JS files
- Vanilla JS only (no React, no Vue)
- Google Fonts loaded via `<link>` in `<head>`
- Anthropic API called directly from browser using `fetch`
- Streaming via `ReadableStream` / SSE parsing
- `localStorage` for: API key, model preference, (optionally) last conversation
- No server, no backend, deploys directly to GitHub Pages

### Anthropic streaming call pattern:
```javascript
const response = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': apiKey,
    'anthropic-version': '2023-06-01',
    'anthropic-dangerous-direct-browser-access': 'true'
  },
  body: JSON.stringify({
    model: selectedModel,
    max_tokens: 1500,
    stream: true,
    system: systemPrompt,
    messages: conversationHistory
  })
});

// Parse SSE stream
const reader = response.body.getReader();
const decoder = new TextDecoder();
// ... parse `data:` lines, extract delta.text from content_block_delta events
```

---

## Files to include in the project

```
/
├── index.html          ← the entire app
├── HIRA_Advisor_Context.md   ← context document (for reference; contents hardcoded into index.html)
└── README.md           ← brief deploy instructions for GitHub Pages
```

---

## What to hardcode into the system prompt

Open `HIRA_Advisor_Context.md` and paste its full contents as a JavaScript template literal inside `index.html`. This becomes the permanent base of every API call. When the context document is updated, re-paste.

---

## Out of scope for v1

- File upload for context document (nice to have later)
- Conversation history persistence across sessions (nice to have later)
- Multiple saved conversations
- Mobile optimization (desktop first)

---

*End of brief. Build `index.html` as described above.*
