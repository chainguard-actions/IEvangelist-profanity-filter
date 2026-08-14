<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.010

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/10.0.1-preview.010** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings, violating rule (a). In dotnet.yml, ${{ matrix.project }} is used directly in shell commands (dotnet publish, cd, and executable invocation). In release-api.yml and release.yml, ${{ github.event.inputs.version }} (attacker-controlled via workflow_dispatch) is interpolated directly in a run: block writing to $GITHUB_ENV, and ${{ env.RELEASE_VERSION }} / ${{ env.IMAGE_NAME }} are interpolated in dotnet publish arguments. In release-nuget.yml, ${{ github.ref }} and ${{ github.ref_name }} are interpolated in echo commands inside run: blocks.

Locations:

- `.github/workflows/dotnet.yml:37`
- `.github/workflows/dotnet.yml:40`
- `.github/workflows/dotnet.yml:44`
- `.github/workflows/release-api.yml:29`
- `.github/workflows/release-api.yml:43`
- `.github/workflows/release-api.yml:44`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:45`
- `.github/workflows/release-nuget.yml:29`
- `.github/workflows/release-nuget.yml:30`
- `.github/workflows/release-nuget.yml:46`

### github-env-injection (severity: high)

Attacker-controlled input ${{ github.event.inputs.version }} is written directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A workflow_dispatch caller can inject newlines to poison the environment file and set arbitrary environment variables for subsequent steps. The offending lines are: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV` in both release-api.yml (line 29) and release.yml (line 30).

Locations:

- `.github/workflows/release-api.yml:29`
- `.github/workflows/release.yml:30`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable branch/tag refs instead of immutable 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action is compromised. Failing references include: actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v3, actions/upload-artifact@main, IEvangelist/profanity-filter@main (in dogfood.yml). Additionally, action.yml uses a mutable Docker image tag `docker://ghcr.io/ievangelist/profanity-filter:latest` instead of a SHA digest.

Locations:

- `.github/workflows/dogfood.yml:16`
- `.github/workflows/dotnet.yml:27`
- `.github/workflows/dotnet.yml:30`
- `.github/workflows/release-api.yml:24`
- `.github/workflows/release-api.yml:32`
- `.github/workflows/release-api.yml:35`
- `.github/workflows/release-api.yml:47`
- `.github/workflows/release-nuget.yml:13`
- `.github/workflows/release-nuget.yml:19`
- `.github/workflows/release-nuget.yml:56`
- `.github/workflows/release-nuget.yml:61`
- `.github/workflows/release.yml:25`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:48`
- `action.yml:47`

### missing-permissions (severity: medium)

dotnet.yml and release-nuget.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/dotnet.yml:1`
- `.github/workflows/release-nuget.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 6 files:

1. action.yml: Pinned Docker image `ghcr.io/ievangelist/profanity-filter:latest` to sha256 digest while preserving `docker://` scheme and `:latest` tag.

2. dogfood.yml: Pinned `IEvangelist/profanity-filter@main` to full SHA.

3. dotnet.yml: Pinned `actions/checkout@main` and `actions/setup-dotnet@main` to full SHAs. Added `permissions: contents: read` top-level block. Fixed script injection by moving `${{ matrix.project }}` into `MATRIX_PROJECT` env var.

4. release-api.yml: Pinned all actions to full SHAs. Fixed github-env-injection by sanitizing `${{ github.event.inputs.version }}` with `printf '%s' | tr -d '\n\r'` before writing to GITHUB_ENV. Fixed script injection by moving `${{ env.RELEASE_VERSION }}` and `${{ env.IMAGE_NAME }}` into env blocks.

5. release.yml: Same fixes as release-api.yml.

6. release-nuget.yml: Pinned all actions to full SHAs. Added `permissions: contents: read` top-level block. Fixed script injection by moving `${{ github.ref }}` and `${{ github.ref_name }}` into env vars.

