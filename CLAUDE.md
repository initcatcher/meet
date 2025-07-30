# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Development server**: `pnpm dev` - starts Next.js development server on http://localhost:3000
- **Build**: `pnpm build` - creates production build
- **Start production**: `pnpm start` - runs production server
- **Linting**: `pnpm lint` (check) or `pnpm lint:fix` (fix automatically)
- **Tests**: `pnpm test` - runs Vitest test suite
- **Formatting**: `pnpm format:check` (check) or `pnpm format:write` (fix automatically)

## Project Architecture

This is a **LiveKit Meet** video conferencing application built with:
- **Next.js 15** with App Router
- **LiveKit Components React** (`@livekit/components-react`) - primary UI library for video conferencing
- **LiveKit Client SDK** (`livekit-client`) - WebRTC client library
- **TypeScript** with strict mode enabled

### Key Architecture Patterns

**App Structure**:
- `/app/page.tsx` - Home page with demo/custom connection tabs
- `/app/rooms/[roomName]/` - Dynamic room pages for video conferences
- `/app/custom/` - Custom server connection flow
- `/app/api/` - API routes for connection details and recording

**Core Components**:
- `PageClientImpl.tsx` - Main video conference room implementation
- `VideoConferenceClientImpl.tsx` - Custom video conference client
- `/lib/` contains utilities: client-utils, settings menus, keyboard shortcuts, performance optimizers

**LiveKit Integration**:
- Uses `@livekit/components-react` for pre-built video conferencing UI
- Implements E2EE (end-to-end encryption) support via `useSetupE2EE` hook
- Performance optimization with `useLowCPUOptimizer` for low-power devices
- Custom components in `/lib/` extend LiveKit functionality (settings, recording indicators, debug mode)

### Environment Setup

Copy `.env.example` to `.env.local` and configure:
- `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET` - LiveKit server credentials
- `LIVEKIT_URL` - WebSocket URL to LiveKit server (e.g., `wss://your-project.livekit.cloud`)
- Optional: S3 settings for recording, Datadog logging, settings menu enablement

### Next.js Configuration

- **Webpack**: Custom config includes source-map-loader for `.mjs` files
- **Headers**: Sets Cross-Origin policies (`COOP: same-origin`, `COEP: credentialless`) required for SharedArrayBuffer/WebAssembly
- **Images**: Configured for WebP format optimization
- **Source maps**: Enabled in production for debugging

### Key Utilities

- `client-utils.ts` - Room ID generation, passphrase encoding, device detection
- `types.ts` - TypeScript interfaces for sessions, tokens, connection details
- `getLiveKitURL.ts` - Server URL construction and validation logic

Package manager is **pnpm** (specified in packageManager field).