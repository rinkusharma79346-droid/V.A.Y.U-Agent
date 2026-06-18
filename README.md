<div align="center">

# V.A.Y.U Agent

### Vision-Assisted Yonder Unit

**An Android AI operator that does not just answer — it reaches into the phone, reads the screen, and acts.**

[![Android](https://img.shields.io/badge/Android-API%2028%2B-3DDC84?logo=android&logoColor=white)](#android-app)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.22-7F52FF?logo=kotlin&logoColor=white)](#tech-stack)
[![MCP](https://img.shields.io/badge/MCP-ready-ff7a18)](#mcp-remote-control)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=111)](#dashboard)
[![License](https://img.shields.io/badge/License-Apache--2.0-blue)](LICENSE)

> Give V.A.Y.U a mission. It captures the phone screen, understands the UI, chooses the next move, taps, swipes, types, waits, recovers, and keeps going until the job is done.

</div>

---

## The idea

Most assistants live behind a text box. V.A.Y.U lives on the glass.

It is built around a simple but powerful loop:

```text
see the screen → reason over the task → perform one Android action → observe again
```

That means prompts can become real phone operations:

- “Open Chrome and compare two products.”
- “Find this restaurant, copy the address, and save it in Notes.”
- “Open WhatsApp and send a short update.”
- “Scroll until the right button appears, tap it, and confirm the result.”
- “Let Claude Desktop control my Android device through MCP.”

V.A.Y.U combines an Android accessibility service, screenshot/UI-tree capture, model-backed planning, a relay server, and MCP tools into one phone-control stack.

---

## What makes it different

| Layer | What V.A.Y.U does |
| --- | --- |
| **Phone body** | Runs a native Android app with an accessibility service for taps, swipes, typing, navigation, screenshots, and app launching. |
| **Agent mind** | Sends screenshots, UI trees, task text, and action history to supported AI providers. |
| **Remote nervous system** | Keeps the device reachable through an HTTP long-polling relay. |
| **Desktop bridge** | Exposes phone actions as MCP tools for Claude Desktop, Cursor, Windsurf, and compatible clients. |
| **Operator console** | Includes a React/Vite dashboard prototype with a cinematic glass UI. |

---

## Repository map

```text
.
├── app/                  # Native Android app: accessibility service, HUD, settings, providers
├── vayu-relay/           # Main relay: device polling, MCP SSE, Streamable HTTP, sync tool bridge
├── mcp-server/           # Local Node MCP server for desktop clients
├── vayu-mcp-server/      # TypeScript MCP server variant
├── src/                  # React + Vite dashboard prototype
├── relay-server/         # Compatibility relay directory
├── jarvis-relay/         # Legacy Render rootDir compatibility alias
├── server.js             # Root fallback that starts the relay
├── render.yaml           # Render deployment config
├── codemagic.yaml        # Android CI build config
└── README.md             # This guide
```

---

## Core features

### Android app

- Accessibility-powered `TAP`, `LONG_PRESS`, `SWIPE`, `SCROLL`, `TYPE`, `OPEN_APP`, `BACK`, `HOME`, and `RECENTS` actions.
- Screenshot and accessibility tree capture for visual and semantic context.
- ReAct-style execution loop with action history to reduce repeated mistakes.
- HUD overlay for live task status.
- App settings for provider, model, API key, max steps, action delay, and relay URL.
- Clipboard-assisted text entry for stubborn Android/WebView fields.
- Direct UI helpers such as find-and-tap and type-in-field.

### Model providers

V.A.Y.U supports:

- Google Gemini
- OpenAI and OpenAI-compatible endpoints
- NVIDIA NIM
- Custom OpenAI-style base URLs

### Relay and MCP

- Device registration and long polling.
- Command queue and response collection.
- Health and warmup endpoints for hosted deployments.
- MCP over SSE and Streamable HTTP.
- Synchronous `/api/call` bridge for simple tool execution.
- Model recommendation endpoint for ranking available LLMs for agentic phone control.

### Dashboard

- React 19 + Vite 6 interface.
- Animated glass-morphism control-room design.
- Demo/prototype state for future live monitoring.

---

## Architecture

```text
┌────────────────────────────────┐
│ Android device                  │
│ V.A.Y.U app                     │
│                                │
│  Accessibility Service          │
│  Screenshot + UI tree capture   │
│  HUD overlay                    │
│  Provider clients               │
└───────────────▲────────────────┘
                │ long-poll / response
                ▼
┌────────────────────────────────┐
│ V.A.Y.U relay                   │
│ vayu-relay/server.js            │
│                                │
│  /api/register                  │
│  /api/poll                      │
│  /api/command                   │
│  /api/response                  │
│  /api/call                      │
│  /sse and /mcp                  │
└───────────────▲────────────────┘
                │ MCP
                ▼
┌────────────────────────────────┐
│ Desktop / web AI clients        │
│ Claude Desktop, Cursor, etc.    │
│                                │
│  vayu_look                      │
│  vayu_tap                       │
│  vayu_type_text                 │
│  vayu_sequence                  │
│  vayu_read_screen               │
└────────────────────────────────┘
```

---

## Quick start

### 1. Clone

```bash
git clone https://github.com/rinkusharma79346-droid/V.A.Y.U-Agent.git
cd V.A.Y.U-Agent
```

If you are using the legacy repository name, clone that URL instead and keep the same commands.

### 2. Configure Android secrets

Create `local.properties` in the repo root:

```properties
sdk.dir=/path/to/android/sdk
GEMINI_API_KEY=your_gemini_key_here
```

You can also enter keys in the app settings UI. Do **not** commit real API keys.

### 3. Build the Android app

```bash
chmod +x gradlew
./gradlew assembleDebug
```

Install it on a connected device:

```bash
adb install app/build/outputs/apk/debug/app-debug.apk
```

### 4. Enable phone permissions

On the Android device:

1. Open **V.A.Y.U**.
2. Grant required accessibility permission.
3. Grant overlay permission if prompted.
4. Choose provider and model.
5. Add your API key.
6. Set the relay URL if using remote control.

---

## Run the relay

The recommended relay lives in `vayu-relay/`.

```bash
cd vayu-relay
npm install
node server.js
```

Useful checks:

```bash
curl http://localhost:10000/api/health
curl http://localhost:10000/api/call
```

Root fallback is also available:

```bash
npm install
npm start
```

---

## Deploy on Render

Recommended Render service values:

| Setting | Value |
| --- | --- |
| Root Directory | `vayu-relay` |
| Build Command | `npm install` |
| Start Command | `node server.js` |

Compatibility root directories also exist:

- `jarvis-relay/`
- `relay-server/`
- blank root directory with `npm start`

After deploy:

```bash
curl https://YOUR-SERVICE.onrender.com/api/health
curl https://YOUR-SERVICE.onrender.com/api/call
```

---

## MCP remote control

### Claude Desktop / Cursor / Windsurf

Install the local MCP bridge:

```bash
cd mcp-server
npm install
```

Add this to your MCP client config:

```json
{
  "mcpServers": {
    "vayu": {
      "command": "node",
      "args": ["/absolute/path/to/V.A.Y.U-Agent/mcp-server/index.js"],
      "env": {
        "RELAY_URL": "https://YOUR-SERVICE.onrender.com"
      }
    }
  }
}
```

Then open the V.A.Y.U app on the phone and make sure the device is registered with the same relay.

---

## MCP tool families

### Coordinate actions

- `vayu_tap`
- `vayu_swipe`
- `vayu_long_press`
- `vayu_type_text`
- `vayu_press_back`
- `vayu_press_home`
- `vayu_press_recents`
- `vayu_open_app`
- `vayu_open_url`
- `vayu_open_chrome_url`
- `vayu_sequence`

### Observation

- `vayu_look`
- `vayu_screenshot`
- `vayu_ui_tree`
- `vayu_read_screen`
- `vayu_screen_text`

### Smart UI helpers

- `vayu_find_and_tap`
- `vayu_type_in_field`
- `vayu_wait_for_text`
- `vayu_scroll_to_text`
- `vayu_assert_element_visible`
- `vayu_open_url_and_wait`

### Device/session tools

- `vayu_status`
- `vayu_kill`
- `vayu_list_apps`
- `vayu_devices`

### Creative workflow helpers exposed by the relay

- `vayu_create_image`
- `vayu_create_video`
- `vayu_edit_in_capcut`
- `vayu_post_content`
- `vayu_quick_sequence`

---

## Synchronous API bridge

The relay can execute selected tools through one HTTP call.

Read the screen:

```bash
curl -X POST https://YOUR-SERVICE.onrender.com/api/call \
  -H "Content-Type: application/json" \
  -d '{"tool":"vayu_read_screen"}'
```

Find text and tap it:

```bash
curl -X POST https://YOUR-SERVICE.onrender.com/api/call \
  -H "Content-Type: application/json" \
  -d '{"tool":"vayu_find_and_tap","args":{"text":"Search"}}'
```

Type into a field:

```bash
curl -X POST https://YOUR-SERVICE.onrender.com/api/call \
  -H "Content-Type: application/json" \
  -d '{"tool":"vayu_type_in_field","args":{"fieldHint":"email","text":"hello@example.com"}}'
```

Recommend models:

```bash
curl -X POST https://YOUR-SERVICE.onrender.com/api/models/recommend \
  -H "Content-Type: application/json" \
  -d '{"provider":"openai","apiKey":"YOUR_KEY"}'
```

For hosted use, prefer setting `VAYU_LLM_API_KEY` as an environment variable instead of sending keys in request bodies.

---

## Dashboard

Run the dashboard prototype:

```bash
npm install
npm run dev
```

Build it:

```bash
npm run build
```

Type-check it:

```bash
npm run lint
```

---

## Configuration reference

### Android settings

| Setting | Purpose | Typical value |
| --- | --- | --- |
| API Provider | Selects Gemini, OpenAI, NVIDIA, or custom endpoint. | `gemini` |
| API Key | Provider credential. | app UI or `local.properties` |
| Model | Model used for action selection. | `gemini-2.0-flash` / `gpt-4o` |
| Base URL | Custom OpenAI-compatible endpoint. | `https://api.openai.com/v1` |
| Max Steps | ReAct loop limit. | `50` |
| Action Delay | Delay between actions. | `800ms` |
| Relay URL | Hosted relay endpoint. | Render URL |

### Relay environment

| Variable | Purpose |
| --- | --- |
| `PORT` | HTTP server port. Render usually injects this automatically. |
| `VAYU_LLM_API_KEY` | Optional server-side key for model recommendation calls. |

### MCP server environment

| Variable | Purpose |
| --- | --- |
| `RELAY_URL` | URL of the running V.A.Y.U relay. |

---

## Build and CI

### Android

```bash
./gradlew assembleDebug
```

### Web dashboard

```bash
npm run build
```

### Codemagic

`codemagic.yaml` is included for Android APK builds. Put provider secrets in Codemagic environment variables, not in the repository.

---

## Tech stack

| Area | Stack |
| --- | --- |
| Android | Kotlin, Android SDK, Accessibility Service, Coroutines, OkHttp, Gson |
| AI providers | Gemini, OpenAI-compatible APIs, NVIDIA NIM |
| Relay | Node.js, Express, CORS, MCP SDK |
| Desktop MCP | Node.js MCP stdio server |
| Dashboard | React 19, Vite 6, TypeScript, Tailwind CSS, Motion |
| Build | Gradle, Android Gradle Plugin, npm |
| Deployment | Render, Codemagic |

---

## Safety notes

V.A.Y.U can operate real apps on a real Android device. Treat it like a remote operator:

- Start with low-risk tasks.
- Watch the screen during sensitive flows.
- Avoid banking, payments, passwords, or irreversible actions unless you are supervising.
- Revoke accessibility permission when you are done testing.
- Keep API keys out of commits, screenshots, logs, and public issue threads.

---

## Troubleshooting

### The MCP client says no device is connected

- Open the Android app.
- Confirm the relay URL matches your MCP `RELAY_URL`.
- Visit `/api/devices` on the relay.
- Wake a free Render instance with `/api/warmup`.

### The agent repeats actions

- Lower max speed by increasing action delay.
- Use smart helpers like `vayu_wait_for_text` or `vayu_find_and_tap`.
- Prefer `vayu_sequence` for deterministic multi-step flows.

### Text entry fails

- Tap the field first.
- Try `vayu_type_in_field` with a field hint.
- Use Chrome/open URL macros for browser navigation.

### Render deploy uses the wrong folder

Set Root Directory to `vayu-relay`. If you cannot change it, the repo includes compatibility fallbacks for legacy relay folders and root `npm start`.

---

## Contributing

Contributions are welcome. Good areas to improve:

- More reliable UI-tree ranking.
- Safer action confirmation policies.
- Better dashboard live telemetry.
- More MCP tools for structured Android workflows.
- Provider-specific prompt tuning.

See [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

---

## License

Apache License 2.0. See [LICENSE](LICENSE).

---

<div align="center">

**V.A.Y.U is not a chatbot with a phone theme.**

It is a phone-native agent loop: eyes, memory, hands, relay, and remote control.

</div>
