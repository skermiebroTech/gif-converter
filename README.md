# chromakey → gif

Turn a green-screen video into a **transparent animated GIF**, entirely in the browser. No upload, no backend, no build step — a single `index.html` you can drop straight onto GitHub Pages.

The whole thing is one self-contained file: the chroma-key/cleanup pipeline **and** a from-scratch GIF89a encoder (with real binary transparency) are written in plain JS with zero dependencies. It works offline once loaded.

## Use it

1. Open the page.
2. Drop in a video (MP4/H.264 or WebM are the safe bets — see caveats).
3. The key colour is auto-detected from the first frame; tweak it with the eyedropper, the colour picker, or a hex value if needed.
4. Adjust **Similarity** (how much of that colour to cut) and **Feather** (soft edge), watching the live checkerboard preview.
5. *(Optional)* Drag the **trim** handles to render only part of the clip, and set a **filename** for the download.
6. Hit **Render GIF**, then download. The live estimate under the button tells you how many frames you'll get before you commit.

Controls: key colour, similarity, feather, output width, fps, palette size, plus toggles for spill suppression, edge cleanup, auto-crop, and loop.

### Usability features

- **Play / pause preview** — a ▶ button next to the scrubber plays the clip live with keying applied, so you can watch the cut in motion instead of stepping frame by frame.
- **Remove-background toggle** — a master on/off switch for the chroma key. Off shows (and exports) the raw footage untouched; on cuts the backdrop. The key-colour controls dim when it's off.
- **Trim range** — a dual-handle slider picks the in/out points, so you can export just a slice of a clip instead of the whole thing.
- **Live frame estimate** — the note under *Render GIF* shows the frame count, duration, and frame-rate you're about to encode (and flags the 450-frame cap) before you start.
- **Solid background** — instead of transparency, flatten every frame onto a colour of your choice (handy when the target won't show GIF transparency well). The checkbox + swatch live under the toggles.
- **Custom filename** — name the downloaded `.gif` right in the panel.
- **Cancel** — abort a long render mid-way without reloading the page.
- **Remembered settings** — your slider/toggle choices are saved to `localStorage` and restored next visit; **Reset settings to defaults** clears them. (The key colour is always re-detected per video.)

## Deploy to GitHub Pages

Put `index.html` at the root of a repo (or in a `/docs` folder), then:

1. Repo → **Settings** → **Pages**.
2. **Source**: Deploy from a branch.
3. **Branch**: `main`, folder `/ (root)` (or `/docs` if you used that).
4. Save. Your app is live at `https://<user>.github.io/<repo>/` within a minute or two.

Nothing else is required — there's no build, no Node, no Actions. It's a static file.

## Good to know (caveats)

- **It keys colour, not subjects.** Anything that *isn't* the key colour stays opaque. If the green screen doesn't reach the floor or the edges of frame, those areas (and anyone/anything else in shot) will remain — that's correct chroma-key behaviour, not a bug. A clean, evenly-lit backdrop that fills the frame gives the best cut.
- **Frame extraction depends on the browser's video decoder.** MP4 (H.264) and WebM are the most reliable. Exotic codecs or some `.mov` files may not seek frame-accurately, or may not decode at all.
- **Keying runs at the output resolution** (the width you pick), not full source res, so previews and renders stay fast. Lower width = faster + smaller file.
- **Frame cap:** rendering stops at 450 frames to stay responsive. For a long clip, drop the frame rate to cover the whole thing.
- **Transparency is binary** (a pixel is either fully opaque or fully clear) — that's a hard limit of the GIF format itself, not the encoder. Soft/feathered edges are approximated against the chosen palette.

## How it's built

Single file, three parts:

- **GIF89a encoder** — median-cut palette (255 opaque colours + a reserved transparent index), nearest-colour cache, and a spec-correct LZW coder. The encoder was validated byte-for-byte against independent decoders (Pillow and ffmpeg) before shipping.
- **Chroma-key pipeline** — YUV chroma-distance keying with an inner (cut) / outer (feather) ramp, green-spill suppression, connected-component cleanup (keeps the main subject, drops stray specks and border islands, and fills interior holes — but **not** holes whose pixels are still key-coloured, so backdrop showing through a gap like the space between two arms stays transparent instead of being patched back in), and auto-crop to content.
- **UI glue** — `<video>` seek-based frame grabbing, live checkerboard preview, click-to-pick eyedropper, and a Blob download.

No frameworks. No CDN. The only browser storage used is a single `localStorage` key (`chromakey-gif-prefs`) that remembers your control settings — clearable any time with **Reset settings to defaults**.
