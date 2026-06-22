<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.010

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **IEvangelist--profanity-filter/10.0.1-preview.010** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag (`docker://ghcr.io/ievangelist/profanity-filter:latest`) instead of a SHA digest. This means the action could silently pull a different (potentially malicious) image on each run. It should be pinned to a specific SHA digest, e.g. `docker://ghcr.io/ievangelist/profanity-filter@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable `docker://ghcr.io/ievangelist/profanity-filter:latest` image reference in action.yml (line 43) with the pinned digest `docker://ghcr.io/ievangelist/profanity-filter@sha256:23ffe1e4e2e72dece675fc6a3ad539e61079e1216d3a928b9ae28f4e04a75ea1` (with `# latest` comment for readability). The digest was resolved via the Docker Registry HTTP API v2.

