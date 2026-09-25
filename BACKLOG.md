# BACKLOG — rdf_core

Open and pending items only. Anything already decided and done is recorded in
`CHANGELOG.md`, not here.

Repo: `staylorx/rdf_core`, default branch `main`, audited at HEAD `0b34b56`
("Start next development cycle on 0.9.18-dev").
Sources: the Windows-lane build & test report (board task `t_cc5f87e1`) and the
`dart-flutter-bible` deviation audit (board task `t_323e031f`). Neither pass
changed a source file, and nothing recorded below has been auto-fixed.

Measured baseline on the Windows lane (Windows 11, git-bash/MSYS; Dart SDK
3.13.1 stable, windows_x64):

| Command | Exit | Result |
|---|---|---|
| `dart pub get` | 0 | `Got dependencies!` |
| `dart analyze` | 0 | `No issues found!` |
| `dart analyze --format=machine` | 0 | 0 lines of output |
| `dart test --reporter=expanded` | 0 | `All tests passed!` — 725/725, 0 skips |
| `dart format --output=none --set-exit-if-changed .` | 1 | `Formatted 73 files (2 changed)` |
| `dart pub publish --dry-run` | 65 | valid 743 KB archive, 1 warning + 1 hint |

## Build & analysis problems

There are no build errors, no analyze lints/warnings and no broken
dependencies. Every open item below is a warning, a hint, format drift or a
documentation/metadata gap. Raw messages are quoted verbatim from the captures
behind `dart pub get` / `dart analyze` / `dart test` / `dart format` /
`dart pub publish --dry-run` above.

### B1 — `dart format` gate fails on two files

`dart format --output=none --set-exit-if-changed .` → exit 1. Raw output:

```
Changed lib\src\plugin\rdf_codec.dart
Changed test\src\iri_util_test.dart
Formatted 73 files (2 changed) in 0.22 seconds.
```

- `lib/src/plugin/rdf_codec.dart:480` — blank line inside class
  `AutoDetectingGraphDecoder` carries two trailing spaces (raw bytes
  `20 20 0A`); `dart format` collapses it to an empty line.
- `test/src/iri_util_test.dart:229` — the line
  `      test('should disable absolute-path when allowAbsolutePath is false 2', () {`
  is 84 columns; `dart format` wraps the closure onto the next line.

Both are deterministic formatter rules (trailing-whitespace strip, 80-column
wrap), so they are genuine drift and not artefacts of the Dart 3.13 tall-style
formatter. Pending: format these two files only — a repo-wide `dart format` on
this SDK rewrites more files than the gate names, so review the diff first.

### B2 — `CHANGELOG.md` does not name the current version as a version heading

Measured at HEAD `0b34b56`, before this pass wrote anything,
`dart pub publish --dry-run` → exit 65. Raw message:

```
Package validation found the following potential issue:
* <checkout>\CHANGELOG.md doesn't mention current version (0.9.18-dev).
  Consider updating it with notes on this version prior to publication.
```

Re-measured after this pass added the `[Unreleased]` section (which names the
working version), with the two files still uncommitted: that warning is gone.
The same command now reports

```
Package validation found the following potential issue:
* 1 checked-in file is modified in git.

  Modified files:

  CHANGELOG.md
```

i.e. the transient "dirty tree" warning that the commit task clears by
committing, plus the B3 hint — still 1 warning + 1 hint, exit 65, 751 KB archive
(BACKLOG.md is now in the archive too).

Pending: add a real `## [0.9.18]` section when 0.9.18 is cut, so the check is
satisfied by the version heading rather than by a mention in prose. The mention
is the whole of what the check found here, so re-verify it if that mention ever
leaves the file.

### B3 — local version `0.9.18-dev` is older than the published `0.9.25`

`pubspec.yaml:3` — `version: 0.9.18-dev`. Raw message from `dart pub publish
--dry-run`:

```
Package validation found the following hint:
* The latest published version is 0.9.25.
  Your version 0.9.18-dev is earlier than that.
The server may enforce additional checks.
```

Pending: settle the fork's version story before any publish — a real
`dart pub publish` of this tree would regress the published 0.9.x line.

### B4 — `pubspec.yaml:4-7` (and `maintainers`) still point at the upstream fork

```
homepage: https://kkalass.github.io/rdf_core/
repository: https://github.com/kkalass/rdf_core
issue_tracker: https://github.com/kkalass/rdf_core/issues
documentation: https://kkalass.github.io/rdf_core/
```

`maintainers` is likewise `Klas Kalaß <habbatical@gmail.com>`, while this
checkout's `origin` is `https://github.com/staylorx/rdf_core.git`. Pending:
decide whether to re-point the metadata at the fork or keep upstream attribution
(re-pointing loses the upstream link; not re-pointing ships upstream URLs on
pub.dev).

### B5 — generated dartdoc tree and a 1 MB fixture are committed and published

`doc/api/**` is 420 tracked files / ~3.9 MB of generated HTML (including
`index.json` 98 KB, `docs.dart.js` 129 KB), and
`test/assets/realworld/schema.org.ttl` is ~1 MB. `dart pub publish --dry-run`
builds a 743 KB compressed archive from them. `.gitignore` does not exclude
`doc/api`. Pending: decide whether the generated tree stays tracked or moves to
CI/Pages only. Not touched by this pass ("do not hand-edit generated blobs").

### B6 — stale dependency versions in `pubspec.lock`

`dart pub get` raw message:

```
23 packages have newer versions incompatible with dependency constraints.
Try `dart pub outdated` for more information.
```

`pubspec.lock` is committed; `dart pub get` did not rewrite it (`git status`
stayed clean). Nothing is broken — maintenance item only. Pending: refresh the
constraints/lock when convenient.

### B7 — `AGENTS.md` does not exist at the repo root

Only `CHANGELOG.md`, `CONTRIBUTING.md` and `README.md` are present at the root
(`doc/COOKBOOK.md`, `doc/DESIGN_PHILOSOPHY.md`, `doc/GETTING_STARTED.md` and
`example/README.md` sit below it). `BACKLOG.md` is created by this pass.
Pending: add `AGENTS.md` recording the repo's declared error style and local
wiring — the bible deviation list below also calls for it.

### B8 — CI does not gate formatting

`.github/workflows/ci.yml` (push/PR to `main`) runs only `dart pub get` →
`dart analyze` → `dart test --coverage=coverage` → lcov/codecov upload. It does
not run `dart format --output=none --set-exit-if-changed`, which is why the B1
drift survived. Pending: decide whether to add the format gate (see also the
§02 CI item in the deviation list below).

## Deviations from `staylorx/dart-flutter-bible`

Each item below is a spot where the code differs from the bible standard while
the bible may itself be the side that is wrong. All are **flagged for later
review**: none was auto-fixed, and none is a build break. Entries are copied
verbatim, in the required `Deviation: <path> - <what diverges and why>` form,
from the audit report (`DEVIATIONS-rdf_core-vs-bible.md`, board task
`t_323e031f`), which walked `lib/` (29 files), `test/` (39 files),
`pubspec.yaml`, `analysis_options.yaml` and `.github/workflows` against
`staylorx/dart-flutter-bible` @ `d4d2ff1` (docs/00-compact.md + docs/01..12).

Applicability, as recorded by the audit: §05 (persistence), §07 (builders —
conforming by absence) and §08 (Flutter ring) have no surface in this pure-Dart,
no-Flutter core and are recorded as N/A rather than as deviations; §12 (sources)
is a reference list, and §09/§11 carry no repo-local content beyond the items
already cited.

### §01 Architecture / §04 Functional core

Deviation: lib/src/exceptions/rdf_exception.dart - failure is modelled as thrown exceptions end-to-end: the base is `class RdfException implements Exception` (line 44) and every failure path throws, so failure is never a value. Diverges from §01 Law 4 ("Exceptions are a UI-boundary phenomenon; everywhere inside the bulls-eye failure is a value") and from §04 (fpdart Either/TaskEither everywhere).
Deviation: pubspec.yaml - no fpdart and no equatable in dependencies (lines 14-19), so the §04 stack is entirely absent from the package rather than pinned ^1.2.0 / ^2.x.
Deviation: lib/src/graph/rdf_term.dart - `throw RdfConstraintViolationException` at lines 134, 144 and 153, i.e. throw inside the entity/value-object layer (IriTerm._validateAbsoluteIri). Diverges from §04 "The exception rule, stated once, loudly" (no throw inside the bulls-eye) and §01 Law 1 (the domain center must be exception-free).
Deviation: lib/src/graph/rdf_term.dart - IriTerm (line 167), BlankNodeTerm (line 189) and LiteralTerm (line 449) hand-write `operator ==`/`hashCode` instead of `extends Equatable` with a `props` list. Diverges from §04 "Equality: equatable — every entity and value object extends Equatable with List<Object?> get props" and §01 Law 2 (value equality via equatable).
Deviation: lib/src/graph/triple.dart - `Triple` (line 117) hand-writes `operator ==`/`hashCode`; same divergence from §04 equatable as the terms above.
Deviation: lib/src/graph/rdf_graph.dart - `RdfGraph` (line 858) hand-writes `operator ==`/`hashCode`; same §04 equatable divergence. (Its immutability and `withX` returning new instances do conform to §01 Law 2.)
Deviation: lib/src/graph/rdf_term.dart - BlankNodeTerm (lines 184-196) defines equality as identity (`identical` / `identityHashCode`), deliberately. Defensible in RDF terms (blank-node labels are scoped to a document), but it is a divergence from §01 Law 2 / §04 equatable value semantics — flag for later review, do not "fix".
Deviation: lib/src/exceptions/rdf_exception.dart - the failure base is an open, extendable `class ... implements Exception` carrying a free-form `message` and a raw `cause: Object?` (lines 46-52), not a closed hierarchy of named cases. Diverges from §04 "Failure hierarchies are per-layer, defined in the domain package, and closed (abstract base + final/`sealed` leaves ... a switch over them must be exhaustive, which is the point)".
Deviation: lib/src/iri_util.dart - failure type `BaseIriRequiredException extends RdfDecoderException` is declared at line 312, far from the rest of the exception hierarchy in lib/src/exceptions/; failures are not per-layer, and no `sealed`/exhaustive hierarchy exists. Diverges from §04/§05 (contracts and failures live in one domain layer). Flag for review.
Deviation: lib/src/jsonld/jsonld_decoder.dart - hand-rolled `try`/`catch` used for imperative control flow at lines 210, 214/216 and 254 (catch, inspect `e is RdfException`, rethrow or wrap). Diverges from §04 "Adapter boundary is a LINE not a zone ... hand-rolled try/catch inside adapters = VIOLATION"; only `Either.tryCatch`/`TaskEither.tryCatch` wrapping the third-party call itself is allowed.
Deviation: lib/src/ntriples/ntriples_decoder.dart - hand-rolled `try`/`catch` at lines 96 and 429 (per-line parse guarding and hex-escape parsing). Same §04 adapter-boundary violation.
Deviation: lib/src/iri_util.dart - hand-rolled `try`/`catch` at lines 83/141 and 361/365 (fall back to manual URI resolution on parse failure). Same §04 adapter-boundary violation.
Deviation: lib/rdf_core.dart - the public seam returns bare values and throws instead of `Future<Either<Failure, T>>`: `RdfGraph convert(String input, {String? documentUrl})` (lib/src/rdf_decoder.dart, "May throw format-specific parsing exceptions"). Diverges from §04 "the public seam is Future<Either<Failure, T>>" — and the §04 escape hatch (a package may present exceptions *if it declares them*) is not satisfied either, see the declaration item below.
Deviation: lib/rdf_core.dart - the error style is never declared. The barrel doc comment (lines 1-128) never states what a consumer holds; README.md only says it at line 325 of 451 (not "near the top"); and there is no AGENTS.md in the repo. Diverges from §04 "Declare the error style, loudly" — the declaration is required in the barrel doc comment, the README near the top, and AGENTS.md on any deviation. Silence is the violation.

### §02 Toolchain / §03 Topology

Deviation: lib/rdf_core.dart - the barrel (lines 178-369) *defines* the implementation: `final class RdfCore` plus a top-level `final rdf = RdfCore.withStandardCodecs();` singleton, instead of only re-exporting `lib/src/`. Diverges from §02 "lib/<package_name>.dart, a hand-written barrel that re-exports the public API from lib/src/" (also makes the barrel a code file with the 128-line header noted below).
Deviation: lib/rdf_core.dart + lib/rdf_core_extend.dart - the package has TWO public entry points (`export`s: 13 and 3). Diverges from §02 "Each package exposes exactly one public entry point" and §10 "One hand-written barrel per package".
Deviation: pubspec.yaml - `sdk: ^3.6.0` at line 21 instead of the doctrine floor `'>=3.10.0 <4.0.0'`. Diverges from §02 SDK constraint and §09 step 2/4.
Deviation: analysis_options.yaml - line 1 is the entire file: `include: package:lints/core.yaml`. `public_member_api_docs` is not enabled, `analyzer: errors: {todo: error}` is absent, and no strict/extra lints are configured. Diverges from §02 "Enforce it: enable public_member_api_docs in analysis_options.yaml" and §09 step 8.
Deviation: .github/workflows/ci.yml - line 20 runs plain `dart analyze`; the fatal flags are missing. Diverges from §02 "The gate is the fatal flags, not plain analyze" and §09 step 10.
Deviation: test/ - there is no `dart_arch_test` architecture test anywhere. Diverges from §02 "Package-boundary rules: dart_arch_test" / §09 step 7 (test-enforced boundary direction + workspace-wide cycle-freedom). Single-package repo, so direction is near-vacuous, but cycle-freedom is still assertable — flag for review.
Deviation: lib/src/iri_compaction.dart, lib/src/graph/rdf_term.dart, lib/src/plugin/rdf_codec.dart, lib/src/exceptions/rdf_validation_exception.dart, lib/src/exceptions/rdf_decoder_exception.dart, lib/src/turtle/turtle_tokenizer.dart, lib/src/jsonld/jsonld_decoder.dart, lib/src/ntriples/ntriples_encoder.dart, lib/src/rdf_encoder.dart, lib/src/turtle/turtle_decoder.dart, lib/src/exceptions/rdf_encoder_exception.dart, lib/src/jsonld/jsonld_encoder.dart, lib/src/ntriples/ntriples_decoder.dart, lib/src/rdf_decoder.dart, lib/src/turtle/turtle_encoder.dart, lib/src/exceptions/rdf_exception.dart - 16 files under lib/ declare between 2 and 10 public classes each (respective counts 10, 7, 5, 4, 4, 4, 3, 3, 3, 3, 3, 2, 2, 2, 2, 2). Diverges from §02/§03 "One class per file, one file per class".
Deviation: lib/src/graph/triple.dart - file-level `///` header at line 1 plus `library rdf_triple;` at line 41; the same pattern occurs in 24 files under lib/ (e.g. lib/src/graph/rdf_term.dart:32 `library rdf_terms;`, lib/src/graph/rdf_graph.dart:49 `library rdf_graph;`, lib/src/iri_util.dart:13, lib/src/iri_compaction.dart:6 `library prefix_generator;`, lib/src/vocab/*.dart:16-19, lib/src/exceptions/*.dart:14-17 `library exceptions.all;`, and a bare `library;` at lib/src/rdf_decoder.dart:6). Diverges from §02 "`///` is for declarations only — never for files ... Adding a `library;` directive just to satisfy a file-level `///` is a violation"; per §02 a `library;` belongs only in barrel files.
Deviation: lib/rdf_core.dart - the barrel's library doc comment occupies lines 1-128 (a full usage walkthrough with several code blocks and a capability catalogue) before `library rdf;` at line 129. Diverges from §02 "Terse docs" (1-2 lines, what+why) / §01 "Terse docs"; also §01 D.R.Y. (large usage prose duplicates README/doc/GETTING_STARTED.md).
Deviation: lib/src/plugin/rdf_codec.dart - line 480 is a blank line carrying two trailing spaces (raw bytes 20 20 0A); `dart format --output=none --set-exit-if-changed .` exits 1 on it. Diverges from §02 "Formatter: dart format only" and the bible AGENTS.md housekeeping rule "keep the working tree clean before committing".
Deviation: test/src/iri_util_test.dart - line 229 is 84 columns and the formatter rewraps the closure, so `dart format --set-exit-if-changed` also fails here. Diverges from §02 formatter rule.
Deviation: lib/src/jsonld/jsonld_decoder.dart - `// ignore: unused_field` at line 86; also lib/src/ntriples/ntriples_decoder.dart:62 and lib/src/ntriples/ntriples_encoder.dart:79. The suppressions are per-line (correct), but carry no reason. Diverges from §02 "with a reason when the intent isn't obvious".
Deviation: test/src/graph/rdf_graph_test.dart - `// ignore: unrelated_type_equality_checks` at line 354 with no reason. Same §02 suppression-reason divergence.
Deviation: lib/ - there is no package boundary between the entity/domain code (lib/src/graph, lib/src/exceptions, lib/src/vocab) and the serialization adapters (lib/src/turtle, lib/src/jsonld, lib/src/ntriples, lib/src/plugin): both are directories in one package and codecs import entities directly. Diverges from §03 Topology B (separate `*_domain` / `*_usecases` / `*_datasource_*` packages, dependencies inward only) and from the §04/§05 rule that repository/datasource contracts live in the domain package. Flag for review — the bible's topology assumes an application; a data-model core with format adapters is not covered by it.
Deviation: lib/src/plugin/rdf_codec.dart + test/src/{turtle,jsonld,ntriples}/ - the three codec "adapters" share one contract (RdfGraphCodec) but each format is tested only by its own per-format suite; there is no shared contract suite that is run against every implementation. Diverges from §05 "one test/ file that takes any adapter implementation and asserts the full contract, run against every adapter in CI. A contract is only real when two implementations agree on it" and from §01's at-least-two-adapter spirit.
Deviation: README.md - lines 18-23 (and the badge/logo block at lines 2-10) point at the upstream `kkalass/*` projects and homepage, and the file carries rules/publishing prose ("All core methods throw ..."); combined with doc/GETTING_STARTED.md + doc/COOKBOOK.md + doc/DESIGN_PHILOSOPHY.md this restates in prose what belongs in code comments/tests. Diverges from §01 D.R.Y. and §02 "Code placement: ... never in READMEs or prose documentation". Flag for review (these predate the bible).

### §06 Testing

Deviation: test/ - 1,864 `expect(...)` call sites across the suite and zero `should.` assertions; `shouldly` is absent from pubspec.yaml. Diverges from §06 "shouldly ... is the assertion library" / "Never mix expect() with shouldly" and the §02 pinned stack.
Deviation: test/src/graph/rdf_graph_test.dart - names are sentence style (`test('should create empty graph')` line 51, `test('should add triples immutably with withTriple')` line 70) instead of Given/When/Then groups; only one Given/When/Then-shaped string exists in the whole suite. Diverges from §06 "Test names are Given/When/Then".
Deviation: test/src/plugin/auto_detecting_codec_test.dart - hand-rolled mock classes `_MockCodec` (line 239), `_MockDecoder` (line 276) and `_MockEncoder` replicate what mocktail provides; `mocktail` is absent from pubspec.yaml. Diverges from §06 "mocktail for mocks — and only at the application layer, mocking interfaces we own".
Deviation: pubspec.yaml - dev_dependencies are only test, yaml, path and lints (lines 24-28): no shouldly, no mocktail, no dart_arch_test. Diverges from §02 pinned stack / §06 tooling.


## Verified clean in this pass

These are recorded so the open items above are not over-read as a broken build:

- `dart pub get` → exit 0, `Got dependencies!`; no broken dependency.
- `dart analyze` → exit 0, `No issues found!`; `dart analyze --format=machine` →
  exit 0 with zero lines. 0 errors, 0 warnings, 0 lints across `lib/` and
  `test/` under the repo's own `analysis_options.yaml` (`lints/core.yaml`,
  lints 6.0.0 resolved).
- `dart test --reporter=expanded` → exit 0, `All tests passed!` — 725 tests, 0
  failures, 0 skips, 0 errors. The real-world fixtures under
  `test/assets/realworld/` were exercised, not skipped.
- `dart pub publish --dry-run` produces a valid 743 KB archive; its only
  findings are B2 and B3 above.
- No `TODO`/`FIXME` in `lib/`, `test/`, `tool/` or `example/` source; the only
  matches are inside the `test/assets/realworld/schema.org.ttl` fixture, which
  is upstream RDF data rather than code.
- The battery of deviations below concerns standards conformance, not
  correctness: the package compiles, analyzes clean and passes its full suite at
  the audited commit.
