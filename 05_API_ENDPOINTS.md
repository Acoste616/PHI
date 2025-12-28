# PHI API ENDPOINTS
## Complete API Integration Documentation
**Version:** 1.0.0  
**Classification:** Integration Reference  
**For:** AI Coding Agents (Cursor), Backend Developers  

---

## 1. OURA API V2 INTEGRATION

### 1.1 Authentication

#### OAuth2 Flow (Recommended for Production)

```typescript
// services/api/oura.ts - OAuth Configuration

import * as AuthSession from 'expo-auth-session';
import * as WebBrowser from 'expo-web-browser';
import { storeRefreshToken, getRefreshToken } from '../../utils/encryption';

WebBrowser.maybeCompleteAuthSession();

const OURA_CONFIG = {
  clientId: process.env.EXPO_PUBLIC_OURA_CLIENT_ID!,
  clientSecret: process.env.EXPO_PUBLIC_OURA_CLIENT_SECRET!,  // Server-side only ideally
  redirectUri: AuthSession.makeRedirectUri({
    scheme: 'phi',
    path: 'oauth/callback',
  }),
  scopes: [
    'daily',
    'heartrate', 
    'personal',
    'session',
    'sleep',
    'workout',
  ],
};

// Discovery document
const discovery = {
  authorizationEndpoint: 'https://cloud.ouraring.com/oauth/authorize',
  tokenEndpoint: 'https://api.ouraring.com/oauth/token',
  revocationEndpoint: 'https://api.ouraring.com/oauth/revoke',
};

export function useOuraAuth() {
  const [request, response, promptAsync] = AuthSession.useAuthRequest(
    {
      clientId: OURA_CONFIG.clientId,
      scopes: OURA_CONFIG.scopes,
      redirectUri: OURA_CONFIG.redirectUri,
      responseType: AuthSession.ResponseType.Code,
    },
    discovery
  );

  const exchangeCodeForToken = async (code: string) => {
    const tokenResponse = await AuthSession.exchangeCodeAsync(
      {
        clientId: OURA_CONFIG.clientId,
        clientSecret: OURA_CONFIG.clientSecret,
        code,
        redirectUri: OURA_CONFIG.redirectUri,
      },
      discovery
    );
    
    if (tokenResponse.refreshToken) {
      await storeRefreshToken(tokenResponse.refreshToken);
    }
    
    return tokenResponse;
  };

  return { request, response, promptAsync, exchangeCodeForToken };
}
```

#### Personal Access Token (Development/Testing)

```typescript
// For development, use PAT from https://cloud.ouraring.com/personal-access-tokens

const OURA_PAT = process.env.EXPO_PUBLIC_OURA_PAT;

const ouraClient = axios.create({
  baseURL: 'https://api.ouraring.com/v2',
  headers: {
    'Authorization': `Bearer ${OURA_PAT}`,
  },
});
```

### 1.2 API Endpoints

#### Daily Readiness

```typescript
/**
 * GET /v2/usercollection/daily_readiness
 * 
 * Returns daily readiness scores and contributing factors.
 */

interface ReadinessResponse {
  data: ReadinessDay[];
  next_token?: string;
}

interface ReadinessDay {
  id: string;
  day: string;  // YYYY-MM-DD
  score: number;  // 0-100
  temperature_deviation: number;  // Celsius
  temperature_trend_deviation: number;
  contributors: {
    activity_balance: number;
    body_temperature: number;
    hrv_balance: number;
    previous_day_activity: number;
    previous_night: number;
    recovery_index: number;
    resting_heart_rate: number;
    sleep_balance: number;
  };
}

// Usage
async function getReadiness(startDate: string, endDate: string): Promise<ReadinessDay[]> {
  const response = await ouraClient.get<ReadinessResponse>(
    '/usercollection/daily_readiness',
    {
      params: {
        start_date: startDate,  // YYYY-MM-DD
        end_date: endDate,      // YYYY-MM-DD
      },
    }
  );
  return response.data.data;
}
```

#### Sleep Data

```typescript
/**
 * GET /v2/usercollection/daily_sleep
 * 
 * Returns aggregated sleep metrics per day.
 */

interface SleepResponse {
  data: SleepDay[];
  next_token?: string;
}

interface SleepDay {
  id: string;
  day: string;
  score: number;
  contributors: {
    deep_sleep: number;
    efficiency: number;
    latency: number;
    rem_sleep: number;
    restfulness: number;
    timing: number;
    total_sleep: number;
  };
}

/**
 * GET /v2/usercollection/sleep
 * 
 * Returns detailed sleep sessions with stages.
 */

interface SleepSessionResponse {
  data: SleepSession[];
  next_token?: string;
}

interface SleepSession {
  id: string;
  day: string;
  bedtime_start: string;  // ISO 8601
  bedtime_end: string;
  type: 'deleted' | 'sleep' | 'long_sleep' | 'late_nap' | 'rest';
  
  // Durations in seconds
  total_sleep_duration: number;
  awake_time: number;
  light_sleep_duration: number;
  rem_sleep_duration: number;
  deep_sleep_duration: number;
  
  // Quality metrics
  sleep_score: number;
  efficiency: number;  // 0-100
  latency: number;     // seconds to fall asleep
  restless_periods: number;
  
  // Heart metrics
  average_heart_rate: number;
  lowest_heart_rate: number;
  average_hrv: number;
  
  // Sleep stages (optional, may require specific scope)
  sleep_phase_5_min?: string;  // Encoded sleep stages
}

// Usage
async function getSleepSessions(startDate: string, endDate: string): Promise<SleepSession[]> {
  const response = await ouraClient.get<SleepSessionResponse>(
    '/usercollection/sleep',
    {
      params: {
        start_date: startDate,
        end_date: endDate,
      },
    }
  );
  return response.data.data;
}
```

#### Heart Rate

```typescript
/**
 * GET /v2/usercollection/heartrate
 * 
 * Returns 5-minute heart rate samples.
 * Note: Large date ranges can return many data points.
 */

interface HeartRateResponse {
  data: HeartRateSample[];
  next_token?: string;
}

interface HeartRateSample {
  bpm: number;
  source: 'awake' | 'rest' | 'sleep' | 'session' | 'live';
  timestamp: string;  // ISO 8601
}

// Usage - Be careful with date ranges (can be large)
async function getHeartRate(startDateTime: string, endDateTime: string): Promise<HeartRateSample[]> {
  const response = await ouraClient.get<HeartRateResponse>(
    '/usercollection/heartrate',
    {
      params: {
        start_datetime: startDateTime,  // ISO 8601
        end_datetime: endDateTime,
      },
    }
  );
  return response.data.data;
}
```

#### Daily Activity

```typescript
/**
 * GET /v2/usercollection/daily_activity
 * 
 * Returns daily activity summaries.
 */

interface ActivityResponse {
  data: ActivityDay[];
  next_token?: string;
}

interface ActivityDay {
  id: string;
  day: string;
  score: number;
  active_calories: number;
  total_calories: number;
  steps: number;
  equivalent_walking_distance: number;  // meters
  high_activity_time: number;           // seconds
  medium_activity_time: number;
  low_activity_time: number;
  sedentary_time: number;
  resting_time: number;
  inactivity_alerts: number;
  target_calories: number;
  target_meters: number;
  contributors: {
    meet_daily_targets: number;
    move_every_hour: number;
    recovery_time: number;
    stay_active: number;
    training_frequency: number;
    training_volume: number;
  };
}

async function getDailyActivity(startDate: string, endDate: string): Promise<ActivityDay[]> {
  const response = await ouraClient.get<ActivityResponse>(
    '/usercollection/daily_activity',
    {
      params: { start_date: startDate, end_date: endDate },
    }
  );
  return response.data.data;
}
```

### 1.3 Complete Oura Service Implementation

```typescript
// services/api/oura.ts

import axios, { AxiosInstance, AxiosError } from 'axios';
import { getRefreshToken, storeRefreshToken, clearRefreshToken } from '../../utils/encryption';

interface OuraTokens {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

class OuraApiService {
  private client: AxiosInstance;
  private tokens: OuraTokens | null = null;
  
  constructor() {
    this.client = axios.create({
      baseURL: 'https://api.ouraring.com/v2',
      timeout: 10000,
    });
    
    // Request interceptor for auth
    this.client.interceptors.request.use(async (config) => {
      const token = await this.getAccessToken();
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    });
    
    // Response interceptor for token refresh
    this.client.interceptors.response.use(
      (response) => response,
      async (error: AxiosError) => {
        if (error.response?.status === 401) {
          const refreshed = await this.refreshAccessToken();
          if (refreshed && error.config) {
            return this.client.request(error.config);
          }
        }
        throw error;
      }
    );
  }
  
  private async getAccessToken(): Promise<string | null> {
    if (this.tokens && Date.now() < this.tokens.expiresAt - 60000) {
      return this.tokens.accessToken;
    }
    
    // Try to refresh
    await this.refreshAccessToken();
    return this.tokens?.accessToken ?? null;
  }
  
  private async refreshAccessToken(): Promise<boolean> {
    const refreshToken = await getRefreshToken();
    if (!refreshToken) {
      return false;
    }
    
    try {
      const response = await axios.post(
        'https://api.ouraring.com/oauth/token',
        {
          grant_type: 'refresh_token',
          refresh_token: refreshToken,
          client_id: process.env.EXPO_PUBLIC_OURA_CLIENT_ID,
          client_secret: process.env.EXPO_PUBLIC_OURA_CLIENT_SECRET,
        }
      );
      
      this.tokens = {
        accessToken: response.data.access_token,
        refreshToken: response.data.refresh_token,
        expiresAt: Date.now() + (response.data.expires_in * 1000),
      };
      
      await storeRefreshToken(response.data.refresh_token);
      return true;
    } catch (error) {
      console.error('Token refresh failed:', error);
      await clearRefreshToken();
      return false;
    }
  }
  
  // ==================== PUBLIC API ====================
  
  async getReadiness(startDate: string, endDate: string): Promise<ReadinessDay[]> {
    const response = await this.client.get('/usercollection/daily_readiness', {
      params: { start_date: startDate, end_date: endDate },
    });
    return response.data.data;
  }
  
  async getSleep(startDate: string, endDate: string): Promise<SleepSession[]> {
    const response = await this.client.get('/usercollection/sleep', {
      params: { start_date: startDate, end_date: endDate },
    });
    return response.data.data;
  }
  
  async getDailySleep(startDate: string, endDate: string): Promise<SleepDay[]> {
    const response = await this.client.get('/usercollection/daily_sleep', {
      params: { start_date: startDate, end_date: endDate },
    });
    return response.data.data;
  }
  
  async getActivity(startDate: string, endDate: string): Promise<ActivityDay[]> {
    const response = await this.client.get('/usercollection/daily_activity', {
      params: { start_date: startDate, end_date: endDate },
    });
    return response.data.data;
  }
  
  async getHeartRate(startDateTime: string, endDateTime: string): Promise<HeartRateSample[]> {
    const response = await this.client.get('/usercollection/heartrate', {
      params: { start_datetime: startDateTime, end_datetime: endDateTime },
    });
    return response.data.data;
  }
  
  // Convenience methods
  
  async getTodayReadiness(): Promise<ReadinessDay | null> {
    const today = new Date().toISOString().split('T')[0];
    const data = await this.getReadiness(today, today);
    return data[0] ?? null;
  }
  
  async getLastNightSleep(): Promise<SleepSession | null> {
    const today = new Date().toISOString().split('T')[0];
    const yesterday = new Date(Date.now() - 86400000).toISOString().split('T')[0];
    const data = await this.getSleep(yesterday, today);
    // Return most recent main sleep session
    return data.find(s => s.type === 'long_sleep' || s.type === 'sleep') ?? null;
  }
  
  async getWeekData(): Promise<{
    readiness: ReadinessDay[];
    sleep: SleepSession[];
    activity: ActivityDay[];
  }> {
    const today = new Date().toISOString().split('T')[0];
    const weekAgo = new Date(Date.now() - 7 * 86400000).toISOString().split('T')[0];
    
    const [readiness, sleep, activity] = await Promise.all([
      this.getReadiness(weekAgo, today),
      this.getSleep(weekAgo, today),
      this.getActivity(weekAgo, today),
    ]);
    
    return { readiness, sleep, activity };
  }
}

export const ouraService = new OuraApiService();
```

---

## 2. HEALTH CONNECT SDK

### 2.1 TypeScript API Reference

```typescript
// services/api/healthConnect.ts - Type definitions and wrapper

import { NativeModules, Platform } from 'react-native';

const { HealthConnect } = NativeModules;

// ==================== TYPE DEFINITIONS ====================

interface HealthConnectAvailability {
  available: boolean;
  reason?: 'not_android' | 'sdk_unavailable' | 'update_required';
}

interface PermissionStatus {
  allGranted: boolean;
  granted: string[];
  denied: string[];
}

interface StepsData {
  startTime: string;
  endTime: string;
  count: number;
}

interface HeartRateData {
  timestamp: string;
  beatsPerMinute: number;
  source: string;
}

interface HeartRateSummary {
  min: number;
  max: number;
  average: number;
  sampleCount: number;
  samples?: HeartRateData[];
}

interface SleepStage {
  stage: 'awake' | 'light' | 'deep' | 'rem' | 'out_of_bed' | 'unknown';
  startTime: string;
  endTime: string;
}

interface SleepSessionData {
  date: string;
  startTime: string;
  endTime: string;
  totalSleepSeconds: number;
  deepSleepSeconds: number;
  remSleepSeconds: number;
  lightSleepSeconds: number;
  awakeSeconds: number;
  efficiency: number;
  stages?: SleepStage[];
}

interface ActivitySummary {
  date: string;
  steps: number;
  activeCalories: number;
  totalCalories: number;
  distanceMeters: number;
}

// ==================== SERVICE IMPLEMENTATION ====================

class HealthConnectService {
  private cachedAvailability: HealthConnectAvailability | null = null;
  
  async checkAvailability(): Promise<HealthConnectAvailability> {
    if (this.cachedAvailability) {
      return this.cachedAvailability;
    }
    
    if (Platform.OS !== 'android') {
      this.cachedAvailability = { available: false, reason: 'not_android' };
      return this.cachedAvailability;
    }
    
    try {
      const available = await HealthConnect.isAvailable();
      this.cachedAvailability = { available };
      return this.cachedAvailability;
    } catch (error) {
      this.cachedAvailability = { available: false, reason: 'sdk_unavailable' };
      return this.cachedAvailability;
    }
  }
  
  async isAvailable(): Promise<boolean> {
    const status = await this.checkAvailability();
    return status.available;
  }
  
  async checkPermissions(): Promise<PermissionStatus> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect not available');
    }
    return HealthConnect.checkPermissions();
  }
  
  async requestPermissions(): Promise<boolean> {
    if (!(await this.isAvailable())) {
      return false;
    }
    return HealthConnect.requestPermissions();
  }
  
  // ==================== DATA METHODS ====================
  
  async getSteps(startTime: Date, endTime: Date): Promise<number> {
    await this.ensureAvailable();
    return HealthConnect.getSteps(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getHeartRate(
    startTime: Date, 
    endTime: Date
  ): Promise<HeartRateSummary | null> {
    await this.ensureAvailable();
    return HealthConnect.getHeartRate(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getSleep(date: string): Promise<SleepSessionData | null> {
    await this.ensureAvailable();
    return HealthConnect.getSleep(date);
  }
  
  async getActiveCalories(startTime: Date, endTime: Date): Promise<number> {
    await this.ensureAvailable();
    return HealthConnect.getActiveCalories(
      startTime.toISOString(),
      endTime.toISOString()
    );
  }
  
  async getDailySummary(date: string): Promise<ActivitySummary | null> {
    await this.ensureAvailable();
    return HealthConnect.getDailySummary(date);
  }
  
  // ==================== CONVENIENCE METHODS ====================
  
  async getTodaySteps(): Promise<number> {
    const now = new Date();
    const startOfDay = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    return this.getSteps(startOfDay, now);
  }
  
  async getTodaySummary(): Promise<ActivitySummary | null> {
    const today = new Date().toISOString().split('T')[0];
    return this.getDailySummary(today);
  }
  
  async getLastNightSleep(): Promise<SleepSessionData | null> {
    const today = new Date().toISOString().split('T')[0];
    return this.getSleep(today);
  }
  
  // ==================== PRIVATE HELPERS ====================
  
  private async ensureAvailable(): Promise<void> {
    if (!(await this.isAvailable())) {
      throw new Error('Health Connect is not available on this device');
    }
  }
}

export const healthConnectService = new HealthConnectService();
```

---

## 3. GEMINI 1.5 PRO API

### 3.1 Configuration

```typescript
// services/api/gemini.ts

import axios, { AxiosInstance } from 'axios';

const GEMINI_CONFIG = {
  apiKey: process.env.EXPO_PUBLIC_GEMINI_API_KEY!,
  model: 'gemini-1.5-pro',
  baseUrl: 'https://generativelanguage.googleapis.com/v1beta',
  
  // Generation config
  temperature: 0.7,
  topP: 0.95,
  topK: 40,
  maxOutputTokens: 2048,
  
  // Safety settings
  safetySettings: [
    { category: 'HARM_CATEGORY_HARASSMENT', threshold: 'BLOCK_MEDIUM_AND_ABOVE' },
    { category: 'HARM_CATEGORY_HATE_SPEECH', threshold: 'BLOCK_MEDIUM_AND_ABOVE' },
    { category: 'HARM_CATEGORY_SEXUALLY_EXPLICIT', threshold: 'BLOCK_MEDIUM_AND_ABOVE' },
    { category: 'HARM_CATEGORY_DANGEROUS_CONTENT', threshold: 'BLOCK_MEDIUM_AND_ABOVE' },
  ],
};
```

### 3.2 Request/Response Types

```typescript
// types/gemini.ts

interface GeminiMessage {
  role: 'user' | 'model';
  parts: GeminiPart[];
}

interface GeminiPart {
  text?: string;
  inlineData?: {
    mimeType: string;
    data: string;  // base64
  };
}

interface GeminiRequest {
  contents: GeminiMessage[];
  systemInstruction?: {
    parts: { text: string }[];
  };
  generationConfig?: {
    temperature?: number;
    topP?: number;
    topK?: number;
    maxOutputTokens?: number;
    responseMimeType?: string;  // 'application/json' for JSON mode
  };
  safetySettings?: Array<{
    category: string;
    threshold: string;
  }>;
}

interface GeminiResponse {
  candidates: Array<{
    content: {
      parts: Array<{ text: string }>;
      role: string;
    };
    finishReason: string;
    safetyRatings: Array<{
      category: string;
      probability: string;
    }>;
  }>;
  usageMetadata: {
    promptTokenCount: number;
    candidatesTokenCount: number;
    totalTokenCount: number;
  };
}
```

### 3.3 Complete Gemini Service

```typescript
// services/api/gemini.ts

import axios from 'axios';
import { SYSTEM_PROMPTS } from '../ai/prompts';

class GeminiService {
  private apiKey: string;
  private model: string;
  private baseUrl: string;
  
  constructor() {
    this.apiKey = process.env.EXPO_PUBLIC_GEMINI_API_KEY!;
    this.model = 'gemini-1.5-pro';
    this.baseUrl = 'https://generativelanguage.googleapis.com/v1beta';
  }
  
  private getEndpoint(): string {
    return `${this.baseUrl}/models/${this.model}:generateContent?key=${this.apiKey}`;
  }
  
  async generateContent(
    userMessage: string,
    systemPrompt: string,
    conversationHistory: GeminiMessage[] = [],
    options: {
      temperature?: number;
      maxTokens?: number;
      jsonMode?: boolean;
    } = {}
  ): Promise<{ text: string; tokens: { input: number; output: number } }> {
    const request: GeminiRequest = {
      systemInstruction: {
        parts: [{ text: systemPrompt }],
      },
      contents: [
        ...conversationHistory,
        {
          role: 'user',
          parts: [{ text: userMessage }],
        },
      ],
      generationConfig: {
        temperature: options.temperature ?? 0.7,
        topP: 0.95,
        topK: 40,
        maxOutputTokens: options.maxTokens ?? 2048,
        ...(options.jsonMode && { responseMimeType: 'application/json' }),
      },
      safetySettings: GEMINI_CONFIG.safetySettings,
    };
    
    const startTime = Date.now();
    
    try {
      const response = await axios.post<GeminiResponse>(
        this.getEndpoint(),
        request,
        { timeout: 30000 }
      );
      
      const result = response.data;
      
      if (!result.candidates || result.candidates.length === 0) {
        throw new Error('No response generated');
      }
      
      const candidate = result.candidates[0];
      
      if (candidate.finishReason === 'SAFETY') {
        throw new Error('Response blocked by safety filters');
      }
      
      const text = candidate.content.parts
        .map(p => p.text)
        .filter(Boolean)
        .join('');
      
      return {
        text,
        tokens: {
          input: result.usageMetadata.promptTokenCount,
          output: result.usageMetadata.candidatesTokenCount,
        },
      };
    } catch (error) {
      console.error('Gemini API error:', error);
      throw error;
    }
  }
  
  // ==================== PHI-SPECIFIC METHODS ====================
  
  async generateMorningBriefing(context: {
    readiness: number;
    sleepScore: number;
    sleepDuration: number;
    hrvStatus: string;
    baselineReadiness: number;
    yesterdayContext: string[];
    commanderMode: string;
  }): Promise<string> {
    const systemPrompt = SYSTEM_PROMPTS.MORNING_COMMANDER
      .replace('{{commander_mode}}', context.commanderMode);
    
    const userMessage = `
Generate my morning briefing.

TODAY'S DATA:
- Readiness Score: ${context.readiness}/100 (baseline: ${context.baselineReadiness})
- Sleep Score: ${context.sleepScore}/100
- Sleep Duration: ${Math.round(context.sleepDuration / 3600 * 10) / 10} hours
- HRV Status: ${context.hrvStatus}

YESTERDAY'S ACTIVITIES:
${context.yesterdayContext.join('\n')}

Generate a concise, actionable morning briefing.
`;
    
    const result = await this.generateContent(userMessage, systemPrompt, [], {
      temperature: 0.7,
      maxTokens: 500,
    });
    
    return result.text;
  }
  
  async generateCheckInPrompt(context: {
    timeOfDay: string;
    morningEnergy: number;
    currentReadiness: number;
    recentActivities: string[];
    commanderMode: string;
  }): Promise<string> {
    const systemPrompt = SYSTEM_PROMPTS.CHECK_IN_PROMPT
      .replace('{{readiness}}', String(context.currentReadiness))
      .replace('{{morning_energy}}', String(context.morningEnergy))
      .replace('{{current_time}}', context.timeOfDay)
      .replace('{{activities}}', context.recentActivities.join(', '));
    
    const result = await this.generateContent(
      'Generate a check-in prompt.',
      systemPrompt,
      [],
      { temperature: 0.8, maxTokens: 100 }
    );
    
    return result.text;
  }
  
  async analyzePatterns(
    healthHistory: any[],
    contextHistory: any[],
    days: number = 14
  ): Promise<{ patterns: any[]; predictions: any[] }> {
    const systemPrompt = SYSTEM_PROMPTS.PATTERN_ANALYSIS
      .replace('{{days}}', String(days));
    
    const userMessage = `
Analyze patterns in this data:

HEALTH DATA (last ${days} days):
${JSON.stringify(healthHistory, null, 2)}

CONTEXT DATA (last ${days} days):
${JSON.stringify(contextHistory, null, 2)}

Return analysis as JSON.
`;
    
    const result = await this.generateContent(
      userMessage,
      systemPrompt,
      [],
      { jsonMode: true, temperature: 0.3, maxTokens: 2000 }
    );
    
    return JSON.parse(result.text);
  }
  
  async chat(
    userMessage: string,
    conversationHistory: GeminiMessage[],
    currentState: {
      readiness: number;
      energy: number;
      commanderMode: string;
    }
  ): Promise<{ response: string; tokens: { input: number; output: number } }> {
    const systemPrompt = `
You are PHI, a Personal Health Intelligence assistant.

CURRENT USER STATE:
- Readiness: ${currentState.readiness}/100
- Reported Energy: ${currentState.energy}/10
- Commander Mode: ${currentState.commanderMode}

COMMUNICATION STYLE:
${currentState.commanderMode === 'high_energy' ? 'Be direct, challenging, action-oriented.' : ''}
${currentState.commanderMode === 'zen_master' ? 'Be calm, supportive, measured.' : ''}
${currentState.commanderMode === 'recovery' ? 'Be gentle, protective, rest-focused.' : ''}

Be concise. Be actionable. Be personal.
`;
    
    const result = await this.generateContent(
      userMessage,
      systemPrompt,
      conversationHistory,
      { temperature: 0.7, maxTokens: 500 }
    );
    
    return {
      response: result.text,
      tokens: result.tokens,
    };
  }
}

export const geminiService = new GeminiService();
```

---

## 4. OLLAMA LOCAL API (Optional)

### 4.1 Configuration

```typescript
// services/api/ollama.ts

const OLLAMA_CONFIG = {
  baseUrl: 'http://localhost:11434',
  model: 'llama3.1:8b',
  timeout: 60000,  // Local processing can be slow
};
```

### 4.2 API Endpoints

```typescript
/**
 * POST /api/generate
 * 
 * Generate completion for a prompt.
 */

interface OllamaGenerateRequest {
  model: string;
  prompt: string;
  system?: string;
  stream?: boolean;
  options?: {
    temperature?: number;
    top_p?: number;
    top_k?: number;
    num_predict?: number;
  };
}

interface OllamaGenerateResponse {
  model: string;
  created_at: string;
  response: string;
  done: boolean;
  context?: number[];
  total_duration?: number;
  load_duration?: number;
  prompt_eval_count?: number;
  prompt_eval_duration?: number;
  eval_count?: number;
  eval_duration?: number;
}

/**
 * POST /api/chat
 * 
 * Chat completion with message history.
 */

interface OllamaChatRequest {
  model: string;
  messages: Array<{
    role: 'system' | 'user' | 'assistant';
    content: string;
  }>;
  stream?: boolean;
  options?: {
    temperature?: number;
    top_p?: number;
    num_predict?: number;
  };
}

interface OllamaChatResponse {
  model: string;
  created_at: string;
  message: {
    role: string;
    content: string;
  };
  done: boolean;
  total_duration?: number;
  eval_count?: number;
}
```

### 4.3 Ollama Service Implementation

```typescript
// services/api/ollama.ts

import axios from 'axios';

class OllamaService {
  private baseUrl: string;
  private model: string;
  private available: boolean | null = null;
  
  constructor() {
    this.baseUrl = 'http://localhost:11434';
    this.model = 'llama3.1:8b';
  }
  
  async isAvailable(): Promise<boolean> {
    if (this.available !== null) {
      return this.available;
    }
    
    try {
      await axios.get(`${this.baseUrl}/api/tags`, { timeout: 2000 });
      this.available = true;
      return true;
    } catch {
      this.available = false;
      return false;
    }
  }
  
  async generate(
    prompt: string,
    systemPrompt?: string,
    options: { temperature?: number; maxTokens?: number } = {}
  ): Promise<string> {
    if (!(await this.isAvailable())) {
      throw new Error('Ollama not available');
    }
    
    const request: OllamaGenerateRequest = {
      model: this.model,
      prompt,
      system: systemPrompt,
      stream: false,
      options: {
        temperature: options.temperature ?? 0.7,
        num_predict: options.maxTokens ?? 1000,
      },
    };
    
    const response = await axios.post<OllamaGenerateResponse>(
      `${this.baseUrl}/api/generate`,
      request,
      { timeout: 60000 }
    );
    
    return response.data.response;
  }
  
  async chat(
    messages: Array<{ role: 'system' | 'user' | 'assistant'; content: string }>,
    options: { temperature?: number; maxTokens?: number } = {}
  ): Promise<string> {
    if (!(await this.isAvailable())) {
      throw new Error('Ollama not available');
    }
    
    const request: OllamaChatRequest = {
      model: this.model,
      messages,
      stream: false,
      options: {
        temperature: options.temperature ?? 0.7,
        num_predict: options.maxTokens ?? 1000,
      },
    };
    
    const response = await axios.post<OllamaChatResponse>(
      `${this.baseUrl}/api/chat`,
      request,
      { timeout: 60000 }
    );
    
    return response.data.message.content;
  }
  
  // PHI-specific: Process raw biometrics locally
  async analyzeRawBiometrics(data: {
    hrvReadings: number[];
    heartRateReadings: number[];
    sleepStages: string[];
  }): Promise<{
    hrvTrend: 'increasing' | 'stable' | 'decreasing';
    anomalies: string[];
    summary: string;
  }> {
    const prompt = `
Analyze this biometric data and provide insights:

HRV Readings (ms): ${data.hrvReadings.join(', ')}
Heart Rate (bpm): ${data.heartRateReadings.join(', ')}
Sleep Stages: ${data.sleepStages.join(', ')}

Return JSON with:
- hrvTrend: "increasing", "stable", or "decreasing"
- anomalies: array of concerning patterns
- summary: one sentence summary

JSON only, no markdown:
`;
    
    const response = await this.generate(prompt, undefined, { temperature: 0.1 });
    
    try {
      return JSON.parse(response);
    } catch {
      // Fallback if JSON parsing fails
      return {
        hrvTrend: 'stable',
        anomalies: [],
        summary: 'Unable to analyze data.',
      };
    }
  }
}

export const ollamaService = new OllamaService();
```

---

## 5. ERROR HANDLING & RETRY LOGIC

```typescript
// utils/apiHelpers.ts

import axios, { AxiosError } from 'axios';

interface RetryConfig {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  retryableStatuses: number[];
}

const DEFAULT_RETRY_CONFIG: RetryConfig = {
  maxRetries: 3,
  baseDelay: 1000,
  maxDelay: 10000,
  retryableStatuses: [408, 429, 500, 502, 503, 504],
};

export async function withRetry<T>(
  fn: () => Promise<T>,
  config: Partial<RetryConfig> = {}
): Promise<T> {
  const { maxRetries, baseDelay, maxDelay, retryableStatuses } = {
    ...DEFAULT_RETRY_CONFIG,
    ...config,
  };
  
  let lastError: Error | null = null;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      // Check if retryable
      if (axios.isAxiosError(error)) {
        const status = error.response?.status;
        
        if (status && !retryableStatuses.includes(status)) {
          throw error;  // Non-retryable error
        }
        
        // Rate limited - use Retry-After header if available
        if (status === 429) {
          const retryAfter = error.response?.headers['retry-after'];
          if (retryAfter) {
            await sleep(parseInt(retryAfter, 10) * 1000);
            continue;
          }
        }
      }
      
      // Exponential backoff
      if (attempt < maxRetries) {
        const delay = Math.min(baseDelay * Math.pow(2, attempt), maxDelay);
        const jitter = delay * 0.1 * Math.random();
        await sleep(delay + jitter);
      }
    }
  }
  
  throw lastError;
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// API-specific error types
export class OuraApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public retryable: boolean
  ) {
    super(message);
    this.name = 'OuraApiError';
  }
}

export class GeminiApiError extends Error {
  constructor(
    message: string,
    public reason: 'safety' | 'rate_limit' | 'invalid_request' | 'server_error'
  ) {
    super(message);
    this.name = 'GeminiApiError';
  }
}

export class HealthConnectError extends Error {
  constructor(
    message: string,
    public reason: 'not_available' | 'permission_denied' | 'data_error'
  ) {
    super(message);
    this.name = 'HealthConnectError';
  }
}
```

---

## 6. API RATE LIMITS & QUOTAS

| API | Limit | PHI Usage Strategy |
|-----|-------|-------------------|
| **Oura** | 50,000 calls/month | Cache aggressively, sync every 4h max |
| **Gemini 1.5 Pro** | 1M tokens/day (free) | Use for complex tasks only |
| **Health Connect** | No external limits | Limited by device battery |
| **Ollama** | Local only | Unlimited, but CPU-bound |

### Quota Management

```typescript
// utils/quotaManager.ts

interface QuotaState {
  oura: { calls: number; resetAt: number };
  gemini: { tokens: number; resetAt: number };
}

class QuotaManager {
  private state: QuotaState = {
    oura: { calls: 0, resetAt: this.getMonthEnd() },
    gemini: { tokens: 0, resetAt: this.getDayEnd() },
  };
  
  private getMonthEnd(): number {
    const now = new Date();
    return new Date(now.getFullYear(), now.getMonth() + 1, 1).getTime();
  }
  
  private getDayEnd(): number {
    const now = new Date();
    return new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1).getTime();
  }
  
  canCallOura(): boolean {
    this.resetIfNeeded();
    return this.state.oura.calls < 45000;  // Leave buffer
  }
  
  canCallGemini(estimatedTokens: number): boolean {
    this.resetIfNeeded();
    return this.state.gemini.tokens + estimatedTokens < 900000;
  }
  
  recordOuraCall(): void {
    this.state.oura.calls++;
  }
  
  recordGeminiTokens(tokens: number): void {
    this.state.gemini.tokens += tokens;
  }
  
  private resetIfNeeded(): void {
    const now = Date.now();
    if (now > this.state.oura.resetAt) {
      this.state.oura = { calls: 0, resetAt: this.getMonthEnd() };
    }
    if (now > this.state.gemini.resetAt) {
      this.state.gemini = { tokens: 0, resetAt: this.getDayEnd() };
    }
  }
}

export const quotaManager = new QuotaManager();
```

---

**END OF API ENDPOINTS**

*This document defines all external API integrations. Use these specifications when implementing data sync and AI features. All error handling patterns should be followed.*
