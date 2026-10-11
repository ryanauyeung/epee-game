# Smoke checks — epee-game

Date: 2026-10-05 (HKT)

## Files
- Canonical: `/workspace/epee-game/epee.html` (~34 KB)
- Also: `/workspace/epee-game/index.html` (identical copy)
- `SPEC.md` left untouched

## Checks performed
1. **Standalone**: single HTML, inline CSS+JS; `rg` found no `http://` or `https://` external URLs.
2. **Syntax**: extracted `<script>` → `node --check` OK.
3. **Headless chrome**: `google-chrome --headless --dump-dom file:///.../epee.html` loaded DOM (canvas 960×540 present). DBus noise only; page rendered.
4. **Programmatic bout** (stubbed canvas/DOM + `__EPEE__` hook):
   - Regular / Beginner autoplay+forced proximity → reached result `win` 5–4 in ~10s sim time; cues HIT/DOUBLE/ALLEZ logged; no throw.
   - Constants: `LOCKOUT_MS=120`, moves light/heavy/feint/parry with SPEC durations/reaches, DIFF beginner/normal/expert present.
5. **Survival**: force AI lead ≥3 → `survive_lose` result with survival scores shown path OK.

## Known polish gaps vs SPEC (honest)
- Timer HUD sits offset beside scoreboard rather than tightly beside it; still visible.
- Stamina attack-duration scaling uses post-cost stamina (may slightly over-penalize the strike that crossed a threshold).
- Auto-play test hook is coarse (not a polished AI-vs-AI demo); real play is human vs AI.
- Parry success uses tip-vs-body plus a generous tip-near-blade helper (game-feel), not a separate blade mesh.
- No Web Audio yet (by SPEC); `playCue` + `console.log` only.
- Fencer “state” debug tags omitted from draw (cleaner); actions still mutually exclusive in the state machine.
