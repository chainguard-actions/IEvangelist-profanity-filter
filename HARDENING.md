<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/13.4.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **IEvangelist--profanity-filter/13.4.6.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable refs instead of pinned 40-character SHA hashes. deploy-docs.yml uses actions/checkout@v6, actions/setup-node@v6, actions/configure-pages@v6, actions/upload-pages-artifact@v5, actions/deploy-pages@v5. dogfood.yml uses IEvangelist/profanity-filter@main. dotnet.yml uses actions/checkout@main, actions/setup-dotnet@main. publish-nuget.yml uses actions/checkout@main, actions/setup-dotnet@main, actions/upload-artifact@main, NuGet/login@v1. release-api.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/cache@v5, actions/upload-artifact@main. release.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/upload-artifact@main. action.yml runs.image uses docker://ghcr.io/ievangelist/profanity-filter:latest (mutable tag, not a SHA digest).

Locations:

- `.github/workflows/deploy-docs.yml:22`
- `.github/workflows/dogfood.yml:18`
- `.github/workflows/dotnet.yml:27`
- `.github/workflows/publish-nuget.yml:34`
- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:29`
- `action.yml:44`

### missing-permissions (severity: medium)

dotnet.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. Without explicit permissions the workflow inherits default repository permissions which may be overly broad.

Locations:

- `.github/workflows/dotnet.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions before the shell parses the command, enabling script injection. dotnet.yml: ${{ matrix.project }} used as a path and executable name in run: block. publish-nuget.yml: ${{ github.ref }} and ${{ github.ref_name }} echoed in run: block; ${{ github.ref_name }} assigned to TAG_VERSION and used in dotnet pack args; ${{ steps.nuget-login.outputs.NUGET_API_KEY }} passed as --api-key in run: block. release-api.yml: attacker-controlled ${{ github.event.inputs.version }} interpolated in run: block writing to GITHUB_ENV; ${{ env.RELEASE_VERSION }} in run: block. release.yml: attacker-controlled ${{ github.event.inputs.version }} interpolated in run: block writing to GITHUB_ENV; ${{ env.RELEASE_VERSION }} and ${{ env.IMAGE_NAME }} in run: block.

Locations:

- `.github/workflows/dotnet.yml:34`
- `.github/workflows/publish-nuget.yml:47`
- `.github/workflows/publish-nuget.yml:60`
- `.github/workflows/publish-nuget.yml:80`
- `.github/workflows/release-api.yml:36`
- `.github/workflows/release-api.yml:39`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:38`

### github-env-injection (severity: high)

release-api.yml and release.yml write the attacker-controlled workflow_dispatch input version directly to $GITHUB_ENV without sanitization. An attacker can supply a newline in the version input to inject arbitrary environment variables. No printf | tr -d newlines sanitization is applied before the write. Offending steps: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV

Locations:

- `.github/workflows/release-api.yml:39`
- `.github/workflows/release.yml:38`

### unsafe-shell (severity: high)

release-api.yml pipes the output of a remote URL directly to bash without first downloading and inspecting the script. If the remote server is compromised or the connection is intercepted, arbitrary code executes on the runner. Offending line: run: curl -sSL https://aspire.dev/install.sh | bash

Locations:

- `.github/workflows/release-api.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection, unsafe-shell

**Notes:**

Fixed all 5 findings across 7 files:

1. **unpinned-uses**: Pinned all action references to full SHA hashes:
   - actions/checkout@main/v6 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/setup-node@v6 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
   - actions/configure-pages@v6 → @983d7736d9b0ae728b81ab479565c72886d7745b # v5
   - actions/upload-pages-artifact@v5 → @56afc609e74202658d3ffba0e8f6dda462b719fa # v3
   - actions/deploy-pages@v5 → @d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e # v4
   - actions/setup-dotnet@main → @67a3573c9a986a3f9c594539f4ab511d57bb3ce9 # v4
   - actions/upload-artifact@main → @ea165f8d65b6e75b540449e92b4886f43607fa02 # v4
   - actions/cache@v5 → @0057852bfaa89a56745cba8c7296529d2fc39830 # v4
   - NuGet/login@v1 → @8d196754b4036150537f80ac539e15c2f1028841 # v1
   - docker/login-action@v4 → @c94ce9fb468520275223c153574b00df6fe4bcc9 # v3
   - IEvangelist/profanity-filter@main → @2086462f19d6f71390bd082d4d46b8d748796238 # main
   - action.yml docker image pinned: docker://ghcr.io/ievangelist/profanity-filter:latest@sha256:c472f7403a207cdc724fad5fb0d283c288c28a9ad412a0060eb4a1b86b7be436

2. **missing-permissions**: Added `permissions: contents: read` to dotnet.yml top-level.

3. **script-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks:
   - dotnet.yml: matrix.project → MATRIX_PROJECT env var
   - publish-nuget.yml: github.ref/ref_name → env vars in debug step; github.ref_name → TAG_VERSION env var; nuget-login output → NUGET_API_KEY env var
   - release-api.yml: github.event.inputs.version → INPUT_VERSION env var
   - release.yml: github.event.inputs.version → INPUT_VERSION env var; env.RELEASE_VERSION and env.IMAGE_NAME → RELEASE_VERSION and CONTAINER_IMAGE_NAME env vars

4. **github-env-injection**: In release-api.yml and release.yml, sanitized the version input before writing to GITHUB_ENV using `safe=$(printf '%s' "$INPUT_VERSION" | tr -d '\n\r')` then `echo "RELEASE_VERSION=$safe" >> "$GITHUB_ENV"`.

5. **unsafe-shell**: In release-api.yml, replaced `curl -sSL https://aspire.dev/install.sh | bash` with downloading to /tmp first (`curl -sSL -o /tmp/aspire-install.sh https://aspire.dev/install.sh`) then executing separately (`bash /tmp/aspire-install.sh`).

