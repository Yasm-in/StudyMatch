# StudyMatch — Complete Setup & Deployment Guide

## 📋 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Firebase Setup](#firebase-setup)
3. [Backend Setup](#backend-setup)
4. [Mobile App Setup](#mobile-app-setup)
5. [Running Locally](#running-locally)
6. [Deploying to Production](#deploying-to-production)
7. [Building the Mobile App](#building-the-mobile-app)
8. [Environment Variables Reference](#environment-variables-reference)
9. [Troubleshooting](#troubleshooting)

---

## 1. Prerequisites

Install these tools before starting:

```bash
# Node.js 18 or higher (required)
node --version   # must be v18+
npm --version    # comes with Node

# EAS CLI — the only Expo-related tool installed globally
npm install -g eas-cli

# Verify npx is available (ships with npm 5.2+, no install needed)
npx --version

# Optional: Docker for running the backend in a container
docker --version
```

> **Warning — do NOT run `npm install -g expo-cli`**
> The global `expo-cli` package is deprecated as of Expo SDK 46 (August 2022).
> The Expo CLI is now bundled inside the `expo` package in your project's
> `node_modules`. Always use `npx expo <command>` from inside the `mobile/`
> directory instead of a globally installed `expo` binary.
>
> If you previously installed the old global CLI, uninstall it first:
> ```bash
> npm uninstall -g expo-cli
> ```

---

## 2. Firebase Setup

### Create Project
1. Go to [Firebase Console](https://console.firebase.google.com)
2. Click **Add project** → name it `studymatch`
3. Disable Google Analytics (optional)

### Enable Authentication
1. Go to **Build → Authentication → Get started**
2. Enable **Email/Password** provider
3. Under **Settings → Authorized domains**, add your domains

### Enable Firestore
1. Go to **Build → Firestore Database → Create database**
2. Start in **Production mode**
3. Choose a region close to your users (e.g. `europe-west1`)
4. Deploy security rules from `backend/firestore.rules`:
   ```bash
   # Install Firebase CLI
   npm install -g firebase-tools
   firebase login
   firebase init firestore  # point to your project
   firebase deploy --only firestore:rules
   ```

### Enable Storage
1. Go to **Build → Storage → Get started**
2. Start in **Production mode**
3. Set CORS rules to allow your domain

### Get Service Account (for Backend)
1. Go to **Project Settings → Service Accounts**
2. Click **Generate new private key**
3. Download the JSON file
4. Extract values for your `.env`:
   - `FIREBASE_PROJECT_ID` from `project_id`
   - `FIREBASE_CLIENT_EMAIL` from `client_email`
   - `FIREBASE_PRIVATE_KEY` from `private_key`

### Get Web Config (for Mobile)
1. Go to **Project Settings → General → Your apps**
2. Click **Add app → Web** (needed for Expo)
3. Copy the config values to `mobile/.env`

---

## 3. Backend Setup

```bash
cd backend

# Install dependencies
npm install

# Copy and fill environment variables
cp .env.example .env
# Edit .env with your Firebase credentials

# Run tests
npm test

# Start development server
npm run dev
# Server starts at http://localhost:3000

# Check health
curl http://localhost:3000/health
```

### API Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/verify-email` | Validate university email |
| POST | `/api/auth/complete-profile` | Save profile after registration |
| GET | `/api/users/me` | Get own profile |
| PUT | `/api/users/me` | Update profile |
| PUT | `/api/users/me/traits` | Update study traits |
| GET | `/api/matches/discover` | Get ranked matches |
| POST | `/api/matches/swipe` | Like or pass |
| GET | `/api/matches` | Get all matches |
| DELETE | `/api/matches/:id` | Unmatch |
| POST | `/api/chats/direct` | Create/get direct chat |
| GET | `/api/chats` | Get all chats |
| GET | `/api/chats/:id/messages` | Get messages |
| POST | `/api/chats/:id/messages` | Send message |
| POST | `/api/groups` | Create study group |
| GET | `/api/groups` | Get user's groups |
| POST | `/api/materials/upload` | Upload study material |
| GET | `/api/materials` | Get materials |

---

## 4. Mobile App Setup

```bash
cd mobile

# Install dependencies
# This automatically installs the local Expo CLI inside node_modules
npm install

# Copy and fill environment variables
cp .env.example .env
# Set EXPO_PUBLIC_API_URL and all Firebase config values

# Place Firebase config files in the mobile/ directory
# Android: google-services.json
# iOS:     GoogleService-Info.plist

# Verify the local Expo CLI installed correctly
npx expo --version
```

---

## 5. Running Locally

### Start both services

```bash
# Terminal 1 — Backend
cd backend
npm run dev

# Terminal 2 — Mobile (always use npx expo, not expo directly)
cd mobile
npx expo start
```

### Option B: Docker (backend only)
```bash
cp backend/.env.example backend/.env
# Fill in backend/.env

docker-compose up --build
# API available at http://localhost:3000
```

### Common npx expo commands

```bash
# Start the dev server and show QR code for Expo Go
npx expo start

# Clear Metro bundler cache and restart (fixes most weird errors)
npx expo start --clear

# Open directly on a connected Android device / emulator
npx expo start --android

# Open directly on an iOS simulator
npx expo start --ios

# Check all installed packages are compatible with your Expo SDK version
npx expo install --check
```

### Testing on a physical device
1. Install **Expo Go** from the App Store or Google Play
2. Run `npx expo start` from inside `mobile/`
3. Scan the QR code shown in the terminal
4. Your phone and computer must be on the **same WiFi network**
5. Set `EXPO_PUBLIC_API_URL` to your computer's local IP  
   (e.g. `http://192.168.1.10:3000` — not `localhost`)

---

## 6. Deploying to Production

### Backend — Deploy to Railway (Recommended)

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login and deploy
railway login
railway init
railway up

# Set environment variables
railway variables set FIREBASE_PROJECT_ID=xxx
railway variables set FIREBASE_CLIENT_EMAIL=xxx
railway variables set FIREBASE_PRIVATE_KEY="xxx"
# ... set all other variables from .env.example
```

### Backend — Deploy to Google Cloud Run

```bash
# Build Docker image
docker build -t gcr.io/YOUR_PROJECT/studymatch-api ./backend

# Push to GCR
docker push gcr.io/YOUR_PROJECT/studymatch-api

# Deploy
gcloud run deploy studymatch-api \
  --image gcr.io/YOUR_PROJECT/studymatch-api \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

### Backend — Deploy to Render

1. Connect your GitHub repo at [render.com](https://render.com)
2. New → Web Service → select `backend/` directory
3. Build command: `npm install`
4. Start command: `node src/index.js`
5. Add all environment variables in the dashboard

---

## 7. Building the Mobile App

EAS Build runs in Expo's cloud — you do not need Xcode or Android Studio
locally for cloud builds. Only `eas-cli` (global) is required.

### One-time EAS setup

```bash
cd mobile

# Log in to your Expo account (free at expo.dev)
eas login

# Link this project to EAS (eas.json is already included)
eas build:configure
```

### iOS builds

```bash
cd mobile

# Development build — install on device via TestFlight internal testing
eas build --platform ios --profile development

# Production build — ready for App Store submission
eas build --platform ios --profile production
```

### Android builds

```bash
cd mobile

# Development build — outputs a debug APK you can sideload
eas build --platform android --profile development

# Production build — outputs an AAB for the Play Store
eas build --platform android --profile production
```

### Submit to app stores

```bash
cd mobile

# Submit to Apple App Store Connect
eas submit --platform ios

# Submit to Google Play Store
eas submit --platform android
```

### Local development builds (advanced)

If your app uses native modules not supported by Expo Go, build a local
development client. This requires Xcode (iOS) or Android Studio (Android):

```bash
cd mobile
npx expo run:ios      # builds and opens in iOS simulator
npx expo run:android  # builds and opens in Android emulator
```

---

## 8. Environment Variables Reference

### Backend (`backend/.env`)

| Variable | Description | Example |
|----------|-------------|---------|
| `FIREBASE_PROJECT_ID` | Your Firebase project ID | `studymatch-abc12` |
| `FIREBASE_CLIENT_EMAIL` | Service account email | `firebase-adminsdk@project.iam.gserviceaccount.com` |
| `FIREBASE_PRIVATE_KEY` | Service account private key | `-----BEGIN PRIVATE KEY-----\n...` |
| `FIREBASE_STORAGE_BUCKET` | Storage bucket URL | `studymatch-abc12.appspot.com` |
| `PORT` | Server port | `3000` |
| `NODE_ENV` | Environment | `production` |
| `JWT_SECRET` | Secret for JWT signing | Random 64+ char string |
| `ALLOWED_EMAIL_DOMAINS` | Comma-separated university domains | `.edu,ac.ke,ac.uk` |
| `RATE_LIMIT_WINDOW_MS` | Rate limit window in ms | `900000` (15 min) |
| `RATE_LIMIT_MAX_REQUESTS` | Max requests per window | `100` |

### Mobile (`mobile/.env`)

| Variable | Description |
|----------|-------------|
| `EXPO_PUBLIC_FIREBASE_API_KEY` | Firebase web API key |
| `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN` | Firebase auth domain |
| `EXPO_PUBLIC_FIREBASE_PROJECT_ID` | Firebase project ID |
| `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET` | Storage bucket |
| `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | FCM sender ID |
| `EXPO_PUBLIC_FIREBASE_APP_ID` | Firebase app ID |
| `EXPO_PUBLIC_API_URL` | Backend API base URL |

> All mobile env vars must be prefixed `EXPO_PUBLIC_` to be included in the
> app bundle. Variables without this prefix are stripped at build time and
> will be `undefined` at runtime.

---

## 9. Troubleshooting

### "expo: command not found"
The global `expo-cli` is deprecated and not installed. The CLI lives in
`node_modules` now. Always use `npx expo` from inside `mobile/`:
```bash
# Wrong — do not use
expo start

# Correct
cd mobile && npx expo start
```

### Metro bundler shows stale changes or crashes on start
```bash
cd mobile && npx expo start --clear
```

### "University email not accepted"
- Check `ALLOWED_EMAIL_DOMAINS` in `backend/.env`
- Example: `.edu,ac.ke,ac.uk,youruniversity.edu.xx`

### Biometric login not working
- Only works on physical devices — not simulators or emulators
- The user must have Face ID or fingerprint enrolled in device Settings
- Android minimum: API level 23 (Android 6.0)

### Real-time messages not updating
- Verify Firestore security rules are deployed
- Ensure the user is authenticated before the Firestore listener attaches
- Check that `chatId` matches exactly in Firestore (it is case-sensitive)

### Build errors on iOS (bare workflow only)
- Run `cd mobile/ios && pod install`
- Confirm `GoogleService-Info.plist` is placed in `mobile/`

### "Firebase App already exists" error
Handled in `firebase.ts` with the `getApps().length` guard. If it still
appears, clear the Metro cache:
```bash
cd mobile && npx expo start --clear
```

### EAS build fails with "project not found"
```bash
cd mobile
eas login             # confirm you are logged into the correct account
eas build:configure   # re-link the project to your EAS account
```

### Matching algorithm returns no results
- Confirm both users have `profileComplete: true` in Firestore
- Confirm both users have a `studyTraits` object populated
- Lower the score threshold in `backend/src/services/matchingAlgorithm.js`
  (change `score >= 60` in `getRankedMatches` to a lower value)

---

## 🔒 Security Checklist Before Launch

- [ ] Change `JWT_SECRET` to a strong random string (64+ characters)
- [ ] Set `NODE_ENV=production` on the backend server
- [ ] Deploy Firestore security rules (`firebase deploy --only firestore:rules`)
- [ ] Restrict Firebase API key to your app's bundle ID and domain
- [ ] Enable Firebase App Check (prevents direct API abuse)
- [ ] Configure CORS to your production domain only — remove the `*` wildcard
- [ ] Confirm HTTPS is active on the backend (handled by Railway/Render/Cloud Run)
- [ ] Set up crash monitoring via Firebase Crashlytics
- [ ] Review rate limiting values for expected production traffic volume

---

## 📊 Firestore Indexes Required

These **must** be created before the app works correctly in production.
Go to **Firebase Console → Firestore → Indexes → Composite** and add:

```
Collection: matches
  participants   (array-contains)
  timestamp      (descending)

Collection: chats
  participants   (array-contains)
  lastMessageAt  (descending)

Collection: groups
  members        (array-contains)
  isActive       (ascending)

Collection: materials
  groupId        (ascending)
  uploadedAt     (descending)

Collection: materials
  subject        (ascending)
  uploadedAt     (descending)
```

> Tip: Firebase automatically suggests missing indexes when a query fails.
> Check the backend console logs — the error message includes a direct link
> to create the missing index with one click.

---

*Built with React Native (Expo SDK 50) + Node.js + Firebase*
