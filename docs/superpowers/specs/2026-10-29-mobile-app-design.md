# Spreekpraktijk mobile app: design

## Intent

Spreekpraktijk (`index.html`, single file, vanilla JS) is a Dutch speaking-practice web app: pick a scenario, talk to an AI character, get live corrections. It works on desktop Chrome but is poor on phones, and browser speech recognition is unreliable on iOS. The owner wants it as a real native app for iPhone, iPad and Android, for their own devices and a few friends. Not the App Store / Play Store for now.

The core flow on a phone: **open app, tap a scenario, start talking.** Everything else stays out of the way.

## Goals

- Installable native app on iPhone, iPad, Android from one codebase.
- Minimalist, modern look (direction "Ink", below). Follows the phone's light/dark setting.
- Voice input and spoken Dutch replies that work reliably on iOS and Android (native plugins, not browser Web Speech).
- All existing behaviour kept: 32 scenarios in 6 groups, corrections/tip/suggestion after each message, session summary, download of conversation, system check, provider/model/API-key setup (OpenRouter, Gemini, Anthropic).
- Sideload/TestFlight distribution to a handful of friends.

## Non-goals

- No App Store / Play Store release, store listing, or review work now.
- No backend, accounts, sync or analytics. Everything stays on the device; the user's own API key is stored on the device and sent only to the chosen provider.
- No rewrite into a framework (React etc.) and no new scenarios or AI-logic changes.
- No Windows/desktop packaging.

## Anti-goals

- Not a "web page in a frame" feel: no browser-looking controls, no tiny desktop-sized tap targets, no pinch/zoom, no rubber-band scroll of the whole page, nothing hidden behind the notch or home indicator.
- Not a cluttered UI. If a feature needs a paragraph of explanation on screen, it is too much.

## Decisions made (by the owner)

- Native app, not a PWA.
- Own devices + a few friends only.
- Design direction **B "Ink"**: dark-first, rounded tiles, category chips, purple-blue accent.
- Theme follows the system setting (light and dark both supported).
- Owner has a Mac; design is finalised on Windows, build/run happens on the Mac.

## Look and feel (direction Ink)

**Picker screen**
- Large heading "Wat oefenen we?", short subtitle.
- Horizontal row of category chips: Social, Vertellen, Office, Praktisch, Shop & restaurant, Admin (existing groups; one selected at a time). Settings (cog) top right.
- Below: two-column grid of rounded scenario tiles (icon top-left, title bottom). Tile = existing scenario icon + Dutch title. Tap starts the chat immediately.
- iPad / wide screens: 3 to 4 columns, content max width around 900 px, chat column capped around 640 px and centred.

**Chat screen**
- Slim top bar: back arrow, scenario title, character name; icons for summary and download.
- Partner messages: dark-grey (light theme: light-grey) bubbles, left. User messages: purple-blue gradient bubbles, right.
- Corrections / tip / suggestion: small tinted notes under the relevant message, tappable suggestion that fills the input.
- Bottom: large round mic button with glow ring while listening, label "Tik om te praten". Tap to start, tap again to stop and send. A small keyboard icon reveals a text-input fallback.
- Respect safe areas (notch, home indicator, Android nav bar) and the on-screen keyboard.

**Tokens (starting point, builder may tune):**

| Token | Dark | Light |
|---|---|---|
| background | #0f1115 | #f6f7fb |
| surface (tiles, partner bubble) | #171a21 / #1a1d24 | #ffffff / #eceef4 |
| text | #f2f3f5 | #14161a |
| muted text | #8c93a0 | #5c636b |
| accent | #7c5cff to #5a7cff gradient | same |
| tip note | #1a1730 bg, #a99bff text | #efeaff bg, #4a37b8 text |

Typography: system font stack (SF on iOS, Roboto on Android) for UI; keep the existing serif (Newsreader) only if it earns its place, otherwise drop it. Fonts and icons must be bundled locally (the app has to launch offline into the picker).

A reference mockup of Ink (picker + chat, dark) is in `.superpowers/brainstorm/1206-1793284781/content/look-and-feel.html`.

## Technical approach (my calls, not agreed with the owner)

- **Capacitor** wrapping the existing web code. Keep `index.html` as the app; move it to a `www/` folder (with local fonts/icons, no CDN dependencies) and let Capacitor copy it into the iOS and Android projects.
- **Speech input:** `@capacitor-community/speech-recognition` with language `nl-NL`, replacing `webkitSpeechRecognition`. Keep the existing "keep listening through pauses" behaviour if the plugin allows.
- **Spoken replies:** `@capacitor-community/text-to-speech`, `nl-NL`, replacing `speechSynthesis`.
- **API key and settings:** `@capacitor/preferences` (or a secure-storage plugin for the key) instead of `localStorage`.
- **Network:** use Capacitor's native HTTP for the provider calls so CORS and the Anthropic browser-access header are not an issue.
- **System check screen:** adapt to native (mic permission, speech availability, Dutch voice, internet). Drop the "secure page" check.
- **Permissions:** microphone and speech-recognition usage strings in iOS `Info.plist` (Dutch practice context), RECORD_AUDIO on Android.
- **App identity:** name "Spreekpraktijk"; icon and splash in the Ink style (dark background, accent mark). Bundle id to be chosen when the Apple account is known (placeholder `app.spreekpraktijk`).
- **Platforms:** iOS 16+, iPadOS (universal app, all orientations), Android 8+ (API 26).

## Distribution

- Android: signed debug/release APK or AAB shared by link; friends enable install from unknown sources.
- iOS (decided by the owner): no paid Apple Developer account for now; the app is for the owner's own use. Install directly from Xcode onto the owner's iPhone and iPad with a free Apple ID (signing lasts 7 days, so re-run from Xcode weekly). Friends on iPhone are not supported in this phase. TestFlight is a possible later step if the owner buys the account; keep the project ready for it (clean bundle id, no hacks).
- Android: also covers any friends who want it.

## Deliverable of the first build

Running on an iPhone simulator, an iPad simulator and an Android emulator (then real devices): picker, chat with native mic + spoken replies, corrections, summary, settings/API-key flow, light and dark themes, safe areas handled.

## Left to the builder

Capacitor version, folder layout, exact plugin choices if a better-maintained one exists, how to structure the code split (keep one file or split CSS/JS), animation details, icon artwork, tile icon set (reuse existing Tabler icons bundled locally, or switch to emoji), loading and error states.

## Further calls (mine, from the builder check; owner to confirm while reading)

- **Existing features:** keep transcript alternatives picker (only if the plugin returns multiple results), live interim transcript, auto-send after a pause (off for long "Vertellen" scenarios), hint toggles, and saved progress. Drop: the mic volume meter (no second mic stream; recognition breaks), the manual light/dark toggle (system setting rules), and the 5-minute cache-buster/reload logic.
- **Speech risk:** native plugin first. If a plugin stops after about a minute or on silence, restart the session and stitch the text together. Whisper/cloud speech-to-text is not part of the first build.
- **Offline:** the app must launch offline into the picker; chat needs network. No chat history beyond what the app already keeps. First launch without a key shows the existing setup step.
- **API key storage:** Keychain / Keystore via a secure-storage plugin, not plain Preferences.
- **Tooling:** the builder works on the Mac, which needs Xcode and Android Studio plus a JDK; the plan includes checking and installing these. iOS-only behaviour (audio session, voices, permissions) is verified on the simulator and a real device.
- **Missing Dutch voice:** the system check shows a warning with a short "how to download a Dutch voice" hint; the app falls back to text-only replies.
- **Mockup:** indicative, not pixel-authoritative. The mockup file lives in `.superpowers/` (untracked), so its key points are in this document.
- **Network:** the app calls providers with plain request/response (non-streaming); the builder should confirm from the `fetch` calls and handle long timeouts for storytelling replies.

## Open items

- Bundle id: the builder proposes one (e.g. `nl.<owner>.spreekpraktijk`); free Apple ID signing needs a unique id, so ask the owner for a name on the Mac.
- Whether `spreekpraktijk.html` (untracked, older, 53 KB) is still wanted; assumed obsolete and ignored.
