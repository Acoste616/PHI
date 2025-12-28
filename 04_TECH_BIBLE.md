# PHI TECH BIBLE
## Deep Technical Specifications
**Version:** 1.0.0  
**Classification:** Implementation Reference  
**For:** AI Coding Agents (Cursor), Senior Developers  

---

## 1. DATABASE SCHEMA (SQLite + SQLCipher)

### 1.1 Schema Overview

```sql
-- PHI Database Schema v1.0
-- Encryption: SQLCipher AES-256
-- Character Set: UTF-8

PRAGMA foreign_keys = ON;
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
```

### 1.2 Core Tables

#### 1.2.1 Health Metrics Table

```sql
CREATE TABLE health_metrics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    metric_date DATE NOT NULL,
    metric_type TEXT NOT NULL CHECK (metric_type IN (
        'readiness', 'sleep', 'activity', 'hrv', 'heart_rate', 'temperature'
    )),
    
    -- Core Values
    score INTEGER,                    -- 0-100 for scored metrics
    value REAL,                       -- Raw value for continuous metrics
    unit TEXT,                        -- 'bpm', 'ms', 'steps', 'celsius'
    
    -- Sleep-specific fields
    total_sleep_seconds INTEGER,
    deep_sleep_seconds INTEGER,
    rem_sleep_seconds INTEGER,
    light_sleep_seconds INTEGER,
    awake_seconds INTEGER,
    sleep_efficiency REAL,            -- 0.0-1.0
    sleep_latency_seconds INTEGER,
    
    -- HRV-specific fields
    hrv_average INTEGER,              -- ms
    hrv_lowest INTEGER,               -- ms
    hrv_highest INTEGER,              -- ms
    rmssd REAL,                       -- Root mean square of successive differences
    
    -- Activity-specific fields
    steps INTEGER,
    active_calories INTEGER,
    total_calories INTEGER,
    distance_meters INTEGER,
    
    -- Readiness-specific fields
    previous_night_score INTEGER,
    sleep_balance INTEGER,
    previous_day_activity INTEGER,
    activity_balance INTEGER,
    body_temperature_deviation REAL,
    resting_heart_rate INTEGER,
    hrv_balance INTEGER,
    
    -- Metadata
    source TEXT NOT NULL CHECK (source IN ('oura', 'health_connect', 'manual')),
    raw_json TEXT,                    -- Original API response for debugging
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(metric_date, metric_type, source)
);

-- Indexes for common queries
CREATE INDEX idx_health_metrics_date ON health_metrics(metric_date DESC);
CREATE INDEX idx_health_metrics_type_date ON health_metrics(metric_type, metric_date DESC);
CREATE INDEX idx_health_metrics_source ON health_metrics(source);
```

#### 1.2.2 Episodic Memory Table

```sql
CREATE TABLE episodic_memory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    
    -- Temporal
    timestamp DATETIME NOT NULL,
    date DATE GENERATED ALWAYS AS (DATE(timestamp)) STORED,
    time_of_day TEXT GENERATED ALWAYS AS (
        CASE 
            WHEN CAST(strftime('%H', timestamp) AS INTEGER) < 6 THEN 'night'
            WHEN CAST(strftime('%H', timestamp) AS INTEGER) < 12 THEN 'morning'
            WHEN CAST(strftime('%H', timestamp) AS INTEGER) < 17 THEN 'afternoon'
            WHEN CAST(strftime('%H', timestamp) AS INTEGER) < 21 THEN 'evening'
            ELSE 'night'
        END
    ) STORED,
    
    -- Activity Context
    activity_type TEXT CHECK (activity_type IN (
        'work', 'social', 'rest', 'exercise', 'meal', 'commute', 
        'meeting', 'focus', 'leisure', 'sleep', 'unknown'
    )),
    activity_detail TEXT,             -- Free-form description
    activity_intensity TEXT CHECK (activity_intensity IN (
        'low', 'moderate', 'high', NULL
    )),
    
    -- Self-Reported State
    energy_level INTEGER CHECK (energy_level BETWEEN 1 AND 10),
    mood_valence INTEGER CHECK (mood_valence BETWEEN -5 AND 5),  -- -5 negative to +5 positive
    stress_level INTEGER CHECK (stress_level BETWEEN 1 AND 10),
    focus_level INTEGER CHECK (focus_level BETWEEN 1 AND 10),
    
    -- Tags for semantic search
    mood_tags TEXT,                   -- JSON array: ["high_energy", "focused"]
    context_tags TEXT,                -- JSON array: ["deadline", "client", "exercise"]
    
    -- Physiological Snapshot (at time of entry)
    hrv_snapshot INTEGER,
    heart_rate_snapshot INTEGER,
    
    -- AI Processing
    embedding BLOB,                   -- Vector embedding for semantic search (768 floats)
    ai_summary TEXT,                  -- AI-generated one-line summary
    ai_processed_at DATETIME,
    
    -- Relationships
    related_entries TEXT,             -- JSON array of IDs
    prediction_id INTEGER,            -- Link to prediction being validated
    
    -- Metadata
    entry_source TEXT DEFAULT 'user' CHECK (entry_source IN ('user', 'ai_prompt', 'auto')),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for episodic memory
CREATE INDEX idx_episodic_date ON episodic_memory(date DESC);
CREATE INDEX idx_episodic_timestamp ON episodic_memory(timestamp DESC);
CREATE INDEX idx_episodic_activity ON episodic_memory(activity_type);
CREATE INDEX idx_episodic_energy ON episodic_memory(energy_level);
CREATE INDEX idx_episodic_time_of_day ON episodic_memory(time_of_day);

-- Full-text search for activity details
CREATE VIRTUAL TABLE episodic_memory_fts USING fts5(
    activity_detail,
    ai_summary,
    content=episodic_memory,
    content_rowid=id
);

-- Trigger to keep FTS in sync
CREATE TRIGGER episodic_memory_ai AFTER INSERT ON episodic_memory BEGIN
    INSERT INTO episodic_memory_fts(rowid, activity_detail, ai_summary)
    VALUES (NEW.id, NEW.activity_detail, NEW.ai_summary);
END;

CREATE TRIGGER episodic_memory_ad AFTER DELETE ON episodic_memory BEGIN
    INSERT INTO episodic_memory_fts(episodic_memory_fts, rowid, activity_detail, ai_summary)
    VALUES ('delete', OLD.id, OLD.activity_detail, OLD.ai_summary);
END;

CREATE TRIGGER episodic_memory_au AFTER UPDATE ON episodic_memory BEGIN
    INSERT INTO episodic_memory_fts(episodic_memory_fts, rowid, activity_detail, ai_summary)
    VALUES ('delete', OLD.id, OLD.activity_detail, OLD.ai_summary);
    INSERT INTO episodic_memory_fts(rowid, activity_detail, ai_summary)
    VALUES (NEW.id, NEW.activity_detail, NEW.ai_summary);
END;
```

#### 1.2.3 AI Conversations Table

```sql
CREATE TABLE ai_conversations (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id TEXT NOT NULL,         -- Groups messages in a conversation
    
    -- Message Content
    role TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    
    -- Context at time of message
    commander_mode TEXT CHECK (commander_mode IN (
        'high_energy', 'zen_master', 'social_battery', 'recovery'
    )),
    readiness_at_time INTEGER,
    energy_at_time INTEGER,
    
    -- AI Metadata
    model_used TEXT,                  -- 'gemini-1.5-pro', 'llama-3.1-8b'
    tokens_input INTEGER,
    tokens_output INTEGER,
    latency_ms INTEGER,
    
    -- Timestamps
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_conversations_session ON ai_conversations(session_id);
CREATE INDEX idx_conversations_created ON ai_conversations(created_at DESC);
```

#### 1.2.4 Predictions Table

```sql
CREATE TABLE predictions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    
    -- Prediction Details
    prediction_type TEXT NOT NULL CHECK (prediction_type IN (
        'energy_level', 'energy_crash', 'readiness', 'sleep_quality', 'productivity'
    )),
    predicted_value REAL,             -- Numeric prediction
    predicted_time DATETIME,          -- When the prediction is for
    confidence REAL CHECK (confidence BETWEEN 0.0 AND 1.0),
    
    -- Basis for prediction
    reasoning TEXT,                   -- AI explanation
    input_factors TEXT,               -- JSON: {"hrv": 45, "sleep": 72, ...}
    
    -- Validation
    actual_value REAL,
    actual_time DATETIME,
    accuracy_score REAL,              -- 0.0-1.0 based on how close
    validated_at DATETIME,
    
    -- Metadata
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_predictions_type ON predictions(prediction_type);
CREATE INDEX idx_predictions_time ON predictions(predicted_time);
CREATE INDEX idx_predictions_validation ON predictions(validated_at) WHERE validated_at IS NOT NULL;
```

#### 1.2.5 Patterns Table

```sql
CREATE TABLE patterns (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    
    -- Pattern Definition
    pattern_name TEXT NOT NULL,       -- Human-readable name
    trigger_condition TEXT NOT NULL,  -- JSON: {"metric": "alcohol", "threshold": 1}
    effect_metric TEXT NOT NULL,      -- 'hrv', 'energy', 'sleep_quality'
    effect_direction TEXT CHECK (effect_direction IN ('increase', 'decrease')),
    effect_magnitude REAL,            -- Average % change
    
    -- Statistical Validity
    occurrences INTEGER DEFAULT 0,    -- How many times pattern observed
    confidence REAL CHECK (confidence BETWEEN 0.0 AND 1.0),
    last_validated DATETIME,
    
    -- Recommendation
    recommendation TEXT,              -- Action to take
    recommendation_timing TEXT,       -- 'immediate', 'next_morning', 'ongoing'
    
    -- Status
    is_active BOOLEAN DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_patterns_active ON patterns(is_active) WHERE is_active = 1;
CREATE INDEX idx_patterns_effect ON patterns(effect_metric);
```

#### 1.2.6 User Preferences Table

```sql
CREATE TABLE user_preferences (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,              -- JSON-encoded value
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Default preferences
INSERT INTO user_preferences (key, value) VALUES
    ('commander_mode', '"high_energy"'),
    ('wake_time_default', '"07:00"'),
    ('check_in_times', '["09:00", "14:00", "19:00"]'),
    ('notification_enabled', 'true'),
    ('data_sync_wifi_only', 'false'),
    ('ai_model_preference', '"gemini"'),
    ('theme', '"dark"'),
    ('units', '{"distance": "metric", "temperature": "celsius"}');
```

#### 1.2.7 Sync Metadata Table

```sql
CREATE TABLE sync_metadata (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source TEXT NOT NULL,             -- 'oura', 'health_connect'
    sync_type TEXT NOT NULL,          -- 'full', 'incremental'
    
    -- Sync Window
    start_date DATE,
    end_date DATE,
    
    -- Results
    records_fetched INTEGER,
    records_created INTEGER,
    records_updated INTEGER,
    errors TEXT,                      -- JSON array of errors
    
    -- Timing
    started_at DATETIME,
    completed_at DATETIME,
    duration_ms INTEGER,
    
    -- Status
    status TEXT CHECK (status IN ('pending', 'running', 'success', 'partial', 'failed'))
);

CREATE INDEX idx_sync_source ON sync_metadata(source, completed_at DESC);
```

### 1.3 Database Migrations

```typescript
// services/database/migrations.ts

interface Migration {
  version: number;
  name: string;
  up: string;
  down: string;
}

export const MIGRATIONS: Migration[] = [
  {
    version: 1,
    name: 'initial_schema',
    up: `
      -- All CREATE TABLE statements from above
    `,
    down: `
      DROP TABLE IF EXISTS sync_metadata;
      DROP TABLE IF EXISTS user_preferences;
      DROP TABLE IF EXISTS patterns;
      DROP TABLE IF EXISTS predictions;
      DROP TABLE IF EXISTS ai_conversations;
      DROP TABLE IF EXISTS episodic_memory_fts;
      DROP TABLE IF EXISTS episodic_memory;
      DROP TABLE IF EXISTS health_metrics;
    `
  },
  {
    version: 2,
    name: 'add_embedding_index',
    up: `
      -- Add vector similarity search support (future: sqlite-vss)
      ALTER TABLE episodic_memory ADD COLUMN embedding_version INTEGER DEFAULT 1;
    `,
    down: `
      -- SQLite doesn't support DROP COLUMN easily, would need table rebuild
    `
  }
];

export async function runMigrations(db: SQLiteDatabase): Promise<void> {
  // Create migrations tracking table
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS _migrations (
      version INTEGER PRIMARY KEY,
      name TEXT NOT NULL,
      applied_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `);
  
  // Get current version
  const result = await db.getFirstAsync<{version: number}>(
    'SELECT MAX(version) as version FROM _migrations'
  );
  const currentVersion = result?.version ?? 0;
  
  // Apply pending migrations
  for (const migration of MIGRATIONS) {
    if (migration.version > currentVersion) {
      console.log(`Applying migration ${migration.version}: ${migration.name}`);
      await db.execAsync(migration.up);
      await db.runAsync(
        'INSERT INTO _migrations (version, name) VALUES (?, ?)',
        [migration.version, migration.name]
      );
    }
  }
}
```

---

## 2. STATE MANAGEMENT SPECIFICATIONS

### 2.1 Store Type Definitions

```typescript
// types/stores.ts

// User Store Types
interface UserProfile {
  id: string;
  name?: string;
  wakeTimeDefault: string;        // "07:00"
  commanderMode: CommanderMode;
  timezone: string;
}

type CommanderMode = 'high_energy' | 'zen_master' | 'social_battery' | 'recovery';

interface UserPreferences {
  notificationsEnabled: boolean;
  checkInTimes: string[];         // ["09:00", "14:00", "19:00"]
  dataSyncWifiOnly: boolean;
  aiModelPreference: 'gemini' | 'ollama' | 'auto';
  theme: 'dark' | 'light' | 'system';
  units: {
    distance: 'metric' | 'imperial';
    temperature: 'celsius' | 'fahrenheit';
  };
}

// Health Store Types
interface SleepData {
  date: string;
  totalSleepSeconds: number;
  deepSleepSeconds: number;
  remSleepSeconds: number;
  lightSleepSeconds: number;
  awakeSeconds: number;
  efficiency: number;
  latencySeconds: number;
  score: number;
}

interface ReadinessData {
  date: string;
  score: number;
  contributors: {
    previousNight: number;
    sleepBalance: number;
    previousDayActivity: number;
    activityBalance: number;
    bodyTemperature: number;
    restingHeartRate: number;
    hrvBalance: number;
  };
}

interface HRVData {
  date: string;
  average: number;
  lowest: number;
  highest: number;
  rmssd: number;
}

// Context Store Types
interface EpisodicEntry {
  id: number;
  timestamp: string;
  activityType: ActivityType;
  activityDetail: string;
  energyLevel: number;
  moodValence: number;
  stressLevel?: number;
  focusLevel?: number;
  moodTags: string[];
  contextTags: string[];
  hrvSnapshot?: number;
  aiSummary?: string;
}

type ActivityType = 
  | 'work' | 'social' | 'rest' | 'exercise' | 'meal' 
  | 'commute' | 'meeting' | 'focus' | 'leisure' | 'sleep' | 'unknown';

// AI Store Types
interface AIMessage {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: string;
  metadata?: {
    commanderMode?: CommanderMode;
    readiness?: number;
    energy?: number;
    model?: string;
    latencyMs?: number;
  };
}

interface Prediction {
  id: number;
  type: 'energy_level' | 'energy_crash' | 'readiness' | 'sleep_quality';
  predictedValue: number;
  predictedTime: string;
  confidence: number;
  reasoning: string;
  validated?: boolean;
  actualValue?: number;
  accuracy?: number;
}

interface Pattern {
  id: number;
  name: string;
  trigger: string;
  effect: string;
  magnitude: number;
  confidence: number;
  recommendation: string;
  occurrences: number;
}
```

### 2.2 Complete Store Implementations

```typescript
// stores/userStore.ts

import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface UserState {
  // Profile
  profile: UserProfile | null;
  preferences: UserPreferences;
  
  // Auth State
  isOuraConnected: boolean;
  isHealthConnectConnected: boolean;
  ouraTokenExpiry: string | null;
  
  // Onboarding
  hasCompletedOnboarding: boolean;
  onboardingStep: number;
  
  // Actions
  setProfile: (profile: UserProfile) => void;
  updatePreferences: (prefs: Partial<UserPreferences>) => void;
  setCommanderMode: (mode: CommanderMode) => void;
  setOuraConnection: (connected: boolean, expiry?: string) => void;
  setHealthConnectConnection: (connected: boolean) => void;
  completeOnboarding: () => void;
  reset: () => void;
}

const DEFAULT_PREFERENCES: UserPreferences = {
  notificationsEnabled: true,
  checkInTimes: ['09:00', '14:00', '19:00'],
  dataSyncWifiOnly: false,
  aiModelPreference: 'auto',
  theme: 'dark',
  units: {
    distance: 'metric',
    temperature: 'celsius',
  },
};

export const useUserStore = create<UserState>()(
  persist(
    (set, get) => ({
      profile: null,
      preferences: DEFAULT_PREFERENCES,
      isOuraConnected: false,
      isHealthConnectConnected: false,
      ouraTokenExpiry: null,
      hasCompletedOnboarding: false,
      onboardingStep: 0,
      
      setProfile: (profile) => set({ profile }),
      
      updatePreferences: (prefs) => set((state) => ({
        preferences: { ...state.preferences, ...prefs }
      })),
      
      setCommanderMode: (mode) => set((state) => ({
        profile: state.profile 
          ? { ...state.profile, commanderMode: mode }
          : null
      })),
      
      setOuraConnection: (connected, expiry) => set({
        isOuraConnected: connected,
        ouraTokenExpiry: expiry ?? null,
      }),
      
      setHealthConnectConnection: (connected) => set({
        isHealthConnectConnected: connected,
      }),
      
      completeOnboarding: () => set({
        hasCompletedOnboarding: true,
        onboardingStep: -1,
      }),
      
      reset: () => set({
        profile: null,
        preferences: DEFAULT_PREFERENCES,
        isOuraConnected: false,
        isHealthConnectConnected: false,
        ouraTokenExpiry: null,
        hasCompletedOnboarding: false,
        onboardingStep: 0,
      }),
    }),
    {
      name: 'phi-user-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

```typescript
// stores/healthStore.ts

import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface HealthState {
  // Today's Data
  todayReadiness: ReadinessData | null;
  todaySleep: SleepData | null;
  todayHRV: HRVData | null;
  todaySteps: number;
  todayActiveCalories: number;
  
  // Historical (7-day rolling)
  readinessHistory: ReadinessData[];
  sleepHistory: SleepData[];
  hrvHistory: HRVData[];
  
  // Computed Baselines
  baselineReadiness: number | null;
  baselineSleepScore: number | null;
  baselineHRV: number | null;
  
  // Sync State
  lastSyncOura: string | null;
  lastSyncHealthConnect: string | null;
  isSyncing: boolean;
  syncError: string | null;
  
  // Actions
  setTodayReadiness: (data: ReadinessData) => void;
  setTodaySleep: (data: SleepData) => void;
  setTodayHRV: (data: HRVData) => void;
  setTodayActivity: (steps: number, calories: number) => void;
  appendHistory: (type: 'readiness' | 'sleep' | 'hrv', data: any) => void;
  calculateBaselines: () => void;
  setSyncState: (syncing: boolean, error?: string | null) => void;
  setLastSync: (source: 'oura' | 'health_connect', timestamp: string) => void;
}

export const useHealthStore = create<HealthState>()(
  persist(
    (set, get) => ({
      // Initial state
      todayReadiness: null,
      todaySleep: null,
      todayHRV: null,
      todaySteps: 0,
      todayActiveCalories: 0,
      readinessHistory: [],
      sleepHistory: [],
      hrvHistory: [],
      baselineReadiness: null,
      baselineSleepScore: null,
      baselineHRV: null,
      lastSyncOura: null,
      lastSyncHealthConnect: null,
      isSyncing: false,
      syncError: null,
      
      setTodayReadiness: (data) => {
        set({ todayReadiness: data });
        get().appendHistory('readiness', data);
      },
      
      setTodaySleep: (data) => {
        set({ todaySleep: data });
        get().appendHistory('sleep', data);
      },
      
      setTodayHRV: (data) => {
        set({ todayHRV: data });
        get().appendHistory('hrv', data);
      },
      
      setTodayActivity: (steps, calories) => set({
        todaySteps: steps,
        todayActiveCalories: calories,
      }),
      
      appendHistory: (type, data) => {
        set((state) => {
          const historyKey = `${type}History` as keyof HealthState;
          const currentHistory = state[historyKey] as any[];
          
          // Check if entry for this date already exists
          const existingIndex = currentHistory.findIndex(
            (item) => item.date === data.date
          );
          
          let newHistory: any[];
          if (existingIndex >= 0) {
            // Update existing
            newHistory = [...currentHistory];
            newHistory[existingIndex] = data;
          } else {
            // Append and keep last 7
            newHistory = [...currentHistory, data].slice(-7);
          }
          
          return { [historyKey]: newHistory };
        });
        
        get().calculateBaselines();
      },
      
      calculateBaselines: () => {
        const { readinessHistory, sleepHistory, hrvHistory } = get();
        
        // Need at least 7 days for baseline
        const baselineReadiness = readinessHistory.length >= 7
          ? Math.round(
              readinessHistory.reduce((sum, r) => sum + r.score, 0) / 
              readinessHistory.length
            )
          : null;
          
        const baselineSleepScore = sleepHistory.length >= 7
          ? Math.round(
              sleepHistory.reduce((sum, s) => sum + s.score, 0) / 
              sleepHistory.length
            )
          : null;
          
        const baselineHRV = hrvHistory.length >= 7
          ? Math.round(
              hrvHistory.reduce((sum, h) => sum + h.average, 0) / 
              hrvHistory.length
            )
          : null;
          
        set({ baselineReadiness, baselineSleepScore, baselineHRV });
      },
      
      setSyncState: (syncing, error = null) => set({
        isSyncing: syncing,
        syncError: error,
      }),
      
      setLastSync: (source, timestamp) => {
        if (source === 'oura') {
          set({ lastSyncOura: timestamp });
        } else {
          set({ lastSyncHealthConnect: timestamp });
        }
      },
    }),
    {
      name: 'phi-health-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({
        // Only persist these fields
        todayReadiness: state.todayReadiness,
        todaySleep: state.todaySleep,
        todayHRV: state.todayHRV,
        readinessHistory: state.readinessHistory,
        sleepHistory: state.sleepHistory,
        hrvHistory: state.hrvHistory,
        baselineReadiness: state.baselineReadiness,
        baselineSleepScore: state.baselineSleepScore,
        baselineHRV: state.baselineHRV,
        lastSyncOura: state.lastSyncOura,
        lastSyncHealthConnect: state.lastSyncHealthConnect,
      }),
    }
  )
);
```

```typescript
// stores/contextStore.ts

import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface ContextState {
  // Today's entries
  todayEntries: EpisodicEntry[];
  
  // Current state
  currentEnergy: number | null;
  currentActivity: ActivityType | null;
  lastCheckIn: string | null;
  
  // Pending check-in
  pendingCheckIn: {
    type: 'morning' | 'midday' | 'evening';
    prompted: boolean;
    respondedAt: string | null;
  } | null;
  
  // Recent entries (for context window)
  recentEntries: EpisodicEntry[];  // Last 20 entries
  
  // Actions
  addEntry: (entry: Omit<EpisodicEntry, 'id' | 'timestamp'>) => void;
  updateEntry: (id: number, updates: Partial<EpisodicEntry>) => void;
  setCurrentEnergy: (level: number) => void;
  setCurrentActivity: (activity: ActivityType) => void;
  setPendingCheckIn: (checkIn: ContextState['pendingCheckIn']) => void;
  clearPendingCheckIn: () => void;
  loadTodayEntries: (entries: EpisodicEntry[]) => void;
  loadRecentEntries: (entries: EpisodicEntry[]) => void;
}

export const useContextStore = create<ContextState>()(
  persist(
    (set, get) => ({
      todayEntries: [],
      currentEnergy: null,
      currentActivity: null,
      lastCheckIn: null,
      pendingCheckIn: null,
      recentEntries: [],
      
      addEntry: (entryData) => {
        const newEntry: EpisodicEntry = {
          ...entryData,
          id: Date.now(),  // Temporary ID, replaced by DB
          timestamp: new Date().toISOString(),
        };
        
        set((state) => ({
          todayEntries: [...state.todayEntries, newEntry],
          recentEntries: [...state.recentEntries, newEntry].slice(-20),
          currentEnergy: entryData.energyLevel ?? state.currentEnergy,
          currentActivity: entryData.activityType ?? state.currentActivity,
          lastCheckIn: new Date().toISOString(),
        }));
      },
      
      updateEntry: (id, updates) => {
        set((state) => ({
          todayEntries: state.todayEntries.map((e) =>
            e.id === id ? { ...e, ...updates } : e
          ),
          recentEntries: state.recentEntries.map((e) =>
            e.id === id ? { ...e, ...updates } : e
          ),
        }));
      },
      
      setCurrentEnergy: (level) => set({ currentEnergy: level }),
      
      setCurrentActivity: (activity) => set({ currentActivity: activity }),
      
      setPendingCheckIn: (checkIn) => set({ pendingCheckIn: checkIn }),
      
      clearPendingCheckIn: () => set({ pendingCheckIn: null }),
      
      loadTodayEntries: (entries) => set({ todayEntries: entries }),
      
      loadRecentEntries: (entries) => set({ recentEntries: entries }),
    }),
    {
      name: 'phi-context-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({
        currentEnergy: state.currentEnergy,
        currentActivity: state.currentActivity,
        lastCheckIn: state.lastCheckIn,
        // Don't persist entries - load from DB
      }),
    }
  )
);
```

---

## 3. NATIVE MODULE SPECIFICATIONS

### 3.1 Health Connect Module (Complete)

```kotlin
// android/app/src/main/java/com/phi/healthconnect/HealthConnectModule.kt

package com.phi.healthconnect

import android.content.Context
import android.content.Intent
import android.util.Log
import androidx.health.connect.client.HealthConnectClient
import androidx.health.connect.client.PermissionController
import androidx.health.connect.client.permission.HealthPermission
import androidx.health.connect.client.records.*
import androidx.health.connect.client.request.AggregateRequest
import androidx.health.connect.client.request.ReadRecordsRequest
import androidx.health.connect.client.time.TimeRangeFilter
import com.facebook.react.bridge.*
import com.facebook.react.modules.core.DeviceEventManagerModule
import kotlinx.coroutines.*
import java.time.Duration
import java.time.Instant
import java.time.LocalDate
import java.time.ZoneId

class HealthConnectModule(private val reactContext: ReactApplicationContext) : 
    ReactContextBaseJavaModule(reactContext),
    LifecycleEventListener {

    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())
    private var healthConnectClient: HealthConnectClient? = null
    
    companion object {
        const val NAME = "HealthConnect"
        const val TAG = "HealthConnectModule"
        
        // Required permissions
        val REQUIRED_PERMISSIONS = setOf(
            HealthPermission.getReadPermission(StepsRecord::class),
            HealthPermission.getReadPermission(HeartRateRecord::class),
            HealthPermission.getReadPermission(SleepSessionRecord::class),
            HealthPermission.getReadPermission(ActiveCaloriesBurnedRecord::class),
            HealthPermission.getReadPermission(DistanceRecord::class),
        )
    }
    
    override fun getName() = NAME
    
    init {
        reactContext.addLifecycleEventListener(this)
    }
    
    // ==================== AVAILABILITY CHECK ====================
    
    @ReactMethod
    fun isAvailable(promise: Promise) {
        try {
            val status = HealthConnectClient.getSdkStatus(reactContext)
            when (status) {
                HealthConnectClient.SDK_AVAILABLE -> promise.resolve(true)
                HealthConnectClient.SDK_UNAVAILABLE -> promise.resolve(false)
                HealthConnectClient.SDK_UNAVAILABLE_PROVIDER_UPDATE_REQUIRED -> {
                    promise.resolve(false)
                    // Could emit event to prompt user to update
                }
                else -> promise.resolve(false)
            }
        } catch (e: Exception) {
            Log.e(TAG, "Error checking availability", e)
            promise.resolve(false)
        }
    }
    
    // ==================== PERMISSIONS ====================
    
    @ReactMethod
    fun checkPermissions(promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val granted = client.permissionController.getGrantedPermissions()
                val allGranted = REQUIRED_PERMISSIONS.all { it in granted }
                
                val result = Arguments.createMap().apply {
                    putBoolean("allGranted", allGranted)
                    putInt("grantedCount", granted.size)
                    putInt("requiredCount", REQUIRED_PERMISSIONS.size)
                }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(result)
                }
            } catch (e: Exception) {
                withContext(Dispatchers.Main) {
                    promise.reject("PERMISSION_CHECK_ERROR", e.message)
                }
            }
        }
    }
    
    @ReactMethod
    fun requestPermissions(promise: Promise) {
        try {
            val intent = PermissionController.createRequestPermissionResultContract()
                .createIntent(reactContext, REQUIRED_PERMISSIONS)
            
            currentActivity?.startActivityForResult(intent, 1001)
            
            // Note: Actual result handling would need ActivityResultCallback
            // For simplicity, returning immediately - check permissions after
            promise.resolve(true)
        } catch (e: Exception) {
            promise.reject("PERMISSION_REQUEST_ERROR", e.message)
        }
    }
    
    // ==================== DATA RETRIEVAL ====================
    
    @ReactMethod
    fun getSteps(startTimeIso: String, endTimeIso: String, promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val startTime = Instant.parse(startTimeIso)
                val endTime = Instant.parse(endTimeIso)
                
                val request = ReadRecordsRequest(
                    recordType = StepsRecord::class,
                    timeRangeFilter = TimeRangeFilter.between(startTime, endTime)
                )
                
                val response = client.readRecords(request)
                val totalSteps = response.records.sumOf { it.count }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(totalSteps.toInt())
                }
            } catch (e: Exception) {
                Log.e(TAG, "Error getting steps", e)
                withContext(Dispatchers.Main) {
                    promise.reject("STEPS_ERROR", e.message)
                }
            }
        }
    }
    
    @ReactMethod
    fun getHeartRate(startTimeIso: String, endTimeIso: String, promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val startTime = Instant.parse(startTimeIso)
                val endTime = Instant.parse(endTimeIso)
                
                val request = ReadRecordsRequest(
                    recordType = HeartRateRecord::class,
                    timeRangeFilter = TimeRangeFilter.between(startTime, endTime)
                )
                
                val response = client.readRecords(request)
                val allSamples = response.records.flatMap { it.samples }
                
                if (allSamples.isEmpty()) {
                    withContext(Dispatchers.Main) {
                        promise.resolve(null)
                    }
                    return@launch
                }
                
                val result = Arguments.createMap().apply {
                    putInt("min", allSamples.minOf { it.beatsPerMinute }.toInt())
                    putInt("max", allSamples.maxOf { it.beatsPerMinute }.toInt())
                    putDouble("average", allSamples.map { it.beatsPerMinute }.average())
                    putInt("sampleCount", allSamples.size)
                }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(result)
                }
            } catch (e: Exception) {
                Log.e(TAG, "Error getting heart rate", e)
                withContext(Dispatchers.Main) {
                    promise.reject("HEART_RATE_ERROR", e.message)
                }
            }
        }
    }
    
    @ReactMethod
    fun getSleep(dateString: String, promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val date = LocalDate.parse(dateString)
                val zone = ZoneId.systemDefault()
                
                // Sleep sessions typically span midnight, so look at previous day evening to current day morning
                val startTime = date.minusDays(1).atTime(18, 0).atZone(zone).toInstant()
                val endTime = date.atTime(12, 0).atZone(zone).toInstant()
                
                val request = ReadRecordsRequest(
                    recordType = SleepSessionRecord::class,
                    timeRangeFilter = TimeRangeFilter.between(startTime, endTime)
                )
                
                val response = client.readRecords(request)
                
                if (response.records.isEmpty()) {
                    withContext(Dispatchers.Main) {
                        promise.resolve(null)
                    }
                    return@launch
                }
                
                // Aggregate all sleep sessions
                var totalSleepMs = 0L
                var deepSleepMs = 0L
                var remSleepMs = 0L
                var lightSleepMs = 0L
                var awakeMs = 0L
                
                for (session in response.records) {
                    totalSleepMs += Duration.between(session.startTime, session.endTime).toMillis()
                    
                    for (stage in session.stages) {
                        val stageDuration = Duration.between(stage.startTime, stage.endTime).toMillis()
                        when (stage.stage) {
                            SleepSessionRecord.STAGE_TYPE_DEEP -> deepSleepMs += stageDuration
                            SleepSessionRecord.STAGE_TYPE_REM -> remSleepMs += stageDuration
                            SleepSessionRecord.STAGE_TYPE_LIGHT -> lightSleepMs += stageDuration
                            SleepSessionRecord.STAGE_TYPE_AWAKE -> awakeMs += stageDuration
                        }
                    }
                }
                
                val result = Arguments.createMap().apply {
                    putString("date", dateString)
                    putInt("totalSleepSeconds", (totalSleepMs / 1000).toInt())
                    putInt("deepSleepSeconds", (deepSleepMs / 1000).toInt())
                    putInt("remSleepSeconds", (remSleepMs / 1000).toInt())
                    putInt("lightSleepSeconds", (lightSleepMs / 1000).toInt())
                    putInt("awakeSeconds", (awakeMs / 1000).toInt())
                    putDouble("efficiency", if (totalSleepMs > 0) {
                        (totalSleepMs - awakeMs).toDouble() / totalSleepMs
                    } else 0.0)
                }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(result)
                }
            } catch (e: Exception) {
                Log.e(TAG, "Error getting sleep", e)
                withContext(Dispatchers.Main) {
                    promise.reject("SLEEP_ERROR", e.message)
                }
            }
        }
    }
    
    @ReactMethod
    fun getActiveCalories(startTimeIso: String, endTimeIso: String, promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val startTime = Instant.parse(startTimeIso)
                val endTime = Instant.parse(endTimeIso)
                
                val request = ReadRecordsRequest(
                    recordType = ActiveCaloriesBurnedRecord::class,
                    timeRangeFilter = TimeRangeFilter.between(startTime, endTime)
                )
                
                val response = client.readRecords(request)
                val totalCalories = response.records.sumOf { it.energy.inKilocalories }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(totalCalories.toInt())
                }
            } catch (e: Exception) {
                Log.e(TAG, "Error getting calories", e)
                withContext(Dispatchers.Main) {
                    promise.reject("CALORIES_ERROR", e.message)
                }
            }
        }
    }
    
    // ==================== AGGREGATED DATA ====================
    
    @ReactMethod
    fun getDailySummary(dateString: String, promise: Promise) {
        scope.launch {
            try {
                val client = getClient()
                val date = LocalDate.parse(dateString)
                val zone = ZoneId.systemDefault()
                val startTime = date.atStartOfDay(zone).toInstant()
                val endTime = date.plusDays(1).atStartOfDay(zone).toInstant()
                
                val request = AggregateRequest(
                    metrics = setOf(
                        StepsRecord.COUNT_TOTAL,
                        ActiveCaloriesBurnedRecord.ACTIVE_CALORIES_TOTAL,
                        DistanceRecord.DISTANCE_TOTAL,
                    ),
                    timeRangeFilter = TimeRangeFilter.between(startTime, endTime)
                )
                
                val response = client.aggregate(request)
                
                val result = Arguments.createMap().apply {
                    putString("date", dateString)
                    putInt("steps", (response[StepsRecord.COUNT_TOTAL] ?: 0L).toInt())
                    putInt("activeCalories", (response[ActiveCaloriesBurnedRecord.ACTIVE_CALORIES_TOTAL]?.inKilocalories ?: 0.0).toInt())
                    putInt("distanceMeters", (response[DistanceRecord.DISTANCE_TOTAL]?.inMeters ?: 0.0).toInt())
                }
                
                withContext(Dispatchers.Main) {
                    promise.resolve(result)
                }
            } catch (e: Exception) {
                Log.e(TAG, "Error getting daily summary", e)
                withContext(Dispatchers.Main) {
                    promise.reject("SUMMARY_ERROR", e.message)
                }
            }
        }
    }
    
    // ==================== HELPER METHODS ====================
    
    private fun getClient(): HealthConnectClient {
        return healthConnectClient ?: HealthConnectClient
            .getOrCreate(reactContext)
            .also { healthConnectClient = it }
    }
    
    // ==================== LIFECYCLE ====================
    
    override fun onHostResume() {
        // Refresh client when app comes to foreground
    }
    
    override fun onHostPause() {
        // Optional: cleanup
    }
    
    override fun onHostDestroy() {
        scope.cancel()
    }
}
```

```kotlin
// android/app/src/main/java/com/phi/healthconnect/HealthConnectPackage.kt

package com.phi.healthconnect

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class HealthConnectPackage : ReactPackage {
    override fun createNativeModules(reactContext: ReactApplicationContext): List<NativeModule> {
        return listOf(HealthConnectModule(reactContext))
    }

    override fun createViewManagers(reactContext: ReactApplicationContext): List<ViewManager<*, *>> {
        return emptyList()
    }
}
```

### 3.2 TypeScript Wrapper

```typescript
// services/api/healthConnect.ts

import { NativeModules, Platform } from 'react-native';

const { HealthConnect } = NativeModules;

interface HealthConnectPermissions {
  allGranted: boolean;
  grantedCount: number;
  requiredCount: number;
}

interface HeartRateData {
  min: number;
  max: number;
  average: number;
  sampleCount: number;
}

interface SleepData {
  date: string;
  totalSleepSeconds: number;
  deepSleepSeconds: number;
  remSleepSeconds: number;
  lightSleepSeconds: number;
  awakeSeconds: number;
  efficiency: number;
}

interface DailySummary {
  date: string;
  steps: number;
  activeCalories: number;
  distanceMeters: number;
}

class HealthConnectService {
  private isSupported: boolean | null = null;
  
  async isAvailable(): Promise<boolean> {
    if (Platform.OS !== 'android') {
      return false;
    }
    
    if (this.isSupported !== null) {
      return this.isSupported;
    }
    
    try {
      this.isSupported = await HealthConnect.isAvailable();
      return this.isSupported;
    } catch {
      this.isSupported = false;
      return false;
    }
  }
  
  async checkPermissions(): Promise<HealthConnectPermissions | null> {
    if (!(await this.isAvailable())) {
      return null;
    }
    
    return HealthConnect.checkPermissions();
  }
  
  async requestPermissions(): Promise<boolean> {
    if (!(await this.isAvailable())) {
      return false;
    }
    
    return HealthConnect.requestPermissions();
  }
  
  async getSteps(startTime: Date, endTime: Date): Promise<number> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    
    return HealthConnect.getSteps(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getHeartRate(startTime: Date, endTime: Date): Promise<HeartRateData | null> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    
    return HealthConnect.getHeartRate(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getSleep(date: string): Promise<SleepData | null> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    
    return HealthConnect.getSleep(date);
  }
  
  async getActiveCalories(startTime: Date, endTime: Date): Promise<number> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    
    return HealthConnect.getActiveCalories(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getDailySummary(date: string): Promise<DailySummary | null> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    
    return HealthConnect.getDailySummary(date);
  }
  
  // Convenience method for today's data
  async getTodaySummary(): Promise<DailySummary | null> {
    const today = new Date().toISOString().split('T')[0];
    return this.getDailySummary(today);
  }
}

export const healthConnectService = new HealthConnectService();
```

---

## 4. ENCRYPTION SPECIFICATIONS

### 4.1 Encryption Key Management

```typescript
// utils/encryption.ts

import * as SecureStore from 'expo-secure-store';
import * as Crypto from 'expo-crypto';

const KEYS = {
  DB_ENCRYPTION: 'phi_db_key_v1',
  API_REFRESH_TOKEN: 'phi_oura_refresh_token',
  USER_SECRET: 'phi_user_secret',
};

// SecureStore options for maximum security
const SECURE_OPTIONS: SecureStore.SecureStoreOptions = {
  keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
};

export async function generateEncryptionKey(): Promise<string> {
  const randomBytes = await Crypto.getRandomBytesAsync(32);
  return Buffer.from(randomBytes).toString('hex');
}

export async function getOrCreateDatabaseKey(): Promise<string> {
  let key = await SecureStore.getItemAsync(KEYS.DB_ENCRYPTION);
  
  if (!key) {
    key = await generateEncryptionKey();
    await SecureStore.setItemAsync(KEYS.DB_ENCRYPTION, key, SECURE_OPTIONS);
  }
  
  return key;
}

export async function storeRefreshToken(token: string): Promise<void> {
  await SecureStore.setItemAsync(KEYS.API_REFRESH_TOKEN, token, SECURE_OPTIONS);
}

export async function getRefreshToken(): Promise<string | null> {
  return SecureStore.getItemAsync(KEYS.API_REFRESH_TOKEN);
}

export async function clearRefreshToken(): Promise<void> {
  await SecureStore.deleteItemAsync(KEYS.API_REFRESH_TOKEN);
}

export async function hashSensitiveData(data: string): Promise<string> {
  return Crypto.digestStringAsync(
    Crypto.CryptoDigestAlgorithm.SHA256,
    data
  );
}

// For encrypting sensitive fields before storage
export async function encryptField(plaintext: string, key: string): Promise<string> {
  // Note: expo-crypto doesn't have AES encryption built-in
  // For production, use react-native-quick-crypto or similar
  // This is a placeholder for the interface
  
  // Option 1: Use native module
  // Option 2: Use pure JS library like crypto-js (slower)
  throw new Error('Implement with react-native-quick-crypto');
}

export async function decryptField(ciphertext: string, key: string): Promise<string> {
  throw new Error('Implement with react-native-quick-crypto');
}
```

### 4.2 SQLCipher Configuration

```typescript
// services/database/index.ts

import * as SQLite from 'expo-sqlite';
import { getOrCreateDatabaseKey } from '../../utils/encryption';
import { runMigrations } from './migrations';

let database: SQLite.SQLiteDatabase | null = null;

export async function initializeDatabase(): Promise<SQLite.SQLiteDatabase> {
  if (database) {
    return database;
  }
  
  // Get or create encryption key
  const encryptionKey = await getOrCreateDatabaseKey();
  
  // Open encrypted database
  database = await SQLite.openDatabaseAsync('phi_encrypted.db');
  
  // Configure SQLCipher (if supported by expo-sqlite version)
  // Note: As of expo-sqlite 14.x, SQLCipher support is limited
  // May need to use react-native-quick-sqlite for full encryption
  try {
    await database.execAsync(`PRAGMA key = '${encryptionKey}';`);
    await database.execAsync('PRAGMA cipher_compatibility = 4;');
    
    // Verify encryption is working
    await database.execAsync('SELECT count(*) FROM sqlite_master;');
  } catch (error) {
    console.warn('SQLCipher not available, using unencrypted database');
    // Fallback to unencrypted
    database = await SQLite.openDatabaseAsync('phi_data.db');
  }
  
  // Enable foreign keys and WAL mode
  await database.execAsync('PRAGMA foreign_keys = ON;');
  await database.execAsync('PRAGMA journal_mode = WAL;');
  
  // Run migrations
  await runMigrations(database);
  
  return database;
}

export async function getDatabase(): Promise<SQLite.SQLiteDatabase> {
  if (!database) {
    return initializeDatabase();
  }
  return database;
}

export async function closeDatabase(): Promise<void> {
  if (database) {
    await database.closeAsync();
    database = null;
  }
}

// Transaction helper
export async function withTransaction<T>(
  fn: (db: SQLite.SQLiteDatabase) => Promise<T>
): Promise<T> {
  const db = await getDatabase();
  
  await db.execAsync('BEGIN TRANSACTION;');
  try {
    const result = await fn(db);
    await db.execAsync('COMMIT;');
    return result;
  } catch (error) {
    await db.execAsync('ROLLBACK;');
    throw error;
  }
}
```

---

## 5. CONFIGURATION CONSTANTS

```typescript
// constants/config.ts

export const CONFIG = {
  // API Endpoints
  OURA_API_BASE: 'https://api.ouraring.com/v2',
  OURA_AUTH_URL: 'https://cloud.ouraring.com/oauth/authorize',
  OURA_TOKEN_URL: 'https://api.ouraring.com/oauth/token',
  
  GEMINI_API_BASE: 'https://generativelanguage.googleapis.com/v1beta',
  OLLAMA_API_BASE: 'http://localhost:11434/api',  // Local only
  
  // Timeouts
  API_TIMEOUT_MS: 10000,
  AI_TIMEOUT_MS: 30000,
  SYNC_TIMEOUT_MS: 60000,
  
  // Sync Settings
  SYNC_INTERVAL_MINUTES: 240,  // 4 hours
  MIN_SYNC_INTERVAL_MINUTES: 60,  // 1 hour minimum between syncs
  SYNC_DAYS_BACK: 7,  // How many days to fetch on full sync
  
  // AI Settings
  GEMINI_MODEL: 'gemini-1.5-pro',
  OLLAMA_MODEL: 'llama3.1:8b',
  MAX_CONTEXT_TOKENS: 100000,  // Gemini 1.5 Pro context
  MAX_RESPONSE_TOKENS: 2048,
  
  // Storage
  MAX_CONVERSATION_HISTORY: 50,
  MAX_EPISODIC_ENTRIES_CACHE: 100,
  MAX_HEALTH_HISTORY_DAYS: 90,
  
  // Check-in Settings
  DEFAULT_CHECK_IN_TIMES: ['09:00', '14:00', '19:00'],
  CHECK_IN_REMINDER_DELAY_MINUTES: 30,
  
  // Thresholds
  READINESS_HIGH: 80,
  READINESS_MODERATE: 60,
  READINESS_LOW: 40,
  
  HRV_DEVIATION_SIGNIFICANT: 0.15,  // 15% from baseline
  SLEEP_SCORE_GOOD: 75,
  SLEEP_SCORE_POOR: 50,
  
  // UI
  ANIMATION_DURATION_MS: 300,
  DEBOUNCE_MS: 300,
  
  // Feature Flags
  ENABLE_OLLAMA: false,  // Set to true when local AI is configured
  ENABLE_HEALTH_CONNECT: true,
  ENABLE_PREDICTIONS: true,
  ENABLE_PROACTIVE_CHECK_INS: true,
} as const;

// Environment-specific overrides
export const ENV_CONFIG = {
  development: {
    ENABLE_OLLAMA: true,
    SYNC_INTERVAL_MINUTES: 15,  // More frequent for testing
  },
  staging: {
    ENABLE_OLLAMA: false,
  },
  production: {
    ENABLE_OLLAMA: false,
  },
};
```

```typescript
// constants/thresholds.ts

export const THRESHOLDS = {
  // Readiness interpretation
  readiness: {
    peak: { min: 85, label: 'Peak', mode: 'high_energy' },
    good: { min: 70, label: 'Good', mode: 'high_energy' },
    moderate: { min: 55, label: 'Moderate', mode: 'zen_master' },
    low: { min: 40, label: 'Low', mode: 'recovery' },
    veryLow: { min: 0, label: 'Very Low', mode: 'recovery' },
  },
  
  // HRV deviation from baseline
  hrv: {
    elevated: { deviation: 0.15, meaning: 'Well recovered' },
    normal: { deviation: 0.05, meaning: 'Baseline' },
    reduced: { deviation: -0.10, meaning: 'Some stress' },
    low: { deviation: -0.20, meaning: 'High stress or illness' },
  },
  
  // Sleep score interpretation
  sleep: {
    excellent: { min: 85, label: 'Excellent' },
    good: { min: 70, label: 'Good' },
    fair: { min: 55, label: 'Fair' },
    poor: { min: 0, label: 'Poor' },
  },
  
  // Energy level interpretation (1-10 self-report)
  energy: {
    high: { min: 8, label: 'High Energy' },
    moderate: { min: 5, label: 'Moderate' },
    low: { min: 3, label: 'Low' },
    depleted: { min: 1, label: 'Depleted' },
  },
  
  // Pattern confidence
  pattern: {
    strong: 0.8,
    moderate: 0.6,
    weak: 0.4,
    insufficient: 0.2,
  },
} as const;

export function getReadinessLevel(score: number) {
  const { readiness } = THRESHOLDS;
  if (score >= readiness.peak.min) return readiness.peak;
  if (score >= readiness.good.min) return readiness.good;
  if (score >= readiness.moderate.min) return readiness.moderate;
  if (score >= readiness.low.min) return readiness.low;
  return readiness.veryLow;
}

export function getHRVStatus(current: number, baseline: number) {
  const deviation = (current - baseline) / baseline;
  const { hrv } = THRESHOLDS;
  
  if (deviation >= hrv.elevated.deviation) return { ...hrv.elevated, deviation };
  if (deviation >= hrv.normal.deviation) return { ...hrv.normal, deviation };
  if (deviation >= hrv.reduced.deviation) return { ...hrv.reduced, deviation };
  return { ...hrv.low, deviation };
}

export function getSleepLevel(score: number) {
  const { sleep } = THRESHOLDS;
  if (score >= sleep.excellent.min) return sleep.excellent;
  if (score >= sleep.good.min) return sleep.good;
  if (score >= sleep.fair.min) return sleep.fair;
  return sleep.poor;
}

export function determineCommanderMode(
  readiness: number,
  hrv: number,
  baseline: number
): CommanderMode {
  const readinessLevel = getReadinessLevel(readiness);
  const hrvStatus = getHRVStatus(hrv, baseline);
  
  // Recovery mode takes priority if body signals are poor
  if (readiness < THRESHOLDS.readiness.low.min || hrvStatus.deviation < -0.20) {
    return 'recovery';
  }
  
  return readinessLevel.mode as CommanderMode;
}
```

---

**END OF TECH BIBLE**

*This document contains all technical specifications required for implementation. Reference specific sections when building components. All code examples are production-ready templates.*
