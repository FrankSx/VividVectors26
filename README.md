# VIVIDVECTORS26 
**Repo:** `FrankSx/VividVectors26` · 6 pieces · all single-file HTML · no build step, no dependencies beyond CDN fonts/Tailwind where noted
**Thesis of the collection (read this first):** every piece is a *parser polyglot* — the same bytes are simultaneously a webpage, a shader stage, an audio engine, and a covert channel. The resume is never "on" the page; it is *hidden in the parsing layers themselves*. The artwork and the steganography are the same object.

**Notation used below:** `::` binds a component to its role · `→` marks a transformation stage · `⊗` marks two systems coupled together · `⟲` marks a feedback/recursion loop · `∴` marks the conclusion a piece is arguing

---

## 0. THE SHARED TOOLKIT (master these five idioms and you can rebuild all six pieces)

### 0.1 The IQ cosine palette
Used verbatim in PsyPFE, QUANTUM_ECHO, and every mode of neural_dissonance. One function → infinite color cycles:
```glsl
vec3 palette(float t, vec3 a, vec3 b, vec3 c, vec3 d){
  return a + b * cos(6.28318 * (c * t + d));
}
// canonical args: a=b=vec3(0.5), c=vec3(1.0), d=vec3(0.3,0.2,0.5) — rotate d for hue shifts
```

### 0.2 The fullscreen-quad WebGL boilerplate
Every shader piece uses the identical pair of triangles, never a mesh:
```js
gl.bufferData(gl.ARRAY_BUFFER,
  new Float32Array([-1,-1, 1,-1, -1,1, -1,1, 1,-1, 1,1]), gl.STATIC_DRAW);
// vertex shader: gl_Position = vec4(position, 0.0, 1.0); — geometry is irrelevant, the fragment shader IS the artwork
```

### 0.3 The inverse-distance glow idiom
The "neon line from nothing" trick — PsyPFE uses it directly, endlessjpeg fakes it in 2D:
```glsl
d = sin(d * 8.0 + u_time * 2.0) / 8.0;   // turn distance into repeating rings
d = abs(d);                               // fold: rings become ridges
d = pow(0.01 / d, 1.2);                   // 1/d inversion: ridges become glowing filaments
finalColor += col * d;                    // accumulate over 3–4 iterations for density
```

### 0.4 The three smoothing constants
Every piece uses the same motion grammar:
- mouse easing: `mouse += (target - mouse) * 0.08`
- pulse/excitation decay: `pulse *= 0.92` (or 0.94)
- audio meter decay when silent: `level *= 0.9`

### 0.5 The steganographic ladder
The resume is encoded at increasing parser depth across the collection:
`plaintext HTML comment` → `base64 in SVG data-* attribute` → `base64 in JS constant` → `ASCII-shift cipher array` → `reversed base64` → `CSS custom properties (\A as newline)` → `XOR-obfuscated byte array` → `raw RGBA canvas pixels`. neural_dissonance stacks five of these simultaneously.

∴ **Collection-level conclusion:** the resume is a unit test for the parser forest. If you can find it, you understand the medium.

---

## 1. `endlessjpeg(bright-hypercolor).html` — ENDLESS JPEG

### What it represents
The simplest and most conceptually pure piece. It is a **live JPEG encoder wearing a screensaver's clothes**. The comment in the source says it outright: *"intentionally wasteful and beautiful."* Every frame is born as a lossless pixel buffer, dies in a JPEG encoder (`toDataURL`), and is reborn as an `<svg><image>` — three different parsers (Canvas2D → JPEG codec → SVG image renderer) cooperating to display one thing. The JPEG artifacts are not a bug; **generation loss is the aesthetic** — digital decay as a real-time medium rather than a archival accident.

### Effect inventory
- 2D plasma field from 5 stacked sine/cosine terms
- Deliberate color quantization (`r & 0xF8` etc.) → forces visible JPEG banding
- Subtle self-feedback smear (`globalAlpha 0.06` screen-blit, 0.5px offset)
- Continuous re-encode at quality 0.8, throttled to ~13fps encode (75ms)
- SVG `preserveAspectRatio="xMidYMid slice"` as the display surface
- HUD: FPS, frame count, live base64 size in KB
- Film grain (SVG `feTurbulence` data-URI, drifting) + vignette

### Replication recipe
```
canvas :: offscreen, 320–640px wide (never in DOM)
  → per-pixel plasma: v = Σ sin/cos(x·f ± t·s, dist·f, atan2·f); normalize ×0.2
  → color rotate: channel = 128 + 127·sin(v·π + t + phase), phases 0 / 2.094 / 4.188 (≈2π/3)
  → quantize: r&0xF8, g&0xFC, b&0xF8   // pre-seeds the artifacts
  → putImageData, then feedback ⟲: alpha 0.06, composite 'screen', drawImage(canvas, 0.5, 0, ...) 
jpegUrl = canvas.toDataURL('image/jpeg', 0.8)     → svgImage.setAttribute('href', jpegUrl)
                                                  → setAttributeNS(xlink, 'href', jpegUrl)  // Safari
throttle: skip frames until 75ms elapsed since last encode
```
The grain overlay is pure CSS: an inline SVG data-URI with `feTurbulence baseFrequency=0.9 numOctaves=3`, `mix-blend-mode: screen`, `opacity: 0.035`, animated `translate(10%,10%)`.

### Variations to try
Drop quality to 0.4 for brutal VHS macro-blocking; encode as `image/webp` and watch the artifacts change personality; feed the JPEG back into the plasma field for true generational recursion.

---

## 2. `franksx_endless_q3-26.html` — Q3-26 PORTFOLIO (Endless JPEG as identity site)

### What it represents
The quarterly drop format: piece #1 was the experiment, piece #2 is the **experiment turned into a home**. The generative JPEG stream becomes a dimmed background texture (`opacity-40`) behind a Tailwind portfolio, and the resume hides in **three parser layers at once**: base64 inside the background SVG's `data-resume` attribute, a glass-modal layer toggled by the user, and plaintext after `</html>`. The site *is* the thing it describes — a page about parser differentials that only fully parses if you know where to look. Named `endless_q3-26` ∴ the collection is explicitly seasonal — each quarter a new polyglot bloom.

### Effect inventory
- Background: orbiting additive radial gradients (`globalCompositeOperation='screen'`, 6 blobs, hueBase rotating 52° apart)
- Ghosting ⟲: translucent black fillRect each frame (`alpha 0.07`) — trails instead of full redraw
- Scanline shimmer: 1px rows every 3px, sinusoidal x-shift, hue-cycled, alpha 0.035
- Noise: 200 random 1px black specks/frame at alpha 0.04
- Re-encode at quality 0.72 every rAF (~60fps); SVG `feGaussianBlur stdDeviation=20` glow filter on the image
- HUD stats: FPS / FRAME / SIZE(KB) read live from the encode loop
- Resume modal: glass + CRT scanline overlay, ESC/backdrop/buttons to close

### Replication recipe
```
canvas :: 960×540 max, 16:9
  → fillRect('#020608', alpha 0.07)                    // decay buffer
  → for i in 0..5: x,y = Lissajous(t·0.00035 + i·1.17)  // orbital blobs
       radialGradient(hsla(h1,100%,65%,.95) → transparent), 'screen'
  → scanlines: fillRect(sin-shift, y, W, 1) per 3rd row
  → specks: 200 × fillRect(rand, rand, 1, 1)
  → jpeg = toDataURL('image/jpeg', 0.72) → svg:image[href] with filter:url(#glow)
stats every 4th frame: fps = 4000/Δt; sizeKB = (len−23)·0.75/1024
resume toggle ⊗ (R key ‖ type "resume" ‖ triple-click < 380ms) → pause loop while open, resume on close
```

### Why the pause matters
`isPaused` stops the rAF loop when the modal opens — the background literally holds its breath while the resume is being read. Attention as a scarce resource, encoded in the render loop.

---

## 3. `PsyPFE.html` — PARSER FOREST ENGINE (Psychedelic Parser Synthesizer)

### What it represents
The thesis statement piece, and the best recruiter-capture device in the set. A real-time WebGL shader whose only *content* is the concealment of a resume. Title doubles as doctrine: **Parser Forest Engine** — one byte-stream, many parsers, divergent realities. The three field modes are three ways of seeing the same space; the three resume layers are three ways of reading the same file. A recruiter sees a pretty screensaver; a security engineer sees a working demonstration of differential parsing and someone who can document it.

### Effect inventory
- Single fragment shader, uniform-driven mode switch (`int u_mode`)
- Mouse gravity well: `uv += (uv − mouse)·exp(−dist·3)·(0.4 + pulse)` — space bends toward the cursor
- Click pulse: `pulse = 1.0` on mousedown, decays ×0.92; adds a cyan flare term
- Mode 0 Kaleidoscope: `uv = fract(uv·1.5) − 0.5` iterated 4× inside the accumulation loop
- Mode 1 Hyper-Tunnel: polar remap `uv = (angle/π, 1/(r+0.1) + t·0.5)`
- Mode 2 Liquid Plasma: `uv = sin(uv·1.8 + t·0.2) + cos(uv.yx·2.2 − t·0.3)` (note the `.yx` swizzle — cross-couples axes)
- 4-iteration accumulation with `d = length(uv)·exp(−length(uv0))` — distance decay anchors the pattern to center
- UI: Tailwind CDN + JetBrains Mono / Space Grotesk, glass HUD, CRT scanline overlay on the modal

### Steganographic stack (3 layers)
```
L1 → hidden modal markup (toggleable DOM)        :: what users find
L2 → base64 resume in <svg data-resume="...">    :: what view-source finds
L3 → plaintext resume in comment AFTER </html>   :: what scrapers-that-stop-at-/html miss
```
Unlock gesture: press **R**, or **type "resume"** (rolling 8-char key buffer), or **triple-click** anywhere. `∴` the unlock itself is a parser differential: humans gesture, scrapers don't.

### Replication recipe
```
uv = (frag·2 − res)/min(res)                       // centered, aspect-correct
uv0 = uv                                           // keep pristine copy
uv += (uv − mouse)·exp(−|uv−mouse|·3)·(0.4+pulse)  // cursor gravity
mode==1: uv = polar remap                          // tunnel
for i in 0..3:
  mode==0: uv = fract(uv·1.5) − 0.5                // kaleido fold
  mode==2: uv = sin/cos plasma step
  d = |uv|·exp(−|uv0|)
  col = palette(|uv0| + i·0.4 + t·0.4, a,b,c,d_col)  // d_col drifts with i and t
  d = |sin(d·8 + t·2)/8|; d = pow(0.01/d, 1.2)     // glow (idiom 0.3)
  final += col·d
final += 0.1/|uv0 − mouse·0.5| · cyan · pulse      // click flare
```
Deploy as a single .html. No assets, no server. The whole thing travels in one file — the polyglot is also the packaging format.

---

## 4. `QUANTUM_ECHO_v2.html` — VISUAL SONIFICATION ENGINE + CRYPTO-VAULT

### What it represents
The **⊗ piece**: sound and light coupled bidirectionally. The generative synth doesn't just accompany the shader — the shader *listens*: low/high FFT bins drive uniforms (`u_low`, `u_high`), so the visuals are a sonogram of their own soundtrack. Each of the three environments has a matching synth voice, so cycling the protocol retunes the instrument. Then the vault: a three-cipher decryption theater that is really a lecture on *why encoding ≠ encryption* — hex, Caesar-shift, and reversed base64 are all "obfuscation of effort," and the piece presents them with full self-aware ceremony ("BYPASSING MAINFRAME ENCRYPTION..."). The title is the argument: an echo is information bouncing between media until it's unrecognizable in one of them.

### Effect inventory
**Visual (GLSL):**
- Mode 0 FOAM_STATE: layered rotation-noise — `p *= rot(t·0.2/i)` per octave, `n += sin(px·i)cos(py·i)/i`, abs-folded
- Mode 1 CYBER_GRID: fake-3D perspective floor — project `p.x/|p.y|`, `1/|p.y|`, scroll `proj.y −= t·2`, smoothstep grid lines fading with `exp(−|y|·2)`
- Mode 2 VOID_CORE: polar ring `rad = 0.3 + sin(angle·8 + t)·0.05`, 12-fold spikes `sin(atan·12)`, spiral-rotated sampling `p·rot(t + d·2)`
- Global vignette `1 − dot(uv,uv)·0.3`

**Audio (WebAudio):**
- FM chain: `modOsc → modGain → osc1.frequency` (true frequency modulation)
- Voice per mode: FOAM = sine+triangle, fifth interval (×1.4983), slow LFO, lowpass swept by mouse; GRID = saw+square arpeggio on a 7-step minor scale, bandpass with Q up to 10; CORE = 65Hz sub, `modOsc = root·12` mirroring the shader's 12 spikes, aggressive FM depth 120–470
- Mic uplink: `getUserMedia → MediaStreamSource → analyser` — the room becomes the oscillator
- Meters: LOW = bins 0–7, HIGH = bins 30–59, normalized /255, decaying when silent

**Vault (theater):**
```
L1: hex string in data-hex attribute      → String.fromCharCode(parseInt(hex,16))
L2: ASCII-shift array                     → String.fromCharCode(code − 5)     // Caesar by 5
L3: reversed base64 constant              → atob(REV_B64.split('').reverse().join(''))
each typed char-by-char (15ms) into a terminal with color-coded layer banners
```

### Replication recipe
```
audio :: AnalyserNode(fftSize 256) ← (synth ‖ mic)
bins → low/high (0–7, 30–59) → u_low/u_high
shader per-mode branch → col; col += vignette
sonifier.update(mode, t, mouse): retarget osc freqs/filter with setTargetAtTime(…, now, 0.02–0.05)
  // exponential-glide smoothing ≈ the 0.08 lerp of the visual side
vault: three decoders + typewriter promise chain
∴ sonification :: the analyser is the bridge; visuals modulated by the sound they produce = feedback without a wire
```

---

## 5. `neural_dissonance.html` — NEURAL_DISSONANCE v1.0 (the apex piece)

### What it represents
The largest, most architecturally ambitious piece — a **two-pass GPU pipeline with a five-layer steganographic vault**, and the fullest expression of the collection's theme: dissonance between what a renderer shows and what a parser sees. While QUANTUM_ECHO sonifies, this piece *psychoacoustically* couples: a beat detector gates bloom and chromatic aberration, so light physically pulses with bass. The six modes are a tour of real-time graphics techniques — agent grids, wave fields, escape-time fractals, SDF raymarching, polar tunnels, procedural glitch — unified by one palette function and one composite pass. And hidden inside: **five simultaneous encodings of the same resume**, each living in a different web platform subsystem (SVG DOM, JS heap, canvas pixels, CSSOM, raw bytes). Finding all five *requires* understanding five parser contexts. That is the interview.

### Effect inventory
**Six shader modes (single `u_mode` int):**
1. **SYNAPSE** — 8×6 grid of oscillating glow nodes + connection lines, audio-swollen amplitude
2. **ACID RAIN** — 5 stacked sine wave sheets + 24 hashed falling drops, drops flare with beat
3. **FRACTAL ZOOM** — Mandelbrot at (−0.74529, 0.11307), 64 iterations, smooth-coloring correction `iter − log2(log2(|z|)) + 4`, zoom driven by high-freq audio
4. **TORUS KNOT** — raymarched SDF: twisted torus `length(vec2(length(q)−0.7, p.y·0.6)) − 0.18`, twist `cos(p.y·3 + t + angle·2)`, 32 march steps, finite-difference normals, diffuse+spec+ambient
5. **DMT TUNNEL** — polar `tunnel = 0.4/(r+0.08)`, angular harmonics × noise, white core flash on beat
6. **GLITCH CITY** — 16×12 block hash displacement, scanline tearing, matrix rain columns, gated by audio energy

**Composite pass (scene → FBO → screen):**
- Chromatic aberration: R/B sampled at `uv ± dist·0.015·(1+beat/2)`, G at center
- Bloom: 3×3 tap, threshold `max(color − 0.6, 0)`, gain ×2.5
- Grain: `hash(uv + fract(t))·0.04` · Scanlines: `sin(y·res·1.5)` modulated 0.85–1.0 · Vignette `1 − dot(c,c)·0.5`
- Beat detect: bass bins 0–5, energy EMA `E = 0.85E + 0.15·bass`, fire when `bass > 1.4E && bass > 60`, decay ×0.92

**Interaction:** SPACE cycles modes · click fires glitch row-displacement (`u_glitch`, decays ×0.94) · ↑/↓ audio sensitivity 0.1–3.0× · R opens vault · S records WebM via `canvas.captureStream(60) + MediaRecorder`

**The five-layer vault:**
```
L1 :: SVG data-cortex base64 attribute      → atob(attr)
L2 :: JS base64 constant RESUME_B64          → atob(const)
L3 :: canvas RGBA pixel stego: 19×19 canvas, ImageData bytes = char codes, read back until 0x00
L4 :: CSS custom properties: :root{--l4-N:"chunk\A chunk"} strings, decoded via
     getComputedStyle + replace(/\\A/g, "\n")  // 16 properties reassemble the resume
L5 :: XOR byte array: XOR_DATA.map(b => String.fromCharCode(b ^ 42))  // key = 42, of course
```

### Replication recipe
```
pass 1 :: render mode-N(uv, t, audio, beat) into texture attached to FBO
pass 2 :: composite: chromatic aberration → bloom(3×3, threshold 0.6) → grain → scanlines → vignette
audio :: analyser.fftSize 256, smoothing 0.7
  → low(0–7) mid(8–39) high(40+) → u_audio0/1/2
  → beat: EMA + threshold trigger → u_beat (gates aberration, tunnel core, bloom tint)
stego :: pick N subsystems, encode resume into each, decode live in a terminal with typewriter reveal
∴ dissonance :: five encodings, one plaintext — the stack is a résumé of parser knowledge wearing a visualizer
```

---

## 6. `Franksx-Resume-meta-macos.html` — THE TROJAN RESUME

### What it represents
The outlier — and the punchline. Everything else is hand-rolled GLSL and canvas; this is a **192KB compiled React 18.3.1 + Tailwind v3.4.18 artifact** whose `<title>` literally reads *"React Artifact."* A resume dressed as a native macOS window: traffic-light dots (#FF5F56 / #FFBD2E / #27C93F), near-black #0D0D0D chrome, chartreuse #CCFF00 accents, Inter with mono data rows, chartreuse-on-black text selection. Next to the five hand-built pieces it reads as deliberate self-parody — the "normal" resume is the strangest object in the repo: a bloated compiled bundle pretending to be a lightweight native window, while the "weird" pieces are lean single files. ∴ **The medium is the message, inverted:** the polyglot HTML files are honest about what they are; the conventional-looking one is the real illusion.

### Effect inventory (static, no runtime effects)
- Inlined Tailwind utility CSS (~30KB of compiled utilities) — the "grid-bg," "glass," and accent classes are all precomputed
- Minified React + ReactDOM inlined as an ES module; the entire resume body lives inside the bundle
- macOS window chrome via styled divs: rounded title bar, three 14px colored dots, blurred backdrop
- Typography: tight tracking (−0.02/−0.05em), mono data rows at 10–11px, oversized 84px nameplate on desktop
- Micro-details: `color-scheme: dark`, selection styling, hover border glows (`hover:border-[#CCFF00]/50`), radial gradient ambient washes

### Replication recipe
```
build a React + Tailwind resume (dark, chartreuse accent, window chrome)
→ compile with Tailwind content-scanning so only used utilities ship
→ inline react, react-dom, the bundle, and the compiled CSS into ONE html file
→ title it "React Artifact" as a wink
theme tokens: bg #0D0D0D, surface #161616, accent #CCFF00, secondary #7A7AFF,
             text #EDEDE8 → #555 gradient, selection: #CCFF00 bg / black text
```
Why replicate this way: the piece's identity *is* the build artifact. Rebuilding it by hand in vanilla HTML would miss the point — the 192KB bundle is the artwork's mass.

---

## 7. HOW THE SIX PIECES FIT TOGETHER

```
endlessjpeg ──technique────→ franksx_endless_q3-26 ──thesis────→ PsyPFE
   (JPEG loop)                   (identity site)            (shader + 3-layer stego)
        │                                                     │
        └──────────── generational ⟲ ─────────┐                │
                                              ▼                ▼
                                    neural_dissonance ◄── QUANTUM_ECHO
                                    (everything at once)   (audio ⊗ vision)
                                              │
                                    Franksx-Resume-meta-macos
                                    (the straight-faced coda)
```

- **One resume, six encodings.** The same plaintext appears in all six files, encoded per piece's theme.
- **One palette function, six palettes.** `a + b·cos(2π(ct+d))` is the collection's only color theory.
- **One motion grammar.** Lerp 0.08 · decay 0.92 · pulse on click · beat on bass. If it moves in this repo, it obeys those constants.
- **Escalating stego depth:** 3 layers (PsyPFE) → 3 layers (QUANTUM_ECHO, theatrical) → **5 layers (neural_dissonance)** → 0 layers, pure surface (meta-macos). The coda's honesty is the final twist.
- **The unlock gestures repeat** across pieces: R key · typing "resume" · triple-click · SPACE to cycle. Learn one piece's interface and you can open all of them — the repo itself is a parser forest with a shared key.

∴ **Final read:** VividVectors26 is not a portfolio *with* steganography; it is a steganography curriculum that happens to be beautiful. Each piece teaches one parser boundary (Canvas/JPEG/SVG, HTML/SVG/DOM, GLSL/WebAudio, CSSOM/canvas pixels/JS heap), and the resume is the checksum that proves you crossed it.


---

# ADDENDUM — THE CORRUPTION ERA (8 new pieces, added Oct 15 2026)

The second wave shifts the collection's center of gravity: away from pure shader/stego synthesis and toward **theatrical failure**. The computer itself becomes the parser being tortured. Every piece in this wave follows a six-act tragedy:

```
ACT I :: BOOT      — power-on theater (BIOS/UEFI/kernel)
ACT II :: LOGIN    — automated credential typing (no user needed)
ACT III :: DESKTOP — a retro OS rendered in DOM, cursor moves by itself
ACT IV :: CASCADE  — error windows breed geometrically
ACT V :: DATAMOSH  — the framebuffer eats itself, resume text inside the corruption
ACT VI :: BSOD     — the crash screen delivers the payload; any key ⟲ reboots
```

∴ The argument of the era: **a system failure is the most honest parser output there is.** When the DOM "crashes," it reveals what it actually parsed.

---

## 8. `errors.html` — ERROR SCREEN MONTAGE

### What it represents
The era's overture. Six static dioramas of historical failure — MS-DOS green panic, Win95 fatal exception, XP Luna stop error, pop-art CMYK overflow, macOS kernel panic, classic blue screen — auto-cycling every 3.5s behind a CRT scanline filter. Each crash is rewritten as *FrankSx's own*: faulting module `FRANKSX_POLYGLOT.SYS`, stop code `0x80004005 (FRANKSX_BUFFER_MISMATCH)`, kernel panic in `"FrankSx parser differential mismatch detected"`. A museum of operating-system death where every exhibit is autobiographical.

### Effect inventory / replication recipe
- Pure CSS/Tailwind dioramas — no canvas, no WebGL. The craftsmanship is in authentic chrome:
  - Win95 bevels: `border-color: #fff #404040 #404040 #fff` + `box-shadow: 6px 6px 0 #000`
  - XP Luna gradients: `linear-gradient(180deg, #0058ee, #3a93ff 8%, #0055ea 90%, #0045ab)`
  - CRT filter: `body::after { background: linear-gradient(transparent 50%, rgba(0,0,0,.35) 50%); background-size: 100% 4px }`
- Montage engine: `slides[current].classList.remove('active')` → index mod N → counter text update; `setInterval(showNextSlide, 3500)`

Replication is a research exercise more than a coding one: pick an era's error screen, get its palette and typography exactly right (Tahoma for XP, VT323/monospace for DOS, Helvetica-bold for macOS panic), then insert your own name into the fault strings. The humor lands only if the chrome is faithful.

---

## 9. `Totalcorrupted.html` — CORRUPTED FIRMWARE BOOT v5.0

### What it represents
The era's first full tragedy. A complete CRT monitor simulation (4:3 aspect, barrel-breathe animation, power LED, bezel shadows) running a corrupted UEFI boot: integrity check fails, "malicious memory patterns" are injected as a live hex dump, a boot menu offers **Boot Unstable Shell / Overwrite Resume Sector / Kernel Panic Simulation**. The resume lives in the shell's `whoami`/`skills` commands. The piece's thesis: *booting a computer is already a negotiation with parsers — corrupt it and you get theater.*

### Effect inventory
- **The CRT housing** (the real star): `perspective(1000px) rotateX(1.2deg)` breathe loop, `aspect-ratio: 4/3`, `border-radius: 28px`, triple inset box-shadow vignette, scanline drift at 20Hz
- **5 glitch types**, each a DOM mutation with a CSS-variable trigger:
  - `chroma_split` → `--glitch-intensity` scales text-shadow R/G/B offsets
  - `scanline_shift` → spawns `.scan-bar` divs (color-dodge blend), removes after 80–230ms
  - `color_invert` → `mix-blend-mode: difference` flash
  - `screen_shake` → translate on a wrapper + `--shear-x`
  - `parser_differential` → diagonal-stripe overlay + scanline alpha spike + sawtooth beep
- **Corruption meter**: every glitch decrements stability ×4; bar turns red under 30
- **Corrupted typewriter**: 25% chance each character renders as a block-drawing glyph (0x2580+) before being corrected
- WebAudio: square-wave beeps + 100ms noise-burst `glitchSound()`

### Replication recipe
```
screen :: 4:3 div → perspective breathe + vignette + scanlines (3 stacked layers)
glitch :: switch(type) → CSS-variable mutation → setTimeout restore (60–250ms)
  ∴ the key discipline: EVERY glitch must be reversible — chaos, then restoration
boot :: async/await stageplay — out() → wait() → typeLine() → showProgress()
progress :: fill bar + percentage that lies (±20% jitter) → ends on "CORRUPTED"
shell :: keydown buffer → prompt() div with live cmdline span → executeCommand switch
```

---

## 10. `franksx_corrupted_v6.html` — CORRUPTED FIRMWARE BOOT v6.0 (the era's apex, 51.8KB)

### What it represents
v5's architecture perfected into a **complete fake operating system**. The UEFI POST enumerates real hardware subsystems (CPU, DRAM walking-ones, CMOS, PCI bus, GPT partition table) and finds the resume living at partition 3, `0x40100800, 4KB, [RAW — NULL BYTES DETECTED]` — then hex-dumps it live with corrupted bytes that heal as you watch. The boot manager offers five paths; the shell has ~20 commands with history; memtest corrupts and ECC-recovers; PXE boot downloads `parser_lab_netboot_corrupt.bin` and fails signature verification (`0xDEADBEEF` vs `0xCAFEBABE`) because of "JPEG↔SVG boundary corruption in network frame." ∴ **The resume is firmware.** It lives in the boot sector, below the OS, where parsers are smallest and most gullible.

### Effect inventory (upgrades over v5)
- **13 glitch types** — adds `noise_burst`, `barrel_warp` (3D rotate on the CRT itself), `interlace_flicker`, `ghost_image` (copies terminal innerHTML into a screen-blend layer offset by random px), `signal_loss` (white overlay + snow), `tear_lines` (vertical difference-blend slivers), `hue_shift`, `datamosh` (6 screen-blend strips translated ±200px)
- **Phosphor themes**: amber(38°)/cyan(180°)/green(120°)/magenta(300°) HSL, toggled F2 or `theme` command; **overdrive mode** hue-cycles +3°/frame
- **Hex dump with healing**: 15% of bytes spawn `.corrupted` (random value, red), then flip to `.healed` (correct value, green) — corruption as a live editorial process
- **Full shell**: `whoami skills projects contact history hexdump glitch overdrive corrupt panic integrity repair theme clear reboot exit` + arrow-up/down history recall
- Boot menu: normal / safe mode / setup / memtest / PXE — five complete sub-routines

### Replication recipe
```
POST :: await-chain per subsystem, each with ✓/⚠/✗ lines + glitch side-effects
hex  :: TextEncoder → bytes → 16-byte rows → per-byte span with corrupted→healed flip
boot :: menuIndex state machine (ArrowUp/Down re-renders ▶ selection) → selectBoot()
shell:: commandBuffer string + keydown capture → executeCommand switch → prompt()
themes:: CSS custom props in HSL → rAF hue cycle in overdrive
∴ the trick that makes it feel real: every command MUTATES the corruption state,
  and the corruption state is visible in the HUD — a closed loop the user can feel
```

---

## 11. `franksx_spectrogram_resume.html` — SPECTROGRAM RESUME TX v1.0

### What it represents
The wave's conceptual outlier and the collection's most complete statement of **audio as a parser**. QUANTUM_ECHO sonified *existing* sound; this piece *transmits text as sound* and lets the spectrogram render it. Every character of the resume becomes a tone: ASCII 32–126 mapped quadratically onto 300–7500Hz, 90ms per character, sawtooth + a detuned square harmonic (×2.01) for spectral richness, 5% chance of a noise-burst "glitch" per character. The waterfall FFT display — a circular ImageData ring buffer, newest column at the right edge — is the primary display surface; the webpage is merely the radio's front panel. A real spectrogram app pointed at this page recovers the resume. ∴ **The resume no longer needs the browser.** It travels through air as pressure waves; any FFT parser can read it. This is the purest "parser forest" piece yet — the intended parser is *your ears' software.*

### Effect inventory
- Frequency map: `freq = 300 + t²·7200` where `t = (code − 32)/94` — quadratic spread keeps low letters visually separated
- Waterfall ring buffer: `wfWidth = innerWidth/2` columns × `fftSize` rows; write column at `wfHead`, draw with a mirrored-translate blit so the head sits at the right edge
- 5-band phosphor palette: black → dark amber → amber → bright amber → white-hot (`ampToColor`)
- Per-section transmit buttons (summary/skills/projects/contact) + full transmission + replay
- HUD: live dB, peak Hz, per-char FREQ/PHASE readout, TX progress bar
- Decode panel: the text is "decoded" live as it transmits (it's the txData itself — the decode is the theatrical mirror of the encode)

### Replication recipe
```
char → freq :: quadratic ASCII map
tone  :: sawtooth osc (0.15 gain, 10ms attack, 90ms hold, exp decay) + square at freq·2.01
waterfall :: analyser.getByteFrequencyData → write ImageData column → blit with wrap
glitch :: 5% chance per char: 50ms white-noise buffer at 0.1 gain
keys   :: SPACE transmit · D decode · R replay
∴ encode :: the piece is its own modulation scheme; take a phone spectrogram app to any demo and it reads the resume aloud in glyphs
```

---

## 12–14. `xp.html` / `y2k.html` / `y2kpopart.html` / `y2kv2.html` — THE OS TRAGEDY QUAD

Four variations on one engine, escalating in branding fidelity and stego depth:

| | xp.html | y2k.html | y2kpopart.html | y2kv2.html |
|---|---|---|---|---|
| **OS skin** | Windows XP Luna | Windows 98 | Pop-art CMYK | FrankSx OS (Win98 base) |
| **Boot** | XP logo + sliding progress block | BIOS memory count to 65536KB | 3-stage dye calibration log | Kernel + parser module logs |
| **Windows** | 13 XP error popups | 12 Win98 popups | 15 CMYK popups | 10 parser-corrupt popups |
| **Mosh** | 130 frames, pop-art resume text | 80 frames, green resume text | 160 frames, CMYK overlay cycle | 90 frames, cyan strips |
| **Crash** | XP BSOD (`FRANKSX.SYS`) | Win95 exception 0E | Pop collapse + **decoded payload card** | FrankSx panic + **intercepted payload card** |
| **Stego layers** | none | none | L2 base64 SVG attr + **L3 plaintext after `</html>`** | L2 base64 SVG attr + **L3 plaintext after `</html>`** |

### What they represent
The quad is the era's narrative engine. An unattended machine boots itself, logs itself in (the password types *itself* — 12 bullets at 150–200ms intervals), a simulated cursor drifts to `Resume.exe`, error windows cascade geometrically, and the framebuffer dissolves into datamosh — with the resume rendered *inside* the corruption. Then the BSOD, and the crash screen itself delivers the payload (popart/v2 put a decoded-resume card on the crash screen and in v2 the crash text *admits* "the resume layer remains embedded across base64, canvas, and HTML comment structures"). The machine fails its way into honesty. ∴ **The crash is the cover letter.**

### Replication recipe — the datamosh algorithm (shared verbatim across all four)
```
canvas :: fullscreen, opacity 0 → 1
loop :: rAF, N frames (80–160)
  1. draw resume text (big display font, pop colors, re-drawn EVERY frame ⟲)
  2. 20–30 × drawImage(canvas, sx,sy,sw,sh, dx,dy,sw,sh)   // self-copy displacement
       dx = sx ± rand(15–60), dy = sy ± rand(5–20)         // horizontal bias = VHS tear
  3. every 2–3 frames: translucent color block (r|g|b|cyan|magenta, alpha .15–.2)
  → after N frames: triggerBSOD()
```
The genius detail: text is redrawn each frame while blocks keep displacing — so the letters smear into themselves while staying legible long enough to read. Persistence through corruption.

The simulated cursor is an inline SVG arrow polygon, absolutely positioned, `transition: transform .05–.1s` for CRT-style stepping.

The Win98 bevel idiom (all four use it): `border: 2px solid; border-color: #fff #404040 #404040 #fff` with an `:active` state that inverts — pure CSS, no images, and instantly reads as 1998.

---

## 15. Updated collection map

```
WAVE 1 (synthesis)                 WAVE 2 (corruption, Oct 15 2026)
─────────────────────────────      ─────────────────────────────────────────────
endlessjpeg ──┐                    errors.html ── montage of failures
q3-26 ────────┤── generative       xp / y2k / y2kpopart / y2kv2 ── OS tragedies
PsyPFE ───────┤── shader+stego         └── engine: boot→login→desktop→cascade→mosh→BSOD
QUANTUM_ECHO ─┤── audio⊗vision       Totalcorrupted v5 ──┐
neural_disson ┘── 5-layer apex        franksx_corrupted v6 ┘── CRT firmware theater
meta-macos ──── the coda              spectrogram_resume ── the outlier: text→tone→FFT
```
