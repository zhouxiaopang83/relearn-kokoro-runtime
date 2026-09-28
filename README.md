# relearn-kokoro-runtime

Runtime payload for the free Mandarin TTS (Kokoro) that **OpenMAIC Desktop**
provisions on the user's own machine. The app does not ship this payload; it
downloads it from this repository's newest release, verifies it, and hands it
to the local Docker engine.

This repository exists separately from the app's update repository on purpose:
GitHub's `releases/latest` resolves to the newest published release, so a
payload release published next to the app's would silently take over the
installer's update feed.

## Assets

| Asset | What it is |
| --- | --- |
| `kokoro-tts-cpu-v1.1-zh.tar` | `docker save` output of the CPU image `openmaic-kokoro-tts:1.1-zh`, which already contains the `hexgrad/Kokoro-82M-v1.1-zh` weights and the 103 v1.1-zh voice packs. Loading it requires no GitHub or HuggingFace access. |
| `SHA256SUMS.txt` | sha256 of the tar, so a download can be verified independently of the app. |

## Contract

- The app pins this payload by **url + sha256 + byte size + image tag** in its
  committed `desktop/resources/kokoro-runtime.json`. A payload whose bytes do
  not match that manifest is rejected before `docker load` runs.
- The image tag inside the tar is app-owned (`openmaic-kokoro-tts:1.1-zh`); the
  upstream `kokoro-fastapi-cpu-kokoro-tts` name is not relied upon at runtime.
- Verified contract of a loaded payload: the container serves exactly 103
  voices on `GET /v1/audio/voices`, accepts model id `kokoro`, defaults to
  voice `zf_001`, and returns a non-empty WAV from `POST /v1/audio/speech`.
- The container listens on `8880`, the port the app's shipped provider config
  (`server-providers.yml`) already names.

## How a payload release is produced

From the app repository's `desktop/` directory:

```
npm run kokoro:pack      # docker save + manifest, from the verified local image
npm run kokoro:verify -- <path-to-tar>   # contract check against a live Docker
```

Then create a release in this repository with a version tag (for example
`kokoro-v1.1-zh-1`), attach the tar and `SHA256SUMS.txt`, and publish. Bump the
app's `desktop/resources/kokoro-runtime.json` to the new url/sha256/size in the
same change that needs it.

## Licenses

Kokoro-FastAPI is Apache-2.0 and the `hexgrad/Kokoro-82M-v1.1-zh` model is
Apache-2.0. Neither Docker Desktop nor any Docker installer is redistributed
here: the app links to Docker's official download page and the user installs it
themselves.
