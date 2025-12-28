# PHI - Personal Health Intelligence
## Digital Biological Twin for High-Energy Executives

[![React Native](https://img.shields.io/badge/React%20Native-0.73+-blue.svg)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo-SDK%2050+-black.svg)](https://expo.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-Proprietary-red.svg)]()

---

## 🎯 Overview

PHI (Personal Health Intelligence) is a mobile application that creates a **Digital Biological Twin** - an AI-powered system that understands your body and mind to optimize energy, focus, and performance.

**Core Capabilities:**
- 🔋 Real-time Readiness Score from Oura Ring
- 💬 AI Wingman for proactive health coaching
- 📊 Pattern detection (sleep, activity, stress correlations)
- ⚡ Commander Modes adapting to your energy state

---

## 📚 Documentation Index

| Document | Purpose |
|----------|---------|
| [01_MANIFEST.md](./01_MANIFEST.md) | Vision, philosophy, core principles |
| [02_WHITE_PAPER.md](./02_WHITE_PAPER.md) | Scientific foundations, psychological models |
| [03_BLUEPRINT.md](./03_BLUEPRINT.md) | System architecture, data flow diagrams |
| [04_TECH_BIBLE.md](./04_TECH_BIBLE.md) | Database schema, stores, native modules |
| [05_API_ENDPOINTS.md](./05_API_ENDPOINTS.md) | Oura, Health Connect, Gemini integration |
| [06_UI_UX_SPECS.md](./06_UI_UX_SPECS.md) | Screen layouts, components, animations |
| [07_CHECKLIST_ROADMAP.md](./07_CHECKLIST_ROADMAP.md) | Step-by-step implementation guide |

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** 18+ ([Download](https://nodejs.org/))
- **Android Studio** with Android SDK ([Download](https://developer.android.com/studio))
- **Expo CLI**: `npm install -g expo-cli`
- **EAS CLI**: `npm install -g eas-cli`

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-org/phi-app.git
cd phi-app

# 2. Install dependencies
npm install

# 3. Create environment file
cp .env.example .env
# Edit .env with your API keys (see Environment Variables section)

# 4. Generate native code (required for Health Connect)
npx expo prebuild

# 5. Start development server
npx expo start --dev-client

# 6. Build development client for Android
npx expo run:android
```

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
# ===========================================
# PHI Environment Configuration
# ===========================================

# Oura API (Required)
# Get from: https://cloud.ouraring.com/oauth/applications
EXPO_PUBLIC_OURA_CLIENT_ID=your_oura_client_id
EXPO_PUBLIC_OURA_CLIENT_SECRET=your_oura_client_secret

# For development only - Personal Access Token
# Get from: https://cloud.ouraring.com/personal-access-tokens
EXPO_PUBLIC_OURA_PAT=your_personal_access_token

# Gemini API (Required)
# Get from: https://makersuite.google.com/app/apikey
EXPO_PUBLIC_GEMINI_API_KEY=your_gemini_api_key

# App Configuration
EXPO_PUBLIC_APP_ENV=development  # development | staging | production

# Optional: Sentry for error tracking
# SENTRY_DSN=your_sentry_dsn

# Optional: Analytics
# MIXPANEL_TOKEN=your_mixpanel_token
```

### API Key Setup Instructions

#### Oura API
1. Go to [Oura Developer Portal](https://cloud.ouraring.com/oauth/applications)
2. Create new application
3. Set redirect URI: `phi://oauth/callback`
4. Copy Client ID and Client Secret

#### Gemini API
1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Create API key
3. Enable Generative Language API

---

## 📁 Directory Structure

```
phi-app/
├── app/                              # Expo Router screens
│   ├── (tabs)/                       # Tab navigation screens
│   │   ├── _layout.tsx               # Tab bar configuration
│   │   ├── index.tsx                 # Commander (Home) screen
│   │   ├── chat.tsx                  # Wingman (Chat) screen
│   │   └── insights.tsx              # Insights screen
│   ├── _layout.tsx                   # Root layout
│   ├── onboarding/                   # Onboarding flow
│   │   ├── welcome.tsx
│   │   ├── oura-connect.tsx
│   │   ├── health-connect.tsx
│   │   └── mode-select.tsx
│   └── settings.tsx                  # Settings screen
│
├── components/                        # Reusable UI components
│   ├── commander/                     # Home screen components
│   │   ├── ReadinessRing.tsx         # Circular progress ring
│   │   ├── MetricCards.tsx           # Sleep/HRV/Energy cards
│   │   ├── EnergyCheckIn.tsx         # Energy logging widget
│   │   ├── EnergyForecast.tsx        # Daily timeline
│   │   └── AIInsightCard.tsx         # AI recommendation card
│   ├── chat/                          # Chat screen components
│   │   ├── MessageBubble.tsx         # Chat message UI
│   │   ├── ChatInput.tsx             # Text input with send
│   │   ├── QuickResponses.tsx        # Pre-set reply buttons
│   │   └── EnergySlider.tsx          # 1-10 energy input
│   ├── insights/                      # Insights screen components
│   │   ├── TrendChart.tsx            # Line chart for trends
│   │   ├── PatternCard.tsx           # Discovered pattern UI
│   │   └── PeriodSelector.tsx        # 7d/14d/30d picker
│   ├── common/                        # Shared components
│   │   ├── LoadingState.tsx          # Skeleton loaders
│   │   ├── ErrorBoundary.tsx         # Error handling
│   │   └── Button.tsx                # Styled button
│   └── themes/                        # Commander mode themes
│       ├── index.ts                  # Theme exports
│       ├── HighEnergy.ts             # Red/Orange theme
│       ├── ZenMaster.ts              # Blue/Teal theme
│       └── Recovery.ts               # Green theme
│
├── services/                          # Business logic layer
│   ├── database/                      # SQLite operations
│   │   ├── index.ts                  # DB initialization
│   │   ├── schema.ts                 # Table definitions
│   │   ├── migrations.ts             # Version upgrades
│   │   └── queries.ts                # CRUD operations
│   ├── api/                           # External API clients
│   │   ├── oura.ts                   # Oura Ring API
│   │   ├── healthConnect.ts          # Health Connect wrapper
│   │   ├── gemini.ts                 # Gemini 1.5 Pro
│   │   └── ollama.ts                 # Local AI (optional)
│   ├── ai/                            # AI orchestration
│   │   ├── brain.ts                  # Main AI controller
│   │   ├── prompts.ts                # System prompts
│   │   ├── correlation.ts            # Pattern detection
│   │   └── predictions.ts            # Energy forecasting
│   └── sync/                          # Data synchronization
│       ├── scheduler.ts              # Background jobs
│       ├── ouraSync.ts               # Oura data sync
│       └── reconciliation.ts         # Conflict resolution
│
├── stores/                            # Zustand state management
│   ├── userStore.ts                  # User profile & prefs
│   ├── healthStore.ts                # Biometric data
│   ├── contextStore.ts               # Episodic memory
│   └── aiStore.ts                    # AI state & history
│
├── hooks/                             # Custom React hooks
│   ├── useReadiness.ts               # Readiness data hook
│   ├── useEpisodicMemory.ts          # Context storage hook
│   ├── useAI.ts                      # AI response hook
│   └── useTheme.ts                   # Commander mode theming
│
├── types/                             # TypeScript definitions
│   ├── health.ts                     # Health data types
│   ├── context.ts                    # Episodic memory types
│   ├── ai.ts                         # AI response types
│   └── api.ts                        # API response types
│
├── utils/                             # Utility functions
│   ├── dateTime.ts                   # Date manipulation
│   ├── scoring.ts                    # PHI score calculations
│   ├── encryption.ts                 # Data encryption helpers
│   ├── haptics.ts                    # Haptic feedback
│   └── validation.ts                 # Input validation
│
├── constants/                         # App configuration
│   ├── config.ts                     # App settings
│   ├── theme.ts                      # Colors, typography
│   ├── prompts.ts                    # AI prompt templates
│   └── thresholds.ts                 # Health thresholds
│
├── native/                            # Native modules (Android)
│   └── healthconnect/                # Health Connect integration
│       ├── HealthConnectModule.kt    # Kotlin implementation
│       └── HealthConnectPackage.kt   # React Native bridge
│
├── assets/                            # Static assets
│   ├── fonts/                        # Custom fonts
│   │   ├── Inter-Regular.ttf
│   │   ├── Inter-Medium.ttf
│   │   ├── Inter-SemiBold.ttf
│   │   └── Inter-Bold.ttf
│   └── images/                       # Images & icons
│       └── icon.png
│
├── android/                           # Android native code (generated)
│   └── app/
│       └── src/main/java/com/phi/
│
├── .env                               # Environment variables (not in git)
├── .env.example                       # Environment template
├── app.json                           # Expo configuration
├── babel.config.js                    # Babel configuration
├── tsconfig.json                      # TypeScript configuration
├── package.json                       # Dependencies
└── README.md                          # This file
```

---

## 🔧 Development

### Running the App

```bash
# Start Expo dev server
npx expo start --dev-client

# Run on Android device/emulator
npx expo run:android

# Run on iOS simulator (Mac only)
npx expo run:ios
```

### Building for Production

```bash
# Configure EAS Build
eas build:configure

# Build Android APK
eas build --platform android --profile preview

# Build Android AAB for Play Store
eas build --platform android --profile production

# Submit to Play Store
eas submit --platform android
```

### Code Quality

```bash
# TypeScript check
npx tsc --noEmit

# ESLint
npx eslint . --ext .ts,.tsx

# Run tests
npm test
```

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      PHI ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│   │   Oura API  │    │   Health    │    │   Manual    │    │
│   │  (Cloud)    │    │  Connect    │    │   Input     │    │
│   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    │
│          │                  │                  │            │
│          └──────────────────┼──────────────────┘            │
│                             ▼                               │
│                  ┌──────────────────┐                       │
│                  │  Services Layer  │                       │
│                  │  (API + Sync)    │                       │
│                  └────────┬─────────┘                       │
│                           │                                 │
│            ┌──────────────┼──────────────┐                  │
│            ▼              ▼              ▼                  │
│     ┌──────────┐   ┌──────────┐   ┌──────────┐            │
│     │ SQLite   │   │ Zustand  │   │ Secure   │            │
│     │(Encrypted)│   │ Stores   │   │ Store    │            │
│     └──────────┘   └──────────┘   └──────────┘            │
│            │              │              │                  │
│            └──────────────┼──────────────┘                  │
│                           ▼                                 │
│                  ┌──────────────────┐                       │
│                  │    AI Brain      │                       │
│                  │ (Gemini/Ollama)  │                       │
│                  └────────┬─────────┘                       │
│                           │                                 │
│                           ▼                                 │
│                  ┌──────────────────┐                       │
│                  │    UI Layer      │                       │
│                  │ (React Native)   │                       │
│                  └──────────────────┘                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Key Features

### Commander Modes

| Mode | Trigger | AI Behavior |
|------|---------|-------------|
| ⚡ High Energy | Readiness > 80 | Direct, challenging |
| 🧘 Zen Master | Readiness 50-80 | Calm, supportive |
| 🔋 Social Battery | Social events | Energy management |
| 🌿 Recovery | Readiness < 50 | Gentle, protective |

### Data Privacy

- **Tier 1 (Critical):** OAuth tokens → SecureStore (Keychain)
- **Tier 2 (Sensitive):** Health data → SQLite + SQLCipher
- **Tier 3 (Standard):** Preferences → AsyncStorage

### AI Integration

- **Gemini 1.5 Pro:** Long-context analysis, natural conversation
- **Ollama (Local):** Raw data processing, privacy-first
- **Fallback:** Rule-based recommendations when offline

---

## 🧪 Testing

### Unit Tests
```bash
npm test
```

### E2E Tests (Detox)
```bash
npm run e2e:build
npm run e2e:test
```

### Manual Testing Checklist
- [ ] Oura OAuth flow completes
- [ ] Readiness score displays correctly
- [ ] Energy check-in saves to database
- [ ] AI responds within 5 seconds
- [ ] Patterns detected after 7+ days
- [ ] Notifications fire at scheduled times

---

## 🐛 Troubleshooting

### Common Issues

#### "Health Connect not available"
```bash
# Ensure Health Connect app is installed on device
# Check: Settings > Apps > Health Connect
```

#### "Oura auth fails"
```bash
# Verify redirect URI in Oura Developer Portal
# Should be: phi://oauth/callback
```

#### "Gemini API error"
```bash
# Check API key in .env
# Verify quota at: https://console.cloud.google.com/
```

#### "Build fails on Android"
```bash
# Clean and rebuild
cd android && ./gradlew clean && cd ..
npx expo prebuild --clean
npx expo run:android
```

---

## 📈 Roadmap

### Phase 1 (Current)
- [x] Basic readiness display
- [x] Energy check-in
- [x] Tab navigation
- [ ] Oura integration
- [ ] AI chat

### Phase 2
- [ ] Health Connect integration
- [ ] Pattern detection
- [ ] Proactive notifications

### Phase 3
- [ ] Advanced predictions
- [ ] Goal setting
- [ ] Export/sharing

---

## 🤝 Contributing

This is a proprietary project. For access or contribution inquiries, contact the Founder.

---

## 📄 License

Proprietary - All Rights Reserved

---

## 📞 Support

For technical issues, refer to:
1. This README
2. Documentation files in this directory
3. Inline code comments
4. Founder contact

---

**Built with ❤️ for High-Energy Executives**

*"Know thyself through data. Act through intelligence. Thrive through optimization."*
