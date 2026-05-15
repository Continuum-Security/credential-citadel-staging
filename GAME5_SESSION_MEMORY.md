# Game 5 — Credential Citadel — Session Memory
_Save this file. Load it in any future Claude session to resume exactly where we left off._

---

## Project Overview

**Game:** CyberIQ Game 5 — Credential Citadel
**Type:** Super Mario-style side-scrolling platformer teaching credential hygiene
**Stack:** Single self-contained HTML file, vanilla JS, HTML5 Canvas — NO framework
**Staging repo:** `Continuum-Security/credential-citadel-staging` (private, GitHub)
**CRITICAL isolation:** This repo is 100% separate from main CyberIQ repo. Denis's staging→prod syncs CANNOT pick it up. He would have to manually deploy from this separate repo.
**Supabase:** Bypassed for now — game has no backend dependencies yet. Will integrate later into main platform.

---

## Story / Design

- Hero's identity stolen by master breach-broker
- 5 worlds: Password Plains → MFA Mountains → Vault Valley → Privilege Peaks → Recovery Realm
- Power-up ladder mirrors credential maturity: Tiny → Strong (Mushroom) → Flame (Fire Flower) → Cape → Star → Crown
- Each world: 4 levels + boss fight
- Boss fights have knowledge-gate quiz at 50% health
- Visual: Phosphor palette — void-black `#020617`, cyan `#22d3ee`, amber `#fbbf24`

---

## Key Contacts

- **Denis** — lead developer, sends build assets
- **Nish** — your user (nish_claude1@continuum-sec.com on this account, nish.pillay@continuum-sec.com on main)

---

## Files

| File | Description |
|------|-------------|
| `GAME-5-CREDENTIAL-CITADEL.html` | Main game file (the one being fixed) |
| `GAME-5-V3-MARIO-VSLICE-MOCK.html` | V3 mock (reference for working physics) |
| `GAME5_FIXED.html` | **The fixed output file** (look in outputs folder) |

Both HTML files are in the uploads folder. The fixed file goes to outputs folder.

---

## Decisions Made (confirmed by Nish)

| Decision | Choice |
|----------|--------|
| Session goal | Fix & playable (physics bugs first) |
| Tile size | T=32 (scale up from T=16, match V3 feel) |
| Camera | Bi-directional — allow backtracking to revisit quiz boxes |
| World priority | Build all worlds one at a time, each must work perfectly |
| Level complete | Summary screen → advance to next level |
| Boss fight | Must work this session (The Reuser, Level 1-Castle) |
| Lives system | Classic Mario: powerup cushions hits, dying tiny costs 1 life |
| GitHub push | Auto-push after each fix session |
| Canvas size | 800×480 (matches Figma spec) |
| World unlock | All unlocked for testing (dev mode) |
| Coin style | Round/key-shaped (amber ellipse) |
| BGM | Procedural background music per world |
| Supabase | Bypassed for now |
| Other games? | All CyberIQ games are vanilla HTML/Canvas, iframed into Next.js platform |

---

## Root Cause of "falls off instantly" Bug

**File:** `GAME-5-CREDENTIAL-CITADEL.html`

**Bug 1 — Spawn inside ground:**
```js
// BROKEN in makePlayer(tx, ty):
y: ty*T   // spawn tile ty=H-2=13 → y=208. Player h=24, feet at 232. Ground top=224. 8px INSIDE ground.

// FIX:
y: ty*T - PH  // place feet AT top of tile ty, body above it
```

**Bug 2 — resolveY too strict:**
```js
// BROKEN condition in resolveY():
if (isSolid(tile) && ent.y+ent.h > ty*T && ent.y+ent.h <= ty*T+ent.vy+1)
// When vy=0 and entity already overlaps floor: 232 <= 224+0+1=225 → FALSE. Falls through.

// FIX — remove the velocity condition:
if (isSolid(tile) && ent.y+ent.h > ty*T)
```

**Bug 3 — Lives on retry:**
After game over, pressing R resets lives correctly in `loadLevel()` (line 382: `STATE.lives=3`). But after respawn, `player.y = cp.y` restores the checkpoint position which was stored with the same broken y-coordinate. Fix: store checkpoint as `{x: player.x, y: player.y}` AFTER makePlayer corrects y.

---

## All Bugs Found

| # | Severity | Bug | Fix |
|---|----------|-----|-----|
| 1 | CRITICAL | Player spawns 8px inside ground, falls through immediately | Fix makePlayer y: `ty*T - PH` |
| 2 | CRITICAL | resolveY velocity condition prevents floor snap | Remove `ent.y+ent.h <= ty*T+ent.vy+1` |
| 3 | HIGH | Lives system broken on retry (checkpoint stores bad y) | Store checkpoint from corrected player.y |
| 4 | HIGH | Jump can't reach platforms at T=16 scale | Scale T=16→32, tune JUMP_VY=-12 |
| 5 | MEDIUM | Camera left-locks (can't backtrack to quiz boxes) | Remove `Math.max(camera.x, ...)` left-lock |
| 6 | LOW | Worlds 2-5 locked | Set all `worldsUnlocked[]=true` |
| 7 | LOW | No background music | Add procedural BGM module |

---

## Physics Constants (for T=32 build)

```js
const T = 32;
const GRAVITY = 0.4;
const JUMP_VY = -12;        // was -9.5, needs to be bigger for T=32
const FRICTION = 0.78;
const MAX_SPEED = 5;
const RUN_SPEED = 9;
const ACCEL = 0.7;
const BOOST_FRAMES = 10;
const BOOST_PER_FRAME = -0.55;
const INVULN_FRAMES = 90;
const GRACE_FRAMES = 6;
// Player size
const PW = 22, PH = 36;   // player width/height in px
```

---

## Entity Sizes at T=32

| Entity | Original (T=16) | Fixed (T=32) |
|--------|-----------------|--------------|
| Player | w:12, h:24 | w:22, h:36 |
| Goomba/Spiny | w:14, h:14 | w:28, h:28 |
| Cheep | w:14, h:14 | w:26, h:22 |
| Boss | w:48, h:64 | w:56, h:72 |
| Mushroom | w:14, h:14 | w:28, h:28 |
| Fire Flower | w:12, h:14 | w:24, h:28 |
| Fireball | w:6, h:6 | w:10, h:10 |

---

## Colour Palette (Phosphor Doctrine)

```js
const C = {
  BG: '#020617',
  CYAN: '#22d3ee',
  CYAN_DIM: '#0e7490',
  AMBER: '#fbbf24',
  EMERALD: '#10b981',
  RED: '#ef4444',
  SLATE: '#94a3b8',
  WHITE: '#f8fafc',
  DARK: '#0f172a',
  BOSS: '#475569',
  GROUND: '#1e3a5f',
  PLATFORM: '#164e63',
  CASTLE: '#1e293b',
  LAVA: '#dc2626',
  WATER: '#0e7490',
};
```

---

## Level Structure

All 5 World 1 levels are FULLY SPECIFIED in the HTML. Worlds 2-5 are scaffold levels (basic platforms, enemies, quiz). Level sequence:

1. **1-1 Plain Beginnings** — flat, 4 quiz boxes, 3 goombas, hidden alcove
2. **1-2 Spiny Crossing** — un-stompable spinies, Fire Flower pocket, 2 quiz boxes
3. **1-3 Brute-Force Brook** — water/bridges, Cheep-Cheeps, hidden underwater quiz
4. **1-4 Repeating Heights** — stepped platforms, all enemy types, 2 quiz boxes
5. **1-Castle The Reuser's Keep** — lava pit, boss The Reuser (8 HP, knowledge gate at 50%)

---

## Quiz System

- Mystery boxes trigger `showQuiz()` → sets `STATE.screen='quiz'`
- Player answers via keyboard 1/2/3/4 or clicking option buttons
- Correct answer → `deliverBoxReward()` (mushroom/fireflower/coins/heart)
- Wrong answer → box resets, player can retry
- Boss knowledge gate: wrong answer → boss heals to full, enters Phase 2 (faster throwing)
- After answering: Continue button (click or Space/Enter) dismisses quiz
- Timeout bar: 6 seconds auto-dismiss with no penalty

---

## Figma Design Reference

URL: `https://www.figma.com/design/2uIrbjaQ3adoNstmo87QTr/CyberIQ-Game-5-%E2%80%94-Mario-Reframe-%E2%80%94-Vertical-Slice-Design-Review`

**HUD (Section 5):** 800×120px strip
- Left: Heart lives + IQ score
- Centre: World label + power-up state badge (BASE/STRONG-PASSWORD/FLAME·UNIQUE-PW etc.)
- Right: Coin count (larger) + Score (smaller)
- Typography: Inter Bold 11px headers, Inter Bold 9px badge

**Canvas:** 800×480px (confirmed from Figma)

**Tile:** 32×32px logical (Phosphor pixel art)

---

## BGM Themes (procedural Web Audio)

| World/Theme | Key | Tempo | Feel |
|-------------|-----|-------|------|
| plains | C major | 140 bpm | Bright, upbeat |
| cave | A minor | 110 bpm | Tense |
| water | G major | 120 bpm | Flowing |
| castle | D minor | 100 bpm | Ominous |
| boss | Chromatic | 130 bpm | Urgent |

---

## GitHub Setup

- **Repo:** `https://github.com/Continuum-Security/credential-citadel-staging`
- **Branch:** `main`
- **File to push:** `GAME-5-CREDENTIAL-CITADEL.html` (or `index.html` in root)
- **Push method:** GitHub web API via browser (no CLI access in sandbox)
- **Isolation:** Completely separate from `Continuum-Security/cyberiq` (or whatever main repo is named). Denis's pipeline never touches this repo.

---

## Previous UAT Findings (Nish played V2/V3)

- Genre reframe works — reads as CyberIQ not Mario ✓
- First power-up (Mushroom) visible and satisfying ✓
- Sprites need polish pass (too low-res currently) — later task
- Coins should be round not square — fixing now
- Quiz pause should stay open until "Continue" clicked — implemented in V3, carry forward
- Lives broken on retry — fixing now
- Jump clips through blocks — fixing with resolveY fix
- Reuse-Goomba meaning not clear on first encounter — add coachmark (already in Level 1-1 code)
- 1-4 needs 2 quiz boxes (had 1) — already fixed in CREDENTIAL-CITADEL.html
- 1-Castle needs gauntlet before boss — noted for later

---

## Supabase Note

You have 2 Supabase projects (free tier limit reached):
1. `nish.pillay@continuum-sec.com's Project` (personal default)
2. `cyber-iq-staging` (main CyberIQ staging)

Game 5 will need a NEW Supabase project when integrating into platform. Options:
- Pause personal project (reversible)
- Upgrade to paid ($25/mo removes 2-project cap)

**Not needed yet** — game has no backend during standalone build phase.

---

## Supabase API Change Warning

From May 30, 2026: New Supabase projects won't expose public schema tables to Data API by default. When Game 5 eventually gets integrated, any `CREATE TABLE` needs explicit GRANTs:
```sql
grant select on public.your_table to anon;
grant select, insert, update, delete on public.your_table to authenticated;
grant select, insert, update, delete on public.your_table to service_role;
alter table public.your_table enable row level security;
```

---

## How to Resume in a New Claude Session

1. Upload `GAME5_FIXED.html` (the fixed output file from this session)
2. Paste this memory file or upload it
3. Tell Claude: _"Continue building CyberIQ Game 5 from the session memory. The fixed file is the current version. Next task is [whatever is next]."_
4. Reference the Figma URL above for design decisions

---
_Last updated: Session 2 — Physics fixes + T=32 scale-up in progress_
