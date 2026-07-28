<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/13.4.6.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/13.4.6.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag/branch refs instead of pinned 40-character SHA hashes, making the action vulnerable to supply-chain attacks.

- action.yml: `image: 'docker://ghcr.io/ievangelist/profanity-filter:latest'` (mutable :latest tag)
- deploy-docs.yml: actions/checkout@v7, actions/setup-node@v6, actions/configure-pages@v6, actions/upload-pages-artifact@v5, actions/deploy-pages@v5
- dogfood.yml: IEvangelist/profanity-filter@main
- dotnet.yml: actions/checkout@main, actions/setup-dotnet@main
- publish-nuget.yml: actions/checkout@main, actions/setup-dotnet@main, actions/upload-artifact@main (×2), NuGet/login@v1
- release-api.yml: actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/cache@v6, actions/upload-artifact@main
- release.yml: actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/upload-artifact@main

Locations:

- `action.yml:52`
- `.github/workflows/deploy-docs.yml:20`
- `.github/workflows/dogfood.yml:18`
- `.github/workflows/dotnet.yml:30`
- `.github/workflows/publish-nuget.yml:31`
- `.github/workflows/release-api.yml:24`
- `.github/workflows/release.yml:24`

### github-env-injection (severity: high)

Two workflow files write the user-controlled input `${{ github.event.inputs.version }}` directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker with workflow_dispatch access can inject arbitrary environment variable definitions by embedding newlines in the version input.

Offending lines:
- release-api.yml: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`
- release.yml: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`

Locations:

- `.github/workflows/release-api.yml:31`
- `.github/workflows/release.yml:31`

### script-injection (severity: high)

Multiple workflow run: blocks interpolate ${{ ... }} expressions directly into shell command strings, allowing expression values to be interpreted as shell code before the shell ever sees them.

(a) dotnet.yml — `matrix.project` interpolated directly into shell commands:
  Line 39: `dotnet publish --configuration Release tests/${{ matrix.project }}/`
  Line 41: `cd artifacts/publish/${{ matrix.project }}`
  Line 44: `./${{ matrix.project }}`

(a) publish-nuget.yml — `github.ref_name` and `steps.nuget-login.outputs.NUGET_API_KEY` interpolated into shell:
  Line 63: `TAG_VERSION="${{ github.ref_name }}"`
  Line 89: `--api-key "${{ steps.nuget-login.outputs.NUGET_API_KEY }}"`

(a) release-api.yml — `github.event.inputs.version` interpolated into shell:
  Line 31: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`

(a) release.yml — `github.event.inputs.version`, `env.RELEASE_VERSION`, and `env.IMAGE_NAME` interpolated into shell:
  Line 31: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`
  Line 44: `-p ContainerImageTags='"latest;${{ env.RELEASE_VERSION }}"'`
  Line 45: `-p ContainerRepository=${{ env.IMAGE_NAME }}`

Locations:

- `.github/workflows/dotnet.yml:39`
- `.github/workflows/dotnet.yml:41`
- `.github/workflows/dotnet.yml:44`
- `.github/workflows/publish-nuget.yml:63`
- `.github/workflows/publish-nuget.yml:89`
- `.github/workflows/release-api.yml:31`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:45`

### unsafe-shell (severity: high)

release-api.yml pipes a remote shell script directly to bash without first downloading and inspecting it. If the remote server (aspire.dev) is compromised or the URL is intercepted, arbitrary code will execute on the runner.

Offending line (line 35): `run: curl -sSL https://aspire.dev/install.sh | bash`

Locations:

- `.github/workflows/release-api.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection, unsafe-shell

**Notes:**

Fixed all four finding types across 7 files:

1. **unpinned-uses**: Pinned all mutable action refs to full 40-char SHAs with tag comments. Container image in action.yml pinned with sha256 digest while preserving docker:// scheme and :latest tag inline.

2. **github-env-injection**: In release-api.yml and release.yml, moved `github.event.inputs.version` into an `INPUT_VERSION` env var and sanitized with `printf '%s' "$INPUT_VERSION" | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. **script-injection**: Moved all `${{ }}` expressions out of run: shell strings into step env: blocks — matrix.project in dotnet.yml, github.ref_name and NUGET_API_KEY in publish-nuget.yml, and env.RELEASE_VERSION/env.IMAGE_NAME in release.yml.

4. **unsafe-shell**: In release-api.yml, replaced `curl ... | bash` with a two-step approach: download to /tmp/aspire-install.sh then execute separately with bash.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/publish-nuget.yml at the 'Pack projects with explicit version from tag' step. The $TAG_VERSION variable (sourced from github.ref_name via the step's env block) was expanded unquoted in two dotnet pack commands as /p:Version=$TAG_VERSION and /p:PackageVersion=$TAG_VERSION. Changed both occurrences in both pack commands to use double-quoted form: /p:Version="$TAG_VERSION" /p:PackageVersion="$TAG_VERSION". This prevents shell metacharacters in a tag name from causing arbitrary command execution.

