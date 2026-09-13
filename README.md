# detour-tts-assets

Mirrored, checksum-verified copies of the binary assets [Detour](https://github.com/proairface/detour)'s
on-device neural voice feature depends on. This repo exists for one reason: **Detour itself is
private, so its own GitHub Releases aren't fetchable without authentication** — a plain,
unauthenticated download URL 404s on a private repo, which breaks both a Gradle build dependency
and, more importantly, an end user's phone downloading a voice at runtime. This repo is public
specifically so those URLs work for both.

Nothing here is built from source — everything is a straight mirror of a third party's own
release, re-published after its SHA-256 is checked against a value pinned in
[`.github/workflows/publish-tts-assets.yml`](.github/workflows/publish-tts-assets.yml).

## What's here (release `tts-assets-v1`)

- `sherpa-onnx-<version>.aar` — the [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) Android
  runtime, **Apache-2.0**. No Maven Central artifact exists for it (confirmed via a direct Maven
  Central search API query) — k2-fsa ships it only as a GitHub Release asset.
- `en_US-ryan-high.onnx` / `.onnx.json` — English voice, **MIT**, from
  [rhasspy/piper-voices](https://huggingface.co/rhasspy/piper-voices) on Hugging Face
  (`en/en_US/ryan/high`). Note this is a separate license from the now-GPL-3.0 `piper1-gpl`
  *code* repo — the model weights were never relicensed.
- `nl_NL-mls-medium.onnx` / `.onnx.json` — Dutch voice, **MIT**, same source
  (`nl/nl_NL/mls/medium`) — medium is the highest tier Piper currently publishes for Dutch.
- `CHECKSUMS.txt` — SHA-256 of every file above.

See the release page itself for the exact source links and checksums for the files actually
published.

## Publishing / updating

`workflow_dispatch` only — these files are essentially immutable once a voice is chosen, so
there's nothing here that benefits from running on every push. Re-run the workflow (Actions tab)
if a URL or pinned checksum ever needs to change; it re-publishes the same fixed tag.

## Consumed by

- Detour's Gradle build, as an ivy-repository dependency (see `settings.gradle.kts` in the
  [detour](https://github.com/proairface/detour) repo) — resolves `sherpa-onnx-<version>.aar`
  directly from this repo's Release assets.
- Detour's own voice-download manager at runtime, once built (see `docs/PROJECT-STATE.md` in
  that repo) — a driver downloads a voice from here the same way, just from the app itself
  instead of Gradle.
