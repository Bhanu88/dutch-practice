# Improvements

## Better Dutch speech recognition (overcome browser limits)

The browser `SpeechRecognition` API is weak on Dutch, can't take vocabulary hints, and ends on pauses. Record audio with `MediaRecorder` (already used for the volume meter) and send it to a transcription model instead.

Options, best first:

1. **Gemini audio transcription** (recommended)
   - Send the clip as inline audio to the existing `generateContent` endpoint, prompt: "Transcribe this Dutch speech verbatim".
   - No new key or backend. Good Dutch. Scenario vocabulary can go in the prompt.
2. **OpenAI `gpt-4o-transcribe` / Whisper** (`/v1/audio/transcriptions`, `language=nl`, `prompt` for hints)
   - Most accurate for Dutch, about $0.006/min. Needs an OpenAI key in the browser.
3. **Local Whisper in the browser** (transformers.js, `whisper-small`)
   - Free and private, but 250 MB+ download, slow on phones, less accurate.

Trade-offs:
- No live preview; about 1s delay after pressing stop.
- Keep the browser recognizer as a fallback or toggle.
- Safari records `audio/mp4`, Chrome `audio/webm`; both work with Gemini and OpenAI.

Implementation sketch: add `transcribeAudio(blob)` and call it from the record button's stop handler.

## Download the conversation as voice (plus text)

The text export already exists. For audio, the two sides are not equally easy:

- **Student side (spoken answers):** easy. The mic stream is already open, so record each spoken turn with `MediaRecorder`, keep the blobs in order, and offer one joined audio download next to the text export. Typed turns have no audio.
- **Character side:** the app uses browser `speechSynthesis`, and browsers don't let a page capture that audio. Needs a real TTS API (Gemini TTS or OpenAI TTS). Costs per reply and needs that service's key.

Plan: start with the student side only, add the character side later if wanted.

Open question: which provider's key to use for character TTS.
