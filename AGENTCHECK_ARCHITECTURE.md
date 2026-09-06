# AgentCheck Architecture

## A2 Boundary

AgentCheck uses Entire as the source of truth for checkpoint and session context.

The current boundary is:

`Entire checkpoint/session readers -> agentcheck.Builder -> agentcheck.Context -> future evaluators`

Future evaluators should depend on `cmd/entire/cli/agentcheck.Context`, not on checkpoint storage paths, refs, branches, or Codex hook internals.

Production callers can use `agentcheck.BuildFromRepository` to open the existing Entire checkpoint facade for a repository root. Tests and future CLI wiring can still inject a `Reader` directly into `Builder` when they need a narrower surface.

## Privacy Boundary

AgentCheck context now carries explicit evidence state for prompt, transcript, Git, Graph, provenance, and aggregate completeness.

Evidence states are:

- `complete`: evidence was available with no detected redaction marker.
- `redacted`: evidence was available but contains redaction markers.
- `unavailable`: evidence was missing, unreadable, not configured, or otherwise unavailable.

Downstream evaluators and reports must treat any non-`complete` aggregate context as incomplete. They may still run local checks, but must not present incomplete context as authoritative.

## Context Sources

- Checkpoint metadata comes from `checkpoint.CheckpointReader`.
- Session metadata, prompt text, and transcript bytes come from `checkpoint.SessionReader`.
- Prompt splitting uses `checkpoint.SplitPromptContent`.
- Commit association uses `trailers.ParseAllCheckpoints` against commit messages, plus checkpoint `CommitSHA` anchors when present.
- Changed file evidence uses `gitops.DiffTreeFileList`.
- Diff evidence uses `git show` against associated commits.
- Graph evidence is optional and injected through `GraphProvider`; `GraphCLIProvider` uses `entire graph checkpoint <id> --json` when configured by a caller.
- `GraphRequest` contains checkpoint ID and file lists only. It does not include raw prompt text or transcript bytes.

## Unavailable Evidence

Missing optional evidence is represented explicitly with evidence state and reasons. The builder does not fabricate prompts, transcripts, commits, diffs, task records, or Graph results. Missing evidence is `unavailable`, not an empty complete value; redacted evidence is `redacted`, not authoritative complete context.

## Out Of Scope

A2 does not include evaluation rules, trust verdicts, verification runners, reports, or command presentation.
