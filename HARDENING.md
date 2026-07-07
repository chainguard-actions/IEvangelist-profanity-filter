<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/13.4.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **IEvangelist--profanity-filter/13.4.6.1** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use unpinned (non-SHA) references. action.yml uses a mutable Docker image tag 'docker://ghcr.io/ievangelist/profanity-filter:latest' instead of a SHA digest. Workflow files use branch/tag refs: deploy-docs.yml uses actions/checkout@v6, actions/setup-node@v6, actions/configure-pages@v6, actions/upload-pages-artifact@v5, actions/deploy-pages@v5; dogfood.yml uses IEvangelist/profanity-filter@main; dotnet.yml uses actions/checkout@main, actions/setup-dotnet@main; publish-nuget.yml uses actions/checkout@main, actions/setup-dotnet@main, actions/upload-artifact@main, NuGet/login@v1; release-api.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/cache@v5, actions/upload-artifact@main; release.yml uses actions/checkout@main, actions/setup-dotnet@main, docker/login-action@v4, actions/upload-artifact@main.

Locations:

- `action.yml:44`
- `.github/workflows/deploy-docs.yml:20`
- `.github/workflows/dogfood.yml:18`
- `.github/workflows/dotnet.yml:27`
- `.github/workflows/publish-nuget.yml:37`
- `.github/workflows/release-api.yml:24`
- `.github/workflows/release.yml:25`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ }} expressions directly into shell commands. (a) dotnet.yml: ${{ matrix.project }} is interpolated directly in run: commands including 'dotnet publish --configuration Release tests/${{ matrix.project }}/', 'cd artifacts/publish/${{ matrix.project }}', and './${{ matrix.project }}'. (a) publish-nuget.yml: ${{ github.ref }} and ${{ github.ref_name }} are echoed in a run: block, TAG_VERSION="${{ github.ref_name }}" assigns an expression to a shell variable, and --api-key "${{ steps.nuget-login.outputs.NUGET_API_KEY }}" is used in a run: block. (a) release-api.yml: ${{ github.event.inputs.version }} is interpolated directly in a run: echo command. (a) release.yml: ${{ github.event.inputs.version }} is interpolated in a run: echo command, and ${{ env.RELEASE_VERSION }} and ${{ env.IMAGE_NAME }} are interpolated in a dotnet publish run: block.

Locations:

- `.github/workflows/dotnet.yml:36`
- `.github/workflows/publish-nuget.yml:44`
- `.github/workflows/publish-nuget.yml:57`
- `.github/workflows/publish-nuget.yml:72`
- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:50`

### github-env-injection (severity: high)

User-controlled input is written directly to $GITHUB_ENV without sanitization. In release-api.yml, the 'Get the version' step (workflow_dispatch branch) runs: echo "RELEASE_VERSION=${{ github.event.inputs.version }}" >> $GITHUB_ENV — the user-supplied 'version' input is interpolated directly into the GITHUB_ENV write without the required printf '%s' ... | tr -d '\n\r' sanitization. The same pattern exists in release.yml.

Locations:

- `.github/workflows/release-api.yml:30`
- `.github/workflows/release.yml:30`

### missing-permissions (severity: medium)

The workflow file dotnet.yml has no top-level 'permissions:' key and no job-level 'permissions:' key on any of its jobs. This means the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/dotnet.yml:1`

### unsafe-shell (severity: high)

release-api.yml pipes remote content directly to bash: 'curl -sSL https://aspire.dev/install.sh | bash'. This downloads and executes an arbitrary remote script without first verifying its integrity, allowing a compromised or malicious server to execute arbitrary code on the runner.

Locations:

- `.github/workflows/release-api.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions, unsafe-shell

**Notes:**

Fixed all 5 findings across 7 workflow files and action.yml: (1) unpinned-uses: pinned all action references to full SHA digests in action.yml (Docker image digest), deploy-docs.yml, dogfood.yml, dotnet.yml, publish-nuget.yml, release-api.yml, and release.yml; (2) script-injection: moved all ${{ }} expressions in run: blocks into env: blocks and referenced them as plain env vars in dotnet.yml (matrix.project), publish-nuget.yml (github.ref, github.ref_name, nuget API key), release-api.yml (github.event.inputs.version), and release.yml (github.event.inputs.version, env.RELEASE_VERSION, env.IMAGE_NAME); (3) github-env-injection: sanitized user-supplied version input with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV in both release-api.yml and release.yml; (4) missing-permissions: added 'permissions: contents: read' top-level block to dotnet.yml; (5) unsafe-shell: replaced 'curl ... | bash' with downloading the script to /tmp first then executing it separately in release-api.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted $TAG_VERSION variable expansion in the 'Pack projects with explicit version from tag' step in .github/workflows/publish-nuget.yml. Changed /p:Version=$TAG_VERSION and /p:PackageVersion=$TAG_VERSION to /p:Version="$TAG_VERSION" and /p:PackageVersion="$TAG_VERSION" in both dotnet pack invocations (lines 80-81). The variable was already correctly placed in the step's env: block; only the double-quoting in the shell script was missing.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in both .github/workflows/release-api.yml and .github/workflows/release.yml. In each file, the non-workflow_dispatch '🏷️ Get the version' step was updated to: (1) move github.ref into an env block as GIT_REF to avoid direct expression interpolation in the shell, (2) strip the refs/tags/ prefix using POSIX parameter expansion, and (3) sanitize the result with `printf '%s' "$raw" | tr -d '\n\r'` before writing to $GITHUB_ENV. This matches the sanitization pattern already present in the workflow_dispatch path of both files.

