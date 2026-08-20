# NyaTerm fork notes

This branch tracks an unmodified upstream `russh` and adds only CI, so
[NyaTerm](https://github.com/nyakang/nyaterm) compiles a revision that has been
verified. It carries no library change of its own any more.

- Fork: <https://github.com/nyakang/russh>
- Upstream: <https://github.com/warp-tech/russh>
- Base revision: `d3ae702a43a163946f258297e398dc216339d5ce` (the `v0.63.1`
  release commit)
- Branch: `nyaterm`

## Patches

None. The branch is upstream plus `.github/workflows/nyaterm.yml` and this file.

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
cargo test -p russh --lib    # 166 passed
```
