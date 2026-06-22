<!-- markdownlint-disable -->

# Hardening Report: IEvangelist--profanity-filter/10.0.1-preview.009

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **IEvangelist--profanity-filter/10.0.1-preview.009** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references a Docker image using a mutable tag (:latest) instead of a SHA digest. The image 'docker://ghcr.io/ievangelist/profanity-filter:latest' can be silently replaced with a different (potentially malicious) image at any time, enabling supply-chain attacks. It should be pinned to a specific SHA digest, e.g. 'docker://ghcr.io/ievangelist/profanity-filter@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image 'ghcr.io/ievangelist/profanity-filter:latest' to its immutable SHA256 digest 'sha256:23ffe1e4e2e72dece675fc6a3ad539e61079e1216d3a928b9ae28f4e04a75ea1' in action.yml line 44. The comment '# latest' is preserved outside the YAML quotes for readability.

