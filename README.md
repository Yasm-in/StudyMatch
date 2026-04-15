# 📚 StudyMatch

> Tinder for studying — find your perfect study partner.

A full-stack mobile application that matches students based on study habits, learning styles, schedules, and academic goals using a smart compatibility algorithm.

---

## 🏗️ Project Structure

```
studymatch/
├── mobile/          # React Native app (iOS & Android)
├── backend/         # Node.js + Express API server
└── shared/          # Shared types and constants
```

---

## 🚀 Tech Stack

| Layer      | Technology                          |
|------------|-------------------------------------|
| Mobile     | React Native (Expo)                 |
| Backend    | Node.js + Express                   |
| Database   | Firebase Firestore                  |
| Auth       | Firebase Auth + Biometrics          |
| Storage    | Firebase Storage (study materials)  |
| Real-time  | Firestore real-time listeners       |
| Push Notif | Firebase Cloud Messaging (FCM)      |

---

## ⚙️ Setup Instructions

### Prerequisites
- Node.js 18+
- EAS CLI: `npm install -g eas-cli` (the only Expo tool installed globally)
- The Expo CLI is bundled in the project — use `npx expo <command>` from inside `mobile/`
- Firebase project (see below)

### 1. Firebase Setup
1. Go to [Firebase Console](https://console.firebase.google.com)
2. Create a new project called `studymatch`
3. Enable **Authentication** → Email/Password + Google
4. Enable **Firestore Database** (production mode)
5. Enable **Storage**
6. Download `google-services.json` (Android) and `GoogleService-Info.plist` (iOS)
7. Place them in `mobile/`

### 2. Backend Setup
```bash
cd backend
cp .env.example .env
# Fill in your Firebase Admin SDK credentials in .env
npm install
npm run dev
```

### 3. Mobile Setup
```bash
cd mobile
cp .env.example .env
# Set EXPO_PUBLIC_API_URL to your backend URL
npm install
npx expo start
```

---

## 🔐 Security Features

- **University Email Verification** — only `.edu` or verified university domains
- **Biometric Authentication** — Face ID / Fingerprint via `expo-local-authentication`
- **Multi-layer Auth** — Firebase Auth + JWT tokens for API calls
- **Rate Limiting** — API protected against brute force
- **Firestore Rules** — users can only read/write their own data

---

## 🧠 Matching Algorithm

The compatibility score (0–100) is calculated using weighted traits:

| Trait                  | Weight |
|------------------------|--------|
| Subject/Course Match   | 30%    |
| Study Schedule Overlap | 25%    |
| Learning Style         | 15%    |
| Study Environment      | 10%    |
| Academic Level         | 10%    |
| Study Goals            | 10%    |

Scores above 60 are surfaced as matches. The algorithm runs server-side to prevent manipulation.

---

## 📱 App Screens

### Auth Flow
- `SplashScreen` — animated logo
- `OnboardingScreen` — feature walkthrough
- `LoginScreen` — email/password + biometric
- `RegisterScreen` — university email verification
- `ProfileSetupScreen` — study traits questionnaire

### Main App
- `DiscoverScreen` — swipe cards (like Tinder)
- `MatchesScreen` — list of mutual matches
- `ChatScreen` — 1-on-1 and group messaging
- `GroupsScreen` — study groups
- `MaterialsScreen` — shared study materials
- `ProfileScreen` — edit your profile & traits

---

## 🗃️ Firestore Collections

```
users/{userId}
  - profile info, study traits, bio

matches/{matchId}
  - userA, userB, compatibilityScore, timestamp

chats/{chatId}
  - participants[], type (direct/group), lastMessage

messages/{chatId}/messages/{messageId}
  - senderId, text, attachments, timestamp

groups/{groupId}
  - name, members[], subject, materials[]

materials/{materialId}
  - uploaderId, fileUrl, subject, description
```

---

## 📄 License
MIT
