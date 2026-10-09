# Motion grammar — the craft, with numbers

**The one law: movement is cheap; timing is the craft.** Judge beats by easing, holds, and whether sound
lands on the exact frame of the hit — never by how much moves.

## Timing & pacing (30fps)
- **Front-load entrances:** most of the distance early, then settle; exits reverse.
- **Hold** after every reveal — the still moment where the eye reads. Copy still moving into the next cut = amateur.
- **Stagger in frames, not pixels — 2–5 frames.** The eye reads time over distance.
- **Anticipation / overshoot / settle** ≈ 2–3f / 1f / 4–6f around a key action.
- **One hero move per beat.** Everything else supports and is offset.
- Beat lengths to TEST: micro-response 4–10f · short enter/exit 10–18f · major transform 18–35f · readability hold 18–60f.

## Easing
- Never linear; default "Easy Ease" is a start, not a finish.
- **Overshoot** snaps/scales to ~105% then settle (value graph for overshoot; speed graph for pacing).
- Ease strength by role (graph influence): ~33% background · 60–75% polished UI · 85–100% kinetic type.
- Consistency per element type = intention. Readability beats flash for copy.

## Staging, type, color
- Decide the eye-path per frame; keep ~40% quiet ground; one dominant focal point.
- **The typeface leads** kinetic type (font → tracking → size → color → ground). Two weights + one accent treatment.
- 2–3 inks held; depth from a value range + one accent, not many hues. Headline legible as a phone thumbnail.

## Secondary motion, arcs, smear
- Offset related elements by a few frames (numbers → icons → highlights); trailing bits lag and settle after the hero.
- Visible travel moves on arcs, not straight lines; secondary motion stays quieter than the hero.
- **Smear/snap** for speed: hold a pose, then break across the gap with a 1–2f stretch/blur along the travel; keep silhouette readable; use sparingly and consistently.

## Camera, depth, texture
- Motion blur is a tell; without it, compensate with shorter travel + tighter timing, or fake it (smear / `filter: blur()`).
- Depth via layered parallax (move + scale layers at different rates); rack-focus by blurring background; match blur softness to grain.
- Grain ~50% opacity, added **last**, masked off text; never scale a grain overlay up.

## Transitions
- **Hard cut** on the beat for impact (the default for fast, confident pieces).
- **Cut on action** — continue the same motion into a new angle.
- **Match cut** — carry a shared element across the cut; cut at its **fastest movement**.
- **Morph vs faked morph** — true morph changes path geometry (logos); most "morphs" are a cut hidden in fast motion (match end/start positions).
- **Dynamic/infinite zoom** — push through a sub-frame to reveal the next scene.
- **Dissolve** — reserve for time passing / montage; rarely right in a punchy launch.

## UI / product-in-motion (Apple HIG register)
- Motion conveys status, feedback, continuity — never decoration.
- Spring, not just ease: critically damped by default; bounce only when a gesture carried momentum.
- UI feels alive when motion starts from the current value, inherits velocity, and is interruptible/reversible.
- Show real frames with depth + legible type; reveal one feature at a time; respect reduced-motion.

## Sound design (half the craft)
- Build bottom-up: sub-boom (weight) → mid impact (attack) → top whoosh (movement). Stack two takes for width.
- **Transient on the exact frame** of the reveal; a logo's swell resolves on the frame the mark appears.
- Match a whoosh's length to the move; its peak sits at the landing. A riser that stops dead sets up a cut.
- Music: a bed with a clear build + drop; put the drop on your single biggest reveal. Check on laptop speakers (subs vanish there).
