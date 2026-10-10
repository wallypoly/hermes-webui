# Jophael World · Hermes Desktop backend and voice research
Research date: 2026-10-11 | Status: documentation audit, **not tested against Wallance's actual running Hermes instance**.

## Executive finding

Official Hermes Desktop is Electron + React, backed by the **same Hermes Agent runtime** as CLI and gateway. It launches `hermes serve` and speaks JSON-RPC over WebSocket. In contrast, this repository's current **Hermes WebUI** voice input is browser Web Speech API only; its TESTING.md marks `/voice` unsupported. These are different products and cannot be assumed feature-equivalent.

**Recommendation:** continue the isolated Jophael visual prototype here while reusing Hermes Agent's existing profile/configuration and adding a narrowly scoped **authenticated server-side voice bridge**. If the target is native desktop only, consider the official Hermes Desktop Plugin SDK instead of patching native internals. Never duplicate the identity or memory store.

## Existing official settings

| Feature | Key / screen | Notes |
| --- | --- | --- |
| Main model and effort | Desktop Settings → Model / provider | Per-profile default; chat picker is session-local |
| Voice turn model | `auxiliary.voice_chat` / Models → Auxiliary models | Chained voice can use a faster model, with tools/memory still available |
| STT | `stt.*` / Settings → Voice | `local`, `groq`, `openai`, etc. |
| Live STT | `stt.streaming: true` | OpenAI/xAI/ElevenLabs and streaming-capable plugins; not local Whisper |
| TTS | `tts.*` | Eleven providers, including keyless Edge and local Piper |
| Voice call behaviour | `voice.*` | Silence timeout, stop phrases, interruption; desktop-specific chat mode |
| Profile | `hermes -p jophael ...` | Separate config, credentials, SOUL, sessions, memory |
| Remote backend | `hermes serve` / Desktop Gateway settings | Authentication required beyond loopback |
| Desktop plugin | `$HERMES_HOME/desktop-plugins/<id>/plugin.js` | Native UI extension uses `@hermes/plugin-sdk` |

### Chained vs GPT-Live

1. `voice.voice_chat_mode: chained` (default): audio → STT → Hermes model turn → TTS. Existing identity, tools, memory and dialogue history remain the core.
2. `voice.voice_chat_mode: gpt-live`: OpenAI full-duplex `gpt-live-1` handles conversational audio and delegates real work to Hermes. Needs direct OpenAI API key and bills the voice session separately. This speech layer may paraphrase Hermes outputs. Not the safest first choice for a character with a carefully maintained voice/persona.

### Remote desktop client-direct audio

Current docs describe `GET /api/audio/voice-config`, which can give the official Desktop ephemeral provider config to call some cloud STT/TTS providers directly. The fallback is gateway relay via `/api/audio/transcribe` and speech WebSocket. `voice.client_direct: false` forces relay. **Do not blindly expose the credential-bearing voice-config route to a browser or public custom UI.** The official Desktop treats keys as ephemeral in desktop memory; a web app needs its own authenticated backend and suitable browser security model.

## Suggested zero/low-subscription-cost Jophael trial

**Modify only Jophael's already-existing configuration, after making a backup. Do NOT replace its whole config.yaml.**

```yaml
# Merge these keys into ~/.hermes/profiles/jophael/config.yaml
stt:
  enabled: true
  provider: local
  language: ""            # auto-detect; test Mandarin + English + Malay clips
  local:
    model: base
    vad: true

tts:
  provider: edge
  speed: 1.0
  edge:
    voice: zh-CN-YunxiNeural  # Mandarin male voice; test actual results

voice:
  voice_chat_mode: chained
  client_direct: false    # useful when experimenting with gateway relay
  barge_in: true

auxiliary:
  voice_chat:
    provider: auto
    reasoning_effort: none
```

Notes:
- Local Faster-Whisper is free/offline *after downloading its model*, but needs the supported runtime/dependency; Hermes Tools manages built-in extras. If initial transcription is slow, try `tiny` before moving to paid cloud STT.
- The default `stt.language: "en"` can hurt Chinese/Malay transcription. `""` requests autodetection, but short code-switched utterances still need real testing. If speech is mainly Chinese, `zh` may perform better.
- Edge TTS needs network access even though it requires no API key. Its `zh-CN-YunxiNeural` Mandarin male voice and `ms-MY-OsmanNeural` Malay male voice are listed by Microsoft; a voice selected in config is not guaranteed to sound natural when switching languages mid-sentence.
- Piper is local and keyless but has a narrower practical set of useful voice models; check actual Malaysian Mandarin/Malay quality before locking it in.
- The Desktop's **Read replies aloud** toggle is a desktop-local preference, not simply `voice.auto_tts` from the gateway.
- This experiment has not installed any voice model and has not accessed Jophael's real profile.

## What to implement in this WebUI fork

1. **Read-only profile diagnostics** first: confirm exact Jophael profile, backend version, chosen STT/TTS, required dependencies, and whether the server-side audio routes exist. Hide all secrets.
2. **STT input**: microphone capture with clear mic permissions / explicit start-stop; authenticated upload to backend transcription route, with a text transcript before sending. Fallback to text if unavailable.
3. **TTS output**: route assistant streaming text to backend speech adapter; buffer for natural chunks; audio controls for play/pause/mute/interrupt.
4. **Voice chat mode**: combine the above into turn-taking with barge-in; add full-duplex premium provider only if explicitly enabled.
5. **Settings UI**: display provider/model/voice/language/speed, mic speaker check, voice sample preview. Write profile config only through validated backend and backup logic. Keep raw keys off the client.
6. **Character animation**: drive limited avatar state `idle / listening / thinking / speaking / interrupted` from real events, not synthetic made-up replies.
7. **Phone access**: prefer VPN/Tailscale + secure authentication, never public raw `hermes serve` with basic auth alone. Note that native Desktop Plugin SDK modules cannot run unchanged in mobile web browsers.

## Backend safety

- Avoid coupling Jophael to the global/default profile; use explicit profile routing and a check in every write path.
- Do not allow arbitrary filesystem/config access from the chat-facing web surface.
- Store credentials in Jophael's Hermes secret store, not in GitHub, JavaScript, localStorage or query strings.
- Private SOUL, journals and images should remain in a separately controlled private storage/import path.
- A Hermes *profile* isolates state but is **not a filesystem sandbox**. Agent tools may still access the OS user's home unless separately restricted.
- This is research, **not proof that the custom prototype can talk to the user's live Hermes yet**.

## Primary sources

- Hermes Desktop https://hermes-agent.nousresearch.com/docs/user-guide/desktop
- Voice Mode https://hermes-agent.nousresearch.com/docs/user-guide/features/voice-mode
- Voice & TTS https://hermes-agent.nousresearch.com/docs/user-guide/features/tts
- Configuration https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- Profiles https://hermes-agent.nousresearch.com/docs/user-guide/profiles
- Profile Commands https://hermes-agent.nousresearch.com/docs/reference/profile-commands/
- Hermes Desktop Plugin SDK https://hermes-agent.nousresearch.com/docs/developer-guide/desktop-plugin-sdk
- Microsoft voices https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support
- Local repository README.md (Voice input) / TESTING.md (deferred /voice)
