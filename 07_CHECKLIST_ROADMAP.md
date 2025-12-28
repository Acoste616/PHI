# PHI CHECKLIST & ROADMAP
## Step-by-Step Implementation Guide for AI Coding Agents
**Version:** 1.0.0  
**Classification:** Execution Playbook  
**For:** Cursor, AI Coding Agents, Developers  

---

## IMPLEMENTATION PHILOSOPHY

### Core Principles
1. **Quick Wins First** - Deliver visible value in 15-60 minute sprints
2. **Incremental Complexity** - Start simple, add features layer by layer
3. **Test Each Step** - Verify before moving to next task
4. **No Dead Ends** - Each step produces working code

### Definition of Done (DoD)
Each task is complete when:
- [ ] Code compiles without errors
- [ ] Feature works as specified
- [ ] No TypeScript errors
- [ ] Console has no warnings related to the feature
- [ ] Tested on Android emulator/device

---

## PHASE 0: PROJECT SETUP
**Estimated Time:** 30 minutes  
**Prerequisites:** Node.js 18+, Android Studio, Expo CLI  

### Task 0.1: Create Expo Project

```bash
# Create project with Expo Dev Client template
npx create-expo-app@latest phi-app --template expo-template-blank-typescript

# Navigate to project
cd phi-app

# Install dev client for native modules
npx expo install expo-dev-client
```

**Verification:**
```bash
npx expo start
# Should open Expo Dev menu
```

---

### Task 0.2: Install Core Dependencies

```bash
# State Management
npm install zustand

# Storage
npx expo install expo-sqlite expo-secure-store @react-native-async-storage/async-storage

# Navigation
npx expo install expo-router react-native-screens react-native-safe-area-context

# UI Components
npx expo install expo-haptics expo-linear-gradient

# HTTP Client
npm install axios

# Date handling
npm install date-fns

# Icons
npm install lucide-react-native react-native-svg
```

**Verification:**
```bash
npm list zustand expo-sqlite expo-router
# All packages should be listed
```

---

### Task 0.3: Configure Expo Router

**File:** `app.json`
```json
{
  "expo": {
    "name": "PHI",
    "slug": "phi-app",
    "version": "1.0.0",
    "scheme": "phi",
    "plugins": [
      "expo-router",
      "expo-secure-store"
    ],
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

**File:** `app/_layout.tsx`
```typescript
import { Stack } from 'expo-router';
import { StatusBar } from 'expo-status-bar';

export default function RootLayout() {
  return (
    <>
      <StatusBar style="light" />
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(tabs)" />
      </Stack>
    </>
  );
}
```

**Verification:**
- App launches without navigation errors
- Status bar is visible

---

### Task 0.4: Create Directory Structure

```bash
mkdir -p app/\(tabs\)
mkdir -p components/{commander,chat,insights,common}
mkdir -p services/{database,api,ai,sync}
mkdir -p stores
mkdir -p hooks
mkdir -p types
mkdir -p utils
mkdir -p constants
```

**DoD:** All directories exist, visible in file explorer

---

## PHASE 1: QUICK WINS (MVP Core)
**Goal:** Working app with basic functionality in <4 hours

---

### 🏆 QUICK WIN 1: Energy Check-In (45 min)

**Goal:** User can log energy level 1-10, see last 3 entries

#### Step 1.1: Create Database Service (15 min)

**File:** `services/database/index.ts`
```typescript
import * as SQLite from 'expo-sqlite';

let db: SQLite.SQLiteDatabase | null = null;

export async function getDatabase(): Promise<SQLite.SQLiteDatabase> {
  if (db) return db;
  
  db = await SQLite.openDatabaseAsync('phi.db');
  
  // Create tables
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS energy_entries (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      energy_level INTEGER NOT NULL CHECK (energy_level BETWEEN 1 AND 10),
      activity TEXT,
      note TEXT,
      timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
    );
  `);
  
  return db;
}

export async function insertEnergyEntry(
  energyLevel: number,
  activity?: string,
  note?: string
): Promise<number> {
  const db = await getDatabase();
  const result = await db.runAsync(
    'INSERT INTO energy_entries (energy_level, activity, note) VALUES (?, ?, ?)',
    [energyLevel, activity ?? null, note ?? null]
  );
  return result.lastInsertRowId;
}

export async function getRecentEntries(limit: number = 5): Promise<EnergyEntry[]> {
  const db = await getDatabase();
  const entries = await db.getAllAsync<EnergyEntry>(
    'SELECT * FROM energy_entries ORDER BY timestamp DESC LIMIT ?',
    [limit]
  );
  return entries;
}

export interface EnergyEntry {
  id: number;
  energy_level: number;
  activity: string | null;
  note: string | null;
  timestamp: string;
}
```

#### Step 1.2: Create Energy Check-In Component (20 min)

**File:** `components/commander/EnergyCheckIn.tsx`
```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, FlatList } from 'react-native';
import { insertEnergyEntry, getRecentEntries, EnergyEntry } from '@/services/database';
import * as Haptics from 'expo-haptics';

const ENERGY_LEVELS = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

export function EnergyCheckIn() {
  const [selectedLevel, setSelectedLevel] = useState<number | null>(null);
  const [recentEntries, setRecentEntries] = useState<EnergyEntry[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    loadEntries();
  }, []);

  const loadEntries = async () => {
    const entries = await getRecentEntries(3);
    setRecentEntries(entries);
  };

  const handleSelect = (level: number) => {
    setSelectedLevel(level);
    Haptics.selectionAsync();
  };

  const handleSubmit = async () => {
    if (!selectedLevel) return;
    
    setLoading(true);
    await insertEnergyEntry(selectedLevel);
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    setSelectedLevel(null);
    await loadEntries();
    setLoading(false);
  };

  const getEnergyColor = (level: number): string => {
    if (level >= 8) return '#2ECC71';
    if (level >= 5) return '#F39C12';
    return '#E74C3C';
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>How's your energy?</Text>
      
      <View style={styles.levelRow}>
        {ENERGY_LEVELS.map((level) => (
          <TouchableOpacity
            key={level}
            style={[
              styles.levelButton,
              selectedLevel === level && { backgroundColor: getEnergyColor(level) },
            ]}
            onPress={() => handleSelect(level)}
          >
            <Text style={[
              styles.levelText,
              selectedLevel === level && styles.levelTextSelected,
            ]}>
              {level}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      <TouchableOpacity
        style={[styles.submitButton, !selectedLevel && styles.submitButtonDisabled]}
        onPress={handleSubmit}
        disabled={!selectedLevel || loading}
      >
        <Text style={styles.submitText}>
          {loading ? 'Saving...' : 'Log Energy'}
        </Text>
      </TouchableOpacity>

      <View style={styles.historySection}>
        <Text style={styles.historyTitle}>Recent</Text>
        {recentEntries.map((entry) => (
          <View key={entry.id} style={styles.historyItem}>
            <View style={[styles.historyDot, { backgroundColor: getEnergyColor(entry.energy_level) }]} />
            <Text style={styles.historyLevel}>{entry.energy_level}/10</Text>
            <Text style={styles.historyTime}>
              {new Date(entry.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
            </Text>
          </View>
        ))}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#141414',
    borderRadius: 12,
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#FFFFFF',
    marginBottom: 16,
  },
  levelRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 16,
  },
  levelButton: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#1E1E1E',
    justifyContent: 'center',
    alignItems: 'center',
  },
  levelText: {
    color: '#A0A0A0',
    fontSize: 14,
    fontWeight: '500',
  },
  levelTextSelected: {
    color: '#FFFFFF',
  },
  submitButton: {
    backgroundColor: '#FF4500',
    paddingVertical: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  submitButtonDisabled: {
    backgroundColor: '#333333',
  },
  submitText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
  historySection: {
    marginTop: 20,
  },
  historyTitle: {
    color: '#666666',
    fontSize: 12,
    marginBottom: 8,
  },
  historyItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 8,
  },
  historyDot: {
    width: 8,
    height: 8,
    borderRadius: 4,
    marginRight: 8,
  },
  historyLevel: {
    color: '#FFFFFF',
    fontSize: 14,
    flex: 1,
  },
  historyTime: {
    color: '#666666',
    fontSize: 12,
  },
});
```

#### Step 1.3: Add to Home Screen (10 min)

**File:** `app/(tabs)/index.tsx`
```typescript
import { View, StyleSheet, SafeAreaView, Text } from 'react-native';
import { EnergyCheckIn } from '@/components/commander/EnergyCheckIn';

export default function CommanderScreen() {
  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.greeting}>Good morning</Text>
        <Text style={styles.subtitle}>⚡ High Energy Mode</Text>
      </View>
      
      <EnergyCheckIn />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0A0A0A',
    padding: 16,
  },
  header: {
    marginBottom: 24,
  },
  greeting: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
  subtitle: {
    fontSize: 14,
    color: '#FF4500',
    marginTop: 4,
  },
});
```

**✅ VERIFICATION:**
- [ ] App displays energy buttons 1-10
- [ ] Selecting a number highlights it
- [ ] Submit saves to database
- [ ] Recent entries appear below

---

### 🏆 QUICK WIN 2: Readiness Display (30 min)

**Goal:** Show mock readiness score with visual ring

#### Step 2.1: Create Readiness Ring Component

**File:** `components/commander/ReadinessRing.tsx`
```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import Svg, { Circle } from 'react-native-svg';

interface ReadinessRingProps {
  score: number;
  size?: number;
}

export function ReadinessRing({ score, size = 180 }: ReadinessRingProps) {
  const strokeWidth = 12;
  const radius = (size - strokeWidth) / 2;
  const circumference = 2 * Math.PI * radius;
  const progress = (score / 100) * circumference;
  
  const getColor = (score: number): string => {
    if (score >= 85) return '#2ECC71';
    if (score >= 70) return '#27AE60';
    if (score >= 55) return '#F39C12';
    if (score >= 40) return '#E67E22';
    return '#E74C3C';
  };

  const getStatus = (score: number): string => {
    if (score >= 85) return 'Peak performance';
    if (score >= 70) return 'Good to go';
    if (score >= 55) return 'Take it easy';
    if (score >= 40) return 'Recovery mode';
    return 'Rest day';
  };

  const color = getColor(score);

  return (
    <View style={styles.container}>
      <Svg width={size} height={size}>
        {/* Background circle */}
        <Circle
          cx={size / 2}
          cy={size / 2}
          r={radius}
          stroke="#1E1E1E"
          strokeWidth={strokeWidth}
          fill="none"
        />
        {/* Progress circle */}
        <Circle
          cx={size / 2}
          cy={size / 2}
          r={radius}
          stroke={color}
          strokeWidth={strokeWidth}
          fill="none"
          strokeDasharray={`${progress} ${circumference}`}
          strokeDashoffset={0}
          strokeLinecap="round"
          transform={`rotate(-90 ${size / 2} ${size / 2})`}
        />
      </Svg>
      
      <View style={[styles.centerContent, { width: size, height: size }]}>
        <Text style={[styles.score, { color }]}>{score}</Text>
        <Text style={styles.status}>{getStatus(score)}</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    justifyContent: 'center',
  },
  centerContent: {
    position: 'absolute',
    justifyContent: 'center',
    alignItems: 'center',
  },
  score: {
    fontSize: 48,
    fontWeight: 'bold',
  },
  status: {
    fontSize: 14,
    color: '#A0A0A0',
    marginTop: 4,
  },
});
```

#### Step 2.2: Update Home Screen

**File:** `app/(tabs)/index.tsx` (update)
```typescript
import { View, StyleSheet, SafeAreaView, Text, ScrollView } from 'react-native';
import { EnergyCheckIn } from '@/components/commander/EnergyCheckIn';
import { ReadinessRing } from '@/components/commander/ReadinessRing';

export default function CommanderScreen() {
  // Mock data - will be replaced with Oura data
  const mockReadiness = 74;

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView showsVerticalScrollIndicator={false}>
        <View style={styles.header}>
          <Text style={styles.greeting}>Good morning</Text>
          <Text style={styles.subtitle}>⚡ High Energy Mode</Text>
        </View>
        
        <View style={styles.readinessContainer}>
          <ReadinessRing score={mockReadiness} />
        </View>
        
        <EnergyCheckIn />
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0A0A0A',
    padding: 16,
  },
  header: {
    marginBottom: 24,
  },
  greeting: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
  subtitle: {
    fontSize: 14,
    color: '#FF4500',
    marginTop: 4,
  },
  readinessContainer: {
    alignItems: 'center',
    marginBottom: 24,
  },
});
```

**✅ VERIFICATION:**
- [ ] Readiness ring displays with score 74
- [ ] Ring color is green (score > 70)
- [ ] Status text shows "Good to go"

---

### 🏆 QUICK WIN 3: Tab Navigation (20 min)

**Goal:** Working tab bar with 3 tabs

#### Step 3.1: Create Tab Layout

**File:** `app/(tabs)/_layout.tsx`
```typescript
import { Tabs } from 'expo-router';
import { Zap, MessageCircle, TrendingUp } from 'lucide-react-native';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarStyle: {
          backgroundColor: '#141414',
          borderTopColor: '#1E1E1E',
          borderTopWidth: 1,
          height: 80,
          paddingBottom: 20,
          paddingTop: 12,
        },
        tabBarActiveTintColor: '#FF4500',
        tabBarInactiveTintColor: '#666666',
        tabBarLabelStyle: {
          fontSize: 12,
          fontWeight: '500',
        },
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Commander',
          tabBarIcon: ({ color, size }) => <Zap size={size} color={color} />,
        }}
      />
      <Tabs.Screen
        name="chat"
        options={{
          title: 'Wingman',
          tabBarIcon: ({ color, size }) => <MessageCircle size={size} color={color} />,
        }}
      />
      <Tabs.Screen
        name="insights"
        options={{
          title: 'Insights',
          tabBarIcon: ({ color, size }) => <TrendingUp size={size} color={color} />,
        }}
      />
    </Tabs>
  );
}
```

#### Step 3.2: Create Placeholder Screens

**File:** `app/(tabs)/chat.tsx`
```typescript
import { View, Text, StyleSheet, SafeAreaView } from 'react-native';

export default function ChatScreen() {
  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>Daily Wingman</Text>
      <Text style={styles.subtitle}>Chat coming soon...</Text>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0A0A0A',
    padding: 16,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
  subtitle: {
    fontSize: 16,
    color: '#666666',
    marginTop: 8,
  },
});
```

**File:** `app/(tabs)/insights.tsx`
```typescript
import { View, Text, StyleSheet, SafeAreaView } from 'react-native';

export default function InsightsScreen() {
  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>Insights</Text>
      <Text style={styles.subtitle}>Patterns & trends coming soon...</Text>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#0A0A0A',
    padding: 16,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#FFFFFF',
  },
  subtitle: {
    fontSize: 16,
    color: '#666666',
    marginTop: 8,
  },
});
```

**✅ VERIFICATION:**
- [ ] Three tabs visible at bottom
- [ ] Tapping each tab switches screens
- [ ] Active tab is highlighted in orange

---

## PHASE 2: OURA INTEGRATION
**Goal:** Connect to Oura API, display real data

---

### Task 2.1: Create Oura Auth Service (45 min)

**File:** `services/api/oura.ts`

Reference: `05_API_ENDPOINTS.md` Section 1.3

**Steps:**
1. Copy OuraApiService class from API_ENDPOINTS.md
2. Create `.env` file with OURA credentials
3. Implement OAuth flow with expo-auth-session
4. Store tokens in SecureStore

**✅ VERIFICATION:**
- [ ] OAuth redirect works
- [ ] Token stored securely
- [ ] Can fetch /usercollection/daily_readiness

---

### Task 2.2: Create Health Store (30 min)

**File:** `stores/healthStore.ts`

Reference: `04_TECH_BIBLE.md` Section 2.2

**Steps:**
1. Copy healthStore implementation from TECH_BIBLE.md
2. Connect to database for persistence
3. Create hook `useHealthData()`

**✅ VERIFICATION:**
- [ ] Store persists across app restarts
- [ ] Baselines calculate correctly

---

### Task 2.3: Sync Service (45 min)

**File:** `services/sync/ouraSync.ts`

```typescript
import { ouraService } from '../api/oura';
import { useHealthStore } from '@/stores/healthStore';
import { getDatabase } from '../database';

export async function syncOuraData(): Promise<void> {
  const today = new Date().toISOString().split('T')[0];
  const weekAgo = new Date(Date.now() - 7 * 86400000).toISOString().split('T')[0];
  
  try {
    // Fetch all data
    const [readiness, sleep, activity] = await Promise.all([
      ouraService.getReadiness(weekAgo, today),
      ouraService.getSleep(weekAgo, today),
      ouraService.getActivity(weekAgo, today),
    ]);
    
    // Store in database
    const db = await getDatabase();
    
    for (const r of readiness) {
      await db.runAsync(
        `INSERT OR REPLACE INTO health_metrics 
         (metric_date, metric_type, score, source, raw_json)
         VALUES (?, 'readiness', ?, 'oura', ?)`,
        [r.day, r.score, JSON.stringify(r)]
      );
    }
    
    // Update store
    const store = useHealthStore.getState();
    const todayReadiness = readiness.find(r => r.day === today);
    if (todayReadiness) {
      store.setTodayReadiness({
        date: todayReadiness.day,
        score: todayReadiness.score,
        contributors: todayReadiness.contributors,
      });
    }
    
    console.log('Oura sync complete');
  } catch (error) {
    console.error('Oura sync failed:', error);
    throw error;
  }
}
```

**✅ VERIFICATION:**
- [ ] Sync completes without errors
- [ ] Data appears in SQLite
- [ ] UI updates with real readiness score

---

## PHASE 3: AI CHAT
**Goal:** Working chat with Gemini 1.5 Pro

---

### Task 3.1: Gemini Service (30 min)

**File:** `services/api/gemini.ts`

Reference: `05_API_ENDPOINTS.md` Section 3.3

**✅ VERIFICATION:**
- [ ] API key configured
- [ ] generateContent returns valid response
- [ ] Error handling works

---

### Task 3.2: Chat UI (60 min)

**Files:**
- `components/chat/MessageBubble.tsx`
- `components/chat/ChatInput.tsx`
- `stores/aiStore.ts`

Reference: `06_UI_UX_SPECS.md` Section 4.2

**✅ VERIFICATION:**
- [ ] Messages display correctly
- [ ] User can type and send
- [ ] AI responds within 5 seconds
- [ ] Conversation persists in memory

---

### Task 3.3: Morning Briefing (45 min)

**File:** `services/ai/brain.ts`

```typescript
import { geminiService } from '../api/gemini';
import { useHealthStore } from '@/stores/healthStore';
import { useContextStore } from '@/stores/contextStore';

export async function generateMorningBriefing(): Promise<string> {
  const health = useHealthStore.getState();
  const context = useContextStore.getState();
  
  const briefing = await geminiService.generateMorningBriefing({
    readiness: health.todayReadiness?.score ?? 70,
    sleepScore: health.todaySleep?.score ?? 70,
    sleepDuration: health.todaySleep?.totalSleepSeconds ?? 25200,
    hrvStatus: getHRVStatus(health),
    baselineReadiness: health.baselineReadiness ?? 70,
    yesterdayContext: context.recentEntries.map(e => e.activityDetail),
    commanderMode: 'high_energy',
  });
  
  return briefing;
}

function getHRVStatus(health: any): string {
  if (!health.baselineHRV || !health.todayHRV) return 'Unknown';
  
  const deviation = (health.todayHRV.average - health.baselineHRV) / health.baselineHRV;
  
  if (deviation > 0.1) return 'Above baseline (+' + Math.round(deviation * 100) + '%)';
  if (deviation > -0.1) return 'At baseline';
  return 'Below baseline (' + Math.round(deviation * 100) + '%)';
}
```

**✅ VERIFICATION:**
- [ ] Morning briefing generates on app open
- [ ] Content reflects actual health data
- [ ] Response time < 5 seconds

---

## PHASE 4: HEALTH CONNECT (Native Module)
**Goal:** Integrate Samsung Health data via Health Connect

---

### Task 4.1: Create Native Module (90 min)

**Files:**
- `android/app/src/main/java/com/phi/healthconnect/HealthConnectModule.kt`
- `android/app/src/main/java/com/phi/healthconnect/HealthConnectPackage.kt`

Reference: `04_TECH_BIBLE.md` Section 3.1

**Steps:**
1. Run `npx expo prebuild` to generate android folder
2. Create Kotlin files from TECH_BIBLE.md
3. Register package in MainApplication.kt
4. Create TypeScript wrapper

**✅ VERIFICATION:**
- [ ] Module accessible from JS
- [ ] Permissions request works
- [ ] Can read step count

---

### Task 4.2: Health Connect Service (30 min)

**File:** `services/api/healthConnect.ts`

Reference: `05_API_ENDPOINTS.md` Section 2.1

**✅ VERIFICATION:**
- [ ] Service checks availability correctly
- [ ] Returns data on supported devices
- [ ] Graceful fallback on unsupported devices

---

## PHASE 5: PATTERNS & PREDICTIONS
**Goal:** Correlation engine and prediction display

---

### Task 5.1: Pattern Detection (60 min)

**File:** `services/ai/correlation.ts`

Reference: `02_WHITE_PAPER.md` Section 5.2

**✅ VERIFICATION:**
- [ ] Detects alcohol → HRV pattern
- [ ] Detects late meal → sleep pattern
- [ ] Confidence calculation works

---

### Task 5.2: Insights Screen (45 min)

**File:** `app/(tabs)/insights.tsx`

Reference: `06_UI_UX_SPECS.md` Section 5

**✅ VERIFICATION:**
- [ ] Trend chart displays
- [ ] Pattern cards show
- [ ] Period selector works

---

## PHASE 6: PROACTIVE FEATURES
**Goal:** Check-in prompts and notifications

---

### Task 6.1: Background Scheduler (45 min)

```bash
npx expo install expo-notifications expo-task-manager expo-background-fetch
```

**File:** `services/sync/scheduler.ts`

**✅ VERIFICATION:**
- [ ] Check-in prompt fires at scheduled times
- [ ] Background sync runs every 4 hours
- [ ] Battery impact < 5%

---

### Task 6.2: Notification Handling (30 min)

**File:** `services/notifications.ts`

**✅ VERIFICATION:**
- [ ] Notifications appear
- [ ] Tapping opens correct screen
- [ ] Quiet hours respected

---

## PHASE 7: POLISH & OPTIMIZATION
**Goal:** Production-ready app

---

### Task 7.1: Error Boundaries (30 min)

**File:** `components/common/ErrorBoundary.tsx`

---

### Task 7.2: Loading States (30 min)

**File:** `components/common/LoadingState.tsx`

---

### Task 7.3: Offline Support (45 min)

- Cache API responses
- Queue mutations
- Sync when online

---

### Task 7.4: Performance Optimization (60 min)

- Memoization
- Lazy loading
- Image optimization

---

## DEPLOYMENT CHECKLIST

### Pre-Release
- [ ] All VERIFICATION checkpoints pass
- [ ] No TypeScript errors
- [ ] No console warnings
- [ ] Memory usage stable
- [ ] Battery usage acceptable
- [ ] Crash-free on 100 test launches

### Release Build
```bash
# Generate Android bundle
eas build --platform android --profile production

# Submit to Play Store
eas submit --platform android
```

### Post-Release
- [ ] Monitor crash reports
- [ ] Check API error rates
- [ ] Review user feedback
- [ ] Plan next iteration

---

## TIME ESTIMATES SUMMARY

| Phase | Tasks | Estimated Time |
|-------|-------|----------------|
| Phase 0 | Setup | 30 min |
| Phase 1 | Quick Wins | 2 hours |
| Phase 2 | Oura Integration | 2 hours |
| Phase 3 | AI Chat | 2.5 hours |
| Phase 4 | Health Connect | 2 hours |
| Phase 5 | Patterns | 2 hours |
| Phase 6 | Proactive | 1.5 hours |
| Phase 7 | Polish | 3 hours |
| **TOTAL** | | **~15 hours** |

---

**END OF CHECKLIST & ROADMAP**

*Follow this document sequentially. Each task builds on previous ones. Skip no steps. Verify before proceeding.*
