# AgentCheck

## One-Sentence Summary

AgentCheck is an Entire-native evidence layer that turns checkpointed AI coding sessions into auditable context, verification evidence, and reviewable provenance without treating missing or redacted checkpoint data as authoritative.

## Problem / User

AI coding agents can produce useful changes, but reviewers often lack a durable answer to "why did this change happen, what evidence supports it, and what verification actually ran?" AgentCheck is for maintainers, reviewers, security-conscious teams, and buildathon judges who need to inspect agent-produced work through source-linked evidence instead of trusting a chat summary.

The key user need is confidence under imperfect context. Real checkpoints can be incomplete, redacted, unavailable, or disconnected from a commit. AgentCheck must still help locally while clearly showing when evidence is incomplete.

## Selected Track

Selected track: BTW Buildathon Track 1, using Entire checkpoints and Entire Graph.

Entire is essential because AgentCheck depends on data that ordinary Git history does not carry:

- Session and checkpoint identity.
- Prompt and transcript provenance.
- Files touched by an agent session.
- Commit-to-checkpoint association through `Entire-Checkpoint` trailers and checkpoint metadata.
- Checkpoint inspection through `entire checkpoint explain`.
- Entire Graph impact/source analysis for code-path validation.

Without Entire, AgentCheck could inspect commits, but it could not tie a change back to the agent session, transcript, checkpoint, and source graph evidence that explain why the change exists.

## Architecture / Workflow

The implemented AgentCheck boundary is:

`Entire checkpoint/session readers -> agentcheck.Builder -> agentcheck.Context -> verification/evidence/reporting surfaces`

Implemented pieces:

- A2 context adapter in `cmd/entire/cli/agentcheck`.
- AgentCheck-owned context types instead of direct downstream reads from checkpoint storage internals.
- Production repository entry point via `agentcheck.BuildFromRepository`.
- Checkpoint/session reads through existing Entire checkpoint reader APIs.
- Git evidence association through `Entire-Checkpoint` trailers and checkpoint commit anchors.
- Changed-file and diff evidence from Git.
- Optional Graph evidence through a provider interface.
- Read-only AgentCheck CLI/status/open surfaces for existing evidence bundles and reports.
- Verification evidence execution and persistence under `.agentcheck`.

The workflow is:

1. Entire captures an AI coding session as a checkpoint.
2. A commit links to the checkpoint through Entire metadata and, when available, an `Entire-Checkpoint` trailer.
3. AgentCheck builds a stable context from checkpoint, session, Git, and optional Graph evidence.
4. Verification evidence is collected and persisted.
5. Downstream evaluators consume AgentCheck context and must honor its evidence completeness state.

## Entire Checkpoint Usage

AgentCheck uses Entire checkpoints as the provenance source of truth. It reads checkpoint summaries, session metadata, prompt text when available, transcript availability, files touched, token/model metadata, and checkpoint-associated commit evidence.

Important checkpoint IDs for this submission:

- Pre-Curveball checkpoint: `01M1TXD79GSQ2NHF970KQFGQGA`
- Final Curveball checkpoint: `01M1TZN957CYHB8H1FG8KFK2C7`

What they prove:

- `01M1TXD79GSQ2NHF970KQFGQGA` proves the pre-curveball intent, architecture, completed work, risks/assumptions, and verification state before the privacy-boundary requirement was introduced.
- `01M1TZN957CYHB8H1FG8KFK2C7` proves the completed Curveball session, including the new privacy requirement, invalidated assumption, revised design, implementation, Graph analysis, tests, verification, and remaining limitations.

The final checkpoint was recovered using Entire's native after-the-fact session attachment mechanism:

```powershell
entire session attach 01a075d9-f3f2-7a82-ad26-9cd7f835d8cc --agent codex
```

The attach created checkpoint `01M1TZN957CYHB8H1FG8KFK2C7` without modifying or rewriting the final commit.

## Graph Findings And Source/Test Verification

Entire Graph was used as the mandated code-navigation and impact-analysis tool for the AgentCheck work.

Observed Graph state:

- `entire graph capabilities --json` confirmed Go semantic relations were available.
- `entire graph search --repo . --profile full --query "AgentCheck checkpoint privacy boundary completeness redacted transcript prompt context verification CLI"` was attempted.
- Narrower Graph search and Graph impact attempts were also attempted.
- The bounded Graph search/impact attempts produced no output within the allotted waits and were interrupted.

Because Graph search/impact did not return usable results in time, the implementation was verified by focused source tracing:

- `checkpoint.ReadCheckpoint` / `ReadSessionContent`
- `agentcheck.Builder.addSessions`
- `Context`, `Session`, `Prompt`, `TranscriptRef`, `GitEvidence`, `GraphContext`, and `Provenance`
- `runAgentCheckOrchestration`
- Current AgentCheck CLI `status` / `open` consumers

Verification performed:

```powershell
gofmt
go test ./cmd/entire/cli/agentcheck/...
go build ./cmd/entire/cli/agentcheck/...
go test ./cmd/entire/cli -run AgentCheck
```

All focused AgentCheck verification passed for the completed work.

## Curveball Adaptation

Curveball: Track 1 Privacy Boundary.

The Curveball invalidated the assumption that AgentCheck could preserve raw prompt/transcript strings and treat missing values as ordinary empty optional fields.

Revised design principle:

Missing is not empty. Redacted is not absent. Unavailable is not verified. Incomplete is not authoritative.

Implemented adaptation:

- Added explicit evidence states: `complete`, `redacted`, and `unavailable`.
- Added `EvidenceStatus` across prompt, transcript, Git, Graph, provenance, and aggregate completeness.
- Added context-level `Completeness`.
- Added prompt-level `PromptEvidence`.
- Marked redacted prompt/transcript evidence as `redacted`.
- Marked missing prompt/transcript evidence as `unavailable`.
- Kept Graph evidence optional and non-fatal.
- Prevented Graph requests from receiving raw prompts or transcripts.
- Ensured incomplete context can still support local checks without being presented as authoritative.

Privacy boundary:

- Raw prompt and transcript strings stay in the local AgentCheck context-building path.
- `GraphRequest` carries checkpoint ID and file lists only.
- Redacted or missing evidence makes the aggregate context incomplete.

## Implemented State

Final implementation commit:

```text
82e11d44d831e490ec48eb79c81042dd6c676606 feat(agentcheck): enforce checkpoint privacy boundary
```

Remote branch:

```text
agentcheck/curveball-privacy
```

Pull request:

```text
https://github.com/LakshyaChauhan304/AgentCheck/pull/2
```

Relevant milestone commits:

- `45b097d3b` - A2 context adapter foundation.
- `7ce230580` - AgentCheck verification/evidence/CLI integration.
- `82e11d44d831e490ec48eb79c81042dd6c676606` - Privacy Boundary Curveball implementation.

The final Curveball commit changes:

- `AGENTCHECK_ARCHITECTURE.md`
- `AGENTCHECK_CODEBASE.md`
- `AGENTCHECK_PROGRESS.md`
- `AGENTCHECK_STAGE_LOG.md`
- `cmd/entire/cli/agentcheck/context.go`
- `cmd/entire/cli/agentcheck/context_builder.go`
- `cmd/entire/cli/agentcheck/context_builder_test.go`

Commit stats:

```text
7 files changed, 338 insertions(+), 40 deletions(-)
```

## Setup / Run / Test Instructions

Install project dependencies using the repository's normal development setup.

```powershell
mise install
mise trust
```

Build the CLI:

```powershell
mise run build
```

Run the focused AgentCheck checks:

```powershell
go test ./cmd/entire/cli/agentcheck/...
go build ./cmd/entire/cli/agentcheck/...
go test ./cmd/entire/cli -run AgentCheck
```

Inspect the final checkpoint:

```powershell
entire checkpoint explain 01M1TZN957CYHB8H1FG8KFK2C7
```

Inspect the pull request branch:

```powershell
git fetch origin
git ls-remote origin refs/heads/agentcheck/curveball-privacy
git show --stat --oneline 82e11d44d831e490ec48eb79c81042dd6c676606
```

## Databricks

Databricks was not used for this submission. This project is an Entire CLI / Entire Graph submission focused on checkpoint provenance, evidence completeness, verification evidence, and privacy-boundary behavior.

## Limitations / Next Steps

Current limitations:

- AgentCheck evaluators and scoring are not fully implemented in this stage.
- The current CLI surfaces are read-only/status-oriented for evidence and report availability.
- Graph evidence is optional and may be unavailable without failing local context construction.
- Graph search/impact commands were attempted but did not produce usable output within bounded waits during final Curveball verification.
- Codex hook trust remains `trust_review_needed` for this working tree until the user approves the configured hooks.
- `entire checkpoint list --json` returned an empty local list with a remote warning, although `entire checkpoint explain 01M1TZN957CYHB8H1FG8KFK2C7` is readable by exact ID.
- Downstream evaluators must be implemented to respect non-`complete` aggregate context and avoid presenting incomplete evidence as authoritative.

Next steps:

- A3 Git and Graph evidence hardening.
- Evaluator implementation that consumes `agentcheck.Context`.
- Human-readable AgentCheck reports that make missing, redacted, unavailable, and complete evidence visible.
- Expanded end-to-end tests over real checkpoint/session scenarios.

## Submission Checklist

- Project name: AgentCheck.
- One-sentence summary: included.
- Problem/user: included.
- Selected Entire track and why Entire is essential: included.
- Architecture/workflow: included.
- Entire checkpoint usage: included.
- Graph findings and source/test verification: included.
- Curveball adaptation: included.
- Checkpoint IDs and what each proves: included.
- Setup/run/test instructions: included.
- Databricks use: explicitly not applicable.
- Limitations/next steps: included.
- A2 context adapter state: included.
- C verification/evidence/CLI state: included.
- Privacy Boundary Curveball state: included.
- Pre-Curveball checkpoint: included.
- Fresh session: `01a075d9-f3f2-7a82-ad26-9cd7f835d8cc`.
- Final Curveball checkpoint: included.
- Final commit: included.
- Pull request: included.
