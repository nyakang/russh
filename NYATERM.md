# NyaTerm fork notes

This branch tracks upstream `russh` with the patches documented below, so
[NyaTerm](https://github.com/nyakang/nyaterm) compiles a revision that has been
verified.

- Fork: <https://github.com/nyakang/russh>
- Upstream: <https://github.com/warp-tech/russh>
- Base revision: `e5d80a07dc554480b6ea3b2e14adfc190407b627` (upstream `main`, 2026-10-13)
- Branch: `nyaterm`

## Patches

- Add `client::KeepaliveMode::{Strict, Compatible}`. Strict preserves upstream;
  Compatible sends no-reply probes and does not close appliances that ignore them.
- No-reply keepalives must not enqueue global-response slots, which would otherwise
  misattribute a later forwarded-port or ping response.
- Regression coverage checks the wire reply flag and global-response queue.

## Not carried here

- `fix(helpers): accept a single trailing comma in a name-list` — dropped when
  this series was rebased from `4882af71` (`v0.62.5`) onto `v0.63.1`. Upstream
  implemented the same behaviour independently in `c465e3f` ("accept a single
  trailing comma in SSH name-lists", #743): it strips exactly one trailing comma
  before splitting, so an empty whole list stays valid while a lone comma, a
  double comma, and leading or mid-list empty entries stay rejected. That is the
  same accept/reject set the NyaTerm patch defined, so keeping both would have
  been duplicate logic on the packet decoding path. The patch's extra test cases
  (compression name-list, leading empty entry, non-ASCII name) went with it;
  upstream covers the behaviour with its own two tests.
- The NyaTerm snapshot adds `!Cargo.lock` to `.gitignore` and keeps a committed
  `Cargo.lock` so vendored validation resolves reproducibly. That is a vendoring
  concern, not a library change, so this branch keeps upstream's ignore rule and
  no lock file.

## Validation

On Windows 11, with the toolchain `rust-toolchain.toml` pins (1.91.0):

```sh
cargo fmt --all -- --check   # clean
cargo test -p russh --lib    # 175 passed
```

The 2026-09-22 merge to `d49f3e7a4d` conflicted only in
`russh/src/client/mod.rs`. The resolution keeps upstream's current session loop
and timer reset structure. Strict mode requests replies, counts missed probes,
and enforces `keepalive_max`; Compatible mode sends no-reply probes without
incrementing the timeout counter or adding a global-response queue entry.

## 2026-10-01 upstream merge

Merged upstream main without content conflicts. Host-key ownership proof and
Pageant principal fixes coexist with the Strict/Compatible keepalive patch.
Windows validation: `cargo test -p russh --lib` (181 passed) and
`cargo fmt --all -- --check` passed with the pinned Rust 1.91.0 toolchain.

## 2026-10-13 upstream rebase

Upstream `main` is fixed at `e5d80a07dc554480b6ea3b2e14adfc190407b627`.
Previous published NyaTerm head: `5ed33bde6e9f0288b7f0d54394d29f436849582b`.
The previous branch is retained locally as `backup/nyaterm-20261013`.

Replay commits before this documentation update:

- `73fa47470efcd0f0ad1211795e0edd7e31d9ba45` ci: verify fmt and library tests on the nyaterm branch
- `d277312d6ead687aa588aa0957aeb091223ae40b` ci: install rustfmt for the pinned toolchain
- `05ed9d8ee78e48df627131e0a446fe1bb878f9aa` docs: record the NyaTerm fork baseline and patch set
- `1793afca8879a59ab57437cc81728c3906d4db4b` feat(client): add compatible automatic keepalive mode
- `448d83f8c670da3147cd6dcb11f396db64c60f33` docs(fork): record russh baseline and validation

Retained compatible no-reply keepalive probes, strict timeout accounting and
the rule that probes without replies never enter the global response queue.
Compared the complete old and new upstream-relative client diffs, including
the strict-mode fix previously recorded only in a merge resolution.

Windows: `cargo test -p russh --lib` (194 passed),
`cargo fmt --all -- --check`, and `cargo clippy -p russh --lib` passed.
All-target clippy cannot build upstream Unix-only examples on Windows;
`--lib --tests` additionally rejects 487 preexisting unwraps in upstream tests.
No unrelated test/lint policy was changed to conceal these baseline failures.


Validation is on Windows; native macOS/Linux checks are delegated to the
branch workflow. Interactive GUI/file-drop and real-server authentication
checks were not performed by this update. Their outcomes must not be inferred
from scripted peers or compile checks.
