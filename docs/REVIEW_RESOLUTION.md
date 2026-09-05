# Review resolution record

- Repository: `Saber5656/datastory-kit`
- Pull request: #47
- Parent head observed before this addendum: `46252f82deadc172678ab68977d2bbd11fd9e4bd`
- Scope: existing review threads only; no new Bot review is requested.
- This document records design-level resolutions and focused verification gates. It does not claim implementation or test completion.

## Thread `PRRT_kwDOTNkKgM6PvAjj`

### Remove .gitkeep from optional content directories

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjj` identifies this contract gap.
- Normative resolution: Do not place `.gitkeep` files under optional content directories. An empty optional directory is represented by absence, and loader presence rules remain `evidence/images.yaml` iff images exist and `data/datapack.yaml` iff tables exist.
- Focused verification before resolving this thread: Build the minimal template and assert optional directories are absent while the loader accepts it without phantom assets or datapack requirements.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAjl`

### Avoid resolving an unexported player package.json

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjl` identifies this contract gap.
- Normative resolution: Remove runtime `require.resolve('@datastory/player/package.json')` assumptions; expose the needed version/metadata through a supported package export or build-time manifest that remains valid under an `exports` map.
- Focused verification before resolving this thread: Install the player package with a restrictive `exports` map and assert version/metadata resolution succeeds without package.json subpath access.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAjn`

### Render question prose in only one compiler stage

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjn` identifies this contract gap.
- Normative resolution: Make the prose-rendering stage the sole owner of question/final prompt, explanation, and hint HTML; the answer stage consumes the rendered fields and must not render them again.
- Focused verification before resolving this thread: Compile a prompt containing markup-sensitive content and assert each prose field produces one diagnostic/rendered fragment, not a duplicate.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAjp`

### Exercise the browser normalizer in vector parity

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjp` identifies this contract gap.
- Normative resolution: Run the shared browser/player `normalizeAnswer` and `hashAnswer` implementation on raw vector inputs before comparison; the vectors must cover NFKC, kana, whitespace, punctuation, and case behavior.
- Focused verification before resolving this thread: Execute parity vectors in the browser/runtime path and assert the produced normalized values and hashes equal the canonical fixtures.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAjr`

### Stop grepping legitimate answer text in bundles

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjr` identifies this contract gap.
- Normative resolution: Replace raw accepted-answer text greps with a structural secret-leak check over the answer manifest/bundle fields and a unique synthetic canary that cannot occur in story content; legitimate narrative text is not treated as a leak.
- Focused verification before resolving this thread: Build the sample containing the culprit name and assert it passes while a synthetic answer canary placed in a forbidden field is detected.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAju`

### Drop lowercase duplicates from accepted answers

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAju` identifies this contract gap.
- Normative resolution: Normalize accepted answers before uniqueness validation and hashing; duplicate normalized values emit DS7101 or are rejected according to the catalog, and the zero-warning sample removes the duplicate.
- Focused verification before resolving this thread: Compile `Riku Sato` and `riku sato` together and assert the defined duplicate diagnostic; compile the corrected sample and assert no duplicate warning.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAjy`

### Exclude sql.js glue from the player size budget

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAjy` identifies this contract gap.
- Normative resolution: Define the size-budget input from the player asset manifest and explicitly exclude only the sql.js wasm plus its known glue/worker assets, while retaining all other JS/CSS assets in the budget.
- Focused verification before resolving this thread: Build with sql.js and assert glue is excluded, wasm/glue are accounted by the explicit rule, and unrelated oversized player assets still fail.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAj1`

### Disambiguate ids reused across shared namespaces

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAj1` identifies this contract gap.
- Normative resolution: Use kind-qualified keys such as `image:<id>` and `character:<id>` in shared media/progress maps, and validate uniqueness within each namespace before writing the bundle.
- Focused verification before resolving this thread: Load a package reusing an id across image and character kinds and assert both records remain addressable with independent read state.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAj2`

### Hide the data section when no datapack exists

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAj2` identifies this contract gap.
- Normative resolution: Expose a capability flag from package loading and render the Data navigation/route only when a datapack and SQL service exist; minimal scenarios must not advertise an unusable view.
- Focused verification before resolving this thread: Load a scenario without tables and assert no Data navigation, route, or SQL service is constructed; load one with a datapack and assert all are present.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAj5`

### Move quoteIdent out of the compiler package

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAj5` identifies this contract gap.
- Normative resolution: Place `quoteIdent()` in a dependency-neutral shared SQL utility package imported by both compiler and player; preserve one implementation and keep the player independent of compiler internals.
- Focused verification before resolving this thread: Run dependency-boundary checks and SQL identifier fixtures, including quotes and reserved words, through both consumers.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAj6`

### Resolve sql-wasm.wasm from the bundle root

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAj6` identifies this contract gap.
- Normative resolution: Resolve the wasm URL against the deployed bundle root/base URL, pass it explicitly into the worker, and prohibit worker-relative bare-name resolution.
- Focused verification before resolving this thread: Serve a built bundle from a nested assets path and assert the worker requests the root-resolved wasm URL successfully.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Thread `PRRT_kwDOTNkKgM6PvAj8`

### Renumber fatal DS11xx diagnostics

- Finding: The existing review thread `PRRT_kwDOTNkKgM6PvAj8` identifies this contract gap.
- Normative resolution: Reserve `x1xx` for warnings and assign the fatal init/build diagnostics to the error range (for example DS1201/DS1202), updating the catalog, issue references, fixtures, and exit-status assertions consistently.
- Focused verification before resolving this thread: Run the diagnostic severity-range check and fatal-path fixtures; assert the codes are errors, return exit 1, and no x1xx warning is treated as fatal.
- Resolution record status: addressed in this design addendum; implementation/test completion is not claimed here.

## Bot review policy

The existing Bot review is not re-triggered for this PR. Replies and thread resolution are performed only after the focused verification conditions above are recorded.