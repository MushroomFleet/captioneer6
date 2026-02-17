# CAPTIONEER6

A unified desktop application that combines video frame extraction with AI-powered batch image captioning into a single drag-and-drop pipeline.

**Video In → Frames Extracted → Captions Generated → Dataset ZIP Out**

Built with Vite + React + TypeScript and packaged as a native Windows desktop app using Tauri v2.

![License](https://img.shields.io/badge/license-MIT-blue)
![Platform](https://img.shields.io/badge/platform-Windows-0078d4)
![Tauri](https://img.shields.io/badge/Tauri-v2-ffc131)

---

## What It Does

CAPTIONEER6 replaces a multi-tool workflow that previously required running separate applications, Python servers, and manual file shuffling between steps. It provides three operational modes through a tabbed interface:

### Splicer Tab — Video Frame Extraction
- Drag and drop MP4 files (batch upload supported)
- Extract keyframes at configurable intervals (0.5–10 seconds)
- Preview extracted frames in a thumbnail gallery
- Download frames as a ZIP archive
- Frames are held in memory for instant handoff to captioning

### Captioner Tab — AI Image Captioning
- Load images directly from the Splicer, from a folder, or from individual files
- Upload custom system prompts (.md or .txt) with lock/unlock persistence
- Configure trigger words and classifiers for training-ready caption prefixes
- Captions generated via OpenRouter Vision API (supports any vision model)
- Post-processing strips brackets, normalizes whitespace, assembles prefixes
- Download all captions as a ZIP archive

### Pipeline Tab — One-Click End-to-End
- Runs the full sequence automatically: extract → caption → package
- Three-stage progress display with per-stage tracking
- Produces a combined dataset ZIP containing `/images/`, `/captions/`, and a `manifest.json`

---

## Prerequisites

Before installing CAPTIONEER6, ensure you have the following:

- **Node.js** (v18 or later) — [nodejs.org](https://nodejs.org/)
- **Rust** (latest stable) — [rustup.rs](https://rustup.rs/)
- **OpenRouter API Key** — [openrouter.ai/keys](https://openrouter.ai/keys) (required for captioning)

On Windows, you also need:
- **Visual Studio Build Tools** with the "Desktop development with C++" workload
- **WebView2** runtime (pre-installed on Windows 10/11)

---

## Installation

### Clone the Repository

```bash
git clone https://github.com/MushroomFleet/captioneer6.git
cd captioneer6
```

### Install Dependencies

```bash
npm install
```

This installs all frontend dependencies. Rust/Tauri dependencies are fetched automatically on first build.

---

## Usage

### Development Mode (Browser)

Run the frontend only in your browser for rapid development:

```bash
npm run dev
```

Opens at [http://localhost:1420](http://localhost:1420).

### Development Mode (Desktop)

Run as a native desktop window with Tauri:

```bash
npm run tauri:dev
```

This starts both the Vite dev server and the Tauri webview window with hot-reload.

### Production Build (MSI Installer)

Build the distributable Windows installer:

```bash
npm run tauri:build
```

The MSI and NSIS installers are output to `src-tauri/target/release/bundle/`.

---

## Configuration

### API Settings

1. Click the **Settings** button in the top-right corner of the app
2. Enter your **OpenRouter API Key** (starts with `sk-or-v1-...`)
3. Set the **Model Name** (default: `x-ai/grok-4-fast`)
4. Click **Save Settings**

Your API key is stored in localStorage and is only sent to the OpenRouter API endpoint. It is never transmitted anywhere else.

### System Prompt

The captioner uses a system prompt to guide the vision model. You can:

- Use the built-in default prompt for general image description
- Upload a custom `.md` or `.txt` file with your own instructions
- Lock the prompt so it persists across sessions

### Trigger Words & Classifiers

For training datasets (LoRA, DreamBooth, etc.):

- **Trigger Word**: A token prepended to every caption (e.g., `sks`, `ohwx`)
- **Classifier**: A subject category appended after the trigger word (e.g., `woman`, `style`, `object`)

Example output: `sks woman, a detailed description of the image...`

---

## Project Structure

```
captioneer6/
├── index.html                  # Entry HTML
├── package.json                # Dependencies and scripts
├── vite.config.ts              # Vite config (port 1420, COOP/COEP headers)
├── src/
│   ├── main.tsx                # React entry point
│   ├── App.tsx                 # Root component with tab routing
│   ├── types/
│   │   └── index.ts            # All TypeScript interfaces and constants
│   ├── styles/
│   │   ├── theme.ts            # Design tokens (colors, fonts, spacing)
│   │   └── globals.css         # Global styles, animations, fonts
│   ├── services/
│   │   ├── frameExtractor.ts   # Canvas API video frame extraction
│   │   ├── imageUtils.ts       # File type checks, base64 conversion
│   │   ├── openrouter.ts       # OpenRouter API client + post-processing
│   │   └── zipService.ts       # JSZip wrappers for frames/captions/datasets
│   ├── hooks/
│   │   ├── useVideoExtractor.ts  # Video loading + extraction state
│   │   ├── useCaptioner.ts       # Captioning pipeline state
│   │   ├── usePipeline.ts        # Unified pipeline orchestration
│   │   ├── useLogger.ts          # Log state management
│   │   └── useLocalStorage.ts    # Generic localStorage hook
│   └── components/
│       ├── Header.tsx            # App header with logo and settings
│       ├── TabNav.tsx            # Three-tab navigation
│       ├── splicer/
│       │   └── SplicerPanel.tsx  # Complete splicer tab UI
│       ├── captioner/
│       │   └── CaptionerPanel.tsx  # Complete captioner tab UI
│       ├── pipeline/
│       │   └── PipelinePanel.tsx   # Complete pipeline tab UI
│       ├── shared/
│       │   ├── LogConsole.tsx      # Color-coded log display
│       │   ├── ProgressBar.tsx     # Reusable progress bar
│       │   ├── SettingsModal.tsx    # API key/model settings
│       │   └── StatusIndicator.tsx  # Status dot with label
│       └── ui/
│           ├── Button.tsx          # Styled button variants
│           ├── Input.tsx           # Styled text input
│           └── Panel.tsx           # Numbered panel container
├── src-tauri/
│   ├── Cargo.toml              # Rust dependencies
│   ├── tauri.conf.json         # Tauri v2 app config (window, CSP, bundling)
│   ├── capabilities/
│   │   └── default.json        # Permission capabilities
│   ├── icons/                  # App icons (all platforms)
│   └── src/
│       ├── main.rs             # Rust entry point
│       └── lib.rs              # Tauri builder setup
```

---

## Supported Formats

### Video Input
- `.mp4`

### Image Input (Captioner)
- `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.bmp`

### Video Input (Captioner — extracts frame 15 for captioning)
- `.mp4`, `.webm`, `.mov`, `.avi`, `.mkv`, `.m4v`

---

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server (browser, port 1420) |
| `npm run build` | TypeScript check + Vite production build |
| `npm run tauri:dev` | Start Tauri desktop app in development |
| `npm run tauri:build` | Build production MSI/NSIS installers |
| `npm run lint` | Run ESLint |
| `npm run preview` | Preview production build locally |

---

## Tech Stack

- **Frontend**: React 19, TypeScript, Vite 7
- **Desktop**: Tauri v2 (Rust backend, WebView2 on Windows)
- **Vision API**: OpenRouter (any vision-capable model)
- **Frame Extraction**: Canvas API (no FFmpeg dependency)
- **ZIP Handling**: JSZip
- **Fonts**: JetBrains Mono, Orbitron

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{captioneer6,
  title = {Captioneer6: Unified Video-to-Caption Dataset Pipeline},
  author = {Drift Johnson},
  year = {2025},
  url = {https://github.com/MushroomFleet/captioneer6},
  version = {1.0.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)
