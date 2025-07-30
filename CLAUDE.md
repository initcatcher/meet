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

## Core Features and Logic Flows

### End-to-End Encryption (E2EE)

**PassPhrase URL Hash System**:
- `encodePassphrase()` / `decodePassphrase()` - Simple URL encoding/decoding (lib/client-utils.ts:1-7)
- URL format: `/rooms/abc-123#encoded_passphrase` 
- `useSetupE2EE()` hook extracts passphrase from `location.hash.substring(1)` (lib/useSetupE2EE.ts:6-7)
- Creates Web Worker: `new Worker('livekit-client/e2ee-worker')` for background encryption
- Key setup: `keyProvider.setKey(decodePassphrase(e2eePassphrase))` (app/rooms/[roomName]/PageClientImpl.tsx:140)

**E2EE Flow**:
1. Generate 64-char random passphrase with `randomString(64)`
2. Encode and append to URL hash on meeting creation
3. Extract from hash on room join, create worker if present
4. Set up ExternalE2EEKeyProvider with decoded passphrase
5. Enable E2EE on room connection: `room.setE2EEEnabled(true)`

### Video Quality and Resolution Management

**Resolution Presets** (app/rooms/[roomName]/PageClientImpl.tsx:113):
```typescript
resolution: props.options.hq ? VideoPresets.h2160 : VideoPresets.h720
```
- **HQ Mode**: 4K capture (h2160) + simulcast [1080p, 720p]
- **Normal Mode**: 720p capture + simulcast [540p, 216p]
- Controlled via URL parameter: `?hq=true`

**Adaptive Quality** (lib/usePerfomanceOptimiser.ts:42-47):
- CPU-constrained event triggers automatic quality reduction
- `publication.setVideoQuality(VideoQuality.LOW)` for all remote participants
- Low-power device detection: `navigator.hardwareConcurrency < 6`

### Participant Management

**Token-based Authentication** (app/api/connection-details/route.ts:40-47):
```typescript
identity: `${participantName}__${randomParticipantPostfix}`
// JWT with VideoGrant permissions: roomJoin, canPublish, canSubscribe
```

**Participant State**:
- Local: `useLocalParticipant()` hook for camera/microphone controls
- Remote: `room.remoteParticipants` Map for all other participants
- Permissions stored in participant.permissions object
- Debug mode (Shift+D) shows detailed participant/track information

### Recording System

**S3-based Recording** (app/api/record/start/route.ts):
- Egress client with `startRoomCompositeEgress()` in "speaker" layout
- File format: `${timestamp}-${roomName}.mp4`
- Check for existing recordings before starting new ones
- **Security Warning**: No authentication - production use requires auth

**Recording UI**:
- RecordingIndicator shows red border overlay when active
- Toast notification: "This meeting is being recorded" 
- Settings menu toggle for start/stop (if `NEXT_PUBLIC_LK_RECORD_ENDPOINT` set)

### Performance Optimization

**Low CPU Optimizer** (lib/usePerfomanceOptimiser.ts):
- Monitors `ParticipantEvent.LocalTrackCpuConstrained`
- Automatically reduces publisher quality: `track.prioritizePerformance()`
- Disables video processing and reduces subscriber quality
- Sets all remote tracks to `VideoQuality.LOW`

**Web Worker Usage**:
- E2EE worker handles encryption/decryption in background thread
- Prevents main thread blocking during crypto operations
- Worker created conditionally only when E2EE enabled

### UI Layout and Responsiveness  

**Screen Size Management**:
- CSS Grid/Flexbox for responsive layout (styles/globals.css, styles/Home.module.css)
- `overflow: hidden` on html/body for full-screen experience
- LiveKit's `VideoConference` component handles participant layout automatically
- Container max-width: 500px with responsive padding

**Keyboard Shortcuts** (lib/KeyboardShortcuts.tsx):
- `Ctrl+A` / `Cmd+A`: Toggle microphone
- `Ctrl+V` / `Cmd+V`: Toggle camera
- `Shift+D`: Open debug panel

### Debug and Development Tools

**Debug Mode** (lib/Debug.tsx):
- Real-time participant tracking and bitrate monitoring
- Track dimensions, subscription status, permissions display
- Datadog integration for production logging (if configured)
- Room simulation capabilities for testing network conditions

**Key Files to Understand**:
- `PageClientImpl.tsx` - Main room logic, E2EE setup, connection handling
- `useSetupE2EE.ts` - E2EE passphrase extraction and worker creation  
- `usePerfomanceOptimiser.ts` - Automatic quality adjustment for low-power devices
- `client-utils.ts` - Utility functions for room IDs, encoding, device detection