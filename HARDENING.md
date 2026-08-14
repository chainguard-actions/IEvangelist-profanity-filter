<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/13.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/13.3.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable refs instead of pinned full-length SHA commits, making the action vulnerable to supply-chain attacks.

action.yml: `image: 'docker://ghcr.io/ievangelist/profanity-filter:latest'` — mutable `:latest` tag instead of a SHA digest.

deploy-docs.yml: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/configure-pages@v6`, `actions/upload-pages-artifact@v5`, `actions/deploy-pages@v5`.

dogfood.yml: `IEvangelist/profanity-filter@main`.

dotnet.yml: `actions/checkout@main`, `actions/setup-dotnet@main`.

publish-nuget.yml: `actions/checkout@main`, `actions/setup-dotnet@main`, `actions/upload-artifact@main` (×2), `NuGet/login@v1`.

release-api.yml: `actions/checkout@main`, `actions/setup-dotnet@main`, `docker/login-action@v4`, `actions/cache@v5`, `actions/upload-artifact@main`.

release.yml: `actions/checkout@main`, `actions/setup-dotnet@main`, `docker/login-action@v4`, `actions/upload-artifact@main`.

Locations:

- `action.yml:44`
- `.github/workflows/deploy-docs.yml:20`
- `.github/workflows/dogfood.yml:18`
- `.github/workflows/dotnet.yml:19`
- `.github/workflows/publish-nuget.yml:32`
- `.github/workflows/release-api.yml:24`
- `.github/workflows/release.yml:24`

### github-env-injection (severity: high)

User-controlled input `${{ github.event.inputs.version }}` is written directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker who triggers a `workflow_dispatch` event can inject arbitrary environment variable definitions by embedding newlines in the version input.

release-api.yml line 30: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`

release.yml line 30: `run: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`

Locations:

- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:30`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands.

dotnet.yml — `${{ matrix.project }}` is interpolated directly in shell commands (lines 33, 38, 41). A matrix value sourced from workflow context is substituted before the shell sees it:
  - `dotnet publish --configuration Release tests/${{ matrix.project }}/`
  - `cd artifacts/publish/${{ matrix.project }}`
  - `./${{ matrix.project }}`

publish-nuget.yml — multiple direct interpolations in run: blocks:
  - Line 50: `echo "GitHub ref: ${{ github.ref }}"`
  - Line 51: `echo "GitHub ref_name: ${{ github.ref_name }}"`
  - Line 65: `TAG_VERSION="${{ github.ref_name }}"`
  - Line 96: `--api-key "${{ steps.nuget-login.outputs.NUGET_API_KEY }}"`

release-api.yml:
  - Line 30: `echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`

release.yml:
  - Line 30: `echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV`
  - Line 43: `-p ContainerImageTags='"latest;${{ env.RELEASE_VERSION }}"'`
  - Line 44: `-p ContainerRepository=${{ env.IMAGE_NAME }}`

Locations:

- `.github/workflows/dotnet.yml:33`
- `.github/workflows/dotnet.yml:38`
- `.github/workflows/dotnet.yml:41`
- `.github/workflows/publish-nuget.yml:50`
- `.github/workflows/publish-nuget.yml:51`
- `.github/workflows/publish-nuget.yml:65`
- `.github/workflows/publish-nuget.yml:96`
- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:44`

### unsafe-shell (severity: high)

release-api.yml pipes a remote script directly to bash without first downloading and inspecting it: `run: curl -sSL https://aspire.dev/install.sh | bash`. If the remote server is compromised or the URL is redirected, arbitrary code will execute on the runner.

Locations:

- `.github/workflows/release-api.yml:34`

### missing-permissions (severity: medium)

dotnet.yml has no top-level `permissions:` key and no job-level `permissions:` key on its `build` job. Without explicit permissions, the workflow inherits the repository default (typically `contents: write` for classic repos), granting broader access than necessary.

Locations:

- `.github/workflows/dotnet.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all 5 findings across 7 files:

1. unpinned-uses: Pinned all action references to full SHA commits using lookup_action_sha. Container image in action.yml pinned with sha256 digest (docker://ghcr.io/ievangelist/profanity-filter:latest@sha256:c472f7403...). All workflow action references now use format 'owner/repo@<full-sha> # tag'.

2. github-env-injection: In release-api.yml and release.yml, moved github.event.inputs.version into an env var (INPUT_VERSION) and sanitized with 'printf "%s" "$INPUT_VERSION" | tr -d "\n\r"' before writing to $GITHUB_ENV.

3. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks. dotnet.yml: matrix.project → MATRIX_PROJECT. publish-nuget.yml: github.ref/ref_name → env vars, nuget API key → env var. release-api.yml and release.yml: version input → env var. release.yml: env.RELEASE_VERSION and env.IMAGE_NAME → step-level env vars for the publish step.

4. unsafe-shell: Fixed release-api.yml's 'curl | bash' pattern by downloading the script to /tmp/aspire-install.sh first, then executing it separately.

5. missing-permissions: Added 'permissions: contents: read' top-level block to dotnet.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 3 issues across 3 files: (1) publish-nuget.yml lines 78-79: added double quotes around $TAG_VERSION in dotnet pack MSBuild property arguments to prevent shell metacharacter injection; (2) release.yml line 32: replaced bare GITHUB_REF bash substitution written to $GITHUB_ENV with a sanitized version using printf+tr to strip newlines; (3) release-api.yml line 30: same sanitization fix as release.yml for the identical step.

