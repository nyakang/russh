# NyaTerm fork notes

This branch tracks upstream `russh` with the patches documented below, so
[NyaTerm](https://github.com/nyakang/nyaterm) compiles a revision that has been
verified.

- Fork: <https://github.com/nyakang/russh>
- Upstream: <https://github.com/warp-tech/russh>
- Base revision: `d49f3e7a4d674beeeac7fdd1f0e7b49352bef488`
  (upstream `main` on 2026-09-22)
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
