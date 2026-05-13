# 🍷 Bella Cucina — AI Concierge

> An elegant, single-file restaurant chatbot powered by **Groq** (LLaMA 3.3 70B) with a full tool-calling agent loop, live order tracking, and table reservations.

---

## Overview

Bella Cucina's AI Concierge, **Sofia**, is a guest-facing chat interface for a fictional Italian restaurant. Guests can explore the menu, get personalised recommendations, build an order, and book (or cancel) a table — all through natural conversation.

The entire application is a **single HTML file** with no backend, no build step, and no npm. The Groq API is called directly from the browser using the guest's own API key (stored in `sessionStorage`).

---

## Features

| Feature | Description |
|---|---|
| **Menu browsing** | Browse antipasti, primi, secondi, dolci, pizze, and beverages — with optional dietary filters (vegetarian / vegan) |
| **Chef's specials** | Nightly specials returned as a separate endpoint |
| **Recommendations** | Sofia suggests dishes based on guest preferences, occasion, or dietary needs |
| **Live order panel** | A persistent sidebar shows items as they're added, with a running subtotal |
| **Table reservations** | Check availability by date and party size, then confirm a booking |
| **Reservation cancellation** | Cancel any booking with a confirmation ID |
| **Order confirmation** | Place and confirm the current order |
| **Suggestion chips** | Quick-tap chips at the top and bottom of the chat for common actions |
| **Rich UI cards** | Reservation confirmations render as styled cards with a badge and booking ID |

---

## Tech Stack

| Layer | Choice |
|---|---|
| **LLM** | Groq API — `llama-3.3-70b-versatile` |
| **Tool calling** | OpenAI-compatible function-calling format over `/openai/v1/chat/completions` |
| **Frontend** | Vanilla HTML + CSS + JS (zero dependencies, zero build) |
| **Fonts** | Google Fonts — Cormorant Garamond + DM Sans |
| **State** | In-memory JS variables (`conversationHistory`, `currentOrder`, `reservations`) |
| **API key storage** | `sessionStorage` (cleared on tab close, never persisted) |

---

## Getting Started

### 1. Get a Groq API key

Sign up at [console.groq.com](https://console.groq.com) and create a free API key (format: `gsk_...`).

### 2. Open the file

Just open `index.html` in any modern browser — no server required.

```bash
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

Or serve it locally if you prefer:

```bash
npx serve .
# then visit http://localhost:3000
```

### 3. Enter your API key

A modal prompts for your Groq key on first load. The key is saved to `sessionStorage` and used for all subsequent API calls. Click **API Key** in the top bar to update it.

---

## Architecture

```
Browser (single HTML file)
│
├── UI Layer
│   ├── Sidebar  — logo, live order panel, restaurant info
│   ├── Topbar   — status indicator, New Chat, API Key buttons
│   ├── Chat     — message bubbles, tool indicators, typing animation
│   └── Input    — textarea with auto-resize, suggestion chips
│
├── Agent Loop  (runAgentLoop)
│   ├── POST /openai/v1/chat/completions  →  Groq API
│   ├── Parse tool_calls from response
│   ├── Execute tool locally (executeTool)
│   ├── Append tool result to conversation history
│   └── Loop (up to 5 iterations) until text-only response
│
├── Tool Registry (8 tools)
│   ├── get_menu            — filter menu by category / dietary
│   ├── get_specials        — tonight's chef specials
│   ├── check_availability  — open slots by date + party size
│   ├── make_reservation    — create a booking, remove the slot
│   ├── cancel_reservation  — cancel by ID, restore the slot
│   ├── add_to_order        — fuzzy-match item, update sidebar
│   ├── view_order          — list items + subtotal
│   └── confirm_order       — finalise and clear the order
│
└── In-Memory Data
    ├── RESTAURANT object   — menu, specials, available slots
    ├── conversationHistory — full message array sent each turn
    ├── currentOrder        — array of { item, qty, price }
    └── reservations        — map of ID → reservation object
```

---

## File Structure

This is a self-contained single file. Logical sections inside `index.html`:

```
index.html
├── <style>          CSS custom properties + all component styles
├── <aside>          Sidebar HTML (logo, order panel, info chips)
├── <main>           Chat area (topbar, messages, input area)
├── <div#api-modal>  API key entry modal
└── <script>
    ├── State variables
    ├── RESTAURANT data (menu, specials, availableSlots)
    ├── TOOLS array (OpenAI function-calling format)
    ├── executeTool()  dispatcher
    ├── Tool functions  (toolGetMenu, toolMakeReservation, …)
    ├── updateOrderPanel()  sidebar renderer
    ├── Chat UI helpers  (addMessage, addTypingIndicator, …)
    ├── SYSTEM_PROMPT  (Sofia's persona + capabilities)
    ├── sendMessage()  entry point
    ├── runAgentLoop()  agentic loop with tool execution
    ├── renderToolResult()  rich card renderer
    └── UI helpers  (sendChip, clearChat, autoResize, modal)
```

---

## Customisation

### Swap the restaurant data

Edit the `RESTAURANT` object to change the menu, specials, opening hours, address, or available booking slots.

### Change the AI model

Replace `"llama-3.3-70b-versatile"` in `runAgentLoop()` with any Groq-supported model.

### Use Anthropic's API instead of Groq

Replace the fetch URL with `https://api.anthropic.com/v1/messages`, update the request body to Anthropic's format, and add your API key with `x-api-key` header. The tool schema is compatible — just rename `type: "function"` wrappers to Anthropic's `type: "custom"` format.

### Add more tools

1. Define the tool schema in the `TOOLS` array.
2. Add a handler in `executeTool()`.
3. Write the handler function.
4. Optionally add a rich card renderer in `renderToolResult()`.

---

## Limitations

- **No persistence** — orders and reservations reset on page refresh. Integrate a backend or localStorage for persistence across sessions.
- **No real payments** — order confirmation is cosmetic only.
- **Availability is hardcoded** — `availableSlots` is a static object. Replace with a real calendar API for production.
- **Single-user** — all state is local to the browser tab.
- **API key in browser** — the Groq key is visible in DevTools. For production, proxy requests through a backend.

---

## License

MIT — use freely for personal or commercial projects.
