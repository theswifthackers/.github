# Writing for Interfaces

Rules for all text shown to users inside the app: labels, errors, empty states, buttons, onboarding, confirmations, notifications.  
Sourced from andrewgleave/skills/writing-for-interfaces.

---

## When this applies

Any time you write, review, or improve text that users see inside the app:
- Button labels, titles, body copy
- Error messages and alerts
- Empty states
- Onboarding and permission rationale strings
- Settings descriptions
- Confirmation dialogs
- Notification content

Does NOT apply to: App Store listing copy, marketing text, API documentation.

---

## Step 1 — Establish voice

Before writing, understand who this product is and who it's for. Check for:
- An existing `AGENTS.md`, `CLAUDE.md`, or style guide that defines voice and tone
- Existing copy to infer patterns from (formal or casual? Technical or plain? Warm or direct?)

If no voice exists, infer from the app purpose and audience. Capture it in 3–4 adjectives.

**Voice** = stable personality. **Tone** = how voice adapts per moment.

| Situation | Tone shift |
|-----------|------------|
| Celebrating a milestone | More warmth, less brevity |
| Error / failure | More clarity, less friendliness |
| Critical / destructive action | Maximum directness, minimal personality |
| Onboarding | Balance helpfulness with warmth |

Precedence: **clarity > voice > craft rules.** When voice gets in the way of understanding, strip it back.

---

## Step 2 — Four core principles

### 1. Purpose
What is the single most important thing this person needs to know right now?
- Use information hierarchy: headline + button tell the primary story; body fills in detail.
- Cut anything that doesn't serve this moment.
- Tell people WHY a feature exists, not just WHAT it does.

### 2. Anticipation
Interface copy is a conversation. Anticipate the next question:
- Error → how to fix it: "Can't connect to Wi-Fi. Check your connection and try again."
- Request → how to do it: "Verify your identity" + a visible action.
- Success → what happens next: "Password changed. Sign in with your new password."
- Lead with the benefit: "To back up your data, enable iCloud." (reason first, action second.)

### 3. Context
Write for the physical and emotional situation:
- Mid-task text should be ultra-brief. Setup flows can afford more.
- Show information when it's relevant, not before.
- "Tap" not "click" on touch surfaces.
- Test on a small screen; long copy breaks layouts.

### 4. Empathy
Write for everyone:
- Plain, direct language — no jargon, idioms, or cultural references.
- Inclusive, neutral language (no unnecessary gender/age/ability references).
- Account for text expansion (German can be 30% longer than English).
- Accessibility labels aren't afterthoughts — they're the entire experience for some users.

---

## Step 3 — Craft rules

Apply these after voice and principles are right.

### Remove filler
Every word must earn its place. **Test:** remove the word; if meaning and tone don't change, cut it.

| Cut | Keep |
|-----|------|
| "Simply tap…" | "Tap…" |
| "Oops!" in errors | (nothing) |
| "Please try again" | "Try again" |
| "You have successfully saved" | "Saved" |
| "Just" | (nothing) |

Exception: a "filler" word that carries voice warmth ("Nothing here yet" vs "Nothing here") stays.

### Avoid repetition
Headline and body saying the same thing → collapse into one. Each element adds new information.

"We're running late. Your driver won't be there on time. They'll arrive in 10 minutes."  
→ "Delivery delayed 10 minutes."

### Be specific
Name the thing and the action:

| Vague | Specific |
|-------|----------|
| "Can't open this file" | "Can't open 'Quarterly Report.pdf'" |
| "Payment error" | "Card ending in 4242 was declined" |
| "Yes" / "No" | "Cancel Subscription" / "Keep Subscription" |

### Consistent terminology
Build a word list. Pick one term per concept; use it everywhere.

If it's "alias" on screen A, don't say "username" on screen B. If "Next" advances a flow, always use "Next".

### Pronouns
"Favorites" = "Your Favorites". Avoid "we" — it obscures what happened. "Unable to load" not "We're having trouble loading".

### Exclamation marks
Rare. Reserve for genuinely celebratory moments (first purchase, milestone reached). Never in errors.

### Capitalisation
Pick title case or sentence case and apply consistently. Sentence case reads more casual; title case more formal.

---

## Common patterns

### Error messages
Structure: what went wrong → how to recover.
```
"Can't sign in. Check your email and password, then try again."
"No internet connection. Connect to Wi-Fi or cellular and retry."
```
Avoid: "An error occurred. Please try again later." (too vague, no recovery path.)

### Empty states
Structure: label → brief explanation → optional action.
```
"No Items Yet
Add your first item to get started."
[Add Item]
```

### Alerts and confirmations
Lead with what will happen; name the destructive action explicitly:
```
"Delete 'Project X'?
This can't be undone."
[Delete] [Cancel]
```

### Onboarding / permission rationale
Lead with the benefit (required by App Review for clarity):
```
"To send you reminders, [App] needs permission to send notifications."
```

### Buttons
- Use verb phrases: "Save", "Delete", "Add Item", "Turn On Notifications"
- The primary action is affirmative; the cancel/secondary is neutral
- Destructive actions use red tint and name the thing being destroyed

### Success / confirmation banners
Point forward: "Photo saved. View in Photos →"  
Don't just say "Done."

---

## Dynamic content

Templates like `"\(count) items selected"` are interface copy too:
- Handle zero, one, many: "No results", "1 result", "24 results"
- Read it with real values — long names and large numbers
- Keep templates simple; complex branching usually means the design needs to change

---

## The simplest test

Read it out loud. If it sounds like how you'd explain something to a friend — clear, natural, no filler — it's good. If it sounds like a robot or legal document, keep refining.
