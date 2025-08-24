# Chat Client

Minimal native chat client for ch.at API using Vue 3, TypeScript, and Electron.

## Prerequisites

- Node.js 16+
- npm

## Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/chat-client.git
cd chat-client
```

### 2. Install dependencies
```bash
npm install
```

### 3. Development

Start the app in development mode:
```bash
npm run electron:dev
```

This will:
- Start Vite dev server on port 5173
- Launch Electron window with hot-reload enabled

### 4. Build

Build native executables:

```bash
# Windows (.exe)
npm run build:win

# macOS (.dmg)
npm run build:mac

# Linux (AppImage)
npm run build:linux

# All platforms
npm run build
```

Built apps will be in the `dist` folder.

## Project Structure

```
chat-client/
├── src/
│   ├── components/       # Vue components
│   │   └── ChatWindow.vue
│   ├── App.vue           # Main Vue app
│   ├── main.ts           # Vue entry point
│   ├── style.css         # Global styles
│   └── env.d.ts          # TypeScript declarations
├── electron/
│   └── main.ts           # Electron main process
├── index.html            # HTML entry point
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript config
├── vite.config.ts        # Vite configuration
└── README.md
```

## Architecture

- **Frontend**: Vue 3 with TypeScript for the UI
- **API Communication**: Direct HTTP requests to ch.at API
- **Native wrapper**: Electron provides cross-platform native app
- **Build tool**: Vite for fast development and optimized builds

## Features

- Clean, minimal interface
- Real-time chat with ch.at API
- Cross-platform native app
- No authentication required
- TypeScript for type safety
- Hot-reload during development