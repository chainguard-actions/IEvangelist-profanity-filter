<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/13.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **IEvangelist--profanity-filter/13.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image pinned to a mutable ':latest' tag instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on each run. The image reference 'docker://ghcr.io/ievangelist/profanity-filter:latest' should be replaced with a SHA-pinned reference such as 'docker://ghcr.io/ievangelist/profanity-filter@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable ':latest' Docker image tag in action.yml (line 44) with the immutable SHA256 digest: 'docker://ghcr.io/ievangelist/profanity-filter@sha256:23ffe1e4e2e72dece675fc6a3ad539e61079e1216d3a928b9ae28f4e04a75ea1' # latest. The comment preserves the original tag name for readability.

