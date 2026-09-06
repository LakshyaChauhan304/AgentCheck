# AgentCheck Progress

## Current Stage

Track 1 Curveball - checkpoint privacy boundary on top of A2.

## Current State

- Added the first AgentCheck context package at `cmd/entire/cli/agentcheck`.
- The context builder consumes existing Entire checkpoint read interfaces.
- `BuildFromRepository` opens the production checkpoint facade with `checkpoint.Open`, scoped by `settings.WithWorktreeRoot`, and uses `strategy.CheckpointReadRemotes`.
- Developer prompts are preserved as raw prompt text split with the existing checkpoint prompt separator.
- Session metadata, files touched, transcript availability, agent/model, token usage, and provenance are adapted into AgentCheck-owned structs.
- Git evidence is associated with the requested checkpoint through `Entire-Checkpoint` trailers and checkpoint metadata commit anchors.
- Graph evidence is optional through an injected provider. A CLI provider is available for `entire graph checkpoint`, and Graph unavailability never fails context construction.
- Prompt, transcript, Git, Graph, and aggregate context completeness are explicitly represented as `complete`, `redacted`, or `unavailable`.
- Redacted or missing prompt/transcript evidence is never represented as authoritative complete context.
- Raw prompt/transcript strings stay inside the local AgentCheck context builder path; Graph requests receive checkpoint/file evidence only.

## Verification

- Environment gate passed after using command-scoped PATH and Go cache settings.
- Required `entire graph search` was attempted with the repaired PATH but produced no output within the bounded wait and was interrupted.
- Focused AgentCheck package tests pass.
- Focused AgentCheck package build passes.
- Curveball-focused AgentCheck package tests pass.

## Next Stage

A3 - Git and Graph evidence hardening, with downstream evaluators required to honor context completeness state.
