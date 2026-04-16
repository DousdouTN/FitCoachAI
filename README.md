# FitCoach AI — Android App

A production-grade AI-powered fitness coaching app built with **Kotlin + Jetpack Compose**, following modern Android architecture best practices.

---

## 📐 Architecture

```
app/
├── core/
│   ├── data/
│   │   ├── local/          # Room DB — entities, DAOs, database
│   │   ├── remote/         # Retrofit services, DTOs
│   │   └── repository/     # Concrete repositories + mappers
│   ├── di/                 # Hilt modules
│   └── utils/              # Interceptors, Result wrapper, Workers
├── domain/
│   ├── model/              # Pure Kotlin domain models + enums
│   └── usecase/            # (Add as complexity grows)
└── presentation/
    ├── auth/               # Login/Register screen + ViewModel
    ├── home/               # Dashboard screen + ViewModel
    ├── workout/            # Builder + Session screens + ViewModel
    ├── exercise/           # Library screen + ViewModel
    ├── progress/           # Progress screen + ViewModel
    ├── chat/               # AI Chat screen + ViewModel
    ├── profile/            # User profile (extend as needed)
    ├── navigation/         # NavHost + routes + BottomNav
    ├── theme/              # Colors, Typography, Shapes
    └── components/         # Shared Compose components
```

**Pattern:** MVVM with unidirectional data flow.  
`UI → ViewModel (StateFlow) → Repository → Room / Retrofit / Firebase`

---

## 🚀 Setup

### 1. Clone & open in Android Studio (Hedgehog or newer)

```bash
git clone https://github.com/your-org/FitCoachAI.git
```

### 2. Add `google-services.json`

- Create a Firebase project at [console.firebase.google.com](https://console.firebase.google.com)
- Enable Authentication (Email/Password + Google), Firestore, and Cloud Messaging
- Download `google-services.json` and place it in `app/`

### 3. Add API keys to `local.properties`

```properties
# OpenAI key — https://platform.openai.com/api-keys
OPENAI_API_KEY=sk-...

# ExerciseDB — https://rapidapi.com/justin-WFnsXH_t6/api/exercisedb
EXERCISE_DB_API_KEY=your_rapidapi_key
```

> **Security note:** Never commit `local.properties`. In CI, inject these as environment secrets.

### 4. Build & run

```
./gradlew :app:assembleDebug
```

---

## 🧠 AI Integration

### OpenAI Chat Completion

All AI calls go through `AiChatRepository.sendMessage()` → `OpenAiService` (Retrofit) → `https://api.openai.com/v1/chat/completions`.

**Model used:** `gpt-4o-mini` (cost-efficient). Swap for `gpt-4o` for higher quality.

**Context window strategy:**  
The full conversation history is sent with every request so the AI has context. For production with many messages, implement a sliding window (last N messages + summary of older ones).

### Prompt Engineering

```kotlin
// System prompt (in Constants.kt)
"You are FitCoach AI, a world-class personal fitness coach..."

// Workout plan generation
Constants.workoutPlanPrompt(goal, experience, equipment, days, weeks)
```

To generate a structured plan programmatically (e.g. on profile setup):
```kotlin
aiChatRepository.generateWorkoutPlan(
    goal        = "Muscle Gain",
    experience  = "Intermediate",
    equipment   = "Dumbbell, Bench",
    daysPerWeek = 4,
    durationWeeks = 8
)
```

---

## 🗄️ Local Database (Room)

| Table                | Purpose                                    |
|---------------------|--------------------------------------------|
| `user_profiles`     | Cached user profile (synced with Firestore)|
| `exercises`         | Exercise library (synced from ExerciseDB)  |
| `workouts`          | Workout templates + logged sessions        |
| `workout_exercises` | Exercise-per-workout join table with sets  |
| `body_stats`        | Weight / body fat history                  |
| `chat_messages`     | Persistent AI chat history                 |

All entities use **Moshi** for complex type serialization (lists, enums).

---

## 🔔 Notifications

Two notification channels are registered in `FitCoachApplication`:

| Channel ID             | Purpose                        |
|-----------------------|--------------------------------|
| `workout_reminders`   | Daily workout reminders (WorkManager) |
| `progress_milestones` | PR alerts, streak milestones   |

**Schedule a daily reminder:**
```kotlin
WorkoutReminderWorker.schedule(context, message = "Time to train! 💪")
```

**Firebase Cloud Messaging** is wired in `FitCoachMessagingService` for server-pushed notifications.

---

## 📊 Charts

Charts use **Vico** (`com.patrykandpatrick.vico`). The integration point in `ProgressScreen.kt` is marked with a commented-out `CartesianChartHost` block — uncomment and configure after adding your data model.

---

## 🏗️ Key Technologies

| Library          | Version  | Purpose                         |
|-----------------|----------|----------------------------------|
| Jetpack Compose  | BOM 2024.06 | Declarative UI               |
| Hilt            | 2.51.1   | Dependency injection            |
| Room            | 2.6.1    | Local SQLite ORM                |
| Retrofit        | 2.11.0   | HTTP client                     |
| Moshi           | 1.15.1   | JSON serialization              |
| Coil            | 2.6.0    | Image / GIF loading             |
| Firebase        | BOM 33.1 | Auth, Firestore, FCM            |
| Vico            | 2.0.0-alpha | Charts                       |
| Lottie          | 6.4.0    | Lottie animations               |
| WorkManager     | 2.9.0    | Background task scheduling      |

---

## 🚧 Extending the App

### Add a new screen

1. Create `presentation/myfeature/MyFeatureScreen.kt` (Composable)
2. Create `presentation/myfeature/MyFeatureViewModel.kt` (HiltViewModel)
3. Add route to `navigation/Navigation.kt`
4. Add bottom nav item if needed

### Add a new data type

1. Add domain model to `domain/model/Models.kt`
2. Add Room entity to `core/data/local/entity/Entities.kt`
3. Add DAO to `core/data/local/dao/Daos.kt`
4. Register in `FitCoachDatabase.kt`
5. Add mappers to `core/data/repository/Mappers.kt`
6. Add repository methods to `core/data/repository/Repositories.kt`

### Swap AI provider

Replace `OpenAiService` interface and `ChatRequest`/`ChatResponse` DTOs with your provider's schema. The `AiChatRepository` abstraction means ViewModels are unaffected.

---

## 🔐 Security Checklist

- [ ] Never embed API keys in source code — use `local.properties` or CI secrets
- [ ] Add Firestore security rules (`users/{uid}/**` → owner-only)
- [ ] Enable ProGuard/R8 for release builds (already configured in `build.gradle.kts`)
- [ ] Use HTTPS for all network calls (enforced by OkHttp defaults)
- [ ] Validate AI response length before displaying (prevent OOM)

---

## 📄 License

MIT — free to use, modify, and distribute.
