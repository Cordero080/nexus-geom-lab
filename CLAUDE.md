# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Manifold is a full-stack 3D geometry exploration platform. Users interact with real-time 3D shapes, save scene configurations, and progressively unlock characters/animations based on how many scenes they save.

## Commands

### Frontend (root directory)
```bash
npm run dev          # Dev server on port 5173
npm run build        # Build for GitHub Pages
npm run build:vercel # Build for Vercel (VITE_DEPLOY_TARGET=vercel, base /)
npm run lint         # ESLint
npm run lint:fix     # ESLint with auto-fix
npm run format       # Prettier
npm test             # Jest (39 tests across 7 suites)
```

### Backend (`manifold-backend/`)
```bash
npm run dev   # nodemon index.js (port 3000)
npm start     # node index.js
npm run seed  # Create/reset dev test user (pablo@test.com / test123)
npm run reset # Reset dev user stats
npm run users # List all users in DB
```

### Deploying Backend Changes to Render
The backend is a monorepo subfolder locally but Render watches a **separate** GitHub repo (`Cordero080/manifold-backend`). After pushing to the main repo, also run:
```bash
git subtree push --prefix=manifold-backend https://github.com/Cordero080/manifold-backend.git main
```
This syncs `manifold-backend/` into the separate repo so Render picks up the changes.

### Local Development
The frontend Vite config proxies `/api` → `http://localhost:3000`, so run both servers simultaneously. No CORS issues locally.

## Architecture

### Stack
- **Frontend**: React 19 + React Three Fiber (Three.js) + Vite, deployed to Vercel
- **Backend**: Express 5 + Mongoose + JWT, deployed to Render (`https://nexus-geom-lab-backend-1.onrender.com`)
- **Database**: MongoDB Atlas (`skyrockit` database, user `vantablvck`)

### Frontend State
Two global contexts:
- **AuthContext** (`src/features/auth/context/AuthContext.jsx`) — `user`, `token`, `isAuthenticated`, `unlockedAnimations`, `unlockedNoetechs`. Persists to localStorage.
- **SceneContext** (`src/context/SceneContext.jsx`) — current scene metadata (`currentSceneId`, `sceneName`, `sceneMode`). `sceneMode` is `'fresh'`, `'loaded'`, or `'remixed'`. Provides `loadedConfig` to hydrate the 3D editor.

### Frontend Routing (`src/App.jsx`)
All routes except `/`, `/login`, `/signup` are protected (redirect to `/login` if not authenticated).
- `/geom-lab` or `/geometry-lab` → main 3D editor
- `/showcase` → animated character viewer
- `/scenes` → saved scenes gallery

### Frontend Feature Structure
```
src/features/sceneControls/   # Main 3D editor — ThreeScene.jsx + Controls panel
src/features/auth/            # Auth pages, context, API service
src/features/audio/           # Web Audio API hooks (FFT mic input → 3D transforms)
src/components/pages/         # HomePage, MyScenesPage, Showcase
```

### Backend Routes
```
POST   /api/auth/signup
POST   /api/auth/login
GET    /api/auth/me           # Protected
POST   /api/scenes            # Protected — creates scene, triggers unlock check
GET    /api/scenes/my-scenes  # Protected
PUT    /api/scenes/:id        # Protected — ownership check
DELETE /api/scenes/:id        # Protected — ownership check
```

### Unlock Progression
On each `POST /api/scenes`, the backend increments `user.scenesSaved` and checks against `UNLOCK_PROGRESSION` milestones. New unlocks are returned in the API response and merged into `AuthContext` state. Characters: Icarus-X, Vectra, Nexus, She-Tech. Animations unlock after characters.

### Scene Config Schema
Scenes store a flat `config` object with all 3D editor state: `scale`, `metalness`, `emissiveIntensity`, `baseColor`, `wireframeIntensity`, `hyperframeColor`, `hyperframeLineColor`, `cameraView`, `environment`, `environmentHue`, `objectCount`, `animationStyle`, `objectType`, `objectSpeed`, `orbSpeed`, and light settings.

## Deployment

- **Frontend**: Vercel — `vercel.json` uses `vite build` → `dist/`, SPA rewrites to `index.html`
- **Backend**: Render — requires env vars `MONGODB_URI`, `JWT_SECRET`, `FRONTEND_URL`
- **Base path**: `/` on Vercel, `/manifold/` on GitHub Pages (controlled by `VITE_DEPLOY_TARGET`)

## Dev User
After running `npm run seed` in the backend: `pablo@test.com` / `test123`. The seed script is idempotent — safe to run multiple times.
