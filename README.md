# Suno Prompt Forge

**Audio → Highly Detailed Suno Prompt (Guidebook-Native)**

A Python CLI that captures or loads a song sample (Shazam-style), runs rigorous MIR deconstruction, and emits copy-paste-ready Suno prompts that obey every constraint, template, negative-prompt rule, slider heuristic, timing table, and artist deconstruction from Metta Thomas’s *Suno + SSML Comprehensive Guidebook V2*.

## Why This Exists

The Guidebook (Ch. 4, 11–13, App. B/C) repeatedly demonstrates that generic prompts fail while precisely-specified, feature-driven, negative-prompt-hardened prompts succeed. This tool automates the reverse process: given real audio, recover the exact parameter surface (BPM, key, silence ratio, glitch density, bass character, spectral texture) and instantiate the validated prompt language that actually works inside Suno.

It is deliberately **not** a black-box “genre classifier”. It is an explicit, inspectable feature → template mapper that stays inside the design rules the Guidebook codifies.

## Installation

```bash
cd suno_prompt_forge
python -m venv .venv
source .venv/bin/activate          # or conda / pyenv
pip install -r requirements.txt
```

Requires portaudio for `sounddevice` (usually present on macOS; on Linux: `sudo apt install portaudio19-dev` or equivalent).

## Core Capabilities

| Stage                    | Implementation                              | Guidebook Anchor                  |
|--------------------------|---------------------------------------------|-----------------------------------|
| Recording / ingest       | `sounddevice` or `librosa.load`             | Sec 3.2 (working inside the app)  |
| Tempo + beats            | `librosa.beat.beat_track`                   | Sec 7.1 (timing as rhythm)        |
| Key estimation           | Chroma-CQT + Krumhansl correlation          | Implicit in all key-tagged prompts|
| Silence / negative space | RMS frame analysis below –42 dB             | Tipper validation checklist       |
| Glitch / micro-edit density | Onset strength + IOI statistics          | Ch. 11.3, 11.5, 12.5              |
| Bass character           | Low-band energy ratio (<150 Hz)             | Mono-sub priority everywhere      |
| Aesthetic mapping        | Rule surface over BPM + glitch + silence    | Ch. 11 case studies + Ch. 13 hybrids |
| Prompt synthesis         | Dynamic clause injection into exact templates | App. B, C; Sec 4.2 templates     |
| SSML skeleton            | Beat-math `<break>` tags + prosody          | Sec 7.1, 8.x cookbook, 9.3        |
| Provenance               | UTC timestamp + full feature JSON           | “keep provenance notes” (multiple)|

## Usage

### 1. Live capture (Shazam workflow)

```bash
python suno_prompt_forge.py --record --seconds 18
```

Play any track near your mic for ~18 s. The tool returns the full analysis + prompt block.

### 2. File ingest (recommended for precision)

```bash
python suno_prompt_forge.py --file my_production_clip.wav --output suno_prompt.txt --json-sidecar
```

### 3. Force a specific aesthetic (when you know the target)

```bash
python suno_prompt_forge.py --file clip.wav --aesthetic tipper_micro_granular --bpm-hint 87
```

Available aesthetics (directly from Guidebook):

- `tipper_micro_granular`
- `max_cooper_generative`
- `nocturnal_organic_downtempo`
- `hyperkinetic_glitch_bass_idm`
- `glitchy_atmospheric_symphonic_rnb`

## Output Example (abridged)

```
micro-granular downtempo; 87.0 BPM; C# minor; SINGLE sine bass hits (32Hz, 8s apart), ultra-dense 1/64th granular foley clouds, NO BREAKBEATS, NO 4/4; single evolving arc; evolving density (sparse → dense → sparse); mix: mono 30Hz sub only, surgical 200-5kHz carving, NO reverb wash, parallel granular tails; mood: surgical stillness, nocturnal micro-edited; instrumental.

Negative Prompt (CRITICAL): drum and bass, 4/4 kicks, breakbeats, repeating basslines, dense percussion, long reverb tails, synth leads, hi-hats, snares, 130+ BPM

Weirdness: 78   Style Influence: 95

## Audio Deconstruction Report
BPM: 87.0 (detected)
Key: C# minor (confidence 0.81)
Silence / negative-space ratio: 0.67 → 67%
Glitch / micro-edit density: 0.71
Bass low-end ratio (<150 Hz): 0.19
...
Closest Guide aesthetic: Tipper Micro Granular (Ch. 11.5 Tipper — Micro-Granular Downtempo + App. B.2.1)

## SSML Vocal Layer Suggestion (Guide Sec 7.1, 8.x, 9.3)
<speak ...>
  Signal rises. <break time="690ms"/> Hold the line.
  <break time="345ms"/> We move in quiet time.
</speak>
```

Exactly the language and structure that the Guidebook’s validation checklists reward.

## How the Mapping Works (Transparency)

The decision surface is intentionally readable:

```python
if 82 <= bpm <= 96 and silence > 0.52 and glitch > 0.38:
    return "tipper_micro_granular"
elif 108 <= bpm <= 122 and glitch < 0.35:
    return "max_cooper_generative"
...
```

Each branch instantiates the precise prompt block that the Guidebook shows “actually works” for that aesthetic, then mutates only the detected scalars (BPM, key, silence %, mood nuance) while preserving the negative-prompt and mix directives that Suno needs.

## Limitations & Future Extensions (First-Principles)

1. **Vocal / SSML fidelity** — True deconstruction of an existing vocal into editable SSML requires source separation + ASR + prosody/forced-alignment. The current SSML is a correctly-timed structural skeleton + mood-matched example. Full pipeline is a natural next node (add `whisper`, `demucs`, `aeneas`).
2. **Key on atonal material** — Confidence drops; the tool still emits the best tonal guess + the prompt still works because Suno templates tolerate “chromatic / atonal” descriptions when needed.
3. **No ML classifier** — By design. A trained genre model would violate the Guidebook’s emphasis on explicit, reproducible, feature-level control. The heuristic surface is inspectable and editable.
4. **Long-form structure** — Currently emits section-level or “single evolving arc” prompts. Full 7–8 min arc detection would need `librosa.segment` + recurrence analysis (straightforward extension).

## Integration with Your Existing Workflow

- Drop the generated prompt directly into Suno Custom or Instrumental mode.
- Render the suggested SSML stem externally (Azure recommended per Guide), import both stems into your DAW, sidechain duck per Sec 9.4.
- The `.analysis.json` sidecar gives you machine-readable features for your own prompt notebooks, versioning, or further automation (e.g. your `ig-automator` or future `BookHunter`-style tools).

## Philosophical Alignment

This tool embodies the Guidebook’s core thesis: **Suno is not magic; it is a constrained generative system whose output quality is a direct function of how precisely you specify physics, timing, negative space, and production moves.** By closing the loop from real audio back to that specification language, Suno Prompt Forge accelerates the very workflow the Guidebook teaches.

Built as cognitive augmentation for an autodidactic polymath workflow.

---

**Author of the underlying Guidebook**: Metta Thomas  
**This implementation**: June 2026 (current time context)  
**License**: Same spirit as the open-source repos in the user’s GitHub (IAmM3ta) — use, modify, extend, but respect the IP notice on the original Guidebook if you redistribute the templates verbatim.

Run it. Break it. Improve the mapping rules. Feed the results back into your own Suno + SSML practice. That is the intended loop.