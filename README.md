# slumegle

**[Try it live](https://raw.githack.com/jay23606/p2p-webcams/main/index.html)**

A minimal Omegle-style random video + text chat. Two strangers who open the same
URL get paired automatically; one button cycles you to the next stranger.

Built in the same spirit as [p2p-webcams](https://github.com/jay23606/p2p-webcams)
and [p2p-chat](https://github.com/jay23606/p2p-chat): one HTML file, plain JS, a few
tiny helper methods, no framework, no build step.

## Features

- Random one-to-one video pairing between strangers.
- Single **Start / Next** button — Start joins the pool, then the same button becomes Next to skip to someone new.
- **Stop** leaves entirely and resets the button back to Start.
- Live **"N online"** count of everyone currently in the app.
- Text chat that overlays the video as a translucent panel, **hidden by default** (most people just want video). A **Chat** button toggles it, and it flashes an unread indicator if a message arrives while closed.
- Chat history clears automatically each time you connect to a new stranger.
- **Connection status** feedback (connecting / connected / unstable / failed), a **"still looking…"** nudge if no match appears within 30s, **Esc** as a shortcut for Next, and a **"Stranger is typing…"** indicator.
- Mobile-friendly layout: the stranger's video fills the screen, your own self-view is a small overlay in the corner, and everything reflows on rotation. Video uses `contain`, so the full camera frame is always shown (letterboxed rather than cropped).

## How it works

- **PeerJS** handles the actual peer-to-peer video and the text `DataConnection` — media and messages flow directly between the two browsers, never through a server.
- **Firebase Realtime DB** is used only for coordination, in two small nodes:
  - `slumegle/lobby` — a pool of peers currently *waiting* to be matched. A waiting peer advertises its PeerJS id here; the next person to arrive claims that slot (removing it) and calls them. If nobody is waiting, you become the one waiting.
  - `slumegle/online` — everyone currently present, used only to compute the live count.
- **Next** removes you from any pairing, sends your partner a `__next__` sentinel so they instantly re-queue, and drops you back into the lobby to find someone new.
- `onDisconnect()` plus a `beforeunload` handler clean up both your lobby and presence entries if you close the tab or lose connection.

## Run it

It's a single static file — open `index.html` directly, or serve it:

```
npx serve .
```

Live (via githack): **[raw.githack.com/jay23606/p2p-webcams](https://raw.githack.com/jay23606/p2p-webcams/main/index.html)**

To test locally, open it in **two tabs** or two browsers. One waits, the other
matches it. Each tab needs its own camera/mic permission.

## Config

The demo points at the same public Firebase URL used by p2p-webcams. To run your
own, replace `databaseURL` in `firebaseConfig` and make the `slumegle` node
world read/write in your Realtime DB rules:

```json
{
  "rules": {
    "slumegle": { ".read": true, ".write": true }
  }
}
```

## Limitations / ideas

- **One-to-one only**, by design (classic Omegle). Group mode would mean tracking multiple simultaneous calls, the way p2p-webcams does.
- **No persistent identity.** PeerJS assigns a fresh random id every page load, so there are no accounts and no stable way to recognize a returning stranger. A blocking feature, for example, could only be session-scoped unless real backend identity is added.
- **No moderation.** This is a demo. An unmoderated public stranger-video service carries real safety and abuse obligations — think hard about that before pointing it at the open internet.
- Camera + mic permission is required; denying it just shows a "blocked" status.
- Autoplay of remote audio relies on the user having clicked Start first (which satisfies the browser's interaction requirement).
