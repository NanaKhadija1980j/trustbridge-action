# Contributing

Thanks for your interest in contributing to TrustBridge! This document explains how to get involved.

## Getting Started

1. Fork the repository and clone your fork.
2. Install dependencies for the action package:

   ```bash
   cd trustbridge-action
   npm install
   ```
3. Create a branch for your change.

## Development Workflow

- Make focused changes that address a single issue.
- Keep existing code style, naming, and structure.
- Run the relevant checks before opening a pull request.

## CI Checks

The `.github/workflows/ci.yml` workflow runs on pushes and pull requests targeting `main`, and can also be started manually. Jobs that set up Node use Node.js 20; the workflow grants read-only repository contents permissions. Before opening a pull request, run the relevant checks from the repository root:

```bash
npm run typecheck
npm test
npm run lint --if-present
```

CI also runs these gates:

- `check`: verifies the action package files, then runs typecheck, tests, and lint.
- `test-coverage`: runs `npm run test:coverage` and enforces the thresholds in `jest.config.js`.
- `license-check`: runs `npm run license:check`.
- `dist-drift`: typechecks tests, builds the distribution, checks source maps, and fails if the build changes committed files in `dist/`.
- `bundle-size`: runs `npm run test:bundle-size`.
- `verify-dist-manifest`: runs `node scripts/verify-dist-manifest.js`.
- `reason_code docs in sync with catalog`: runs `npm run reason-codes:check` and fails when the generated table in `docs/USAGE.md` drifts from `schemas/reason-codes.json`. Fix with `npm run reason-codes:doc`.
- `check-unpinned-actions`: runs `bash scripts/check-unpinned-actions.sh .github/workflows docs/examples`.

## Reason code catalog and generated docs

[`schemas/reason-codes.json`](schemas/reason-codes.json) is the single source of truth for every `reason_code` the action emits. It is a **public API surface** — dashboards and workflow gates switch on these strings — so renames and removals are BREAKING CHANGES requiring a MAJOR version bump. Adding a code is additive and safe in a minor/patch release.

The operator-facing table in [`docs/USAGE.md`](docs/USAGE.md#reason_code-reference-issue-297) is **generated** from the catalog, between these sentinel comments:

```markdown
<!-- BEGIN GENERATED: reason-codes (source: schemas/reason-codes.json) -->
<!-- END GENERATED: reason-codes -->
```

### Adding or changing a reason code

1. Add the entry to `schemas/reason-codes.json` (`code`, `description`, `accountFunded`, `trustlineExists`, `xlmReserveMet`, `valid`, `emittedBy`, `since`). Set `deprecated: true` when retiring a code — it is then rendered with strikethrough instead of being deleted.
2. Regenerate the docs table:

   ```bash
   npm run reason-codes:doc
   ```

3. Verify there is no drift (this is what CI runs):

   ```bash
   npm run reason-codes:check
   ```

4. Extend `KNOWN_REASON_CODES` in `__tests__/reason-codes.test.ts` when introducing a new code so the rename lock covers it.

### Rules for the generated region

- **Never hand-edit** between the sentinel comments — the next `npm run reason-codes:doc` run overwrites it and the `reason-codes:check` CI step fails on drift.
- Column order is fixed (`reason_code`, `valid`, `account_funded`, `trustline_exists`, `xlm_reserve_met`, `Description`) so a regeneration produces a minimal diff.
- Prose *outside* the sentinels is preserved verbatim; put any narrative explanation there.
- `|` characters in a description are escaped so they cannot break the Markdown table.

### Running it in a pre-commit hook

`npm run reason-codes:check` exits non-zero on drift and writes nothing, so it is safe to run from a hook or a CI step:

```json
{
  "scripts": {
    "pre-commit": "npm run reason-codes:check"
  }
}
```

The script uses only Node built-ins, so it runs on Node 20+ with no install step.

## Mutation Testing

For changes to security-critical code in `src/validation.ts` (SSRF validation and StrKey address checks), run mutation tests locally to ensure test coverage catches weakened checks:

```bash
cd trustbridge-action
npm run mutation
```

Mutation testing generates synthetic code defects (mutants) and verifies that your test suite catches each one. For changes to security-critical validation, run `npm run mutation`; results are saved to `stryker-report.html`. Mutation testing is not part of `.github/workflows/ci.yml`. See `stryker.config.mjs` for score thresholds and configuration.

## Pull Requests

- Reference the issue your change addresses.
- Describe what changed and why.
- Ensure the checks above pass.

## Reporting Issues

Open a GitHub issue with a clear description, reproduction steps, and expected behavior.
