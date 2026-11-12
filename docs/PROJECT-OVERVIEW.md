# Project Overview: Frontend-Only Notes Application

## 1. Executive Summary

This project is a cross-platform, client-side **Notes Application** targeting both **Android** and **Web** (Wasm/JS). Built using **Kotlin Multiplatform (KMP)** and **Compose Multiplatform (CMP)**, the application delivers a shared, unified codebase across mobile and browser environments.

A defining architectural characteristic of this application is that it is **strictly frontend-only**:
* **Zero Backend Dependency**: No remote servers, cloud databases, or external APIs are required to run or use the application.
* **Privacy & Local Ownership**: All user notes are created, modified, stored, and managed entirely on the user's local device/browser runtime.
* **Instant Responsiveness & Offline-First**: With zero network latency, all CRUD operations (Create, Read, Update, Delete) are instantaneous and work reliably offline by design.

---

## 2. Core Capabilities & Feature Scope (CRUD)

The application provides a clean, responsive notes management experience supporting full CRUD operations:

### 2.1. Create (Note Creation)
* **New Note Entry**: Floating action button (FAB) or dedicated action to open a note composer.
* **Data Fields**: Note title, body content, creation timestamp (`createdAt`), and last modification timestamp (`updatedAt`).
* **Optional Enhancements**: Note categorization/tagging, color coding, or pinned status.

### 2.2. Read (Browse & Inspect)
* **Note Listing**: Display notes in a clean list or staggered grid view.
* **Search & Filter**: Real-time client-side search filtering notes by title, content, or tags.
* **Sorting**: Sort notes by modification date (newest/oldest), creation date, or alphabetically by title.
* **Note Detail View**: Read-only or direct-editing inspection of individual notes.

### 2.3. Update (Editing & Revisions)
* **In-Place / Editor Mode**: Update title and note body seamlessly.
* **Automatic Timestamping**: Automatically updates `updatedAt` upon saving changes.
* **Auto-Save / Explicit Save**: Non-destructive editing with immediate local persistence to prevent accidental loss of text.

### 2.4. Delete (Removal)
* **Individual Note Deletion**: Swipe-to-delete or explicit trash action inside the note viewer/editor.
* **Confirmation Safety**: Action confirmations or an undo action (Snackbar) to safeguard against accidental deletions.
* **Bulk Operations (Future)**: Ability to select multiple notes for batch deletion.

---

## 3. Architecture & Project Structure

The project leverages Kotlin Multiplatform with a shared business logic and UI layer, accompanied by lightweight platform launcher applications.

```
MyApplication/
├── androidApp/               # Android entry point & native configurations
│   ├── src/main/
│   │   ├── AndroidManifest.xml
│   │   └── kotlin/com/example/myapplication/
│   │       └── MainActivity.kt       # Android Activity hosting shared Compose UI
│   └── build.gradle.kts
│
├── webApp/                   # Web entry point & web assets
│   ├── src/webMain/
│   │   ├── kotlin/com/example/myapplication/
│   │   │   └── main.kt               # Web application entry point (ComposeViewport)
│   │   └── resources/
│   │       ├── index.html            # Web HTML host page
│   │       └── styles.css            # Canvas styling & layout setup
│   └── build.gradle.kts
│
├── shared/                   # Core shared module (Multiplatform logic & CMP UI)
│   ├── src/
│   │   ├── commonMain/               # Common code shared across Android & Web
│   │   │   ├── composeResources/     # Shared multiplatform assets (drawables, strings, etc.)
│   │   │   └── kotlin/com/example/myapplication/
│   │   │       ├── App.kt            # Root Compose UI component
│   │   │       ├── model/            # Domain entities (e.g., Note)
│   │   │       ├── data/             # Local data storage interfaces & implementations
│   │   │       ├── viewmodel/        # MVVM state management & ViewModels
│   │   │       └── ui/               # Compose screens, components, theme
│   │   ├── androidMain/              # Android-specific implementations & previews
│   │   ├── wasmJsMain/               # Kotlin/Wasm platform hooks
│   │   ├── jsMain/                   # Kotlin/JS platform hooks
│   │   └── commonTest/               # Shared unit tests for domain & logic
│   └── build.gradle.kts
│
└── docs/                     # Documentation & specifications
    ├── PROJECT-OVERVIEW.md           # This document
    ├── design/DESIGN.MD              # UI/UX design specifications
    └── templates/                    # Process and logging templates
```

### 3.1. Architectural Pattern: MVVM & Unidirectional Data Flow (UDF)
* **UI Layer**: Built entirely with Compose Multiplatform using Material 3 design components. Stateless composables observe reactive state and dispatch user intents.
* **Presentation Layer**: ViewModels manage UI state using Kotlin Coroutines `StateFlow` and handle business events (e.g., `AddNote`, `UpdateNote`, `DeleteNote`, `FilterNotes`).
* **Data / Repository Layer**: An abstract `NotesRepository` defines CRUD methods returning coroutine flows, decoupling UI components from the underlying local storage mechanism.

---

## 4. Local Storage Strategy (Frontend-Only Persistence)

Because the application has **no backend**, persistent data must reside on the client device. The architecture uses a unified `NotesRepository` interface in `commonMain`:

```kotlin
interface NotesRepository {
    fun getAllNotes(): Flow<List<Note>>
    suspend fun getNoteById(id: String): Note?
    suspend fun insertNote(note: Note)
    suspend fun updateNote(note: Note)
    suspend fun deleteNote(id: String)
}
```

### 4.1. Platform Persistence Implementations
* **Android Platform**:
  * Utilizes local SQLite databases via Room (with Kotlin Multiplatform support) or SQLDelight, or key-value storage (Android DataStore / SharedPreferences) for lightweight data.
  * Ensures notes persist across app terminations and device reboots within Android's protected application sandbox.
* **Web Platform (Wasm & JS)**:
  * Leverages browser client storage: **IndexedDB** for structured record persistence or **LocalStorage** for lightweight JSON serialization via `kotlinx-browser` / `kotlinx.serialization`.
  * Preserves user data locally across page refreshes and browser sessions without contacting any external web server.

---

## 5. Technology Stack & Tooling

| Component | Technology | Version / Details |
| :--- | :--- | :--- |
| **Language** | Kotlin | `2.4.21` (Multiplatform) |
| **UI Framework** | Compose Multiplatform | `1.12.2` (Material 3) |
| **Android Target** | Android SDK | Compile/Target SDK `37`, Min SDK `24` |
| **Android Build** | Android Gradle Plugin (AGP) | `9.1.1` |
| **Web Targets** | Kotlin/Wasm (`wasmJs`) & Kotlin/JS (`js`) | High-performance Wasm Canvas with JS compatibility |
| **Lifecycle & ViewModel** | JetBrains AndroidX Lifecycle | `2.11.0` (ViewModel & Runtime Compose) |
| **Build System** | Gradle (Kotlin DSL) | Gradle Wrapper |

---

## 6. Data Model Specification

The central data entity of the application is the `Note` model:

```kotlin
data class Note(
    val id: String,                 // Unique identifier (UUID string)
    val title: String,              // Title header of the note
    val content: String,            // Note body text (plain text or markdown)
    val createdAt: Long,            // Creation epoch timestamp (milliseconds)
    val updatedAt: Long,            // Last modified epoch timestamp (milliseconds)
    val isPinned: Boolean = false,  // Pinned priority flag for quick access
    val colorHex: String? = null    // Optional card background color for visual categorization
)
```

---

## 7. Development & Build Instructions

### 7.1. Prerequisites
* **JDK 11** or higher.
* **Android SDK** configured (with platform tools and SDK version 37).
* Modern web browser supporting WebAssembly with Garbage Collection (WasmGC) for the Wasm target (Chrome 119+, Firefox 120+, Safari 18+).

### 7.2. Running the Applications

#### Android Application
To build and install the debug APK onto a connected Android device or emulator:
```bash
./gradlew :androidApp:assembleDebug
```

#### Web Application
The web app supports both Wasm and JS targets:
* **Wasm Target** (Recommended - fastest, modern browsers):
  ```bash
  ./gradlew :webApp:wasmJsBrowserDevelopmentRun
  ```
* **JS Target** (Compatibility fallback for older browsers):
  ```bash
  ./gradlew :webApp:jsBrowserDevelopmentRun
  ```

### 7.3. Running Tests
Run automated multiplatform unit and host tests using Gradle:
* **Shared Logic Common Tests**:
  ```bash
  ./gradlew :shared:allTests
  ```
* **Android Host Tests**:
  ```bash
  ./gradlew :shared:testAndroidHostTest
  ```
* **Web Tests**:
  ```bash
  ./gradlew :shared:wasmJsTest
  ./gradlew :shared:jsTest
  ```

---

## 8. Privacy, Security, and Offline Guarantees

* **Zero Telemetry & Zero External Requests**: No outbound network requests are initiated for note management or analytics.
* **Offline-Ready**: The application requires no active network connection at any point during its lifecycle.
* **Data Portability (Roadmap Recommendation)**: To ensure data longevity without a cloud backend, client-side JSON/Markdown export and import functionality can be provided to allow users to backup and restore their notes locally.
