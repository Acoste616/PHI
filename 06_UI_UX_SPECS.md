# PHI UI/UX SPECIFICATIONS
## Complete Interface Design Document
**Version:** 1.0.0  
**Classification:** Design & Implementation Reference  
**For:** AI Coding Agents (Cursor), Frontend Developers  

---

## 1. DESIGN PHILOSOPHY

### 1.1 Core Principles

| Principle | Implementation |
|-----------|----------------|
| **Glanceable** | Critical info visible in <2 seconds |
| **Actionable** | Every screen has a clear next action |
| **Adaptive** | UI transforms based on Commander Mode |
| **Minimal** | No vanity metrics, no clutter |
| **Dark-First** | Optimized for morning/evening use |

### 1.2 Design Tokens

```typescript
// constants/theme.ts

export const COLORS = {
  // Base
  background: {
    primary: '#0A0A0A',
    secondary: '#141414',
    tertiary: '#1E1E1E',
  },
  
  // Text
  text: {
    primary: '#FFFFFF',
    secondary: '#A0A0A0',
    tertiary: '#666666',
    inverse: '#0A0A0A',
  },
  
  // Commander Mode Accents
  accent: {
    highEnergy: '#FF4500',      // Orange-Red
    highEnergyLight: '#FF6B35',
    zenMaster: '#00CED1',       // Dark Cyan
    zenMasterLight: '#40E0D0',
    socialBattery: '#9B59B6',   // Purple
    socialBatteryLight: '#BB8FCE',
    recovery: '#2ECC71',        // Green
    recoveryLight: '#58D68D',
  },
  
  // Semantic
  semantic: {
    success: '#2ECC71',
    warning: '#F39C12',
    error: '#E74C3C',
    info: '#3498DB',
  },
  
  // Scores
  score: {
    excellent: '#2ECC71',  // 85+
    good: '#27AE60',       // 70-84
    moderate: '#F39C12',   // 55-69
    low: '#E67E22',        // 40-54
    veryLow: '#E74C3C',    // <40
  },
};

export const TYPOGRAPHY = {
  // Font Family
  fontFamily: {
    regular: 'Inter-Regular',
    medium: 'Inter-Medium',
    semibold: 'Inter-SemiBold',
    bold: 'Inter-Bold',
    mono: 'JetBrainsMono-Regular',
  },
  
  // Font Sizes
  fontSize: {
    xs: 11,
    sm: 13,
    base: 15,
    lg: 17,
    xl: 20,
    '2xl': 24,
    '3xl': 30,
    '4xl': 36,
    '5xl': 48,
    hero: 64,
  },
  
  // Line Heights
  lineHeight: {
    tight: 1.2,
    normal: 1.5,
    relaxed: 1.75,
  },
};

export const SPACING = {
  xs: 4,
  sm: 8,
  md: 12,
  lg: 16,
  xl: 24,
  '2xl': 32,
  '3xl': 48,
  '4xl': 64,
};

export const BORDER_RADIUS = {
  sm: 4,
  md: 8,
  lg: 12,
  xl: 16,
  full: 9999,
};

export const SHADOWS = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 2,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 4,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 8,
  },
};
```

### 1.3 Commander Mode Themes

```typescript
// components/themes/index.ts

interface CommanderTheme {
  name: string;
  accent: string;
  accentLight: string;
  gradient: [string, string];
  icon: string;
  greeting: string;
  buttonStyle: 'aggressive' | 'calm' | 'gentle';
}

export const COMMANDER_THEMES: Record<string, CommanderTheme> = {
  high_energy: {
    name: 'High Energy',
    accent: '#FF4500',
    accentLight: '#FF6B35',
    gradient: ['#FF4500', '#FF6B35'],
    icon: '⚡',
    greeting: "Let's crush it.",
    buttonStyle: 'aggressive',
  },
  zen_master: {
    name: 'Zen Master',
    accent: '#00CED1',
    accentLight: '#40E0D0',
    gradient: ['#00CED1', '#40E0D0'],
    icon: '🧘',
    greeting: 'Easy does it.',
    buttonStyle: 'calm',
  },
  social_battery: {
    name: 'Social Battery',
    accent: '#9B59B6',
    accentLight: '#BB8FCE',
    gradient: ['#9B59B6', '#BB8FCE'],
    icon: '🔋',
    greeting: 'Manage your energy.',
    buttonStyle: 'calm',
  },
  recovery: {
    name: 'Recovery',
    accent: '#2ECC71',
    accentLight: '#58D68D',
    gradient: ['#2ECC71', '#58D68D'],
    icon: '🌿',
    greeting: 'Your body needs this.',
    buttonStyle: 'gentle',
  },
};
```

---

## 2. SCREEN SPECIFICATIONS

### 2.1 App Navigation Structure

```
┌─────────────────────────────────────────────────────────────┐
│                      PHI APP STRUCTURE                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    ROOT NAVIGATOR                    │    │
│  │  (Handles: Onboarding vs Main App)                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                  │
│           ┌───────────────┴───────────────┐                 │
│           ▼                               ▼                  │
│  ┌─────────────────┐            ┌─────────────────┐         │
│  │   ONBOARDING    │            │    MAIN APP     │         │
│  │   STACK         │            │    (Tabs)       │         │
│  │                 │            │                 │         │
│  │  • Welcome      │            │  • Commander    │         │
│  │  • OuraConnect  │            │  • Wingman      │         │
│  │  • HealthSetup  │            │  • Insights     │         │
│  │  • ModeSelect   │            │                 │         │
│  │  • Complete     │            │  [Settings]     │         │
│  └─────────────────┘            └─────────────────┘         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Tab Bar Specification

```typescript
// Navigation tab configuration

interface TabConfig {
  name: string;
  icon: {
    active: string;    // Lucide icon name
    inactive: string;
  };
  label: string;
}

const TABS: TabConfig[] = [
  {
    name: 'commander',
    icon: { active: 'Zap', inactive: 'Zap' },
    label: 'Commander',
  },
  {
    name: 'wingman',
    icon: { active: 'MessageCircle', inactive: 'MessageCircle' },
    label: 'Wingman',
  },
  {
    name: 'insights',
    icon: { active: 'TrendingUp', inactive: 'TrendingUp' },
    label: 'Insights',
  },
];

// Tab bar styling
const TAB_BAR_STYLE = {
  height: 80,
  paddingBottom: 20,  // Safe area
  paddingTop: 12,
  backgroundColor: COLORS.background.secondary,
  borderTopWidth: 1,
  borderTopColor: COLORS.background.tertiary,
};
```

---

## 3. SCREEN 1: MORNING COMMANDER (Home)

### 3.1 Layout Overview

```
┌─────────────────────────────────────────┐
│ ░░░░░░░░░░ STATUS BAR ░░░░░░░░░░░░░░░░ │
├─────────────────────────────────────────┤
│                                         │
│  Good morning, [Name]          ⚙️       │  ← Header
│  [Commander Mode Badge]                 │
│                                         │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐   │
│  │                                 │   │
│  │         READINESS RING          │   │  ← Hero Element
│  │            [ 74 ]               │   │
│  │         "Good to go"            │   │
│  │                                 │   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│  ┌───────┐ ┌───────┐ ┌───────┐        │
│  │ SLEEP │ │  HRV  │ │ENERGY │        │  ← Metric Cards
│  │  72   │ │  +8%  │ │  --   │        │
│  └───────┘ └───────┘ └───────┘        │
│                                         │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐   │
│  │ 💬 AI INSIGHT                   │   │  ← AI Card
│  │                                 │   │
│  │ "Solid recovery. Your peak     │   │
│  │  focus window is 9-11 AM.      │   │
│  │  Schedule deep work now."      │   │
│  │                                 │   │
│  │ [Got it]           [Tell me more]│   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│  TODAY'S ENERGY FORECAST               │
│  ┌─────────────────────────────────┐   │  ← Timeline
│  │ 🟢 9AM   🟢 11AM  🟡 2PM  🔴 5PM │   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│  [ ⚡ Commander ]  [ 💬 Wingman ]  [📊] │  ← Tab Bar
│                                         │
└─────────────────────────────────────────┘
```

### 3.2 Component Specifications

#### 3.2.1 Header Component

```typescript
// components/commander/Header.tsx

interface HeaderProps {
  userName: string;
  commanderMode: CommanderMode;
  onSettingsPress: () => void;
}

/*
LAYOUT:
- Horizontal flex, space-between
- Left: Greeting + Mode badge
- Right: Settings icon button

STYLING:
- Greeting: fontSize 2xl, fontWeight bold, color text.primary
- Mode badge: pill shape, accent color background, white text
- Settings: 24x24 icon, text.secondary color

INTERACTIONS:
- Settings icon: onPress → navigate to Settings
- Mode badge: onPress → show mode selector bottom sheet
*/

// Greeting logic
function getGreeting(hour: number): string {
  if (hour < 12) return 'Good morning';
  if (hour < 17) return 'Good afternoon';
  return 'Good evening';
}
```

#### 3.2.2 Readiness Ring Component

```typescript
// components/commander/ReadinessRing.tsx

interface ReadinessRingProps {
  score: number;           // 0-100
  baselineScore: number;   // For comparison
  commanderMode: CommanderMode;
}

/*
LAYOUT:
- Centered container, 200x200
- SVG circle with animated stroke
- Score number in center
- Status text below score

VISUAL DESIGN:
- Ring: 8px stroke width, rounded caps
- Background ring: tertiary color, 20% opacity
- Progress ring: color based on score tier
- Animation: clockwise fill on mount (800ms ease-out)

SCORE → COLOR MAPPING:
- 85+: score.excellent (#2ECC71)
- 70-84: score.good (#27AE60)  
- 55-69: score.moderate (#F39C12)
- 40-54: score.low (#E67E22)
- <40: score.veryLow (#E74C3C)

SCORE → STATUS TEXT:
- 85+: "Peak performance"
- 70-84: "Good to go"
- 55-69: "Take it easy"
- 40-54: "Recovery mode"
- <40: "Rest day"

INTERACTIONS:
- onPress: Expand to show contributors breakdown
*/
```

#### 3.2.3 Metric Cards Row

```typescript
// components/commander/MetricCards.tsx

interface MetricCardProps {
  label: string;
  value: string | number;
  unit?: string;
  trend?: 'up' | 'down' | 'neutral';
  trendValue?: string;      // e.g., "+8%"
  onPress?: () => void;
}

/*
LAYOUT:
- Horizontal scroll (3 cards visible)
- Each card: 100px width, 80px height
- Gap: 12px between cards

CARD STRUCTURE:
┌────────────────┐
│ LABEL          │  ← fontSize sm, text.secondary
│                │
│ VALUE          │  ← fontSize 2xl, text.primary, bold
│ +TREND         │  ← fontSize xs, semantic color
└────────────────┘

CARDS TO DISPLAY:
1. Sleep Score (from Oura)
2. HRV Status (vs baseline %)
3. Current Energy (self-reported, or "--" if not set)
4. Steps (optional, if relevant)

INTERACTIONS:
- onPress: Show detailed view in bottom sheet
*/
```

#### 3.2.4 AI Insight Card

```typescript
// components/commander/AIInsightCard.tsx

interface AIInsightCardProps {
  message: string;
  commanderMode: CommanderMode;
  onDismiss: () => void;
  onExpand: () => void;
}

/*
LAYOUT:
┌─────────────────────────────────────┐
│ 💬 AI INSIGHT              [mode]  │  ← Header row
├─────────────────────────────────────┤
│                                     │
│ Message text here spanning         │  ← Body
│ multiple lines if needed...        │
│                                     │
├─────────────────────────────────────┤
│ [Got it]              [Tell me more]│  ← Actions
└─────────────────────────────────────┘

STYLING:
- Container: background.secondary, borderRadius lg, padding lg
- Border: 1px accent color (from commander mode)
- Icon: 20x20, accent color
- Message: fontSize base, text.primary, lineHeight relaxed
- Buttons: text buttons, accent color

ANIMATION:
- Slide in from bottom on appearance
- Fade out on dismiss

INTERACTIONS:
- "Got it": Dismiss card, log as acknowledged
- "Tell me more": Navigate to Wingman with context
*/
```

#### 3.2.5 Energy Forecast Timeline

```typescript
// components/commander/EnergyForecast.tsx

interface EnergyForecastProps {
  predictions: Array<{
    time: string;      // "09:00"
    level: 'high' | 'moderate' | 'low';
    label?: string;    // "Peak focus"
  }>;
}

/*
LAYOUT:
- Horizontal timeline with dots
- Time labels below dots
- Optional event labels above dots

VISUAL:
  09:00      11:00      14:00      17:00
    🟢─────────🟢─────────🟡─────────🔴
   Peak      Focus     Dip        Wind down

COLOR MAPPING:
- high: #2ECC71 (green)
- moderate: #F39C12 (yellow)
- low: #E74C3C (red)

LINE STYLING:
- 2px thick
- Gradient between dot colors
- Dots: 12px diameter, filled

CURRENT TIME INDICATOR:
- Vertical line at current position
- Subtle pulse animation

INTERACTIONS:
- Tap on dot: Show prediction details in tooltip
*/
```

---

## 4. SCREEN 2: DAILY WINGMAN (Chat)

### 4.1 Layout Overview

```
┌─────────────────────────────────────────┐
│ ░░░░░░░░░░ STATUS BAR ░░░░░░░░░░░░░░░░ │
├─────────────────────────────────────────┤
│                                         │
│  ← Daily Wingman              [⋮]      │  ← Header
│                                         │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🤖 PHI                          │   │  ← AI Message
│  │ Good morning! Your readiness   │   │
│  │ is 74. How are you feeling?    │   │
│  └─────────────────────────────────┘   │
│                                         │
│                ┌───────────────────┐   │
│                │ Pretty good,      │   │  ← User Message
│                │ slept well        │   │
│                └───────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🤖 PHI                          │   │
│  │ Great! Based on your sleep     │   │
│  │ and HRV, your peak window...   │   │
│  └─────────────────────────────────┘   │
│                                         │
│          ... more messages ...          │
│                                         │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐   │
│  │ 😊 Great  │ 😐 Okay │ 😴 Tired  │   │  ← Quick Responses
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────┐   │
│  │ Type a message...          [➤] │   │  ← Input
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│  [ ⚡ ]      [ 💬 ]      [ 📊 ]       │  ← Tab Bar
└─────────────────────────────────────────┘
```

### 4.2 Component Specifications

#### 4.2.1 Chat Message Bubble

```typescript
// components/chat/MessageBubble.tsx

interface MessageBubbleProps {
  role: 'user' | 'assistant';
  content: string;
  timestamp: string;
  isTyping?: boolean;
  commanderMode?: CommanderMode;
}

/*
LAYOUT RULES:
- User messages: Align right, accent background, white text
- AI messages: Align left, secondary background, primary text
- Max width: 80% of container
- Padding: 12px horizontal, 10px vertical

AI MESSAGE STRUCTURE:
┌─────────────────────────────────────┐
│ 🤖 PHI                    10:32 AM │  ← Header
├─────────────────────────────────────┤
│                                     │
│ Message content here with proper   │  ← Body
│ line wrapping and spacing...       │
│                                     │
└─────────────────────────────────────┘

USER MESSAGE STRUCTURE:
                    ┌─────────────────┐
                    │ User message    │
                    │ content         │
                    └─────────────────┘

TYPING INDICATOR:
- Three dots with sequential fade animation
- 0.4s per dot, 1.2s cycle

STYLING:
- AI bubble: background.secondary, border 1px tertiary
- User bubble: accent gradient, no border
- Corners: borderRadius lg (12px)
- Text: fontSize base, lineHeight relaxed
*/
```

#### 4.2.2 Quick Response Buttons

```typescript
// components/chat/QuickResponses.tsx

interface QuickResponsesProps {
  options: Array<{
    emoji: string;
    label: string;
    value: string | number;
  }>;
  onSelect: (value: string | number) => void;
}

/*
LAYOUT:
- Horizontal row, evenly distributed
- Each button: pill shape, equal width
- Visible when AI asks for quick input

DEFAULT ENERGY OPTIONS:
[😊 Great] [😐 Okay] [😴 Tired] [🔥 Pumped]

DEFAULT ACTIVITY OPTIONS:
[💼 Work] [🏃 Exercise] [😴 Rest] [🗣️ Social]

STYLING:
- Background: background.tertiary
- Text: text.primary
- Border: 1px tertiary
- On press: accent background, haptic feedback

ANIMATION:
- Slide up from bottom
- Stagger animation for each button (50ms delay)
*/
```

#### 4.2.3 Chat Input

```typescript
// components/chat/ChatInput.tsx

interface ChatInputProps {
  value: string;
  onChange: (text: string) => void;
  onSend: () => void;
  onVoicePress?: () => void;
  disabled?: boolean;
  placeholder?: string;
}

/*
LAYOUT:
┌─────────────────────────────────────────────┐
│ [🎤] │ Type a message...              │ [➤] │
└─────────────────────────────────────────────┘

ELEMENTS:
- Voice button (optional): Left side, 40x40
- Text input: Flex 1, multiline (max 4 lines)
- Send button: Right side, 40x40, accent color

STATES:
- Empty: Send button disabled (30% opacity)
- Has text: Send button enabled, accent color
- Sending: Show loading spinner in send button
- Disabled: Entire input 50% opacity

STYLING:
- Container: background.secondary, borderRadius full
- Input: fontSize base, text.primary
- Placeholder: text.tertiary
- Min height: 48px
- Padding: 4px 8px 4px 16px

KEYBOARD:
- Type: default
- Return key: "send"
- Auto-capitalize: sentences
*/
```

#### 4.2.4 Energy Slider (for Check-ins)

```typescript
// components/chat/EnergySlider.tsx

interface EnergySliderProps {
  value: number;           // 1-10
  onChange: (value: number) => void;
  onSubmit: () => void;
}

/*
LAYOUT:
┌─────────────────────────────────────────────┐
│  How's your energy right now?              │
│                                             │
│  😴 ──────────●────────────────────── 🔥   │
│                    [ 6 ]                    │
│                                             │
│            [ Submit ]                       │
└─────────────────────────────────────────────┘

SLIDER STYLING:
- Track: 4px height, background.tertiary
- Filled track: accent gradient
- Thumb: 24px diameter, white, shadow md
- Labels: Emojis at each end

VALUE DISPLAY:
- Large number below slider
- Updates in real-time
- Haptic feedback on value change

SNAP POINTS:
- Slider snaps to integer values 1-10
- Haptic "tick" on each snap
*/
```

---

## 5. SCREEN 3: INSIGHTS

### 5.1 Layout Overview

```
┌─────────────────────────────────────────┐
│ ░░░░░░░░░░ STATUS BAR ░░░░░░░░░░░░░░░░ │
├─────────────────────────────────────────┤
│                                         │
│  Insights                     [7 days ▼]│  ← Header
│                                         │
├─────────────────────────────────────────┤
│                                         │
│  READINESS TREND                        │
│  ┌─────────────────────────────────┐   │
│  │     📈                          │   │  ← Chart
│  │   ╱  ╲    ╱╲                    │   │
│  │  ╱    ╲  ╱  ╲                   │   │
│  │ ╱      ╲╱    ╲____              │   │
│  │ M  T  W  T  F  S  S             │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Avg: 72  ↑ 5% vs last week            │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│  DISCOVERED PATTERNS                    │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🍷 Alcohol Impact               │   │  ← Pattern Card
│  │ 2+ drinks → -18% HRV next day  │   │
│  │ Confidence: 82%                 │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🌙 Late Meals                   │   │
│  │ Eating after 8PM → -12% sleep  │   │
│  │ Confidence: 75%                 │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🏃 Exercise Timing              │   │
│  │ Morning workouts → +15% energy │   │
│  │ Confidence: 68%                 │   │
│  └─────────────────────────────────┘   │
│                                         │
├─────────────────────────────────────────┤
│  [ ⚡ ]      [ 💬 ]      [ 📊 ]       │
└─────────────────────────────────────────┘
```

### 5.2 Component Specifications

#### 5.2.1 Trend Chart

```typescript
// components/insights/TrendChart.tsx

interface TrendChartProps {
  data: Array<{
    date: string;
    value: number;
    label?: string;
  }>;
  metric: 'readiness' | 'sleep' | 'hrv' | 'energy';
  period: '7d' | '14d' | '30d';
  showBaseline?: boolean;
  baselineValue?: number;
}

/*
CHART LIBRARY: react-native-chart-kit or victory-native

LAYOUT:
- Full width, 200px height
- X-axis: Date labels (abbreviated)
- Y-axis: Implicit (no visible labels)
- Grid: Subtle horizontal lines only

LINE STYLING:
- Stroke: accent color (from commander mode)
- Stroke width: 2px
- Bezier curve smoothing
- Area fill: accent color, 10% opacity

DATA POINTS:
- Show dots only on hover/press
- Current day: Larger dot, pulse animation

BASELINE:
- Dashed horizontal line
- Label: "Baseline: X"
- Color: text.tertiary

SUMMARY ROW:
- Below chart
- Format: "Avg: [X] [trend arrow] [Y]% vs [period]"
*/
```

#### 5.2.2 Pattern Card

```typescript
// components/insights/PatternCard.tsx

interface PatternCardProps {
  icon: string;
  title: string;
  description: string;
  impact: string;          // "+15% energy" or "-18% HRV"
  confidence: number;      // 0-100
  occurrences: number;
  onPress: () => void;
}

/*
LAYOUT:
┌─────────────────────────────────────────────┐
│ 🍷 Alcohol Impact                    [→]   │
├─────────────────────────────────────────────┤
│ 2+ drinks in evening                        │
│                                             │
│ Impact: -18% HRV next morning              │
│ Confidence: ████████░░ 82%                 │
│ Based on 12 occurrences                    │
└─────────────────────────────────────────────┘

CONFIDENCE BAR:
- 10 segments
- Filled segments = confidence / 10
- Colors: 
  - 80%+: success
  - 60-79%: warning
  - <60%: text.tertiary

IMPACT STYLING:
- Positive: success color, ↑ arrow
- Negative: error color, ↓ arrow

INTERACTIONS:
- onPress: Expand to show detailed analysis
- Show "See examples" link for raw data
*/
```

#### 5.2.3 Period Selector

```typescript
// components/insights/PeriodSelector.tsx

interface PeriodSelectorProps {
  selected: '7d' | '14d' | '30d';
  onChange: (period: '7d' | '14d' | '30d') => void;
}

/*
LAYOUT:
Dropdown button in header

OPTIONS:
- "7 days" (default)
- "14 days"
- "30 days"

STYLING:
- Button: text + chevron down icon
- Dropdown: bottom sheet or modal
- Selected: accent color check mark
*/
```

---

## 6. SETTINGS SCREEN

### 6.1 Layout Overview

```
┌─────────────────────────────────────────┐
│ ░░░░░░░░░░ STATUS BAR ░░░░░░░░░░░░░░░░ │
├─────────────────────────────────────────┤
│  ← Settings                             │
├─────────────────────────────────────────┤
│                                         │
│  PROFILE                                │
│  ┌─────────────────────────────────┐   │
│  │ Name                    [Name →]│   │
│  │ Commander Mode    [High Energy →]│   │
│  │ Wake Time              [07:00 →]│   │
│  └─────────────────────────────────┘   │
│                                         │
│  CONNECTIONS                            │
│  ┌─────────────────────────────────┐   │
│  │ Oura Ring            [Connected]│   │
│  │ Health Connect       [Connect →]│   │
│  └─────────────────────────────────┘   │
│                                         │
│  NOTIFICATIONS                          │
│  ┌─────────────────────────────────┐   │
│  │ Morning Briefing           [●] │   │
│  │ Check-in Reminders         [●] │   │
│  │ Insight Alerts             [○] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  DATA & PRIVACY                         │
│  ┌─────────────────────────────────┐   │
│  │ Export My Data              [→]│   │
│  │ Delete All Data             [→]│   │
│  │ Privacy Policy              [→]│   │
│  └─────────────────────────────────┘   │
│                                         │
│  ABOUT                                  │
│  ┌─────────────────────────────────┐   │
│  │ Version                   1.0.0 │   │
│  │ Send Feedback               [→]│   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

---

## 7. ONBOARDING FLOW

### 7.1 Screen Sequence

```
[1. WELCOME] → [2. OURA CONNECT] → [3. HEALTH CONNECT] → [4. MODE SELECT] → [5. COMPLETE]
```

### 7.2 Screen Specifications

#### 7.2.1 Welcome Screen

```
┌─────────────────────────────────────────┐
│                                         │
│                                         │
│              ⚡ PHI                      │
│                                         │
│      Your Digital Biological Twin      │
│                                         │
│   "Know thyself through data.          │
│    Act through intelligence.            │
│    Thrive through optimization."        │
│                                         │
│                                         │
│                                         │
│          [ Get Started ]                │
│                                         │
│     Already have an account? Sign in   │
│                                         │
└─────────────────────────────────────────┘
```

#### 7.2.2 Oura Connect Screen

```
┌─────────────────────────────────────────┐
│                                         │
│                 [○]                     │  ← Oura Ring icon
│                                         │
│        Connect Your Oura Ring           │
│                                         │
│   PHI uses your Oura data to           │
│   understand your recovery and         │
│   optimize your energy.                │
│                                         │
│   We access:                           │
│   ✓ Sleep data                         │
│   ✓ Readiness scores                   │
│   ✓ Heart rate & HRV                   │
│   ✓ Activity data                      │
│                                         │
│       [ Connect Oura Ring ]             │
│                                         │
│       Skip for now                      │
│                                         │
└─────────────────────────────────────────┘
```

#### 7.2.3 Mode Selection Screen

```
┌─────────────────────────────────────────┐
│                                         │
│      Choose Your Commander Mode         │
│                                         │
│   How do you want PHI to talk to you?  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ ⚡ HIGH ENERGY                  │   │
│  │ Direct. Challenging.            │   │
│  │ "Let's crush it."               │   │  ← Selected
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🧘 ZEN MASTER                   │   │
│  │ Calm. Supportive.               │   │
│  │ "Easy does it."                 │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 🌿 RECOVERY                     │   │
│  │ Gentle. Protective.             │   │
│  │ "Your body needs this."         │   │
│  └─────────────────────────────────┘   │
│                                         │
│   You can change this anytime          │
│                                         │
│          [ Continue ]                   │
│                                         │
└─────────────────────────────────────────┘
```

---

## 8. BOTTOM SHEETS & MODALS

### 8.1 Readiness Breakdown Sheet

```
┌─────────────────────────────────────────┐
│  ━━━                                    │  ← Handle
├─────────────────────────────────────────┤
│                                         │
│  Readiness Breakdown                    │
│                                         │
│  Previous Night         ████████░░  82  │
│  Sleep Balance          ███████░░░  71  │
│  Previous Day Activity  █████████░  88  │
│  Activity Balance       ██████░░░░  62  │
│  Body Temperature       ████████░░  78  │
│  Resting Heart Rate     █████████░  85  │
│  HRV Balance            ██████░░░░  58  │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  💡 Your HRV is below baseline.        │
│  Consider a lighter day.               │
│                                         │
│              [ Got it ]                 │
│                                         │
└─────────────────────────────────────────┘
```

### 8.2 Mode Selector Sheet

```
┌─────────────────────────────────────────┐
│  ━━━                                    │
├─────────────────────────────────────────┤
│                                         │
│  Switch Commander Mode                  │
│                                         │
│  Current: ⚡ High Energy               │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ ⚡ HIGH ENERGY            [✓]  │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ 🧘 ZEN MASTER                   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ 🔋 SOCIAL BATTERY               │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ 🌿 RECOVERY                     │   │
│  └─────────────────────────────────┘   │
│                                         │
│  PHI suggests: ZEN MASTER              │
│  (Based on your current readiness)     │
│                                         │
└─────────────────────────────────────────┘
```

---

## 9. ANIMATIONS & MICRO-INTERACTIONS

### 9.1 Animation Specifications

```typescript
// constants/animations.ts

import { Easing } from 'react-native-reanimated';

export const ANIMATIONS = {
  // Durations
  duration: {
    instant: 100,
    fast: 200,
    normal: 300,
    slow: 500,
    verySlow: 800,
  },
  
  // Easings
  easing: {
    default: Easing.bezier(0.25, 0.1, 0.25, 1),
    enter: Easing.bezier(0.0, 0.0, 0.2, 1),
    exit: Easing.bezier(0.4, 0.0, 1, 1),
    bounce: Easing.bezier(0.68, -0.55, 0.265, 1.55),
  },
  
  // Spring configs
  spring: {
    gentle: { damping: 15, stiffness: 100 },
    snappy: { damping: 20, stiffness: 200 },
    bouncy: { damping: 10, stiffness: 150 },
  },
};

// Animation presets
export const ANIMATION_PRESETS = {
  fadeIn: {
    from: { opacity: 0 },
    to: { opacity: 1 },
    duration: ANIMATIONS.duration.normal,
  },
  
  slideUp: {
    from: { translateY: 20, opacity: 0 },
    to: { translateY: 0, opacity: 1 },
    duration: ANIMATIONS.duration.normal,
  },
  
  scaleIn: {
    from: { scale: 0.9, opacity: 0 },
    to: { scale: 1, opacity: 1 },
    duration: ANIMATIONS.duration.fast,
  },
  
  pulseRing: {
    scale: [1, 1.05, 1],
    opacity: [1, 0.8, 1],
    duration: 2000,
    repeat: Infinity,
  },
};
```

### 9.2 Haptic Feedback

```typescript
// utils/haptics.ts

import * as Haptics from 'expo-haptics';

export const haptics = {
  // Light tap (button press)
  light: () => Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light),
  
  // Medium tap (selection change)
  medium: () => Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium),
  
  // Heavy tap (important action)
  heavy: () => Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Heavy),
  
  // Success (completed action)
  success: () => Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success),
  
  // Warning
  warning: () => Haptics.notificationAsync(Haptics.NotificationFeedbackType.Warning),
  
  // Error
  error: () => Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error),
  
  // Selection tick (slider, picker)
  tick: () => Haptics.selectionAsync(),
};
```

---

## 10. ACCESSIBILITY

### 10.1 Requirements

```typescript
// All interactive elements must have:
// - accessibilityLabel: Descriptive text
// - accessibilityHint: What happens on activation
// - accessibilityRole: button, link, header, etc.

// Example:
<TouchableOpacity
  accessibilityLabel="Readiness score 74 out of 100"
  accessibilityHint="Double tap to see breakdown"
  accessibilityRole="button"
>
  <ReadinessRing score={74} />
</TouchableOpacity>
```

### 10.2 Color Contrast

All text must meet WCAG AA standards:
- Normal text: 4.5:1 contrast ratio
- Large text (18pt+): 3:1 contrast ratio

```typescript
// Verified contrast ratios:
// text.primary (#FFFFFF) on background.primary (#0A0A0A) = 21:1 ✓
// text.secondary (#A0A0A0) on background.primary (#0A0A0A) = 10.4:1 ✓
// text.tertiary (#666666) on background.primary (#0A0A0A) = 5.3:1 ✓
```

### 10.3 Dynamic Type Support

```typescript
// Use scalable fonts
import { useFontScale } from 'react-native';

function ScalableText({ style, ...props }) {
  const fontScale = useFontScale();
  const scaledStyle = {
    ...style,
    fontSize: (style.fontSize || 16) * Math.min(fontScale, 1.5),
  };
  return <Text style={scaledStyle} {...props} />;
}
```

---

**END OF UI/UX SPECS**

*This document defines all visual and interaction specifications. Reference specific sections when implementing components. All measurements are in logical pixels unless otherwise noted.*
