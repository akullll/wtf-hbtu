# Campus Video Chat — Run Tonight, Free

Random 1-on-1 video/audio chat (Omegle-style). Video/audio streams peer-to-peer
via WebRTC — the server only pairs people up and relays connection setup info,
so it stays cheap and fast even with real-time video.

## Run locally to test
```bash
npm install
npm start
```
Open http://localhost:3000 in two different browser tabs (or two devices) to test pairing.

## Deploy for free tonight (pick one)

### Option A — Render.com (recommended, real public URL)
1. Push this folder to a GitHub repo (see "Updating the code" below if you're
   not using git commands — the website editor works fine).
2. Go to render.com → New → Web Service → connect the repo.
3. Build command: `npm install`   Start command: `npm start`
4. Free tier deploys in a couple minutes, gives you a public `https://...onrender.com` URL.
5. Render auto-redeploys every time you commit a change on GitHub — no extra steps needed after the first setup.

### Option B — Railway.app
Same idea: connect repo, it auto-detects Node, deploys, gives a public URL. Free tier has some monthly credit — plenty for one night with ~100 users.

### Option C — ngrok (zero deploy, runs off your own laptop)
```bash
npm install
npm start
# in another terminal:
ngrok http 3000
```
ngrok gives you a public URL that tunnels to your laptop. Simplest for tonight,
but your laptop has to stay on and awake, and free ngrok URLs are randomly
generated each time you restart it.

**Note:** browsers require HTTPS (or localhost) for camera/mic access.
Render/Railway give you HTTPS automatically. ngrok's free URLs are HTTPS too, so both work.

## Updating the code without git commands

If you set this repo up by dragging files onto github.com (no local git), keep doing it that way:
1. Open the file on github.com (e.g. `index.html`).
2. Click the pencil/edit icon in the top right.
3. Select all, delete, paste in the new version.
4. Scroll down, click **Commit changes**.
5. Render/Railway picks up the commit and redeploys automatically.

## TURN server — read this if connections are unreliable

WebRTC needs a **STUN** server (to discover a public IP) and, for stricter
networks like mobile data or CGNAT wifi, a **TURN** server (to relay media
when a direct connection can't be made). Both are configured in the
`ICE_SERVERS` array near the top of `index.html`.

**Don't use the shared public demo TURN server**
(`openrelay.metered.ca` / `openrelayproject`). It's free but shared by
thousands of unrelated projects, so it gets overloaded and connections
intermittently hang or fail — especially on mobile data, which *requires*
TURN and can't fall back to a direct connection.

**Use your own free TURN project instead:**
1. Sign up at [dashboard.metered.ca/signup](https://dashboard.metered.ca/signup) (free, ~50GB/month).
2. Create a credential in the TURN Server section of the dashboard.
3. Copy the `iceServers` code block it gives you.
4. Paste it in as `ICE_SERVERS` in `index.html`, replacing the existing array.

The app already includes automatic retry logic (`onconnectionstatechange` →
`restartIce()`) for when a TURN allocation is briefly slow rather than
permanently unreachable, so a failed first attempt isn't necessarily final.

**To sanity-check your TURN server on its own** (isolate it from the rest of
the app), use [webrtc.github.io/samples/.../trickle-ice](https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/)
with your exact URLs/credentials and confirm you get a `relay` candidate back.

## Important things to know

- **~100 concurrent users is easily fine.** The server does almost no work — it's
  just a matchmaking queue and small signaling messages, not video. Video never
  touches your server.
- **Switching hosting providers (Render/ngrok/Cloudflare) does not affect
  WebRTC connection reliability.** Those only affect whether people can reach
  the matchmaking server; the actual video/audio is peer-to-peer and depends
  entirely on STUN/TURN, not on where the signaling server is hosted.
- **No moderation exists yet.** With zero content filtering, expect chaos —
  add a simple "report" button or a rule at the top of the page if you want
  some guardrails. Consider requiring a college email or student ID check if
  you want to keep it campus-only, since the link could otherwise spread
  beyond your campus.
- **Camera/mic permissions:** users must click "Allow" when the browser prompts them.
- **No database, no accounts, no chat history stored** — fully ephemeral by design.

## How it works (quick mental model)
1. User clicks Start → grabs their camera/mic → tells server "find me a partner."
2. Server keeps one waiting user in memory; when a second user arrives, it pairs them.
3. Server relays a WebRTC "offer/answer" handshake between the two browsers.
4. Once connected, video/audio flows directly between the two browsers (peer-to-peer) — the server steps out of the way.
5. If the connection briefly fails, the client automatically retries via ICE restart (up to 2 times) before asking the user to hit Skip.
6. "Skip" disconnects and re-queues.
