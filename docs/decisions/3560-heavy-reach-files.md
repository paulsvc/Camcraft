## #3560 - the heavy reach reads test-only and binary-only source files

**Decided.**

- `gate heavy` classifies a changed file `crates/<d>/src/<rel>` with
  `heavy_files::seen_by`:
  - A module whose declaration, or the declaration of an ancestor module,
    is `#[cfg(test)]` reaches only `<d>:lib` (with its known-fail checks).
    The library that the other targets and the other crates link does not
    contain this code.
  - `src/main.rs` and `src/bin/..` reach only `<d>:bins` and every
    integration binary of `<d>`. No crate links a binary, but an
    integration test can run it through `CARGO_BIN_EXE_<name>`.
  - Each other file keeps the crate reach: the whole heavy lane of `<d>`
    and of each crate that depends on `<d>`.
- A narrower answer must be a proof. Each doubt gives the crate reach:
  - the crate manifest sets a target `path`, or cannot be read;
  - a line of any `.rs` file in the crate directory (`src/`, `src/bin/`,
    `tests/`, `benches/`, `examples/`) names the file as a path part
    (`"x.rs`, `/x.rs`), for example in `include!` or in a `#[path]`
    attribute;
  - a file other than the proven parent has a `mod <seg>;` that can load
    the file or one of its ancestor modules (for example `mod t;` in
    `main.rs`), or an indented `mod <seg>;`. A file is test-only only if no
    other compile root can name it (review of #3737);
  - the declaration is missing, indented (in an inline module), inline
    (`mod x {`), has a `#[path]`, or is declared again without
    `#[cfg(test)]` in the same parent file;
  - the attribute is not exactly `#[cfg(test)]` (for example
    `#[cfg(any(test, feature = ".."))]`);
  - the parent module file is not found, or both `x.rs` and `x/mod.rs`
    exist.
- `--dry-run` and the run print one `not run: <step> - the diff does not
  reach it` line for each heavy step that the diff does not reach, and the
  line `heavy reach: <n> step(s) reached, <m> skipped (not run)`.

**Why.** A narrower reach must never drop a heavy test that the change can
affect. Test-only modules and binaries are the two cases that the compiler
itself keeps out of the other targets, so the rule is sound by the Rust
reference and the Cargo target layout. motion has 37 such test-only files.

**Considered and dropped.**

- A per-module `use` graph, to map a library file to the integration
  binaries that load its module. It is not sound without more work: glob
  imports and re-exports hide the module path, a trait impl changes the
  behaviour of each caller of the trait, a method call does not name its
  module, and a registry (`inventory`, `ctor`) runs code that no caller
  names. The guards that this needs are new design (effort:high), so the
  issue is escalated for that part.
- The gate map comment of each test: it names what a test pins, not what
  it runs. A change in `motion` can break an `app` test whose gate map names
  only `app`.

**Cost to change later.** `heavy_files.rs` is one module with its pins in
`heavy_files_tests.rs`. A `use` graph can add a new `Seen` variant next to
the two present ones.
