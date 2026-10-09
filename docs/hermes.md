# Hermes integration

The native entry point is `__init__.py`, with metadata in `plugin.yaml`. It
registers Learn and Reset from the shared skill files and one `pre_llm_call` hook.
No external Python packages, custom model calls, global learner cache, or host
monkey-patches are required.

## Restoration

The current hook payload has session identifiers but no authoritative project
directory. Reading Python's process cwd would risk restoring another project's
notes in gateway sessions. Instead, the hook returns bounded instructions for
the agent to discover state using its session's tools. Git roots and worktree
boundaries limit lookup; without Git, only the starting directory is inspected.
Absent or paused profiles do not activate learning. Delegated child sessions
with `parent_session_id` receive no learning bootstrap.

Hermes adds this context to the current turn. Repeating the bootstrap each turn
supports restoration after restarts and between-turn compression without storing
learner history in the plugin. It is not a dedicated mid-turn compression hook;
restoration during a long tool loop still depends on Hermes retaining the current
turn's instructions. Instruction-following and teaching quality need live testing.

## Validation

```sh
python3 -B -m unittest discover -s tests -v
hermes plugins doctor . --ci
git diff --check
```

The tests cover the existing note lookup/reset behavior and the Hermes adapter's
registration, bounded per-turn context, host-filesystem independence, and child
session exclusion. Plugin Doctor checks the real loader and hook registration.
Run tests outside an ancestor Git boundary for temporary fixtures: in environments
with `/tmp/.git`, use an appropriate non-repository `TMPDIR`.

Validated initially with Hermes checkout `0be2d562b0`. Older versions lacking
`register_skill` are unsupported; no minimum release number has been established.

Before release, use a disposable project in an authenticated Hermes conversation:

1. Load `vibe-wise:learn`; complete onboarding with `clarify` or chat fallback.
2. Work through a Build checkpoint, discuss the proposal, then explicitly approve
   implementation. Verify the agent preserves that distinction.
3. Restart with a pending checkpoint, then compress the conversation and continue.
   Verify notes restore without another onboarding or assumed approval.
4. Pause and restart; confirm learning remains paused. Explicitly load Learn to resume.
5. Load `vibe-wise:reset`, cancel, and verify no changes. Repeat and confirm; check
   backups, fresh onboarding, and unchanged source files.
6. Switch projects and verify no preferences or pending decisions cross boundaries.

Automated checks do not claim these model-driven scenarios have been completed.

Reference: [Hermes native plugin guide](https://hermes-agent.nousresearch.com/docs/developer-guide/plugins).
