# Context File Acknowledgment — Design Spec
*Date: 2026-05-13*

## Problem

When a user attaches a `.md` context file, Claude receives it via the system prompt but gives no visible signal it was read. The user has to trust that the attachment worked and that Claude is applying it — there's no confirmation.

## Goal

On the first message of a session where a context file is attached, Claude's response should clearly surface what it understood from the file — without auto-sending any messages unprompted.

## Approach: First-message injection (Option B)

**Where:** `sendMessage()` function in `index.html`, just before the API call is assembled.

**Trigger condition:** `sessionContext` is set AND `conversationHistory` is empty before the current push (i.e., this is the first user message of the session).

**Mechanism:** When triggered, prepend a hidden instruction to the user content sent to the API. This instruction is NOT stored in `conversationHistory` (so it doesn't persist to history or Supabase) and NOT rendered in the UI.

**Injection text:**
```
[Context note: You have been given a document called "${sessionContext.name}". In this response, briefly surface 2–3 things you've understood from it that are relevant to what I'm asking, before answering.]
```

**Handling both content shapes:**
- Plain string (no images): prepend the injection as a prefix with a newline separator.
- Content block array (with images): prepend a `{ type: 'text', text: injection }` block at index 0.

**Subsequent messages:** No injection. Claude continues using the context via the system prompt but doesn't repeat the acknowledgment.

## What does NOT change

- The system prompt `buildSystemPrompt()` is unchanged.
- The UI pill rendering is unchanged.
- `conversationHistory` storage format is unchanged.
- The injected text is never persisted to Supabase or localStorage.

## Success criterion

After attaching a `.md` file and sending the first message, Claude's response opens with an explicit reference to content from the file — relevant to the question asked — before answering.
