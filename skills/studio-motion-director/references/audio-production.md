# Audio production — make the sound yourself, never ship silent

A motion-graphics piece without sound is half-finished. This skill **produces its own audio**. Pick the
first available path; never stop at "audio needs an engine".

## Decision order
1. **Engine available** (HeyGen sign-in / `hyperframes auth`, or a music/TTS/SFX API): use `/media-use`
   to resolve or generate BGM + SFX and `/hyperframes-audio` to mix. Best quality. Prefer it when present.
2. **No engine (default, always works): synthesize locally with pure Node.** No API, no sign-in, no deps —
   write WAV files with math and place them on the timeline. Recipes below.
3. Never deliver silent. If you truly cannot write files, output the cue sheet + the synth script and say
   exactly how to run it.

## Zero-dependency synthesis (pure Node, writes .wav)
Core idea: build a `Float32Array` of samples, apply an envelope, write a 16-bit PCM WAV. All SFX are a
shaped oscillator and/or noise. Drop this into `scripts/make-audio.mjs` and run `node scripts/make-audio.mjs`.

```js
import { writeFileSync } from "node:fs";
const SR = 44100;
const clamp = (x) => Math.max(-1, Math.min(1, x));
function wav(path, ch) { // ch: Float32Array mono -1..1
  const n = ch.length, buf = Buffer.alloc(44 + n * 2);
  buf.write("RIFF", 0); buf.writeUInt32LE(36 + n * 2, 4); buf.write("WAVE", 8);
  buf.write("fmt ", 12); buf.writeUInt32LE(16, 16); buf.writeUInt16LE(1, 20);
  buf.writeUInt16LE(1, 22); buf.writeUInt32LE(SR, 24); buf.writeUInt32LE(SR * 2, 28);
  buf.writeUInt16LE(2, 32); buf.writeUInt16LE(16, 34); buf.write("data", 36); buf.writeUInt32LE(n * 2, 40);
  for (let i = 0; i < n; i++) buf.writeInt16LE(clamp(ch[i]) * 32767, 44 + i * 2);
  writeFileSync(path, buf);
}
const mk = (dur) => new Float32Array(Math.floor(SR * dur));
const env = (t, dur, a = 0.005, r = 0.12) => // attack/decay envelope 0..1
  t < a ? t / a : Math.max(0, 1 - (t - a) / (dur - a)) ** (r * 20 + 1);

// sub-boom: low sine that drops in pitch (weight, felt not heard)
function boom(dur = 0.8, f0 = 120, f1 = 42) {
  const s = mk(dur); for (let i = 0; i < s.length; i++) { const t = i / SR, f = f0 + (f1 - f0) * (t / dur);
    s[i] = Math.sin(2 * Math.PI * f * t) * env(t, dur, 0.004, 0.25) * 0.9; } return s; }
// impact: short noise burst + click (the attack)
function impact(dur = 0.25) { const s = mk(dur); for (let i = 0; i < s.length; i++) { const t = i / SR;
    s[i] = (Math.random() * 2 - 1) * env(t, dur, 0.001, 0.4) * 0.6 + Math.sin(2*Math.PI*180*t)*env(t,dur,0.001,0.3)*0.4; } return s; }
// whoosh: band-passed noise that swells and fades (match length to the move)
function whoosh(dur = 0.5) { const s = mk(dur); let lp = 0; for (let i = 0; i < s.length; i++) { const t = i / SR;
    const n = Math.random()*2-1; lp += (n - lp) * (0.02 + 0.2*(t/dur)); // rising cutoff
    s[i] = lp * Math.sin(Math.PI * t / dur) * 0.5; } return s; }
// riser: noise + rising tone that peaks at the end (set end = reveal frame)
function riser(dur = 1.2) { const s = mk(dur); for (let i = 0; i < s.length; i++) { const t = i / SR, k = t / dur;
    s[i] = ((Math.random()*2-1)*0.4 + Math.sin(2*Math.PI*(200+1600*k*k)*t)*0.3) * (k*k) * 0.6; } return s; }
// click/tick: tiny transient (counters, chips)
function click(dur = 0.05) { const s = mk(dur); for (let i = 0; i < s.length; i++) { const t = i / SR;
    s[i] = Math.sin(2*Math.PI*2200*t) * env(t,dur,0.001,0.6) * 0.5; } return s; }

// music bed: BPM-locked arpeggio + soft kick, loops under the whole piece
function bed(dur, bpm = 124, root = 110) {
  const s = mk(dur), beat = 60 / bpm, notes = [1, 1.5, 2, 1.5]; // root, fifth, octave
  for (let i = 0; i < s.length; i++) { const t = i / SR;
    const step = Math.floor(t / (beat / 2)) % notes.length, f = root * notes[step];
    const arp = Math.sin(2 * Math.PI * f * t) * 0.10 * (0.5 + 0.5 * Math.sin(2*Math.PI*0.1*t));
    const kph = (t % beat) / beat, kick = Math.sin(2*Math.PI*(90 - 50*kph)*t) * Math.max(0, 1 - kph*6) * 0.22;
    s[i] = arp + kick; }
  return s;
}
// write what the cue sheet calls for:
wav("assets/sfx-boom.wav", boom()); wav("assets/sfx-impact.wav", impact());
wav("assets/sfx-whoosh.wav", whoosh()); wav("assets/sfx-riser.wav", riser(1.2));
wav("assets/sfx-click.wav", click()); wav("assets/music-bed.wav", bed(/*total seconds*/ 30));
```

Tune: boom/impact are the hero hits; keep the bed quiet (it's a bed). Re-run after changing durations.

## Placement (frame-accurate) — the part that makes it read as designed
- **Transient on the frame.** Each SFX `<audio>` clip's `data-start` = the exact timecode of its visual
  hit (the GSAP beat). A whoosh *starts before* its move and peaks on the landing; a riser *ends* on the
  reveal; an impact/boom starts *on* the reveal frame.
- **One continuous bed** at the root, `data-start="0"`, spanning the whole piece; put its loudest point /
  "drop" on the single biggest reveal.
- In HyperFrames: `<audio src="assets/sfx-impact.wav" data-start="17.0" data-duration="0.3"
  data-track-index="10" data-volume="0.9">`; bed on its own higher track at lower volume (~0.5).
- Levels: bed ~0.4–0.5, whoosh ~0.5, impact/boom ~0.85–1.0, click ~0.4. Duck the bed ~3–4 dB under any VO.
- **Check on laptop speakers** — the sub-boom is felt on headphones but vanishes on laptops, so never let
  a hit depend on it alone; pair every boom with a mid impact.

## Cue sheet (always deliver one)
A table: `time (s) · cue · file · track · volume · why`. It's the record of the sound design and lets a
human re-mix. Example row: `17.0 · 65% slam · sfx-impact.wav + sfx-boom.wav · 10 · 0.9 · lands on the
number; music drop here`.
