# AgentCheck Stage Log

## A2 - Entire Checkpoint to AgentCheckContext

Owner: Teammate A

Goal: Build the context foundation that adapts Entire checkpoint/session/Git/Graph evidence into a stable AgentCheck contract.

Implemented:
- Added `Context`, `Checkpoint`, `Session`, `Prompt`, `GitEvidence`, `GraphContext`, and provenance structs.
- Added `Builder.Build` to read checkpoint summaries and session content through existing Entire APIs.
- Added `BuildFromRepository` to open the real Entire checkpoint facade for a repository root.
- Preserved raw developer prompts and subsequent scoped prompts.
- Preserved task checkpoint markers when session metadata exposes them.
- Associated Git commits by scanning `Entire-Checkpoint` trailers and honoring checkpoint `CommitSHA` anchors when present.
- Collected changed files with `gitops.DiffTreeFileList` and patch evidence with `git show`.
- Added optional Graph provider interface, `GraphCLIProvider` for `entire graph checkpoint`, and graceful unavailable state.
- Added focused tests for the A2 required cases.

Verification:
- `entire graph search --repo . --profile full --query "AgentCheck checkpoint context adapter using Entire checkpoint session git trailer graph evidence APIs"` was run with the repaired PATH but returned no output within a bounded wait and was interrupted.
- `gofmt` completed for AgentCheck Go files.
- `go test ./cmd/entire/cli/agentcheck/...` passed.
- `go build ./cmd/entire/cli/agentcheck/...` passed.

Scope deviations:
- No evaluation, trust scoring, verification runner, HTML report, or final CLI was implemented.
- No checkpoint storage, Codex hooks, transcript collectors, or Entire internals were modified.

## Track 1 Curveball - Checkpoint Privacy Boundary

Owner: Curveball implementation

Goal: Keep AgentCheck useful when checkpoint prompt/transcript evidence is redacted or unavailable, without sending raw prompts or transcripts to any new external service.

Invalidated assumption:
- AgentCheck could treat empty prompt/transcript values as simply empty optional data while preserving raw prompt strings for later evaluators.

New design principle:
- Missing != empty, redacted != absent, unavailable != verified, and incomplete != authoritative. Every prompt, transcript, Git, and Graph evidence path must carry explicit state: `complete`, `redacted`, or `unavailable`.

Graph impact analysis:
- `entire graph capabilities --json` confirmed Go semantic relations are supported.
- `entire graph search --repo . --profile full --query "AgentCheck checkpoint privacy boundary completeness redacted transcript prompt context verification CLI"` and narrower `graph search`/`graph impact` attempts produced no output within bounded waits and were interrupted.
- Source/test verification therefore traced the actual affected path: `checkpoint.ReadCheckpoint` and `ReadSessionContent` -> `agentcheck.Builder.addSessions` -> `Context`, `Session`, `Prompt`, `TranscriptRef`, `GitEvidence`, `GraphContext`, and `Provenance` -> `runAgentCheckOrchestration` and current AgentCheck CLI status/open consumers.

Implemented:
- Added `EvidenceState` and `EvidenceStatus` to represent `complete`, `redacted`, and `unavailable` evidence.
- Added context-level `Completeness` and prompt-level `PromptEvidence`.
- Added prompt, transcript, Git, Graph, and provenance state.
- Redacted prompt/transcript content is marked `redacted`; missing prompt/transcript content is marked `unavailable`.
- Graph provider requests continue to receive checkpoint ID and file lists only, not raw prompt or transcript text.
- Graph unavailable remains non-fatal and marks context incomplete rather than failing local functionality.

Verification:
- `gofmt` completed for AgentCheck Go files.
- `go test ./cmd/entire/cli/agentcheck/...` passed.
- `go build ./cmd/entire/cli/agentcheck/...` passed.
- `go test ./cmd/entire/cli -run AgentCheck` passed.
