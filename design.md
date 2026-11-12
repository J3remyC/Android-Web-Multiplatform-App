# System Architecture & Technical Design Specification
## Notes Application — Frontend-Only Compose Multiplatform (CMP)

---

## 1. Executive Summary & Overview

This document specifies the technical design, architectural patterns, and file structure for the **Notes Application**. Built with **Kotlin Multiplatform (KMP)** and **Compose Multiplatform (CMP)**, the application targets both **Android** and **Web** (Wasm/JS) from a unified codebase.

A core tenet of this system is that it is **strictly frontend-only**:
* **Zero Backend Dependency**: No remote servers, cloud databases, or external REST/GraphQL APIs are involved.
* **Local Data Ownership**: All notes are stored, indexed, and processed directly on the client device or browser runtime.
* **Instant Responsiveness & Offline-First**: With zero network latency, all CRUD (Create, Read, Update, Delete) operations execute instantaneously and remain fully operational offline.

To ensure long-term maintainability, strict separation of concerns, and robust testability, the system adopts the **MVVM (Model-View-ViewModel)** architectural pattern augmented with **Unidirectional Data Flow (UDF)** and **MVI (Model-View-Intent)** principles.

---

## 2. Core Architectural Pattern: MVVM + UDF (Screen - Intent - UIState - ViewModel)

The presentation layer is structured around four distinct, decoupled responsibilities:

```
    ┌────────────────────────────────────────────────────────┐
    │                         Screen                         │
    │                 (Declarative Compose UI)                │
    └──────────────┬──────────────────────────▲──────────────┘
                   │                          │
      Dispatches   │                          │ Observes
        Intent     │                          │  UIState
                   ▼                          │
    ┌──────────────────────────────┐          │
    │            Intent            │          │
    │    (Sealed Action Types)     │          │
    └──────────────┬───────────────┘          │
                   │                          │
                   ▼                          │
    ┌─────────────────────────────────────────┴──────────────┐
    │                       ViewModel                        │
    │              (Inherits from BaseViewModel)             │
    │  - Processes Intents via onIntent(intent)              │
    │  - Emits immutable UIState via StateFlow<S>            │
    └──────────────┬──────────────────────────▲──────────────┘
                   │                          │
       Performs    │                          │ Emits Data
      CRUD Actions │                          │   Streams
                   ▼                          │
    ┌─────────────────────────────────────────┴──────────────┐
    │                      Repository                        │
    │                 (NotesRepository Flow)                 │
    └────────────────────────────────────────────────────────┘
```

### 2.1. The Four Key Components

1. **Screen (View)**:
   * Built entirely with Compose Multiplatform using Material 3 composables.
   * **Strictly Stateless**: The screen composable does not hold mutable business state or perform repository calls directly.
   * It observes an immutable `UIState` exposed by the `ViewModel` and renders the visual interface.
   * User interactions (taps, text input, button clicks) are converted into `Intent` objects and forwarded to the ViewModel.

2. **Intent (User Actions / Events)**:
   * Defined as a type-safe `sealed interface`.
   * Encapsulates all possible actions, gestures, and lifecycle triggers that can originate from the screen (e.g., `SearchQueryChanged`, `DeleteNote`, `SaveNote`, `TogglePin`).
   * Eliminates ad-hoc public method exposure on ViewModels, providing a single, traceable entry point for state changes.

3. **UIState (Immutable View State)**:
   * Defined as an immutable Kotlin `data class`.
   * Serves as the **Single Source of Truth (SSOT)** for everything rendered on the Screen.
   * Contains primitive values, collections, loading flags, error messages, and computed helper getters (e.g., `pinnedNotes`, `isEmpty`, `canSave`).
   * State transitions are atomic: the ViewModel produces a new instance via `copy()` rather than mutating properties in place.

4. **ViewModel (Which Inherits from BaseViewModel)**:
   * Inherits from `BaseViewModel<S, I>` which in turn inherits from `androidx.lifecycle.ViewModel`.
   * Survives Android configuration changes and manages asynchronous coroutine execution within lifecycle-aware scopes (`viewModelScope`).
   * Exposes a read-only `StateFlow<UIState>` to the Screen.
   * Implements `onIntent(intent: I)` to process incoming user actions, coordinate repository calls, and update state via `setState { copy(...) }`.

---

## 3. Strict File Separation Policy

### 3.1. Rationale for File Separation

A key architectural mandate is to **separate each role into its own dedicated file**:
* **Screen File**: Contains only UI layout, styling, and composable functions. Free of business logic.
* **Intent File**: Contains only the sealed interface defining actions. Can be inspected at a glance to understand all possible user interactions.
* **UIState File**: Contains only the state data class and view-specific computed properties.
* **ViewModel File**: Contains only state management, repository coordination, and intent handling. Inherits from `BaseViewModel`.
* **BaseViewModel File**: Contains the reusable generic base class managing `StateFlow`, `setState`, and lifecycle coroutines.

**Key Benefits**:
* **Single Responsibility Principle (SRP)**: Each file has exactly one reason to change.
* **Merge Conflict Elimination**: Multiple developers can work on the UI layout (Screen) and business logic (ViewModel) simultaneously without git conflicts.
* **Isolated Unit Testing**: ViewModels and UIStates can be tested in pure JVM/Common tests without loading Compose UI runtimes.
* **UI Previewability**: Screens can be previewed or tested with mock UIState without instantiating actual ViewModels or repositories.

### 3.2. Project Directory & File Separation Map

```text
shared/src/commonMain/kotlin/com/example/myapplication/
├── App.kt                                      # Navigation coordinator & app root
│
├── model/                                      # Domain Entities
│   └── Note.kt                                 # Core Note data model
│
├── data/                                       # Data Layer / Repositories
│   ├── NotesRepository.kt                      # CRUD repository interface
│   └── InMemoryNotesRepository.kt              # Reactive local storage implementation
│
├── viewmodel/                                  # Presentation Architecture
│   ├── BaseViewModel.kt                        # Generic abstract BaseViewModel (inherits ViewModel)
│   │
│   ├── notelist/                               # Note List Feature (SEPARATE FILES)
│   │   ├── NoteListUiState.kt                  # File 1: NoteList UI State
│   │   ├── NoteListIntent.kt                   # File 2: NoteList User Intents
│   │   └── NoteListViewModel.kt                # File 3: NoteList ViewModel (inherits BaseViewModel)
│   │
│   └── notedetail/                             # Note Detail Feature (SEPARATE FILES)
│       ├── NoteDetailUiState.kt                # File 1: NoteDetail UI State
│       ├── NoteDetailIntent.kt                 # File 2: NoteDetail User Intents
│       └── NoteDetailViewModel.kt              # File 3: NoteDetail ViewModel (inherits BaseViewModel)
│
└── ui/                                         # Declarative Compose UI (SEPARATE FILES)
    ├── notelist/
    │   └── NoteListScreen.kt                   # File 4: NoteList Screen composable
    └── notedetail/
        └── NoteDetailScreen.kt                 # File 4: NoteDetail Screen composable
```

---

## 4. Architectural Hierarchy: BaseViewModel Specification

Every feature ViewModel inherits from `BaseViewModel<S, I>`, which inherits from `androidx.lifecycle.ViewModel`:

```kotlin
// shared/src/commonMain/kotlin/com/example/myapplication/viewmodel/BaseViewModel.kt
package com.example.myapplication.viewmodel

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update

/**
 * Abstract generic BaseViewModel adhering to Unidirectional Data Flow (UDF).
 * Inherits from [androidx.lifecycle.ViewModel].
 *
 * @param S The immutable UI State type representing screen rendering state.
 * @param I The sealed Intent type representing user interactions or events.
 * @param initialState The initial state for this ViewModel.
 */
abstract class BaseViewModel<S, I>(initialState: S) : ViewModel() {

    private val _uiState = MutableStateFlow(initialState)
    val uiState: StateFlow<S> = _uiState.asStateFlow()

    /**
     * Synchronous access to the current state snapshot.
     */
    protected val currentState: S
        get() = _uiState.value

    /**
     * Atomically updates the current UI state using a reducer function.
     */
    protected fun setState(reduce: S.() -> S) {
        _uiState.update(reduce)
    }

    /**
     * Entry point for processing user actions / intents dispatched from the Screen.
     */
    abstract fun onIntent(intent: I)
}
```

### Key Design Highlights:
* **Encapsulated State**: `_uiState` is private; only the immutable `StateFlow<S>` is public.
* **Atomic State Reduction**: `setState(reduce: S.() -> S)` uses `MutableStateFlow.update`, guaranteeing thread safety and eliminating race conditions.
* **Intent Dispatch Contract**: Forces derived ViewModels to implement `onIntent(intent: I)` via a standard pattern.

---

## 5. Feature 1 Design: Note List (Browsing, Search & Management)

### 5.1. File 1: `NoteListUiState.kt`
Represents the complete state required to render the list of notes.

```kotlin
package com.example.myapplication.viewmodel.notelist

import com.example.myapplication.model.Note

data class NoteListUiState(
    val notes: List<Note> = emptyList(),
    val filteredNotes: List<Note> = emptyList(),
    val searchQuery: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null
) {
    val pinnedNotes: List<Note>
        get() = filteredNotes.filter { it.isPinned }

    val otherNotes: List<Note>
        get() = filteredNotes.filterNot { it.isPinned }

    val isEmpty: Boolean
        get() = !isLoading && notes.isEmpty()

    val isSearchEmpty: Boolean
        get() = !isLoading && notes.isNotEmpty() && filteredNotes.isEmpty()
}
```

### 5.2. File 2: `NoteListIntent.kt`
Encapsulates all actions the user can trigger from the Note List interface.

```kotlin
package com.example.myapplication.viewmodel.notelist

sealed interface NoteListIntent {
    data class SearchQueryChanged(val query: String) : NoteListIntent
    data class DeleteNote(val id: String) : NoteListIntent
    data class TogglePin(val id: String) : NoteListIntent
    data object ClearError : NoteListIntent
    data object LoadNotes : NoteListIntent
}
```

### 5.3. File 3: `NoteListViewModel.kt`
Inherits from `BaseViewModel<NoteListUiState, NoteListIntent>`. Coordinates reactive repository flows, handles live search filtering, and handles note pinning and deletion.

```kotlin
package com.example.myapplication.viewmodel.notelist

import androidx.lifecycle.viewModelScope
import com.example.myapplication.data.NotesRepository
import com.example.myapplication.model.Note
import com.example.myapplication.viewmodel.BaseViewModel
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.launch

class NoteListViewModel(
    private val repository: NotesRepository,
    private val coroutineScope: CoroutineScope? = null
) : BaseViewModel<NoteListUiState, NoteListIntent>(NoteListUiState(isLoading = true)) {

    private val effectiveScope: CoroutineScope
        get() = coroutineScope ?: viewModelScope

    init {
        observeNotes()
    }

    override fun onIntent(intent: NoteListIntent) {
        when (intent) {
            is NoteListIntent.SearchQueryChanged -> handleSearchQueryChanged(intent.query)
            is NoteListIntent.DeleteNote -> handleDeleteNote(intent.id)
            is NoteListIntent.TogglePin -> handleTogglePin(intent.id)
            is NoteListIntent.ClearError -> setState { copy(errorMessage = null) }
            is NoteListIntent.LoadNotes -> observeNotes()
        }
    }

    private fun observeNotes() {
        setState { copy(isLoading = true, errorMessage = null) }
        effectiveScope.launch {
            repository.getAllNotes()
                .catch { throwable ->
                    setState { copy(isLoading = false, errorMessage = throwable.message ?: "Failed to load notes") }
                }
                .collectLatest { notesList ->
                    setState {
                        copy(
                            notes = notesList,
                            filteredNotes = filterNotes(notesList, searchQuery),
                            isLoading = false
                        )
                    }
                }
        }
    }

    private fun handleSearchQueryChanged(query: String) {
        setState {
            copy(
                searchQuery = query,
                filteredNotes = filterNotes(notes, query)
            )
        }
    }

    private fun handleDeleteNote(id: String) {
        effectiveScope.launch {
            try {
                repository.deleteNote(id)
            } catch (throwable: Throwable) {
                setState { copy(errorMessage = throwable.message ?: "Failed to delete note") }
            }
        }
    }

    private fun handleTogglePin(id: String) {
        effectiveScope.launch {
            try {
                val note = repository.getNoteById(id) ?: return@launch
                val updatedNote = note.copy(isPinned = !note.isPinned)
                repository.updateNote(updatedNote)
            } catch (throwable: Throwable) {
                setState { copy(errorMessage = throwable.message ?: "Failed to toggle pin") }
            }
        }
    }

    private fun filterNotes(notes: List<Note>, query: String): List<Note> {
        val trimmed = query.trim()
        if (trimmed.isEmpty()) return notes
        return notes.filter { note ->
            note.title.contains(trimmed, ignoreCase = true) ||
                note.content.contains(trimmed, ignoreCase = true)
        }
    }
}
```

### 5.4. File 4: `NoteListScreen.kt`
Stateless Compose Multiplatform view rendering the search bar, pinned and regular note lists, note cards, and FAB.

* **Parameters**:
  * `uiState: NoteListUiState`
  * `onIntent: (NoteListIntent) -> Unit`
  * `onNoteClick: (noteId: String) -> Unit`
  * `onAddNoteClick: () -> Unit`
* **UI Features**:
  * Outlined search bar with real-time text query forwarding.
  * Separate sections for "PINNED" notes and "OTHERS".
  * Material 3 Card components with title truncation, preview body, pin toggle, and delete triggers.
  * Informative empty states for zero notes or no search results.
  * FloatingActionButton (FAB) with primary container styling to initiate note creation.

---

## 6. Feature 2 Design: Note Detail & Editor

### 6.1. File 1: `NoteDetailUiState.kt`
Represents the editor's draft state, validation flags, and navigation signals.

```kotlin
package com.example.myapplication.viewmodel.notedetail

data class NoteDetailUiState(
    val noteId: String? = null,
    val title: String = "",
    val content: String = "",
    val isPinned: Boolean = false,
    val colorHex: String? = null,
    val isLoading: Boolean = false,
    val isSaved: Boolean = false,
    val isDeleted: Boolean = false,
    val errorMessage: String? = null
) {
    val isNewNote: Boolean
        get() = noteId == null

    val canSave: Boolean
        get() = title.isNotBlank() || content.isNotBlank()
}
```

### 6.2. File 2: `NoteDetailIntent.kt`
Captures all user actions inside the note viewer and editor.

```kotlin
package com.example.myapplication.viewmodel.notedetail

sealed interface NoteDetailIntent {
    data class LoadNote(val id: String) : NoteDetailIntent
    data class TitleChanged(val title: String) : NoteDetailIntent
    data class ContentChanged(val content: String) : NoteDetailIntent
    data object TogglePin : NoteDetailIntent
    data class ColorChanged(val colorHex: String?) : NoteDetailIntent
    data object SaveNote : NoteDetailIntent
    data object DeleteNote : NoteDetailIntent
    data object ClearError : NoteDetailIntent
}
```

### 6.3. File 3: `NoteDetailViewModel.kt`
Inherits from `BaseViewModel<NoteDetailUiState, NoteDetailIntent>`. Handles creating new notes with unique IDs and timestamps, updating existing notes, and handling deletion.

```kotlin
package com.example.myapplication.viewmodel.notedetail

import androidx.lifecycle.viewModelScope
import com.example.myapplication.data.NotesRepository
import com.example.myapplication.model.Note
import com.example.myapplication.viewmodel.BaseViewModel
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.launch
import kotlin.random.Random

class NoteDetailViewModel(
    private val repository: NotesRepository,
    private val idGenerator: () -> String = { "note_${Random.nextLong().toString(16)}" },
    private val timeProvider: () -> Long = { 1731420000000L },
    private val coroutineScope: CoroutineScope? = null
) : BaseViewModel<NoteDetailUiState, NoteDetailIntent>(NoteDetailUiState()) {

    private val effectiveScope: CoroutineScope
        get() = coroutineScope ?: viewModelScope

    override fun onIntent(intent: NoteDetailIntent) {
        when (intent) {
            is NoteDetailIntent.LoadNote -> handleLoadNote(intent.id)
            is NoteDetailIntent.TitleChanged -> setState { copy(title = intent.title) }
            is NoteDetailIntent.ContentChanged -> setState { copy(content = intent.content) }
            is NoteDetailIntent.TogglePin -> setState { copy(isPinned = !isPinned) }
            is NoteDetailIntent.ColorChanged -> setState { copy(colorHex = intent.colorHex) }
            is NoteDetailIntent.SaveNote -> handleSaveNote()
            is NoteDetailIntent.DeleteNote -> handleDeleteNote()
            is NoteDetailIntent.ClearError -> setState { copy(errorMessage = null) }
        }
    }

    private fun handleLoadNote(id: String) {
        setState { copy(isLoading = true, errorMessage = null) }
        effectiveScope.launch {
            try {
                val note = repository.getNoteById(id)
                if (note != null) {
                    setState {
                        copy(
                            noteId = note.id,
                            title = note.title,
                            content = note.content,
                            isPinned = note.isPinned,
                            colorHex = note.colorHex,
                            isLoading = false
                        )
                    }
                } else {
                    setState { copy(isLoading = false, errorMessage = "Note not found") }
                }
            } catch (throwable: Throwable) {
                setState { copy(isLoading = false, errorMessage = throwable.message ?: "Failed to load note") }
            }
        }
    }

    private fun handleSaveNote() {
        if (!currentState.canSave) return
        val state = currentState
        val now = timeProvider()

        effectiveScope.launch {
            try {
                if (state.isNewNote) {
                    val newNote = Note(
                        id = idGenerator(),
                        title = state.title.trim(),
                        content = state.content.trim(),
                        createdAt = now,
                        updatedAt = now,
                        isPinned = state.isPinned,
                        colorHex = state.colorHex
                    )
                    repository.insertNote(newNote)
                } else {
                    val existing = repository.getNoteById(state.noteId!!)
                    val updatedNote = Note(
                        id = state.noteId,
                        title = state.title.trim(),
                        content = state.content.trim(),
                        createdAt = existing?.createdAt ?: now,
                        updatedAt = now,
                        isPinned = state.isPinned,
                        colorHex = state.colorHex
                    )
                    repository.updateNote(updatedNote)
                }
                setState { copy(isSaved = true) }
            } catch (throwable: Throwable) {
                setState { copy(errorMessage = throwable.message ?: "Failed to save note") }
            }
        }
    }

    private fun handleDeleteNote() {
        val noteId = currentState.noteId ?: return
        effectiveScope.launch {
            try {
                repository.deleteNote(noteId)
                setState { copy(isDeleted = true) }
            } catch (throwable: Throwable) {
                setState { copy(errorMessage = throwable.message ?: "Failed to delete note") }
            }
        }
    }
}
```

### 6.4. File 4: `NoteDetailScreen.kt`
Stateless Compose Multiplatform view for inspecting and editing note title and body content.

* **Parameters**:
  * `uiState: NoteDetailUiState`
  * `onIntent: (NoteDetailIntent) -> Unit`
  * `onNavigateBack: () -> Unit`
* **UI Features**:
  * Dynamic TopAppBar title ("New Note" vs. "Edit Note").
  * Action items: Back button, Pin/Unpin status toggle, Delete button (for existing notes), and Save button (enabled only when `uiState.canSave` is true).
  * `LaunchedEffect` observing `uiState.isSaved` and `uiState.isDeleted` to automatically trigger `onNavigateBack()` when operations succeed.
  * Distinct OutlinedTextField for single-line Title and expanding multiline Content.

---

## 7. Domain & Data Persistence Layer

### 7.1. Domain Entity: `Note.kt`
The fundamental entity of the system:

```kotlin
package com.example.myapplication.model

data class Note(
    val id: String,                 // Unique string identifier
    val title: String,              // Note title header
    val content: String,            // Note body text (plain text / markdown)
    val createdAt: Long,            // Creation timestamp in epoch milliseconds
    val updatedAt: Long,            // Last modification timestamp in epoch milliseconds
    val isPinned: Boolean = false,  // Pin priority flag
    val colorHex: String? = null    // Optional visual categorization color
)
```

### 7.2. Repository Contract: `NotesRepository.kt`
The interface decoupling the presentation layer from the persistence engine:

```kotlin
package com.example.myapplication.data

import com.example.myapplication.model.Note
import kotlinx.coroutines.flow.Flow

interface NotesRepository {
    fun getAllNotes(): Flow<List<Note>>
    suspend fun getNoteById(id: String): Note?
    suspend fun insertNote(note: Note)
    suspend fun updateNote(note: Note)
    suspend fun deleteNote(id: String)
}
```

### 7.3. Persistence Implementations

1. **In-Memory Reactive Implementation (`InMemoryNotesRepository.kt`)**:
   * Uses `MutableStateFlow<List<Note>>` as the in-memory store.
   * Emits sorted lists (pinned notes first, then ordered by `updatedAt` descending).
   * Seeded with initial sample notes for immediate out-of-the-box demonstration.

2. **Android Platform Storage Strategy**:
   * Local SQLite storage using **Android Room (KMP)** or **SQLDelight**.
   * Resides in the private internal app storage sandbox (`/data/data/<package>/databases/`).
   * Survives app restarts, OS process death, and device reboots.

3. **Web Platform Storage Strategy (Wasm & JS)**:
   * Client-side browser persistence using **IndexedDB** or **LocalStorage**.
   * Serialized with `kotlinx.serialization.json`.
   * Requires zero network roundtrips and retains notes across tab closes and browser restarts.

---

## 8. Root Coordination & Navigation (`App.kt`)

The root `App.kt` composable manages top-level screen switching between `NoteList` and `NoteDetail`:

```kotlin
package com.example.myapplication

import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.example.myapplication.data.InMemoryNotesRepository
import com.example.myapplication.ui.notedetail.NoteDetailScreen
import com.example.myapplication.ui.notelist.NoteListScreen
import com.example.myapplication.viewmodel.notedetail.NoteDetailIntent
import com.example.myapplication.viewmodel.notedetail.NoteDetailViewModel
import com.example.myapplication.viewmodel.notelist.NoteListViewModel

private sealed interface AppScreen {
    data object NoteList : AppScreen
    data class NoteDetail(val noteId: String?) : AppScreen
}

@Composable
@Preview
fun App(modifier: Modifier = Modifier) {
    MaterialTheme {
        val repository = remember { InMemoryNotesRepository() }
        var currentScreen by remember { mutableStateOf<AppScreen>(AppScreen.NoteList) }

        val noteListViewModel = remember(repository) {
            NoteListViewModel(repository)
        }

        when (val screen = currentScreen) {
            is AppScreen.NoteList -> {
                val listUiState by noteListViewModel.uiState.collectAsState()
                NoteListScreen(
                    uiState = listUiState,
                    onIntent = noteListViewModel::onIntent,
                    onNoteClick = { noteId ->
                        currentScreen = AppScreen.NoteDetail(noteId)
                    },
                    onAddNoteClick = {
                        currentScreen = AppScreen.NoteDetail(null)
                    },
                    modifier = modifier
                )
            }
            is AppScreen.NoteDetail -> {
                val detailViewModel = remember(screen.noteId, repository) {
                    NoteDetailViewModel(repository).apply {
                        if (screen.noteId != null) {
                            onIntent(NoteDetailIntent.LoadNote(screen.noteId))
                        }
                    }
                }
                val detailUiState by detailViewModel.uiState.collectAsState()

                NoteDetailScreen(
                    uiState = detailUiState,
                    onIntent = detailViewModel::onIntent,
                    onNavigateBack = {
                        currentScreen = AppScreen.NoteList
                    },
                    modifier = modifier
                )
            }
        }
    }
}
```

---

## 9. UI / UX Design & Layout Mockups

### 9.1. Note List Screen Layout (ASCII Wireframe)

```text
┌────────────────────────────────────────────────────────┐
│ [My Notes]                                             │ TopAppBar (PrimaryContainer)
├────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────┐ │
│ │ 🔍 Search notes by title or content...             │ │ OutlinedTextField
│ └────────────────────────────────────────────────────┘ │
│                                                        │
│ PINNED                                                 │ Category Header
│ ┌────────────────────────────────────────────────────┐ │
│ │ Welcome to Notes CMP             [Unpin]  [Delete] │ │ NoteCard (Pinned)
│ │ This is a frontend-only Compose Multiplatform...   │ │
│ └────────────────────────────────────────────────────┘ │
│                                                        │
│ OTHERS                                                 │ Category Header
│ ┌────────────────────────────────────────────────────┐ │
│ │ Architecture Overview              [Pin]  [Delete] │ │ NoteCard
│ │ Each feature cleanly separates files into...       │ │
│ └────────────────────────────────────────────────────┘ │
│                                                        │
│                                                        │
│                                                 ┌────┐ │
│                                                 │ +  │ │ FAB (Add Note)
│                                                 └────┘ │
└────────────────────────────────────────────────────────┘
```

### 9.2. Note Detail / Editor Screen Layout (ASCII Wireframe)

```text
┌────────────────────────────────────────────────────────┐
│ [< Back]  Edit Note            [Pin] [Delete]  [Save]  │ TopAppBar
├────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────┐ │
│ │ Title: Architecture Overview                       │ │ Title TextField
│ └────────────────────────────────────────────────────┘ │
│                                                        │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Content:                                           │ │
│ │                                                    │ │
│ │ Each feature cleanly separates files into:         │ │
│ │ - Screen (UI composables)                          │ │ Multiline Content
│ │ - Intent (User actions)                            │ │ TextField
│ │ - UiState (Immutable view state)                   │ │
│ │ - ViewModel (Inheriting from BaseViewModel)        │ │
│ │                                                    │ │
│ └────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

---

## 10. Verification & Quality Assurance Strategy

### 10.1. Testing Architecture
* **Unit Testing Domain & ViewModels**: ViewModels accept an optional `coroutineScope` parameter (defaulting to `viewModelScope`). In unit tests, an unconfined or background test dispatcher allows deterministic, synchronous testing without requiring Android `MainLooper` mocking.
* **Repository Verification**: Thread-safe validation of CRUD invariants, sorting criteria (pinned priority), and flow emissions.
* **Build Targets Validated**:
  * Android compilation and Host tests: `./gradlew :shared:testAndroidHostTest`
  * Android application packaging: `./gradlew :androidApp:assembleDebug`
  * Shared target compilation: `./gradlew :shared:assemble`

---

## 11. Summary of Architectural Adherence

| Requirement | Architectural Solution | Implementation Location |
| :--- | :--- | :--- |
| **MVVM Architecture** | Model-View-ViewModel with reactive `StateFlow` | `shared/src/commonMain/kotlin/com/example/myapplication/viewmodel/` |
| **Inherited ViewModel** | Generic `BaseViewModel<S, I>` inheriting AndroidX `ViewModel` | `viewmodel/BaseViewModel.kt` |
| **Separated Screen** | Stateless Compose Multiplatform composables | `ui/notelist/NoteListScreen.kt`, `ui/notedetail/NoteDetailScreen.kt` |
| **Separated Intent** | Sealed action interfaces for every user interaction | `viewmodel/notelist/NoteListIntent.kt`, `viewmodel/notedetail/NoteDetailIntent.kt` |
| **Separated UIState** | Immutable data classes with computed state helpers | `viewmodel/notelist/NoteListUiState.kt`, `viewmodel/notedetail/NoteDetailUiState.kt` |
| **Actual System Design** | Full Notes CRUD domain, repository, navigation, and UX | `model/Note.kt`, `data/NotesRepository.kt`, `App.kt` |
