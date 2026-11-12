# AI Work Log Template

This template defines the standard format and guidelines for documenting AI-assisted development sessions across the project. Use this template to log prompts, architectural decisions, code changes, verification steps, and AI-related findings.

---

## Guidelines & Conventions

### 1. File Naming Convention
* **Standard Guideline**: `File Naming Convention - MM/DD/YYYY - Log Title`
* **Filesystem-Safe Naming**: Because forward slashes (`/`) are reserved path separators across operating systems (Windows, Linux, macOS), log files must be named using hyphens for the date component:
  ```text
  MM-DD-YYYY - <Log Title>.md
  ```
  *Example*: `11-12-2026 - Notes Repository and Model Setup.md`
* **Document Heading**: The Markdown title inside the log file can use either format:
  ```markdown
  # [MM/DD/YYYY] - <Log Title>
  ```
  or
  ```markdown
  # MM-DD-YYYY - <Log Title>
  ```

### 2. File Location
* Save all completed AI logs in the `docs/ai-logs/` directory.

### 3. When to Log
* Implementing new features, components, or CRUD operations.
* Architectural changes, refactoring, or state management adjustments.
* Debugging cross-platform issues (Android, Web/Wasm, Web/JS).
* Introducing or updating libraries, dependencies, or build configurations.
* Extended problem-solving sessions involving AI assistants (e.g., Junie, Copilot, Claude, ChatGPT).

---

## AI Log Template

Copy and paste the template below into a new file inside `docs/ai-logs/` and fill in the sections.

```markdown
# [MM/DD/YYYY] - <Log Title>

## Metadata
* **Date**: MM/DD/YYYY
* **Author / Developer**: <Name / Handle>
* **AI Tool & Model**: <e.g., Junie (gemini-3.7-flash), Claude 3.7 Sonnet, ChatGPT 4o, GitHub Copilot>
* **Task / Issue Reference**: <Link or brief description of the task or ticket>
* **Target Platform(s)**: [ ] Android  [ ] Web (Wasm)  [ ] Web (JS)  [ ] Shared (commonMain)
* **Status**: [Completed | In Progress | Blocked]

---

## 1. Objective & Context
* **Goal**: <Brief description of what you aimed to accomplish in this session>
* **Context**: <Background context, problem statement, or requirements>
* **Prerequisites & References**: <Relevant files, specifications, or docs consulted (e.g., `docs/PROJECT-OVERVIEW.md`)>

---

## 2. Prompts & AI Interaction
* **Key Prompts**:
  1. `<Prompt 1>`
  2. `<Prompt 2>`
* **AI Responses & Strategy**: <Summary of the approach proposed by the AI>
* **Guidance & Course Corrections**: <How you steered, corrected, or refined the AI's suggestions>

---

## 3. Technical & Architectural Decisions
* **Decisions Made**: <Key architectural choices, patterns, or data structures selected>
* **Rationale & Trade-offs**: <Why this approach was chosen over alternatives>
* **Platform Considerations**: <Cross-platform considerations for Android, Web (Wasm/JS), and shared Kotlin Multiplatform code>

---

## 4. Implementation & Code Changes
* **Summary of Changes**: <High-level overview of what was implemented>
* **File Changes**:
  * `[NEW]` `<path/to/NewFile>`: <Purpose of file>
  * `[MODIFIED]` `<path/to/ExistingFile>`: <Description of changes>
  * `[DELETED]` `<path/to/RemovedFile>`: <Reason for removal>
* **Key Code Snippets / Details**:
  ```kotlin
  // Add key code snippets here if relevant
  ```

---

## 5. Verification & Testing
* **Automated Tests Executed**:
  * Shared tests: `./gradlew :shared:allTests` -> [Pass/Fail]
  * Android host tests: `./gradlew :shared:testAndroidHostTest` -> [Pass/Fail]
  * Web tests: `./gradlew :shared:wasmJsTest` / `./gradlew :shared:jsTest` -> [Pass/Fail]
  * Build checks: `./gradlew :androidApp:assembleDebug` / `./gradlew :webApp:wasmJsBrowserDevelopmentRun` -> [Pass/Fail]
* **Manual Verification**: <Manual steps taken to test on Android emulator, device, or browser>
* **Results & Observations**: <Outcome of the tests, performance, or behavior observed>

---

## 6. Challenges, AI Limitations & Learnings
* **Issues / Hallucinations**: <Any incorrect assumptions, deprecated APIs, or platform-incompatible code suggested by the AI>
* **Resolution**: <How the issues were identified and resolved>
* **Key Learnings**: <Tips, prompts, or techniques that worked well or should be avoided in future sessions>

---

## 7. Next Steps & Follow-ups
* [ ] <Action item 1>
* [ ] <Action item 2>
* **Known Limitations / Tech Debt**: <Any items deferred for future work>
```

---

## Example Log

Below is an example of a completed AI log using this template:

```markdown
# [11/12/2026] - Notes Repository and Model Setup

## Metadata
* **Date**: 11/12/2026
* **Author / Developer**: Jane Doe (@janedoe)
* **AI Tool & Model**: Junie (gemini-3.7-flash)
* **Task / Issue Reference**: Setup core Notes domain model and repository interface
* **Target Platform(s)**: [x] Shared (commonMain)  [x] Android  [x] Web (Wasm)  [x] Web (JS)
* **Status**: Completed

---

## 1. Objective & Context
* **Goal**: Establish the core domain entity `Note` and `NotesRepository` interface in `shared/commonMain` following the specifications in `docs/PROJECT-OVERVIEW.md`.
* **Context**: The Notes application is strictly frontend-only with no remote backend. All CRUD operations must be defined through reactive Coroutines `Flow` interfaces to support client-side persistence on Android and Web.
* **Prerequisites & References**: `docs/PROJECT-OVERVIEW.md` (Sections 4 & 6).

---

## 2. Prompts & AI Interaction
* **Key Prompts**:
  1. "Create the Note data class and NotesRepository interface in shared/src/commonMain/kotlin based on docs/PROJECT-OVERVIEW.md."
  2. "Ensure that Note uses immutable fields with copy support and include timestamps in milliseconds."
* **AI Responses & Strategy**: The AI provided the `Note` data class and a clean `NotesRepository` interface with coroutines flow support.
* **Guidance & Course Corrections**: Reminded the AI to keep the repository strictly in `commonMain` without platform-specific dependencies (such as Android Context or browser window).

---

## 3. Technical & Architectural Decisions
* **Decisions Made**:
  * Used `kotlinx.coroutines.flow.Flow` for reactive query observation.
  * Used `Long` epoch timestamps (milliseconds) for `createdAt` and `updatedAt`.
* **Rationale & Trade-offs**: Keeps the domain layer completely agnostic of whether data is stored in SQLite (Android) or IndexedDB/LocalStorage (Web).
* **Platform Considerations**: Pure Kotlin commonMain code with zero platform dependencies.

---

## 4. Implementation & Code Changes
* **Summary of Changes**: Created domain model `Note` and repository contract `NotesRepository`.
* **File Changes**:
  * `[NEW]` `shared/src/commonMain/kotlin/com/example/myapplication/model/Note.kt`: Core Note entity.
  * `[NEW]` `shared/src/commonMain/kotlin/com/example/myapplication/data/NotesRepository.kt`: Repository interface.

---

## 5. Verification & Testing
* **Automated Tests Executed**:
  * Unit tests: `./gradlew :shared:allTests` -> PASSED
* **Manual Verification**: Verified code builds cleanly across both Android and Web targets without compilation errors.
* **Results & Observations**: All shared targets compiled cleanly.

---

## 6. Challenges, AI Limitations & Learnings
* **Issues / Hallucinations**: AI initially suggested using `java.util.UUID` for generating IDs, which is not available in commonMain on Kotlin Multiplatform.
* **Resolution**: Adjusted to use a multiplatform UUID or string-based ID passed into the constructor.
* **Key Learnings**: Specify target source sets (`commonMain`) explicitly in prompts to avoid platform-specific imports.

---

## 7. Next Steps & Follow-ups
* [ ] Implement in-memory or SQLite repository for Android.
* [ ] Implement browser local storage / IndexedDB repository for Web.
* [ ] Add unit tests covering repository CRUD operations.
```