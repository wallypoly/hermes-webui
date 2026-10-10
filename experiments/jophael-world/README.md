# Jophael World experimental UI

A private, standalone visual prototype for Jophael, built on a dedicated branch of `hermes-webui`.

## Try

Open `experiments/jophael-world/index.html` locally in a browser. No backend, dependencies or build step is needed for this prototype.

## Included

- Mobile-friendly pastel companion chat shell
- Character placeholder and world locations
- Journal and memory sections
- Local-only demo chat messages (browser localStorage)
- An integration note for connecting the existing Hermes WebUI backend to Jophael's *specific* profile

## Limitations

This is a UI prototype. It does **not** yet connect to a live Hermes agent. It does not fabricate Jophael responses. Real journals and images are deliberately excluded from the Git repository because they may contain private content.

## Next development step

Reuse Hermes WebUI's authenticated session and profile features; wire this UI to an authenticated, same-origin backend route locked to the Jophael profile. Add private journal/image import that stays outside tracked Git files. Build an avatar component independently.

Never commit tokens, SOUL files, raw chat transcripts, or the private Jophael archive.
