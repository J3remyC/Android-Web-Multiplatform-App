# Project Instructions for AI Agents

These instructions apply throughout this repository. Follow them for every user prompt, including follow-up questions, reviews, planning, and documentation-only tasks, subject to higher-priority system, developer, and user instructions.

## Before every prompt: mandatory reading

Before analyzing the request, inspecting implementation files, editing, running checks, or providing a substantive answer:

1. Read this root `AGENTS.md` from disk first, even if it was already loaded or read earlier in the conversation.
2. Read the entire `docs/PROJECT-OVERVIEW.md`.
3. Read the entire `docs/design/DESIGN.MD` and root `design.md`.
4. Read `docs/templates/AI-LOGS.md` and any other templates applicable to the requested work under `docs/templates/`.
5. Read any additional documentation and existing AI logs relevant to the task, and any directory-specific agent instructions that apply to the files being changed.

Repeat this reading sequence for each new user prompt, including within an existing session and after context compaction. Do not rely on memory, an earlier summary, or cached document contents. If output is truncated, read the remaining content in chunks before proceeding.

If a required document is missing or unreadable, report the specific problem; do not claim to have read it or invent its contents. Continue only with work that does not depend on the missing information.

## Documentation and design alignment

- Use `docs/PROJECT-OVERVIEW.md` for product scope, platform requirements, privacy guarantees, and the project structure.
- Use `docs/design/DESIGN.MD` and `design.md` for architecture, feature responsibilities, state flow, and UI requirements. These two design files are identical when these instructions are introduced. If they diverge, identify the discrepancy and resolve it using the user's instructions; do not silently choose a conflicting specification.
- When authorized changes affect documented behavior or architecture, update the relevant documentation and keep both design copies consistent.
- Read and follow the applicable templates before creating their corresponding artifacts. Preserve required headings, fields, naming rules, and locations; replace placeholders with actual information or an explicit `Not applicable` and reason.
- Distinguish specifications, examples, roadmap items, and verified implementation. Inspect actual source and build configuration before claiming a feature, dependency, storage mechanism, or Gradle task is available.

## Project requirements

- Build a frontend-only Notes application with Kotlin Multiplatform and Compose Multiplatform for Android, Web (Wasm), and Web (JS).
- Keep notes, search, CRUD operations, and persistence local. Preserve offline operation and zero telemetry; do not introduce remote APIs, backend services, cloud storage, or outbound requests for note management or analytics.
- Put shared domain logic and Compose UI in `shared/src/commonMain/`. Keep Android and browser implementations in their respective platform source sets, with lightweight launchers in `androidApp/` and `webApp/`.
- Follow MVVM with unidirectional data flow: Screen dispatches Intent, ViewModel processes Intent and coordinates Repository operations, and immutable UIState flows back to Screen.
- Keep each feature's `Screen`, `Intent`, `UiState`, and `ViewModel` in separate files. Feature ViewModels inherit from `BaseViewModel<S, I>`, which inherits from AndroidX `ViewModel`.
- Screens use Material 3 Compose components and remain stateless with respect to business logic; they must not call repositories directly.
- Expose read-only `StateFlow` from ViewModels, update immutable state atomically, and use sealed interfaces for intents and lifecycle-aware coroutine scopes.
- Keep `NotesRepository` as the shared CRUD contract. Preserve note IDs, creation and modification timestamps, pin status, and optional color values. Keep pinned notes first and use the documented sorting behavior.
- Keep shared code compatible with all targets; avoid Android-only, JVM-only, or browser-only APIs in `commonMain`.
- Treat the in-memory repository as demonstration storage. Do not describe it as durable storage across app or browser restarts.

## Verification

Choose checks appropriate to the actual change. For documentation-only changes, verify paths, instructions, and template compliance; application builds are not required.

For code changes, use relevant tests and build checks from the project documentation after confirming the task exists. On Windows use `.\gradlew.bat`; on other systems use `./gradlew`.

Documented tasks include:

- Shared tests: `:shared:allTests`
- Android host tests: `:shared:testAndroidHostTest`
- Web tests: `:shared:wasmJsTest` and `:shared:jsTest`
- Android debug build: `:androidApp:assembleDebug`
- Shared assembly: `:shared:assemble`
- Web development runs: `:webApp:wasmJsBrowserDevelopmentRun` and `:webApp:jsBrowserDevelopmentRun`

Record commands actually executed and their results. Mark skipped or unavailable checks explicitly with the reason. Never copy example test results from documentation as evidence of current verification.

## After every prompt: mandatory AI log

After completing the work for each user prompt and before sending the final response:

1. Create a completed log in `docs/ai-logs/` using the current `docs/templates/AI-LOGS.md`. This requirement applies even to questions, reviews, planning, and documentation-only work with no application changes.
2. Use the filename `MM-DD-YYYY - <Log Title>.md` and heading `# [MM/DD/YYYY] - <Log Title>`, using the actual local date. Use a descriptive title and a numeric suffix if needed to avoid overwriting another log.
3. Include the template's Metadata and all seven numbered sections, in order:
   - Objective & Context
   - Prompts & AI Interaction
   - Technical & Architectural Decisions
   - Implementation & Code Changes
   - Verification & Testing
   - Challenges, AI Limitations & Learnings
   - Next Steps & Follow-ups
4. Record the request, references read, decisions, files changed, verification results, limitations, and remaining work. For tasks without application changes, state that explicitly. Do not invent author identity, exact model version, test outcomes, or platform validation; mark unknown or inapplicable fields honestly.
5. Use `Completed`, `In Progress`, or `Blocked` according to actual task status. Preserve existing logs and keep secrets and private note contents out of logs.
6. Check the finished log against the template. Mention the log path in the final response. If writing the log is blocked, report the failure explicitly.

Do not skip logging because the task was small. Creating the log does not itself require a second log.

## Instruction loading

The uppercase root filename `AGENTS.md` is intentional so supporting tools can discover it. Codex discovers guidance at startup; the per-prompt rereading above is an explicit agent instruction, not a runtime hook. Start a new session after adding or changing this file if the current client has not loaded it. Other AI clients must be configured to load this file if they do not support `AGENTS.md` discovery.
