---
name: higgsfield-video-editing
description: Edit talking-head and YouTube videos with Higgsfield AI by planning precise cuts, generating supporting visuals, and assembling a polished final edit with captions, motion graphics, transitions, audio cleanup, and a branded outro. Use when a user asks to edit a video with Higgsfield, remove pauses or repetitions, add B-roll or motion graphics, or make a talking-head video more engaging.
---

# Higgsfield Video Editing

Use Higgsfield AI for the parts it does well—visual generation, motion assets, backgrounds, and supported video effects—and use a deterministic video editor for precise cuts, exact text, compositing, audio, and final quality control.

This skill is intentionally brand-agnostic. Replace the placeholders below with the user's channel name, colors, fonts, logo, outro, and editorial preferences.

## Non-negotiable rules

- Preserve the source's resolution, frame rate, aspect ratio, and audio quality unless the user requests a change.
- Inspect the source before editing. Never assume the duration, frame rate, resolution, codec, or audio layout.
- Apply the user's explicit time ranges and end phrase exactly. They take priority over automatic silence detection.
- Cut speech only when the intent remains clear. Remove long silences, working pauses, false starts, repeated words, and self-corrections conservatively.
- Cut and verify the editorial timeline before adding graphics.
- Render exact words—names, channel titles, labels, captions, and calls to action—in the editor. Do not rely on generative video to spell important text correctly.
- Keep graphics away from the speaker's face and mouth. Use the left or right third, lower band, or a deliberate split-screen layout.
- Do not add copyrighted music or stock footage unless the user supplied it or confirmed they have the necessary rights.
- Do not claim that a Higgsfield feature is available if it is not exposed in the current account or integration. Map to the closest available feature or clearly mark that step for a local editor.

## Inputs to collect

Ask only for information that is missing:

1. Source video path or upload, plus the desired aspect ratio if it is not obvious.
2. Script or transcript. If none is provided, create a word-level transcript locally.
3. Required cuts:
   - starting point or where the speaker actually begins;
   - explicit ranges to remove;
   - long pauses, working pauses, repetitions, false starts, or corrections;
   - the phrase that should end the spoken section.
4. Brand details: channel name, colors, fonts, logo, caption style, and the standard outro file.
5. Target platform and output constraints. Default to the source's native quality in a 16:9 YouTube export.

If a standard outro is requested but no outro asset is available, ask for it before rendering the final version. Do not invent a brand outro and present it as the user's standard one.

## Phase 1: Inspect and transcribe

### Inspect the source

Read the source metadata with `ffprobe` or the equivalent media-inspection tool:

- duration;
- width and height;
- frame rate and time base;
- video codec, pixel format, and color metadata;
- audio codec, sample rate, channel layout, and bitrate.

Also inspect a few frames and listen to the opening and closing. Record the metadata in the edit plan.

### Create a word-level transcript

Use a local transcription tool such as faster-whisper when available. Keep:

- each word's start and end time;
- speaker pauses;
- false starts and repeated phrases;
- the exact ending phrase;
- technical terms and proper names that need proofreading.

If the transcript is uncertain, inspect the relevant audio or video before cutting. Never infer a correction from text alone when it could change the meaning.

### Build the editorial decision list

Create a machine-readable EDL or equivalent JSON alongside a human-readable plan. Use source timecodes:

```json
{
  "source": "input.mp4",
  "keep": [
    {"in": 0.0, "out": 12.4, "reason": "opening"},
    {"in": 15.1, "out": 86.7, "reason": "main explanation"}
  ],
  "remove": [
    {"in": 12.4, "out": 15.1, "reason": "long working pause"}
  ],
  "end_phrase": "I’m going to see you in the next one"
}
```

Prefer keep-ranges because they make the final order explicit. Add 10–40 ms audio crossfades at non-natural cut points to prevent clicks, but do not crossfade in a way that blends two incompatible words.

### Silence and speech-editing guidance

- Treat a gap of roughly 0.8–1.2 seconds as a candidate, not an automatic cut.
- Preserve short breaths and pauses that help the explanation sound natural.
- Remove the incorrect fragment of a false start, then keep the complete corrected sentence.
- Remove immediate duplicate words only when the remaining waveform and sentence boundary are clean.
- Keep enough room around a cut to avoid chopping consonants.
- Verify every automatic cut with the transcript and a short audio/video preview.

## Phase 2: Plan the visual edit

Break the edited speech into beats. A beat is one idea, normally 8–25 seconds. Plan visual change without turning every sentence into an effect.

Use this table:

| Time | Spoken trigger | Editorial beat | Hero visual | Supporting element | Transition |
|---|---|---|---|---|---|
| 0:00–0:15 | `[key phrase]` | `[idea]` | `[Higgsfield asset]` | `[caption/chip/lower third]` | `[style]` |

Pacing defaults:

- Something may change every 3–5 seconds, but calm explanatory sections can breathe.
- Use one hero graphic at a time.
- Use no more than two recurring transition styles.
- Keep transitions about 0.3–0.8 seconds.
- Use a punch-in of about 105–115% only where it helps emphasis or hides a jump cut.
- Keep the last 10–20 seconds visually calm when the platform's end-screen elements need space.

## Phase 3: Generate assets in Higgsfield

First inspect the current Higgsfield integration or workspace and use only the available generation/editing features. Generate each reusable visual as a separate short asset so it can be timed precisely in the editor.

Recommended asset types:

- 2–6 second motion backgrounds;
- short B-roll clips or stills for concepts that are difficult to show with the speaker on screen;
- abstract tech backgrounds;
- icon or object animations;
- simple transition elements;
- end-card or thumbnail backgrounds.

For every asset, specify:

- duration and aspect ratio;
- intended placement and source timestamp;
- color palette and lighting;
- camera or motion behavior;
- whether the background should be transparent, solid, or full frame;
- `no text, no letters, no logos, no watermark` unless the user explicitly wants generated text.

### Reusable prompt templates

**Abstract background**

> Create a `[duration]` `[aspect ratio]` motion background for a technology YouTube video. Use `[brand palette]`, subtle `[motif]`, clear negative space on the `[left/right/center]` for exact text added later, polished and cinematic, restrained motion, no text, no letters, no logos, no watermark.

**Concept B-roll**

> Create a `[duration]` `[aspect ratio]` visual representing `[concept]`. Show `[subject/action]` with `[lighting/style]` in `[brand palette]`. Keep the composition readable behind an overlay, avoid readable screen text, no logos, no watermark.

**Motion graphic element**

> Create a `[duration]` transparent or solid-background motion graphic of `[object/icon/shape]`. Animate it with `[motion]`, using `[colors]`, with clean edges and no text, no letters, no logos, no watermark.

**Transition element**

> Create a `[duration]` transition element in `[brand palette]`: `[whip pan/light flash/particle dissolve/zoom streak]`. Fast entrance, soft settle, no text, no logos, no watermark, suitable for compositing over a cut.

If Higgsfield generates garbled text or an incorrect logo, regenerate without text and add the exact element locally. For precise branding, prefer supplied logo assets over generated logos.

## Phase 4: Assemble the edit

### Editing order

1. Make the clean cut from the EDL.
2. Place the speaker footage and preserve the original audio sync.
3. Add the supplied branded outro after the requested end phrase.
4. Add captions, lower thirds, punch-ins, B-roll, and motion graphics.
5. Add transitions and subtle sound effects.
6. Apply only the requested or necessary audio and color treatment.
7. Render a draft and review it before the final export.

Use any suitable deterministic editor available in the project—ffmpeg plus Python/Pillow, Remotion, DaVinci Resolve, Premiere, or another NLE. Higgsfield-generated assets are inputs to the edit; they are not a reason to hand off exact timing to a generative model.

### Brand system defaults

Use these only when the user has not supplied a brand system:

- deep navy `#0B1020`;
- electric blue `#2F6BFF`;
- cyan `#22D3EE`;
- white `#FFFFFF`;
- warm highlight `#FFB020` for emphasis and calls to action;
- a bold geometric sans such as Montserrat, Poppins, or Inter;
- JetBrains Mono for small technical labels.

Replace the defaults consistently when the user supplies colors, fonts, or a logo.

### Captions and graphics

- Use 2–4 words per caption line.
- Highlight the active word sparingly.
- Proofread names, technical terms, product names, and the channel name against the transcript.
- Hide or move captions while a lower third, large title, split-screen panel, or full-screen B-roll is active.
- Keep safe margins of at least 5% from the frame edges.
- Never place a large element over the speaker's face or mouth.

### Audio

- Keep the original voice recording as the primary track.
- Use short audio crossfades at edits to avoid clicks.
- Apply denoise, compression, or loudness normalization only when needed and document it.
- A good YouTube target is about `-14 LUFS integrated` with true peak below `-1 dBTP`, but do not crush clean source audio to hit a number.
- Keep sound effects subtle, generally around `-18` to `-24 dB` below the voice.
- Duck any licensed music under speech by roughly 8–12 dB.

## Quality-preserving export

The final export must not silently downgrade the source:

- match the source resolution unless the user requests a different delivery size;
- match the source frame rate and aspect ratio;
- avoid unnecessary intermediate re-encoding and upscaling;
- retain the source color range and color metadata when supported;
- copy the audio stream when it remains compatible and untouched;
- otherwise encode audio with a high-quality setting or the source codec;
- use a high-quality video encode, such as H.264 CRF 16–20 with a slow preset, or a bitrate appropriate for the source;
- use `+faststart` for a web-delivered MP4.

Example only—replace settings after inspecting the source:

```bash
ffmpeg -i edited_video.mov \
  -c:v libx264 -preset slow -crf 18 \
  -c:a aac -b:a 320k \
  -movflags +faststart final_video.mp4
```

Do not force `-r 30`, `-s 1920x1080`, or another fixed setting when the source uses a different frame rate or resolution.

## QA before delivery

Check the full export, not only a short preview:

- [ ] The requested start point is correct.
- [ ] Every explicit removal range is absent.
- [ ] Long silences, working pauses, repetitions, and false starts were handled without clipped words.
- [ ] The video ends after the requested phrase.
- [ ] The branded outro starts cleanly and is correctly centered.
- [ ] The source resolution, frame rate, aspect ratio, and audio layout are retained.
- [ ] There are no clicks, pops, abrupt audio changes, or lip-sync drift.
- [ ] All on-screen words are spelled correctly.
- [ ] Graphics do not cover the speaker's face or mouth.
- [ ] Captions are readable on a phone.
- [ ] Transitions are consistent and not distracting.
- [ ] The final audio is clear and not clipped.
- [ ] The last 10–20 seconds leave room for platform end-screen elements when applicable.

Record the verification results and output metadata in the delivery note.

## Required output when using this skill

Return:

1. A short assumptions list.
2. The final EDL or a readable cut list.
3. The beat plan.
4. The Higgsfield prompts/assets used, including which steps happened in Higgsfield and which happened in the editor.
5. The output file path and export metadata.
6. The QA results and any remaining limitations.

This file is a reusable starting point. Modify the brand defaults, beat structure, cut thresholds, prompt templates, and output settings for each creator and video rather than copying the original channel-specific choices unchanged.
