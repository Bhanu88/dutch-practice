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

## More natural bot voice

The browser's built-in `speechSynthesis` quality depends on the device. The app picks the best Dutch voice it finds (neural/"Natural", Premium/Enhanced, then Google).

For a clearly human-sounding voice, use a TTS API instead. An optional paid "Natural voice" button was built and then removed; notes for next time:

- **Gemini TTS** (`gemini-3.8-flash-lite-tts`): needs a Gemini key. About $0.0015 per typical 10-second reply today, about $0.003 from 2027 (25 audio tokens per second, $6 per 1M now, $12 from 1 Jan 2027). Request shape follows Google's docs (`/v1beta/interactions`, `speech_metadata` style) and was never tested with a real key.
- **OpenRouter `openai/gpt-audio-mini`** (tried): a chat model, not a TTS engine, so it can answer instead of reading. Needs streaming with `modalities: ["text","audio"]` and `pcm16` chunks wrapped in a WAV header. Compare the returned transcript with the on-screen text and fall back to the browser voice if it differs. No clear audio price found.
- **OpenAI TTS (`gpt-4o-mini-tts`)**: a real TTS endpoint with a style instruction; needs an OpenAI key.
- Design that worked: off by default, cost warning when switched on, key entered only in the key popup, audio cached per reply, fallback to the browser voice on error.
- Safari/iPad only plays audio that starts after a network wait if the same `<audio>` element was started during a tap, so unlock one shared element on the first gesture.
- Real TTS audio can also be saved for the voice-download idea above.
