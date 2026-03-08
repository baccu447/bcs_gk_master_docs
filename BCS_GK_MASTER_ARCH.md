# BCS/GK Master - Technical Architecture

## 1. Database Schema (ERD Text Format)

```mermaid
erDiagram
    USERS ||--o{ QUIZ_RESULTS : takes
    USERS ||--o{ SUBSCRIPTIONS : has
    USERS {
        bigint id PK
        string name
        string email
        string password_hash
        string avatar_url
        boolean is_pro
        integer coins
        datetime last_login
        timestamps created_at, updated_at
    }

    CATEGORIES ||--o{ QUESTIONS : contains
    CATEGORIES {
        bigint id PK
        string name "e.g., BCS, Admission, Jobs"
        string icon_url
        boolean is_active
        timestamps created_at, updated_at
    }

    QUESTIONS {
        bigint id PK
        bigint category_id FK
        text question_text
        string option_a
        string option_b
        string option_c
        string option_d
        string correct_answer "A, B, C, or D"
        text explanation
        string type "standard, daily_quiz, mock_test"
        string difficulty "easy, medium, hard"
        string ai_generated_metadata "JSON: source, model, prompt_ref"
        timestamps created_at, updated_at
    }

    QUIZ_RESULTS ||--o{ LEADERBOARD : updates
    QUIZ_RESULTS {
        bigint id PK
        bigint user_id FK
        bigint category_id FK
        integer total_questions
        integer correct_answers
        integer time_taken_seconds
        integer coins_earned
        date quiz_date
        timestamps created_at, updated_at
    }

    LEADERBOARD {
        bigint id PK
        bigint user_id FK
        bigint category_id FK "nullable for global"
        integer total_score
        string period "daily, weekly, all_time"
        date start_date
        date end_date
        timestamps created_at, updated_at
    }

    SUBSCRIPTIONS {
        bigint id PK
        bigint user_id FK
        string plan_name
        datetime start_date
        datetime end_date
        string status "active, expired, cancelled"
        timestamps created_at, updated_at
    }
```

*Note: The `ai_generated_metadata` column in the `QUESTIONS` table is designed to store metadata from automated question generation pipelines (via Gemini/OpenRouter).*

---

## 2. REST API Map (v1)

Base URL: `/api/v1`

### Authentication
- `POST /auth/register` - Register a new user
- `POST /auth/login` - Login & get token (Sanctum/Passport)
- `POST /auth/logout` - Invalidate token
- `GET /auth/me` - Get current user profile (including `is_pro` & `coins`)

### Categories
- `GET /categories` - List all active categories

### Questions
- `GET /questions` - Fetch questions (Query Params: `category_id`, `limit`, `offset`)
- `GET /questions/daily` - Fetch today's daily quiz questions
- `GET /questions/mock-test` - Fetch full mock test questions (Query Params: `category_id`)
- `POST /questions/ai-generate` - Admin/System endpoint to trigger AI generation pipeline

### Quiz & Sync
- `POST /quiz/submit` - Submit quiz results & update coins/score
- `POST /sync/push` - Sync local offline results to server (bulk update)
- `GET /sync/pull` - Fetch updated questions & categories since last sync timestamp

### Leaderboard
- `GET /leaderboard` - Get top users (Query Params: `category_id`, `period=daily|weekly|all_time`, `limit`)

### Subscriptions
- `POST /subscriptions/verify` - Verify in-app purchase and update `is_pro` status

---

## 3. UI/UX Map (A to Z)

### 1. Splash Screen
- App Logo
- Loading indicator
- Background initialization (checking auth, triggering silent sync)

### 2. Authentication Flow
- **Login/Register Screen:** Email/password forms, Google/Apple Sign-in buttons.
- **Onboarding:** Brief walkthrough of features.

### 3. Home Screen (Dashboard)
- User Profile Summary (Avatar, Name, Coins, Pro Status Badge)
- Daily Reward/Streak indicator
- **Sections:**
  - Daily Quiz Card (Call to Action)
  - Categories Grid (BCS, Bank Jobs, University Admission)
  - Recent Performance Mini-Chart
- Bottom Navigation Bar (Home, Leaderboard, Store/Pro, Profile)

### 4. Quiz Screen (The Core)
- Timer App Bar
- Question Counter (e.g., 5/20)
- Question Text (Large, readable font)
- 4 Option Buttons (Highlight Green/Red on selection depending on quiz mode)
- "Next" / "Submit" Button
- *Pro Feature:* "Hint" or "50/50" power-ups (costs coins)

### 5. Result Screen
- Circular Progress/Score indicator
- Coins earned animation
- Review Answers Button
- "Share Score" Button
- Call to Action: Take another quiz or Return Home

### 6. Leaderboard Screen
- Segmented Control (Daily, Weekly, All Time)
- Dropdown for Category filtering
- Top 3 Podium UI
- Scrollable list of ranks (highlighting current user's rank)

### 7. Pro / Store Screen
- Premium Subscription Plans (Monthly/Yearly)
- Benefits list (Ad-free, Mock Tests, Advanced Analytics)
- Coin Store (Buy coins for power-ups)

### 8. Profile Screen
- Detailed Stats (Total quizzes, accuracy rate)
- Edit Profile
- Settings (Theme toggle, Notification preferences, Sync status)
- Logout

---

## 4. Offline Sync Logic (Offline-First)

**Architecture:**
- **Local DB:** `sqflite` (relational structure is better suited for questions/categories than Hive, though Hive/Isar is fine for simple key-value prefs).
- **Backend:** Laravel API.

**Sync Strategy:**
1. **Initial Load / Pull:**
   - On first launch, the app downloads a static SQLite database bundle or fetches a large JSON payload of base categories and questions.
   - Saves a `last_sync_timestamp` in shared preferences/Hive.
2. **Delta Updates (Pull):**
   - Every app launch (while online), call `GET /sync/pull?since={last_sync_timestamp}`.
   - Server returns only categories/questions updated or added since that time.
   - Update local SQLite DB and update `last_sync_timestamp`.
3. **Offline Play:**
   - User takes quizzes entirely offline. Reads directly from SQLite.
   - Results are stored in a local `pending_results` table.
4. **Push Sync:**
   - When network is restored (listened via `connectivity_plus`), read all rows from `pending_results`.
   - Send payload to `POST /sync/push`.
   - On success response (200 OK), delete those rows from the local `pending_results` table.
5. **Conflict Resolution:**
   - Server is the source of truth for `coins` and `is_pro` status. The pull sync will overwrite local user profile data.

---

## 5. Color & Theme Palette

**Premium Dark / Professional Theme**
Designed to reduce eye strain during long study sessions while feeling modern and trusted.

- **Background (Primary Dark):** `#0F172A` (Slate 900) - Deep, rich background.
- **Surface / Cards:** `#1E293B` (Slate 800) - Elevated elements, quiz cards.
- **Primary Accent (Brand):** `#3B82F6` (Blue 500) - Primary buttons, active tabs, highlights. Trustworthy and professional.
- **Secondary Accent (Success/Correct):** `#10B981` (Emerald 500) - Correct answers, positive reinforcement, coins.
- **Error / Incorrect:** `#EF4444` (Red 500) - Wrong answers, destructive actions.
- **Warning / Pro Badge:** `#F59E0B` (Amber 500) - Premium features, warnings, streaks.
- **Text (High Emphasis):** `#F8FAFC` (Slate 50) - Headings, primary text.
- **Text (Medium Emphasis):** `#94A3B8` (Slate 400) - Subtitles, secondary info, disabled states.

*Note on AI readiness:* The UI surface uses distinct card layers (`#1E293B`) which provides excellent contrast for displaying AI-generated explanations or source citations in a slightly dimmer text (`#94A3B8`).