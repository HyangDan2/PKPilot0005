# HD2Lab | Solver (PKPilot0005)

A static web app that reads a problem and solves it. Give it text, a screenshot or a photo of a textbook page and
it produces one answer card: **problem-type read → answer → step-by-step solution → wrong-option elimination and
key points**.

- Live: <https://hyangdan2.github.io/PKPilot0005/>
- Family: the same HD2Lab shell as Web-HQ and PKPilot0001–0004 (Soft Iris light, Navy & Lavender dark)
- Runtime: static, no build. With no server it runs in **demo mode**; connect the Server-HQ API for **real vision
  model reading**

## Two run modes

| Mode | Condition | Behaviour |
|------|------|------|
| Demo (default) | `API connection` is empty | Works entirely in the browser. Three built-in examples (multiple choice, coding, screenshot) plus linear equations and arithmetic are actually computed. For anything else it reads the type and topic and offers an approach — and **never invents an answer** |
| API | a `…/api/solver/solve` URL saved in `API connection` | Server-HQ `hd2lab-solver` → AI gateway (`hd2ai-solver` route) → free vision model |

Even in API mode a failure does **not** silently fall back to the demo: the error is shown, and the demo returns
when the user presses `Disconnect`.

## Features

- **Input** — paste text (8,000 characters recommended maximum), pick an image by click, drag and drop,
  `Ctrl`/`Cmd`+`V` paste, or the phone camera/album, with a thumbnail preview and a clear button
- **Image handling** — 1600px on the long edge, JPEG quality stepping from 0.9 down to 0.5, kept **under 1.5 MB
  of base64**. Canvas re-encoding also **strips EXIF and GPS**
- **Problem type** — `Auto read` · `Multiple choice` · `Short answer / essay` · `Coding / debugging` ·
  `Enpago / language` plus one extra hint line
- **Answer card** — type, topic and mode badges, a highlighted answer, numbered steps, wrong-option elimination
  and key points, and copy-answer
- **History** — the latest 30 entries in `localStorage`, with open, delete and delete-all (two-step confirmation)
- **Keyboard** — `⌘`/`Ctrl` + `Enter` solves immediately
- **Responsive** — two-column studio at 1440px, one column at 390px, zero horizontal overflow

## Files

| File | Purpose |
|------|------|
| `index.html` | header lockup `[mark] HD2Lab \| Solver`, input and answer panels, history, `API connection` dialog |
| `style.css` | Soft Iris tokens (`#FAFAFD` ground, `#8B6CE8` accent) + Navy & Lavender dark, dual-skew `.mark`, responsive |
| `app.js` | demo solver, image compression, API client, history |
| `assets/hd2-theme.*` | shared theme tokens and the 44px floating light/dark toggle |
| `assets/sample-problem.png` | screenshot example (an English fill-in-the-blank worksheet) |
| `og-image.png` | 1200×630 social preview |
| `favicon.svg` · `favicon.ico` · `favicon-32x32.png` · `apple-touch-icon.png` · `icon-192.png` · `icon-512.png` | favicon and PWA icons |
| `manifest.json` | PWA manifest |
| `HANDOFF_ANTIGRAVITY.md` | server handover (AI gateway multimodal, routes, vision model, FastAPI, Nginx) |

## Storage keys (localStorage)

| Key | Contents |
|----|------|
| `pkpilot0005_api_v1` | `{ url, auth: 'none'\|'bearer'\|'cookie', token }` |
| `pkpilot0005_history_v1` | the latest 30 `{ id, at, type, topic, question, hint, mode, result }` |

The token is stored in this browser only and never appears on screen or in a log.

## API contract (summary)

```http
POST /api/solver/solve
{ "text": "...", "image_base64": "data:image/jpeg;base64,...", "mode": "auto|quiz|essay|code|enpago", "hint": "..." }
→ 200 { "answer": "④ x = 10", "answer_detail": "...", "steps": [...], "key_points": [...],
        "problem_type": "quiz", "topic": "Math · linear equation", "confidence": "high",
        "model": "...", "elapsed_ms": 4210 }
```

`/api/solver/health` checks the connection. When the endpoint ends in `/solve` the front end derives `/health` by
itself. The full contract and error table are in chapter 9 of
[`HANDOFF_ANTIGRAVITY.md`](HANDOFF_ANTIGRAVITY.md).

## Run locally

```bash
python3 -m http.server 8899 --directory .
# http://127.0.0.1:8899/
```

`file://` still runs the demo, but `localStorage` and `fetch` restrictions make a local server the better choice.

## Verification (2026-11-16, headless Chromium)

At `1440×900` and `390×844`: zero console and page errors, zero horizontal overflow.

- multiple-choice example → `④ x = 10` (five steps, five key points)
- coding example → `ZeroDivisionError` diagnosis plus fixed code
- screenshot example → 1100×780 thumbnail preview, `③ went ahead`
- file upload → confirmed 1600px and JPEG compression (74KB)
- `API connection` dialog → test, save and disconnect, and the `API` badge switching in a mocked API mode
- history → survives a reload, opens, deletes, deletes all
- copy answer → clipboard `[answer] ④ x = 10`
- **E2E stack verification** from browser → FastAPI → Unix socket → gateway response (in the documentation
  environment)

## Branding

- Soft Iris light: `#FAFAFD` ground, `#FFFFFF` card, `#E8E4F2` border, `#8B6CE8` accent
- Navy & Lavender dark: `#0B1020` ground, `#12192E` card, `#253154` border, `#A78BFA` accent
- Header lockup: dual-skew `.mark` + `HD2Lab` (25px/800/-1.5px) + divider + `Solver` (12px)
- OG: 1200×630, mark on the left + `62px HD2Lab | Solver`
