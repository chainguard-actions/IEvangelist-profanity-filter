<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.009

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/10.0.1-preview.009** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable refs instead of pinned SHA hashes. action.yml references 'docker://ghcr.io/ievangelist/profanity-filter:latest' (mutable tag). All workflow files use @main (branch) or @v3 (version tag): dogfood.yml uses IEvangelist/profanity-filter@main; dotnet.yml uses actions/checkout@main and actions/setup-dotnet@main; release-api.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v3, actions/upload-artifact@main; release-nuget.yml uses actions/checkout@main, actions/setup-dotnet@main, actions/upload-artifact@main (×2); release.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v3, actions/upload-artifact@main. None of these are pinned to a full 40-character commit SHA.

Locations:

- `action.yml:43`
- `.github/workflows/dogfood.yml:16`
- `.github/workflows/dotnet.yml:27`
- `.github/workflows/dotnet.yml:29`
- `.github/workflows/release-api.yml:24`
- `.github/workflows/release-api.yml:33`
- `.github/workflows/release-api.yml:36`
- `.github/workflows/release-api.yml:48`
- `.github/workflows/release-nuget.yml:11`
- `.github/workflows/release-nuget.yml:18`
- `.github/workflows/release-nuget.yml:62`
- `.github/workflows/release-nuget.yml:68`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:53`

### missing-permissions (severity: medium)

dotnet.yml and release-nuget.yml have no top-level 'permissions:' key and no job-level 'permissions:' key on any job. This means the workflows run with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/dotnet.yml:1`
- `.github/workflows/release-nuget.yml:1`

### github-env-injection (severity: high)

Attacker-controlled input is written directly to $GITHUB_ENV without sanitization. In both release-api.yml and release.yml, the workflow_dispatch input 'version' (github.event.inputs.version) is interpolated directly into an echo command that writes to $GITHUB_ENV: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`. A malicious version value containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:32`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ ... }} expressions directly into shell command strings, enabling script injection. (a) dotnet.yml: ${{ matrix.project }} is interpolated directly in run: commands — `dotnet publish --configuration Release tests/${{ matrix.project }}/`, `cd artifacts/publish/${{ matrix.project }}`, `./${{ matrix.project }}`. A malicious matrix value could inject shell commands. (b) release-api.yml and release.yml: ${{ env.RELEASE_VERSION }} and ${{ env.IMAGE_NAME }} are interpolated directly in a dotnet publish run: block — `-p ContainerImageTags='"latest;${{ env.RELEASE_VERSION }}"'` and `-p ContainerRepository=${{ env.IMAGE_NAME }}`. (c) release-nuget.yml: ${{ github.ref }} and ${{ github.ref_name }} are interpolated directly in run: echo commands — `echo "GitHub ref: ${{ github.ref }}"`, `echo "GitHub ref_name: ${{ github.ref_name }}"`, `echo "Packing with tag: ${{ github.ref_name }}"`.

Locations:

- `.github/workflows/dotnet.yml:35`
- `.github/workflows/dotnet.yml:38`
- `.github/workflows/dotnet.yml:41`
- `.github/workflows/release-api.yml:43`
- `.github/workflows/release-api.yml:44`
- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:48`
- `.github/workflows/release-nuget.yml:24`
- `.github/workflows/release-nuget.yml:25`
- `.github/workflows/release-nuget.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection, script-injection

**Notes:**

Fixed all 4 findings across 6 files:

1. unpinned-uses: Pinned all mutable refs to full 40-character commit SHAs. action.yml container image pinned with digest. All workflow action references pinned: actions/checkout@3d3c42e5, actions/setup-dotnet@a98b5685, docker/login-action@c94ce9fb, actions/upload-artifact@043fb46d, IEvangelist/profanity-filter@2086462f.

2. missing-permissions: Added `permissions: contents: read` top-level block to dotnet.yml and release-nuget.yml.

3. github-env-injection: In release-api.yml and release.yml, moved `github.event.inputs.version` into step env block as INPUT_VERSION, then sanitized with `printf '%s' "$INPUT_VERSION" | tr -d '\n\r'` before writing to GITHUB_ENV.

4. script-injection: (a) dotnet.yml: moved matrix.project into MATRIX_PROJECT env var; (b) release-api.yml and release.yml: moved env.RELEASE_VERSION and env.IMAGE_NAME into step-level env blocks; (c) release-nuget.yml: moved github.ref and github.ref_name into GITHUB_REF_VALUE and GITHUB_REF_NAME_VALUE env vars.

