# Paste this into Claude Code on the Mac

Open Terminal, `cd` into the `dutch-practice` repo (pull the latest first), run `claude`, and paste everything below the line.

---

I want to turn this repo's web app (`index.html`, Spreekpraktijk, a Dutch speaking-practice app) into a native app for iPhone, iPad and Android, using Capacitor.

Read `docs/superpowers/specs/2026-10-29-mobile-app-design.md` first. It is the approved design: the look and feel ("Ink": dark-first tiles + chips + purple-blue accent, follows the system light/dark setting), the decisions already made, the non-goals, and the calls I (the previous session) made. Follow it. Don't re-ask what it already answers.

Summary of what matters:
- Keep the existing app logic (scenarios, AI providers, corrections, summary, system check). Redesign the UI for phones/tablets, and replace browser speech with native speech recognition and text-to-speech (Dutch, nl-NL).
- Own use. No paid Apple account, so iOS is installed from Xcode with a free Apple ID. Android can also be shared as an APK.
- The first build must run on an iPhone simulator, iPad simulator and Android emulator.

Process:
1. Check this Mac has Xcode, Xcode command line tools, Node, CocoaPods, Android Studio and a JDK. Tell me what is missing and how to install it; don't install big things without asking.
2. Use `superpowers:writing-plans` to write the implementation plan from the design document, then show it to me before building.
3. Build in small steps, running the app in a simulator after each one. Show me screenshots of the picker and chat in dark and light mode on iPhone and iPad.
4. Ask me only for things the document marks as open (bundle id name) or that you truly can't decide.

Start with step 1.
