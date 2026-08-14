<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.008

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/10.0.1-preview.008** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use unpinned action references (branch names or version tags instead of full 40-character SHA digests), making the action vulnerable to supply-chain attacks. Unpinned references: dogfood.yml uses IEvangelist/profanity-filter@main; dotnet.yml uses actions/checkout@main and actions/setup-dotnet@main; release-api.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v3, and actions/upload-artifact@main; release-nuget.yml uses actions/checkout@main, actions/setup-dotnet@main, and actions/upload-artifact@main (×2); release.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v3, and actions/upload-artifact@main. Additionally, action.yml uses a mutable Docker image tag: image: 'docker://ghcr.io/ievangelist/profanity-filter:latest' instead of a SHA digest.

Locations:

- `.github/workflows/dogfood.yml:18`
- `.github/workflows/dotnet.yml:26`
- `.github/workflows/dotnet.yml:28`
- `.github/workflows/release-api.yml:23`
- `.github/workflows/release-api.yml:34`
- `.github/workflows/release-api.yml:36`
- `.github/workflows/release-api.yml:50`
- `.github/workflows/release-nuget.yml:11`
- `.github/workflows/release-nuget.yml:16`
- `.github/workflows/release-nuget.yml:44`
- `.github/workflows/release-nuget.yml:50`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:50`
- `action.yml:44`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell commands, violating rule (a). (1) dotnet.yml: ${{ matrix.project }} is interpolated directly in the run: block — used in dotnet publish path, cd path, and as an executable name (e.g., `./${{ matrix.project }}`). (2) release-api.yml: ${{ github.event.inputs.version }} is interpolated directly in a run: echo command (line 30); ${{ env.RELEASE_VERSION }} and ${{ env.IMAGE_NAME }} are interpolated directly inside a dotnet publish run: block (lines 44-45). (3) release.yml: same patterns as release-api.yml — ${{ github.event.inputs.version }} in run: echo (line 30), ${{ env.RELEASE_VERSION }} and ${{ env.IMAGE_NAME }} in dotnet publish run: block (lines 44-45).

Locations:

- `.github/workflows/dotnet.yml:35`
- `.github/workflows/dotnet.yml:37`
- `.github/workflows/dotnet.yml:39`
- `.github/workflows/dotnet.yml:41`
- `.github/workflows/release-api.yml:30`
- `.github/workflows/release-api.yml:44`
- `.github/workflows/release-api.yml:45`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:45`

### github-env-injection (severity: high)

Two workflow files write user-controlled input directly to $GITHUB_ENV without sanitization. In release-api.yml (line 30): `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV` — the workflow_dispatch input 'version' is attacker-controlled and written unsanitized to GITHUB_ENV, enabling environment variable injection (e.g., newline injection to set arbitrary env vars). The same pattern appears in release.yml (line 30). Neither write is preceded by the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

Locations:

- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:30`

### missing-permissions (severity: medium)

Two workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions. (1) dotnet.yml: no permissions block at top level or on the 'build' job. (2) release-nuget.yml: no permissions block at top level or on the 'publish' job.

Locations:

- `.github/workflows/dotnet.yml:1`
- `.github/workflows/release-nuget.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across 6 files:

1. unpinned-uses: Pinned all action references to full SHA digests with tag comments for readability. Pinned Docker image in action.yml to sha256 digest while preserving the docker:// scheme and :latest tag.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into step-level env: blocks. In dotnet.yml, matrix.project is now accessed via $MATRIX_PROJECT. In release-api.yml and release.yml, RELEASE_VERSION and IMAGE_NAME are re-exposed via env: in the Publish app step.

3. github-env-injection: In release-api.yml and release.yml, the workflow_dispatch version input is now moved to an env var (INPUT_VERSION) and sanitized with `printf '%s' "$INPUT_VERSION" | tr -d '\n\r'` before writing to GITHUB_ENV.

4. missing-permissions: Added `permissions: contents: read` at the top level of dotnet.yml and release-nuget.yml. The release-api.yml and release.yml already had job-level permissions blocks (contents: read, packages: write).

