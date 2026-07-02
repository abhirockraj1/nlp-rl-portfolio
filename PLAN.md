# Coding RAG Application Roadmap

## Summary
Build a local workstation application that connects to one GitHub repo, indexes code/docs/tests, and supports repo-aware coding work through chat, search, code generation, patch suggestions, and safe test execution.

The app includes:
- React + TypeScript web UI.
- Python FastAPI backend.
- Typer CLI.
- SQLite metadata store.
- Chroma vector store.
- ripgrep keyword search.
- tree-sitter symbol indexing.
- LLM provider adapter for OpenAI API and Ollama/local models.
- Plugin-ready tool slots for Jira, GitHub, Confluence, CI, and future tools.

v1 supports one repo at a time. It can generate code and propose patches, but it does not directly edit, commit, push, or open PRs.

## Milestone 1: Project Foundation
Goal: create the runnable app skeleton.

Deliverables:
- App structure:
  - `backend/`: FastAPI service.
  - `frontend/`: React + TypeScript + Vite.
  - `cli/`: Typer CLI.
  - `storage/`: local SQLite + Chroma data.
- Config system for:
  - repo workspace path
  - OpenAI config
  - Ollama/local model config
  - indexing exclusions
  - allowed test commands
- Backend health endpoint.
- CLI command: `coding-rag doctor`.
- Frontend shell with navigation:
  - Repo
  - Chat
  - Search
  - Code Generator
  - Patches
  - Tests
  - Tools

Acceptance:
- Backend starts locally.
- Frontend connects to backend.
- CLI validates Git, ripgrep, Python, Node, OpenAI config, and Ollama availability.

## Milestone 2: Repo Ingestion
Goal: connect the app to one GitHub repo or local repo path.

Deliverables:
- Add repo from GitHub URL or local path.
- Clone/fetch repo into managed workspace.
- Store repo metadata in SQLite:
  - repo URL
  - local path
  - branch
  - commit SHA
  - index status
  - last indexed time
- File discovery with exclusions:
  - `.git`
  - build outputs
  - IDE files
  - caches
  - test reports
  - binaries
  - credentials/secrets
  - large generated files
- CLI commands:
  - `coding-rag repo add <url-or-path>`
  - `coding-rag repo status`
  - `coding-rag repo refresh`

Acceptance:
- User can register one repo.
- App shows branch, commit SHA, file count, and index status.
- Excluded folders/files are not indexed.

## Milestone 3: Indexing + Retrieval
Goal: make the repo searchable by meaning, keyword, path, and symbol.

Deliverables:
- Code/document chunking for:
  - Groovy
  - Java
  - Python
  - JS/TS
  - Markdown
  - YAML
  - Gradle files
- tree-sitter symbol extraction where supported.
- SQLite symbol table with:
  - file path
  - language
  - package/class/function/method name
  - symbol type
  - imports
  - line range
  - commit SHA
- Chroma vector index for semantic search.
- ripgrep-backed keyword search for exact strings, class names, errors, and config keys.
- Hybrid retriever combining:
  - exact symbol match
  - keyword match
  - vector similarity
  - file path/name heuristics
  - test/source path mapping
- CLI commands:
  - `coding-rag index`
  - `coding-rag find "<query>"`

Acceptance:
- User can find exact classes/methods.
- User can search natural language like “where is login validation implemented?”
- Results include file paths, line ranges, and ranked context.

## Milestone 4: Repo Q&A + Chat Code Generator
Goal: let users ask repo questions and generate repo-aware code from chat.

Deliverables:
- Chat intent router with modes:
  - `explain_code`
  - `find_implementation`
  - `find_tests`
  - `debug_error`
  - `generate_code`
  - `modify_existing_code`
  - `write_test`
- LLM provider interface:
  - `OpenAIProvider`
  - `OllamaProvider`
- Chat API endpoints:
  - `POST /chat`
  - `POST /generate-code`
  - `POST /suggest-tests`
  - `POST /patch/draft`
- CLI commands:
  - `coding-rag ask "<question>"`
  - `coding-rag generate "<request>"`
  - `coding-rag testgen "<class-or-feature>"`
- Chat UI with:
  - answer area
  - mode badge
  - cited files/line ranges
  - retrieved context preview
  - generated code blocks
  - proposed files
  - suggested tests
  - “Promote to Patch” action

Code generator behavior:
- For new code:
  - Return complete code blocks.
  - Suggest exact file paths.
  - Explain imports/dependencies.
- For existing code changes:
  - Prefer unified diff draft.
  - Show affected files.
  - Do not write to the repo directly.
- For test generation:
  - Find similar tests first.
  - Match existing framework and naming conventions.
  - Return test code plus command to run it.
- For unclear requests:
  - Ask one concise follow-up only when required.
  - Otherwise generate the safest minimal implementation.

Acceptance:
- User can ask repo questions and receive cited answers.
- User can ask “create a helper for X” and receive repo-aware code.
- User can ask “add a test for this behavior” and receive matching test code.
- User can ask “modify this implementation” and receive a unified diff draft.
- Generated output cites repo context and avoids invented internal APIs.

## Milestone 5: Patch Suggestions + Validation
Goal: convert chat-generated code into validated patch suggestions without touching the main checkout.

Deliverables:
- Coding task flow:
  - retrieve relevant context
  - generate implementation plan
  - generate unified diff
  - validate diff applies
- Temporary worktree manager.
- Patch validation:
  - apply patch in temp worktree
  - capture apply errors
  - run suggested tests when requested
- Patch Review UI:
  - summary
  - changed files
  - diff viewer
  - risks/assumptions
  - suggested tests
  - validation status
- CLI command:
  - `coding-rag patch "<task>"`

Acceptance:
- User can promote generated code to patch validation.
- App returns a valid unified diff when possible.
- Main repo checkout remains untouched.
- Failed patches return useful diagnostics.
- Suggested tests are preserved with the patch.

## Milestone 6: Test Execution + Tool Slots
Goal: validate changes locally and prepare for Jira/GitHub/CI integrations.

Deliverables:
- Safe command runner:
  - runs only in temp worktree
  - uses configured allowlist
  - supports timeout handling
  - captures stdout/stderr
- Test UI:
  - command
  - status
  - output
  - linked patch/task
- CLI command:
  - `coding-rag test "<command>"`
- Plugin registry:
  - `ToolAdapter` interface
  - disabled/configurable cards for Jira, GitHub, Confluence, CI
  - source-tagged external context model

Tool adapter shape:
- `name`
- `capabilities`
- `auth_config`
- `search(query)`
- `get(id)`
- optional `create`
- optional `update`
- optional `comment`

Acceptance:
- User can run configured tests locally in a temp worktree.
- Test output can be used in follow-up debugging chat.
- Jira/GitHub/Confluence/CI slots exist without blocking v1.
- External tool content is tagged by source and kept separate from repo index metadata.

## Post-MVP Roadmap
- Real GitHub adapter:
  - issue lookup
  - PR lookup
  - branch metadata
  - PR comment drafting
- Real Jira adapter:
  - ticket lookup
  - acceptance criteria retrieval
  - linked issue context
  - comment drafting
- Real Confluence adapter:
  - design doc retrieval
  - team knowledge search
- Real CI adapter:
  - failed build lookup
  - test logs as debugging context
- Multi-repo support.
- Team-server deployment with auth, job isolation, and secret management.
- Explicitly approved patch application, branch creation, commit, push, and PR creation.

## Test Plan
- Unit tests:
  - repo exclusion rules
  - GitHub URL parsing
  - local path registration
  - file chunking
  - symbol extraction
  - SQLite metadata writes
  - Chroma indexing wrapper
  - hybrid retrieval ranking
  - provider adapter behavior
  - chat intent routing
  - tool adapter registry

- Integration tests:
  - index a small fixture repo
  - find a class/method implementation
  - find likely tests for a source file
  - answer code explanation question with citations
  - generate new code from chat
  - generate test code from chat
  - generate a patch and verify it applies in a temp worktree
  - run a safe test command and capture output

- Acceptance scenarios:
  - User can add a GitHub repo and index it locally.
  - User can ask “where is this implemented?” and receive cited files/lines.
  - User can ask “which tests cover this?” and receive likely tests plus commands.
  - User can request new code from chat and receive repo-aware generated code.
  - User can request a coding change and receive a valid unified diff.
  - User can validate a patch in a temporary worktree.
  - User can choose OpenAI or Ollama in config.
  - Jira/GitHub/Confluence/CI plugin slots exist without requiring integrations in v1.

## Assumptions
- v1 supports one repo at a time.
- The app runs locally on a developer workstation.
- Web app and CLI are both included.
- OpenAI API and Ollama/local models are both supported through one provider abstraction.
- Chat-based code generation is included in Milestone 4.
- Patch application and validation are handled in Milestone 5.
- The app suggests patches but does not directly edit, commit, push, or create PRs.
- Tests run only in temporary worktrees.
- Jira and other external tools are reserved through plugin interfaces, not fully implemented in v1.
