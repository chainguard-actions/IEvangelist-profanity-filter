<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.008

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **IEvangelist--profanity-filter/10.0.1-preview.008** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml Docker image reference uses a mutable ':latest' tag instead of a SHA digest. This means the action could silently pull a different (potentially malicious) image on each run. The failing reference is: `image: 'docker://ghcr.io/ievangelist/profanity-filter:latest'`. It should be pinned to a specific SHA digest, e.g. `image: 'docker://ghcr.io/ievangelist/profanity-filter@sha256:<64-hex-char-digest>'`.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable ':latest' Docker image tag in action.yml line 44 with the pinned SHA digest: 'docker://ghcr.io/ievangelist/profanity-filter@sha256:23ffe1e4e2e72dece675fc6a3ad539e61079e1216d3a928b9ae28f4e04a75ea1' # latest. The comment preserves the original tag name for readability.

