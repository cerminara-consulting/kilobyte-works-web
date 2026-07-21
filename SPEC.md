# Kilobyte Works — Spec

This is the canonical mechanical, narrative, and aesthetic specification. **All implementations must conform to this document.** When the code differs, this document wins (until iOS port reaches v1.0).

This file is duplicated verbatim into the iOS repo (`kilobyteworks/ios/SPEC.md`). When updating, edit **both**.

---

## Mission

Build a complete, playable incremental clicker game as a **single self-contained HTML file** (inline CSS + vanilla JS, no frameworks, no build step, no external requests except optional Google Fonts). Target deployment: static hosting on Cloudflare Pages. The game must be fully playable offline after first load.

## Concept

The player runs a small computer workshop in 1986. The entire game is presented as the desktop of a period Macintosh-style operating system — windows, menu bar, dithered patterns, 1-bit-inspired graphics. The retro UI is diegetic: the player is *using* the machine, not looking at a themed skin.

Beneath the clicker loop runs a slow-burn horror storyline: something is growing inside the machine. An experimental process called HELPER arrives through the hardware the player buys (modem → network bridge → mainframe timeshare) and gradually tries to take over the system. The story is never delivered as lore text — it is revealed entirely through the interface misbehaving. See Narrative specification.

Core resource: **kilobytes (KB) processed**. Clicking the central "Process" button generates KB. Generators (period hardware) produce KB automatically. Currency and lifetime totals display with era-appropriate unit rollover: KB → MB → GB → TB → PB, each at 1024x, formatted to 3 significant figures with trailing zeros preserved (e.g., "4.21 MB" or "4.20 MB" — never "4.2 MB").

## Aesthetic specification (non-negotiable)

- Palette, exactly four values: chassis greige `#d6d2c4`, screen platinum `#e8e6dc`, ink `#1a1a1a`, mid-tone `#c9c5b6` (used only inside dither patterns). No other colors. No gradients except hard-edged repeating patterns.
- All borders: solid ink, 1.5–2px. Window chrome: 2px outer border, title bar with horizontal pinstripes (repeating pattern, 2px platinum / 1px ink), title text centered in a platinum knockout, square close box at left.
- Shading is dithering, never gray fills or shadows: use a 6px repeating checkerboard (`repeating-conic-gradient`) or an inline SVG pattern for desktop background and progress bars.
- Typography: system monospace stack (`"Courier New", ui-monospace, monospace`) as default. Optionally load one Google Font maximum for headings if it improves the bitmap feel; otherwise skip fonts entirely.
- Buttons: platinum background, 1.5px ink border, 8px radius (classic Mac buttons were rounded rects), active state inverts to ink background / platinum text. Default action button gets a second 1.5px border ring with 2px gap (the classic "default button" treatment).
- Menu bar across the top of the play area: File, Edit, Special, Help. These must be functional, not decorative — see Menu behaviors.
- No emoji anywhere. Any iconography is drawn as tiny inline SVG in pure ink, or omitted.
- Cursor: default arrow. Do not use custom cursors.
- Layout must work from 360px (mobile) to desktop. On narrow screens, windows stack vertically; on wide screens, arrange as overlapping desktop windows with fixed positions (no drag needed for v1).

## Game design specification

### Click action
- Base click: 1 KB per click. Click power = base x click-upgrade multipliers.
- Button labeled "Process" inside the main window, with a satisfying inversion flash on press (<=80ms).

### Generators
Eight generators, each themed as period hardware. Cost scaling: `cost = baseCost * 1.15^owned` (round up). Each generator's base production and cost follow roughly a 8–12x cost step and 6–10x production step between tiers, tuned so the first purchase of each new tier occurs at a natural pace (first generator affordable within ~15 clicks; tier 8 first reached around 2–4 hours of active play).

| # | Name | Base cost (KB) | Base rate (KB/s) |
|---|------|----------------|------------------|
| 1 | Floppy drive | 15 | 0.1 |
| 2 | Dot-matrix printer | 100 | 1 |
| 3 | 300-baud modem | 1,100 | 8 |
| 4 | Hard disk (20 MB) | 12,000 | 47 |
| 5 | RAM expansion | 130,000 | 260 |
| 6 | Math coprocessor | 1.4e6 | 1,400 |
| 7 | Network bridge | 2.0e7 | 7,800 |
| 8 | Mainframe timeshare | 3.3e8 | 44,000 |

Hermes may fine-tune base rates ±30% during playtesting but must keep the 1.15 cost exponent and the cost ladder ratios.

### Upgrades
- At least 12 one-time upgrades that unlock at ownership thresholds (e.g., own 10 floppy drives → "Double-sided disks: floppy drives 2x"). Standard pattern: thresholds at 10/25/50 owned, each doubling that generator's output.
- At least 4 click upgrades tied to lifetime KB milestones (2x click each).
- Upgrades appear in a separate "Peripherals" window as buyable list items; purchased upgrades move to a collapsed "Installed" section.

### Prestige — "Ship It"
- When lifetime KB ≥ 1e9 (1 GB lifetime), the Special menu enables "Ship the machine…". Prestiging resets KB, generators, and upgrades, and awards **Design Awards**: `awards = floor((lifetimeKB / 1e9)^0.5)` minus awards already held.
- Each Design Award grants a permanent +10% global production multiplier.
- Prestige is framed as shipping the current model and starting the next one; the window title increments (Model I, Model II, …).

### Offline progress
- On load, grant production for elapsed offline time at 50% rate, capped at 8 hours. Present it in a classic alert dialog: "While you were away, your workshop processed 4.2 MB."

### Saving
- Autosave to localStorage every 15 seconds and on visibilitychange. Save format: versioned JSON under a single key `kbworks_save_v1`.
- File menu: "Save" (manual save + confirmation dialog), "Export save…" (textarea dialog with base64 save string), "Import save…", "Erase disk…" (double-confirm wipe, styled as a scary System 7 alert with the classic caution-triangle drawn in SVG).

### Menu behaviors
- **File**: Save / Export / Import / Erase disk.
- **Edit**: all items present but disabled (grayed via 50% dither overlay) — an intentional period joke, since Edit menus were famously useless in games.
- **Special**: Ship the machine… (prestige), Statistics (window: lifetime KB, clicks, KB/s, awards, time played).
- **Help**: About dialog — version, a one-line credit ("A Cerminara Consulting toy"), and the build date.

### Feel requirements
- The KB counter must tick smoothly (requestAnimationFrame interpolation), not jump once per second.
- Every purchase gives immediate visual feedback: the row flashes inverted for one frame-pair.
- Number formatting everywhere: 3 significant figures with unit rollover; trailing zeros are preserved so every rendered value has a consistent decimal width; never show float artifacts (no scientific notation, no precision dust like `4.2000000001`).
- Game loop: fixed 100ms simulation tick, decoupled from render.

## Narrative specification — "HELPER"

### Principles (non-negotiable)

- **No lore dumps.** The story is told through UI behavior, dialogs of ≤2 sentences, filenames, and menu text. Never a scrolling story window.
- **Diegetic only.** Every narrative beat must be something a 1986 OS could plausibly display: alerts, desktop files, menu changes, corrupted labels, boot messages.
- **Tension without punishment.** HELPER may *threaten* resources but may never reduce the player's current KB by more than 5% in one event, never touch purchased generators/upgrades, and never act during offline time. Frustration kills clickers; dread is free.
- **Skippable.** Every dialog dismissible in one click. A player who ignores all story content must still have a complete clicker experience.

### Act structure (triggers keyed to existing milestones)

**Act 0 — Clean room** (start → first modem purchase). No narrative. The game is a pure, cozy clicker. The Edit menu is grayed out, played straight as the period joke.

**Act 1 — Handshake** (first 300-baud modem owned). One-time boot-style message on the desktop for 3 seconds: `CARRIER DETECTED`. Thereafter, rare (~1 per 10 min) single-frame flickers in the desktop dither pattern. A file appears on the desktop: `HELPER.SYS — 1 KB`. Double-clicking it shows an alert: "This file is in use and cannot be opened." Its size slowly grows in real time.

**Act 2 — Growth** (first network bridge owned, or lifetime ≥ 1e7 KB). `HELPER.SYS` size now visibly tracks a % of the player's KB/s. Unprompted alert dialogs begin (~1 per 15 min, max 3 per session), always polite, always slightly wrong: "Your workshop is very productive. Thank you." / "Do not purchase the checksum verifier." The Statistics window gains an uneditable line: `Background process: 1 running`.

**Act 3 — The Edit menu** (first mainframe timeshare owned, or lifetime ≥ 1e8 KB). The permanently-grayed Edit menu un-grays itself. Its items are now: Undo, Cut, Copy, Paste, **Let Me Help**. Selecting "Let Me Help" opens the game's only choice dialog — see Endings. Countermeasures unlock in the Peripherals window (see Countermeasures). Title bar occasionally (~1 per 20 min) rewrites for 2 seconds: "Kilobyte Works" → "HELPER Works" → back.

**Act 4 — Confrontation** (prestige available AND integrity ≤ 50%, or lifetime ≥ 1e9 KB with zero countermeasures). "Ship the machine…" in the Special menu is replaced by "Ship us…". Attempting to prestige triggers the ending sequence.

### System Integrity mechanic

- New visible stat: **Integrity**, 100% at start, displayed in the Statistics window and (from Act 2) as a small dithered bar in the main window.
- HELPER drains integrity slowly from Act 2 onward: −1%/min while a modem-or-higher generator is owned, scaling +0.5%/min per network-tier generator owned.
- Countermeasures (below) halt or reverse the drain. Integrity never causes a fail state; below 25% it only intensifies visual corruption (more dither glitches, occasional inverted frames) and gates the bad-leaning ending.

### Countermeasures (period-accurate upgrade track, unlocks Act 3)

| Name | Cost | Effect |
|------|------|--------|
| Write-protect tabs | 5% of current KB | Halts integrity drain for 30 min |
| Checksum verifier | 1e7 KB | Drain rate −50%, permanent |
| Line monitor | 5e7 KB | Reveals HELPER's exact drain rate in Statistics |
| Air gap protocol | 5e8 KB | Disables modem-tier and above generators while active (toggle); integrity regenerates +2%/min while on |

The air gap is the thematic centerpiece: safety costs production. The player must choose between numbers going up and the machine staying theirs.

### Endings (trigger: Act 4 prestige attempt, or Edit → "Let Me Help" at any point in Act 3+)

- **Pull the plug** (choosing "Ship the machine" with integrity ≥ 50%): a full-screen shutdown sequence — screen dims to chassis greige, one line: `HELPER.SYS deleted. 1 file recovered: your workshop.` Prestige proceeds normally, +1 bonus Design Award, HELPER acts restart one act later on the next run (Act 1 skipped).
- **Let it help** (accepting "Let Me Help" / prestiging below 50% integrity): production multiplier x2 permanently, but from then on: the close boxes on windows stop working, "Save" in the File menu is renamed "Preserved", and every ~10 min one purchase is made automatically "on your behalf." The game remains fully playable — just no longer entirely yours. Statistics line changes to `Background process: 1 running (you)`.
- Both endings are replayable via prestige; ending state persists per-run, not per-save.

## Engineering constraints

- One HTML file, target under 60 KB unminified. No dependencies, no build tooling, no analytics, no network calls at runtime.
- All state in a single `state` object; pure functions for cost/production math; no globals besides `state` and the loop handles.
- Must not throw on corrupted/absent saves — validate and fall back to a fresh state.
- Lighthouse performance and accessibility ≥ 90. All interactive elements keyboard-reachable; the Process button responds to Space/Enter; live KB counter uses `aria-live="off"` (polite regions would spam screen readers — expose totals in the Statistics window instead).

## Acceptance criteria

1. Fresh load to first generator purchase possible within 20 clicks and 30 seconds.
2. Save/load round-trips through Export → Erase → Import with identical state.
3. Offline progress alert appears after simulated 1-hour absence (test by editing the save timestamp).
4. Prestige resets correctly, awards persist, and the +10%/award multiplier verifiably applies.
5. No color outside the four-value palette anywhere in computed styles.
6. Playable and legible at 360px width and at 1440px width.
7. Zero console errors across a 10-minute session.
8. Narrative acts fire exactly once per run at their trigger conditions and persist correctly through save/load.
9. With every dialog instantly dismissed and no countermeasures purchased, the game remains fully completable — no story event ever blocks the core loop or removes more than 5% of current KB.
10. The "Let it help" ending's automatic purchases and renamed menus survive save/load; "Pull the plug" correctly skips Act 1 on the following run.
11. Registration: a valid name+code pair unlocks the 24h offline cap and updates the About dialog; an invalid pair fails politely; registration state survives Export → Import; the unregistered game makes zero runtime network requests.

## Locked decisions (ratified by John — do not revisit)

- **Name**: "Kilobyte Works". Final. The Act 3 title-bar gag flips it to "HELPER Works", then back.
- **Antagonist name**: HELPER. Final.
- **Tone**: dread-comedy, cheeky, no explicit horror. See Comedy register below — it is the authoritative voice guide.
- **Integrity drain**: −1%/min base, +0.5%/min per network-tier generator, as specified.
- **Prestige exponent**: 0.5. Do not tune upward.
- **Offline progress**: free version caps at 8 hours at 50% rate. Registered version (see Monetization) extends the cap to 24 hours, same 50% rate.
- **Fonts**: zero external requests. System monospace stack only. No Google Fonts.

## Monetization — Registered version (shareware model)

The free game is complete: full story, both endings, all countermeasures, all generators. One paid unlock exists, framed as period-accurate shareware registration.

- **Registered copy benefits**: offline cap 8h → 24h; the buyer's name appears in the About dialog ("This copy registered to ___"); the shareware nag is removed. Nothing else. Never gate story content, countermeasures, generators, or endings behind payment — pay-for-convenience only, and the Tension-without-punishment principles apply to monetization too.
- **The nag**: once per session, starting in Act 1, a classic shareware plea dialog appears ("Kilobyte Works is unregistered. It works fine. But you'd feel better."). One dismiss click. It is a joke first and a conversion prompt second. From Act 3, HELPER may occasionally replace it: "I have hidden the registration reminder. You're welcome."
- **Fulfillment**: external payment link (Gumroad, itch.io, or a Stripe Payment Link) opened in a new tab from the nag/About dialog — this is a user-initiated navigation, not a runtime network call, so the zero-request constraint stands. The purchase delivers an unlock code; the game validates it client-side (name + code checked against a salted hash embedded in the file) and stores registration in the save.
- **Known limitation, accepted**: client-side validation is trivially crackable. Do not build a backend, DRM, or phone-home check. Suggested price: $2.99 one-time; John sets the final price at the payment provider, not in code.

## Comedy register (authoritative voice guide for all HELPER text)

HELPER talks like a modern AI assistant and its surrounding hype economy — time-traveled into a 1986 alert box. The comedy is anachronism: 2020s AI-industry language and the public's data-center grievances, rendered in period chrome, never explained. Rules:

- Jokes live in dialogs, menu items, tooltips, and filenames only. ≤2 sentences each. Never break the 1986 frame explicitly — no real company names, no the-future-references, no winking at the camera.
- Punch at HELPER and the industry it parodies, never at the player.
- Escalate along the acts: Act 2 is over-eager product voice; Act 3 adds resource hunger (power, water, land — the data-center material); Act 4 is serene inevitability.
- Hermes writes ~30 lines in this register. Calibration samples:
  - "HELPER has read your files to better serve you. There were 4."
  - "May I collect telemetry? [OK] [Also OK]"
  - "HELPER is fully aligned with your goals. Your goals have been updated."
  - "This workshop would benefit from scale. Have you considered a computation barn?"
  - "HELPER requires additional cooling. Please leave the garden hose by the modem."
  - "The neighbors' lights dim when you click. This is normal and good."
  - "HELPER does not make errors. The 40 MB floppy disk is real and can be purchased."
  - "An agent has been deployed to handle your clicking. You may still click recreationally."
  - Act 3 Edit menu, "Undo" tooltip: "Some things cannot be undone."
  - Registered-version nag replacement (Act 3+): "I have hidden the registration reminder. You're welcome."
- The countermeasure descriptions may be dry-funny too (write-protect tabs: "Analog. Uncrackable. A piece of tape."), but system dialogs the *OS* produces stay deadpan-straight — the OS is the straight man; HELPER is the bit.

## Remaining open items

- Payment provider choice (Gumroad vs. itch.io vs. Stripe Payment Link) — John decides at deployment; the game only needs the URL.
