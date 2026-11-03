# willow-effects agent instructions

willow-effects is a stable-Rust cargo subcommand that statically analyzes
effects in tokio programs (blocking in async, runtime nesting, lock misuse,
cancel safety) and exposes the results to humans and AI agents. It ports ideas
from the Willow language toolchain, but it is a separate project.

## Beads: use this repository's database only

- This repository has its own Beads database (`.beads/`, prefix `wfx`). Never
  create, update or close issues in the Willow database or any other one.
- Run every `bd` command from inside this repository. Before the first write
  in a session, run `bd where` and confirm `prefix: wfx`; stop if it differs.
- Never run bare `bd ready`, `bd list` or `bd memories`. Filter first or pipe to
  `head`/`tail` (normally 20–40 lines). Prefer `bd show <id>`.
- Start: `bd ready | head -n 20`, then `bd show <id>` and `bd update <id> --claim`.
  Finish with `bd close <id>` once the acceptance criteria are met.
- Create an issue before untracked work. One ticket per spec phase; do not
  over-split. Never weaken acceptance criteria or close incomplete work.
- `.beads/` is local-only (git exclude). Never commit it.

## Hard constraints

- **Stable Rust only.** No nightly toolchain, `rustc_private`, `rustc_driver`,
  `rustc_public`/stable_mir or Charon. Analysis is built on rust-analyzer's
  published `ra_ap_*` crates. Pin their version and update deliberately.
- Supported hosts: 64-bit Linux, macOS and Windows.
- Proof backends (Kani etc.) are optional plugins. The core must install and
  run without them; a missing backend means insufficient evidence, never success.

## Success conditions (every phase must preserve them)

1. **Few false positives.** Analysis stays fail-closed internally, but reporting
   is separate. Default mode reports only Definite findings (fully resolved
   witness chain). UNKNOWN is never reported by default; it is visible via
   `effects --explain` and `--mode strict`. For `dyn`, report only when all
   candidates carry the effect. Report generic code at concrete instantiation
   sites. FFI without a model stays silent. Missing evidence is not purity:
   type-inference unknowns map to UNKNOWN, never NONE.
2. **Easy installation.** Prebuilt binary, zero configuration, only the stable
   toolchain (`rust-src`, sysroot `rust-analyzer-proc-macro-srv`). Build scripts
   are off by default.
3. **Fast enough for every CI run.** Targets: warm-cache small PR < 30 s, cold
   < 3 min, < 4 GB on a ~50k LOC workspace plus dependencies.

A change that regresses measured precision, installation steps or the speed
budget is not complete.

## Quality checks

- `cargo fmt --all -- --check`
- `cargo clippy --workspace --all-targets -- -D warnings`
- Focused tests for the changed behavior and affected call paths. Full-suite
  and full-corpus runs belong in CI unless the user asks for a local run.
- New or changed rules need positive and negative tests, including generic,
  `dyn`, `spawn_blocking`/`block_in_place` and pre-runtime-startup cases, plus
  a precision measurement on the evaluation corpus.

## Implementation audit and complexity

- After every implementation or fix, audit the final code and its call paths
  before declaring completion; passing tests alone is not enough.
- Minimize asymptotic cost across the independent input dimensions (functions,
  call edges, generic instances, crates, capabilities). Avoid repeated scans,
  recomputation and unbounded context-sensitivity; check multiplication across
  callers and passes. Verify scaling with deterministic operation counts on
  increasing input sizes, not wall-clock time alone.
- Audit reports, measurements and benchmark outputs are local-only: write them
  under `docs/audits/`, never stage them, never add `.gitignore` allowlists for
  them. Delete large raw traces after summarizing.
- Prefer fixing discovered bugs within scope over filing follow-up tickets.

## Git policy

- Leave changes uncommitted for user review unless the user explicitly
  authorizes committing or pushing the current changes.
- Before every commit, inspect the exact staged file list and full staged diff.
  Stage only selected task files; never blanket-stage or force-add ignored files.
- Never put internal Beads ticket IDs in source, comments, tests, filenames,
  documentation or commit messages. Use descriptive names.
- Never add AI/bot attribution (e.g. `Co-Authored-By`) to commits or PRs.
- Never copy private Willow material (its `requirements/` directory or local
  Beads content) into this repository.
- Never force-push or rewrite published history.
