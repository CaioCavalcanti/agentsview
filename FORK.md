# Fork policy — fenced-JSON tolerance for the extraction client

This repository is a downstream fork of
[kenn-io/agentsview](https://github.com/kenn-io/agentsview). It exists for
one patch and is meant to be retired.

## Why the fork exists

Some OpenAI-compatible gateways accept `response_format: json_schema` but
answer with the payload wrapped in a markdown code fence, optionally with
prose around it. Upstream's extraction client treats such responses as
protocol violations, so recall extraction can never succeed against those
endpoints. This fork normalizes only a leading fence before the unchanged
strict validation; truncated, non-JSON, or prose-only responses still fail
closed.

## Base and patch

- Upstream base: tag `v0.43.0`, commit `9be7745ad1906ee24e04eb05bb86c872ef0939a1`.
- Patch branch: `fenced-json-0.43.0`, one code commit on top of the base
  (fence-strip normalization, `Client.Warnf` hook, extraction tests).
- Fork plumbing (this file and `.github/workflows/publish-fork-ghcr.yml`)
  sits in its own commit so the code patch stays rebase-clean.
- Image: `ghcr.io/caiocavalcanti/agentsview:0.43.0-fencedjson-1`, built by
  the workflow on every push to the patch branch and via manual dispatch.

## Rebase procedure (per upstream release)

1. Add upstream as a remote and fetch:
   `git remote add upstream https://github.com/kenn-io/agentsview.git && git fetch upstream --tags`
2. Rebase the patch branch onto the new tag:
   `git rebase --onto vX.Y.Z v0.43.0 fenced-json-0.43.0`
   (resolve conflicts in `internal/recall/extract/client.go`, keeping
   upstream's validation logic intact — the patch only inserts the
   fence-strip step before `parseEntries`).
3. Update the image tag: in
   `.github/workflows/publish-fork-ghcr.yml`, bump
   `0.43.0-fencedjson-1` to `X.Y.Z-fencedjson-1`.
4. Push and let the workflow publish, then re-pin the compose digest via
   `docker buildx imagetools inspect` (see the harness `fork-agentsview/README.md`).

## Drop condition

Drop this fork as soon as upstream tolerates fenced responses (or enforces
`response_format` server-side tracking makes it moot): revert the harness
compose pins to the upstream image and archive this fork. Rebase the fork
only while the fence gap exists; never carry unrelated patches here.
