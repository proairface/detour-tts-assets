# detour-tts-assets

Mirrored, checksum-verified copies of the binary assets [Detour](https://github.com/proairface/detour)'s
on-device neural voice feature depends on. This repo exists for one reason: **Detour itself is
private, so its own GitHub Releases aren't fetchable without authentication** — a plain,
unauthenticated download URL 404s on a private repo, which breaks both a Gradle build dependency
and, more importantly, an end user's phone downloading a voice at runtime. This repo is public
specifically so those URLs work for both.

## `voice-catalog.json` — the modular-TTS manifest

A plain, directly-committed (not release-published) JSON file listing every voice Detour's
`VoiceCatalog` can offer — the same shape as that Kotlin file's own hardcoded `bundledDefaults`
list, which stays as the safe fallback. Detour's `VoiceCatalogUpdater` fetches this file
unauthenticated via `raw.githubusercontent.com` (no release, no workflow, no checksum-pinning
step of its own — a plain `git commit` to this file is the entire publish step) and merges its
entries into the running app's voice catalog, keyed by (`languageTag`, `id`). This is what lets a
new or updated voice — a corrected checksum, a renamed display label, a new picker entry —
reach installed copies of Detour with **no app release at all**, since `detour` being private is
exactly what blocks Detour from checking its own GitHub Releases for an update in the first
place (see `AppInfoViewModel.releasesUrl`'s doc comment in that repo for the separate,
browser-hand-off mechanism that covers actual app updates).

**This only ever changes which voices the app *offers* and where to download them from — never
the TTS engine itself.** Every `modelUrl`/`configUrl`/`previewUrl` here must still point at a
real `detour-tts-assets` release asset (or a `raw.githubusercontent.com` URL under this same
repo) — Detour's own `VoiceCatalogUpdater` drops any entry whose URLs don't, and
`VoiceDownloadManager` still SHA-256-verifies every byte it downloads before ever touching
onnxruntime, exactly as it always has. Editing this file doesn't change what those checksums are
allowed to verify against.

To add or change a voice: publish/verify the actual binary the normal way (see "Publishing /
updating" below for the release-asset half, unchanged), then edit this file directly to add or
update its entry, matching the shape of `com.detour.app.data.voice.RemoteVoiceOption` in the
`detour` repo. Keep `com.detour.app.data.voice.VoiceCatalog`'s own `bundledDefaults` in sync too
when a change is meant to be permanent, not just an early rollout — this manifest is a fast path
for reaching already-installed apps, not a replacement for the bundled list new installs still
ship with.

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
- `en_US-libritts-high.onnx` / `.onnx.json` — a **second** English option, offered *alongside*
  `ryan-high`, not a replacement for it (the app owner hasn't decided on `ryan-high`'s own
  license question — see above). One 904-speaker Piper voice, **CC BY 4.0** (confirmed against
  its own real `MODEL_CARD`), trained from scratch on LibriTTS (OpenSLR/60) — no `lessac`-derived
  taint, a cleaner license than `ryan-high`'s. Same source (`en/en_US/libritts/high`). Detour
  offers 3 of its 904 speakers as separate voice-picker entries — `f3` (`p6341`), `m6` (`p5239`),
  `f9` (`p6206`) — picked by ear across several rounds of random sampling filtered to
  LibriSpeech's cleaner training subsets (see `docs/PROJECT-STATE.md` in the detour repo for the
  full selection method and the by-ear verdicts, including why a 4th candidate, `m9`, was
  dropped). All 3 read the *same* `en_US-libritts-high.onnx`/`.onnx.json` pair, differing only by
  the `sid` (speaker id) each sends to the ONNX graph at inference — Detour's
  `VoiceOption.sharedAssetId` is what lets its `VoiceDownloadManager` download this one ~130 MB
  file once and reuse it across all 3 picker entries, rather than a driver paying for the same
  bytes three times over.
- `nl_NL-pim-medium-preview.wav` / `nl_NL-alex-medium-preview.wav` /
  `en_US-libritts-high-f3-preview.wav` / `en_US-libritts-high-m6-preview.wav` /
  `en_US-libritts-high-f9-preview.wav` — short (2-4 second) preview clips for Detour's
  voice-picker UI, so a driver can hear a voice before committing to the full download. **Not
  mirrored** — generated locally via a scratch `onnxruntime` harness running the real `DutchG2P`/
  `G2P` phonemizer against these exact voice weights (the LibriTTS previews with each speaker's
  own `sid`), then committed straight into [`previews/`](previews/) in this repo. No upstream
  source or checksum to verify these against; this repo *is* the source, same as any other
  committed file.
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
