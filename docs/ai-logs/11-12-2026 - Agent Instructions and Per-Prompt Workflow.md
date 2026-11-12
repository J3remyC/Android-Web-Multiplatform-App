# [11/12/2026] - Agent Instructions and Per-Prompt Workflow

## Metadata
* **Date**: 11/12/2026
* **Author / Developer**: Requesting project owner; name not supplied in the request.
* **AI Tool & Model**: Codex (GPT-6-based agent; exact model version not exposed).
* **Task / Issue Reference**: Read the entire docs directory and create root agent instructions requiring per-prompt reading and template-based AI logs.
* **Target Platform(s)**: [ ] Android  [ ] Web (Wasm)  [ ] Web (JS)  [ ] Shared (commonMain). Not applicable: repository documentation and AI workflow only.
* **Status**: Completed

---

## 1. Objective & Context
* **Goal**: Establish persistent repository instructions for reading project context before every prompt and logging work afterward.
* **Context**: The user requested an agents.md based on all existing documentation, with mandatory project overview and design reading and adherence to templates.
* **Prerequisites & References**: Read all existing files under `docs/`: `docs/PROJECT-OVERVIEW.md`, `docs/design/DESIGN.MD`, and `docs/templates/AI-LOGS.md`. Also read root `design.md`; SHA-256 comparison confirmed both design copies are identical. Consulted the OpenAI Docs skill and [official OpenAI documentation for AGENTS.md discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

---

## 2. Prompts & AI Interaction
* **Key Prompts**:
  1. "Read the entire docs dierctory and make me an agents.md"
  2. "make it so that every prompt the ai reads agents.md first"
  3. "before every prompt read the project overview, the design.md and after write the ai-logs and follow the templates"
* **AI Responses & Strategy**: Read all documentation, establish a mandatory per-prompt reading order, summarize architectural requirements, and require a completed log before each final response.
* **Guidance & Course Corrections**: Included both existing design paths to cover the root file named by the user and the design document referenced by the project overview.

---

## 3. Technical & Architectural Decisions
* **Decisions Made**: Use root `AGENTS.md` with standard uppercase spelling. Require fresh reads for follow-up prompts and documentation-only tasks. Require logs for every prompt and retain the existing template structure and naming convention.
* **Rationale & Trade-offs**: Standard naming supports instruction discovery. Explicit rereading instructions cover subsequent prompts, while avoiding a claim that a Markdown file provides runtime enforcement.
* **Platform Considerations**: Preserve Android, Wasm, JS, shared-code boundaries, local-only storage, and the documented MVVM/UDF architecture. No runtime changes.

---

## 4. Implementation & Code Changes
* **Summary of Changes**: Added repository agent instructions and this template-based record. No application code changed.
* **File Changes**:
  * `[NEW]` `AGENTS.md`: Mandatory reading order, documentation alignment, architecture rules, verification guidance, per-prompt logging, and discovery limitations.
  * `[NEW]` `docs/ai-logs/11-12-2026 - Agent Instructions and Per-Prompt Workflow.md`: Log of this request.
* **Key Code Snippets / Details**: Not applicable; documentation-only changes.

---

## 5. Verification & Testing
* **Automated Tests Executed**: Application tests and builds not run; no application code changed. PowerShell structural checks verified required document paths, log headings, and the reading/logging instructions.
* **Manual Verification**: Read the complete existing docs directory, reread the added instructions and log, and compared the log against the template. Confirmed the two design copies have matching SHA-256 hashes.
* **Results & Observations**: Required paths exist; all seven template sections and metadata are present. Fresh-client instruction discovery and future agent compliance have not been exercised.

---

## 6. Challenges, AI Limitations & Learnings
* **Issues / Hallucinations**: Initial combined documentation output was truncated. Both a root and docs design file exist. Automatic discovery occurs at startup rather than before each message.
* **Resolution**: Reread the documentation in smaller complete outputs, confirmed matching design hashes, and documented explicit per-prompt rereading and client discovery limits.
* **Key Learnings**: Keep agent instructions concise and reference complete specifications. A repository instruction file depends on the AI client loading it and the agent following it.

---

## 7. Next Steps & Follow-ups
* [ ] Start a new AI session to verify automatic discovery of root `AGENTS.md`.
* **Known Limitations / Tech Debt**: No runtime hook was added. Clients without AGENTS.md support require configuration to load the file. Both design copies must remain consistent when future design edits are authorized.
