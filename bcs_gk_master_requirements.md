# Requirements Analysis: BCS/GK Master (Quiz App)

## 1. Project Overview
- **Name:** BCS/GK Master 🧠
- **Target:** Job Seekers (BCS, Bank, Govt Jobs) and Admission Candidates in Bangladesh.
- **Tech Stack:** Flutter (Frontend) + Laravel (Backend/Admin Panel).
- **Primary Goal:** AdMob Passive Income with Freemium Subscription.

## 2. Core Features (A to Z)
### A. User Experience (Flutter)
- **Offline-First Mode:** Questions loaded via API are cached locally (Hive/SQLite) for offline practice.
- **Daily Quiz:** A fresh set of 10-20 questions every day to drive retention.
- **Category-wise Practice:** Subject-based (Bangla, English, Math, GK, etc.) practice sets.
- **Live Leaderboard:** Real-time ranking to foster competition (Gamification).
- **Bookmarks:** Save difficult questions for later review.
- **Performance Analytics:** Visual charts showing progress and weak areas.
- **Mock Tests:** Time-bound exams mimicking real BCS preliminary conditions.

### B. Monetization (Freemium Model)
- **AdMob Integration:**
    - Banner ads on list views.
    - Interstitial ads between quiz transitions.
    - Rewarded Videos to unlock "Hints" or "Extra Life" during a quiz.
- **Subscription (Pro Version):**
    - Ad-free experience.
    - Premium mock tests.
    - PDF downloads of question sets.

### C. Backend & Management (Laravel)
- **Admin Dashboard:** Manage categories, questions, and users.
- **AI Question Generator:** Integration with Gemini/OpenRouter to fetch and format latest GK/Current Affairs.
- **Notification Manager:** Send push notifications for new quizzes or exam alerts.

## 3. UI/UX Design Strategy
- **Primary Color:** `#0A0A0F` (Deep Dark) or `#1E3A8A` (Deep Navy) for a professional look.
- **Accent Colors:** `#F0C14B` (Gold) for rewards/premium and `#10B981` (Success Green).
- **Typography:** Tiro Bangla (for Bengali text) & Poppins/Inter (for UI/English).
- **Screens:**
    1. Splash (Animated Logo).
    2. Home (Stats summary, Daily Quiz button, Categories).
    3. Quiz Screen (Clean, distraction-free, progress bar).
    4. Result Screen (Score, correct/wrong review, social share).
    5. Leaderboard (Global & Monthly ranking).
    6. Profile & Subscription (User stats, Go Pro button).

## 4. Database Schema (Draft)
- `users`: id, name, email, is_pro, coins.
- `categories`: id, name, icon.
- `questions`: id, category_id, question_text, options(json), correct_option, explanation.
- `quiz_results`: id, user_id, score, time_taken.
- `leaderboard`: id, user_id, total_points.

---
*Prepared by Bondhu 🛠️*
