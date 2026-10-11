# ACCEPTANCE — epee-game vs SPEC §4.3

**Reviewer:** Review & QA  
**Date:** 2026-10-05 (HKT)  
**Canonical:** `/workspace/epee-game/epee.html` (34 242 bytes ≈ 34 KB)  
**Also:** `/workspace/epee-game/index.html` — **byte-identical** (`diff` empty)  
**SPEC:** `/workspace/epee-game/SPEC.md` §4.3 (lines 179–193) — not modified  

---

## 【結論】

**通過 (PASS)**

All 13 §4.3 criteria PASS. No Blockers / Majors. Remaining gaps are Minor polish already documented by Indie in `SMOKE.md` (or explicitly allowed by SPEC, e.g. no Web Audio). Required gameplay behavior is intact.

---

## 【§4.3 清單】

| # | Criterion | Verdict | Evidence |
|---|-----------|---------|----------|
| 1 | Open HTML standalone; no network / no 404 | **PASS** | Single-file inline CSS+JS; canvas `960×540`. Headless Chrome `file:///…/epee.html` dumps full DOM (canvas present). Screenshot `/tmp/epee_menu.png` renders menu+venue. Zero external loads. |
| 2 | Main menu: Regular / Survival; Beginner / Normal / Expert | **PASS** | `buildMenuButtons`:「正規局 · 先到 5 分」「生存模式」. `buildDiffButtons`:「初學 Beginner」「普通 Normal」「高手 Expert」+ back. Screenshot confirms menu. |
| 3 | Regular: first to 5 or 3:00 settle; Sudden Death on tie | **PASS** | `TARGET_SCORE=5`, `BOUT_TIME=180`. Time expire with equal scores → `G.suddenDeath=true` + reset. Decisive point ends bout; double in SD continues (`sudden_double_continue` sim). Autoplay reached `win` 5–1. |
| 4 | Survival: AI lead ≥3 ends; shows survival score | **PASS** | `SURVIVAL_LEAD=3`; `checkEndConditions` → `survive_lose`. `drawResult` line:「生存得分（玩家）：」+ tier. Sim: score 1–4 → `survive_lose`. |
| 5 | Keys: A/D or arrows; J/Z light, K/X heavy, L/C feint, S/; parry | **PASS** | `updatePlayerInput`: `a`/`arrowleft`, `d`/`arrowright`, `j`/`z`, `k`/`x`, `l`/`c`, `s`/`;`/`semicolon`. HUD help string lists same bindings. |
| 6 | Light fast/short; heavy slow/long; feint no score | **PASS** | `MOVE.light` 120 ms / tipReach 42 / tipR 6; `heavy` 220 ms / 68 / 9; `feint.scores===false` (combat only scores when `MOVE[move].scores`). |
| 7 | Double within lockout both +1; HUD lockout note | **PASS** | `LOCKOUT_MS=120`; `flushHits` double path both `++`, cue `DOUBLE`,「雙中！」. HUD:「雙中鎖時 120 ms（已遊戲化）」at bottom. |
| 8 | After score → en garde + short prêts/allez | **PASS** | `resetToEnGarde(true)` → phase `reset` 800 ms → `pret` 400 → `allez` 400 → `fight`. Positions snap/tween to `ENGARDE_L/R` (±180 from piste CX 480 → 300/660). Announce「預備？」「開始！」+ `playCue('ALLEZ')`. |
| 9 | Low stamina slows move + blocks heavy; distance bar affects reach | **PASS** | Stamina `<20`: move ×0.6, heavy blocked (`startAttack`); `20–39`: ×0.8, duration ×1.15. Dist `<45` move ×0.85. Tip reach from `MOVE.*.tipReach`; HUD `drawDistanceBar` 遠/中/近 bands at 45/90. |
| 10 | Kai Tak indoor venue recognizable | **PASS** | `drawVenue`: arc roof (`quadraticCurveTo`), truss lines, `drawStands`, big screen「KAI TAK MAIN VENUE」, ads PEAK BLADE / VICTORIA BAY / KOWLOON STEEL / HONG KONG FENCE, sides `HKG` + `Hong Kong, China`. Confirmed in headless screenshot. |
| 11 | Top bar: scores, time, combo/perfect | **PASS** | `drawHUD` + venue screen: scores, mode, difficulty, timer (`M:SS` or survival elapsed), `Combo ×N`,「完美刺中：N」, stamina bars. |
| 12 | Beginner vs Expert distinguishable in params | **PASS** | Beginner `reactionMs=380`, `feintRate=0.08`, interval `[900,1200]`, parry 25%, move 0.85. Expert `140`, `0.40`, `[320,520]`, parry 70%, move 1.1. Matches SPEC §1.5. |
| 13 | No external resource requests | **PASS** | `rg` no `http://`/`https://`; no `src=`/`href=` external; no CDN/fetch. System font stack only. |

---

## 【問題清單】

### Blockers
*None.*

### Major
*None.*

### Minor (polish; do not fail acceptance)
1. **Timer HUD offset** — regular/survival time drawn at `(W/2+160, 70)` beside scoreboard rather than tightly adjacent (Indie SMOKE). Still visible; SPEC “分數旁” met loosely.
2. **Stamina duration scaling uses post-cost stamina** — `startAttack` deducts cost then scales duration from remaining (`<20` / `<40`). Strike that crosses a threshold is slightly over-penalized (Indie SMOKE). Move slow + heavy block still correct.
3. **Parry tip+near-blade helper** — `tipNearBlade` (28 px) supplements tip-vs-body for game-feel; not a separate blade mesh (Indie SMOKE / SPEC allows game-feel).
4. **No Web Audio** — `playCue` → `console.log` only. **SPEC §3.3 explicitly allows** for v1.
5. **Fencer state debug tags omitted from draw** — actions still mutually exclusive via `canAct` → idle-only (`Indie SMOKE`).
6. **Dead key alias** — `anyKey(...,'semicolon')` never matches (`e.code` stored as `Semicolon`); `;` and `s` still work. Cosmetic.

---

## 【已知缺口】vs Indie `SMOKE.md`

| Indie gap | Review verdict |
|-----------|----------------|
| Timer HUD offset | **Confirm** — Minor only |
| Stamina post-cost duration scaling | **Confirm** — Minor; required slow/block still present |
| Auto-play test hook coarse | **Confirm** — test aid only; human vs AI is real play |
| Parry tip+near-blade helper | **Confirm** — Minor feel choice; required parry behavior works |
| No Web Audio / `playCue`+console | **Confirm** — SPEC-allowed |
| State debug tags omitted | **Confirm** — mutual exclusion intact (`canAct` idle-only; attacking/parrying/recovering/stunned/frozen) |

No disputed gaps; none elevate to Major/Blocker.

---

## English bullets — code findings

- `epee.html` ≡ `index.html` (34 242 B); standalone Canvas 960×540; `node --check` on extracted `<script>` OK.
- Constants match SPEC §1.2–1.5: `LOCKOUT_MS=120`, en garde ±180, light/heavy/feint/parry timings & reaches, DIFF table exact, `STAMINA_REGEN=12`, retreat ×1.15.
- State machine: `canAct` = idle only; attacks/parry mutually exclusive; recovery durations light 80 / heavy 140 / feint 60.
- Scoring: pendingHits + lockout flush → single or double; Sudden Death on 3:00 tie; doubles in SD continue; first-to-5 / survival lead-3 end paths verified by stubbed `__EPEE__` simulation.
- Post-touch: freeze → `resetToEnGarde(true)` → reset 0.8 s → short prêts/allez 0.4 s each.
- Venue draw includes all required Kai Tak identifiers; screenshot visually confirms.
- Survival starts from Normal base params (`freshAiParams(..., true)`); selected difficulty supplies ladder deltas (Expert milder), per SPEC.
- `playCue('HIT'|'DOUBLE'|'PARRY'|'ALLEZ')` present; no external URLs.

---

## Verification commands (repro)

```text
diff epee.html index.html          # empty
rg -n 'https?://' epee.html        # no matches
node --check <extracted script>    # OK
google-chrome --headless --dump-dom file:///workspace/epee-game/epee.html
# + __EPEE__ stub sim: survival_lead3, time_tie_sudden, sudden_win, first_to_5, engarde_reset, autoplay_regular
```

**Verdict: 通過 (PASS) — blocker count 0.**
