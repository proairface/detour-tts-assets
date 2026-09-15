# detour-tts-assets

Mirrored, checksum-verified copies of the binary assets [Detour](https://github.com/proairface/detour)'s
on-device neural voice feature depends on. This repo exists for one reason: **Detour itself is
private, so its own GitHub Releases aren't fetchable without authentication** — a plain,
unauthenticated download URL 404s on a private repo, which breaks both a Gradle build dependency
and, more importantly, an end user's phone downloading a voice at runtime. This repo is public
specifically so those URLs work for both.

Almost everything here is a straight mirror of a third party's own release, re-published after
its SHA-256 is checked against a value pinned in
[`.github/workflows/publish-tts-assets.yml`](.github/workflows/publish-tts-assets.yml) — the one
exception is the two `previews/*.wav` clips, which are generated locally and committed directly
(see below).

## What's here (release `tts-assets-v1`)

- `en_US-ryan-high.onnx` / `.onnx.json` — English voice, from
  [rhasspy/piper-voices](https://huggingface.co/rhasspy/piper-voices) on Hugging Face
  (`en/en_US/ryan/high`). ⚠️ Its own `MODEL_CARD` names the training dataset's license as
  **CC BY-NC-SA 4.0 (NonCommercial)** — not MIT as earlier notes here claimed; that blanket "Piper
  voices are MIT" claim wasn't checked per-voice at the time. Since Detour is commercially
  distributed, this is under the app owner's review (see `docs/PROJECT-STATE.md` in the detour
  repo) rather than silently resolved here either direction. Note this is also a separate
  question from the now-GPL-3.0 `piper1-gpl` *code* repo's license — the model weights aren't
  affected by that regardless of how the dataset question resolves.
- `nl_NL-pim-medium.onnx` / `.onnx.json` and `nl_NL-alex-medium.onnx` / `.onnx.json` — two
  single-speaker Dutch voices, **CC0** (confirmed against each voice's own real `MODEL_CARD`),
  same source (`nl/nl_NL/pim/medium`, `nl/nl_NL/alex/medium`). Both offered in the app; the
  driver picks. Replace the single multi-speaker `nl_NL-mls-medium` voice this repo mirrored
  previously — a real by-ear comparison found every one of that voice's 52 speakers sounded
  "artificial and robotic," while both of these single-speaker voices sounded clearly better.
  `pim` is also the exact voice
  [KotlinG2P](https://github.com/proairface/KotlinG2P)'s `DutchG2P` phonemizer was itself tuned
  and by-ear-verified against during its own development.
- `nl_NL-pim-medium-preview.wav` / `nl_NL-alex-medium-preview.wav` — short (2-4 second) preview
  clips for Detour's voice-picker UI, so a driver can hear a voice before committing to a ~60 MB
  download of the real thing. **Not mirrored** — generated locally via a scratch `onnxruntime`
  harness running the real `DutchG2P` phonemizer against these exact voice weights, then
  committed straight into [`previews/`](previews/) in this repo. No upstream source or checksum
  to verify these against; this repo *is* the source, same as any other committed file.
- `CHECKSUMS.txt` — SHA-256 of every file above.

See the release page itself for the exact source links and checksums for the files actually
published.

## Publishing / updating

`workflow_dispatch` only — these files are essentially immutable once a voice is chosen, so
there's nothing here that benefits from running on every push. Re-run the workflow (Actions tab)
if a URL or pinned checksum ever needs to change; it re-publishes the same fixed tag.

## Consumed by

- Detour's own `VoiceDownloadManager` at runtime (see `docs/PROJECT-STATE.md` in the
  [detour](https://github.com/proairface/detour) repo) — a driver downloads a chosen voice
  straight from this repo's Release assets, checksum-verified again on-device before use.
- Detour's voice-picker UI downloads the small preview clips the same way, separately from (and
  before) the full voice a driver eventually chooses to install.
