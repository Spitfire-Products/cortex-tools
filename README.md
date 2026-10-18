# cortex-tools

Prebuilt binaries for the in-house projects (nexus-cortex harness, the bench boxes, the hosted sandbox images). Source stays in the
upstream repositories; this repo only holds the build recipe, the upstream pin, and the GitHub Releases that every consumer downloads
by pinned version with a sha256 check.

| tool | upstream | pin | release | assets |
|---|---|---|---|---|
| sgrep | https://github.com/context-dot-dev/sgrep (MIT) | `tools/sgrep/PIN` | `sgrep-0.3.0` | `sgrep-0.3.0-x86_64-linux-gnu`, `sgrep-0.3.0-x86_64-linux-musl` (static, Alpine), `sgrep-model-cache-potion-code-16M-v2.tgz` (the 32 MB model sgrep downloads on first use; untar into `~/.cache`), `SHA256SUMS` |

## Consuming a release (pattern every consumer follows)

```sh
V=0.3.0; T=x86_64-linux-gnu   # or x86_64-linux-musl on Alpine
B=https://github.com/Spitfire-Products/cortex-tools/releases/download/sgrep-$V
curl -fsSL -o /usr/local/bin/sgrep $B/sgrep-$V-$T && curl -fsSL $B/SHA256SUMS | grep " sgrep-$V-$T$" | sed "s# .*# /usr/local/bin/sgrep#" | sha256sum -c -
chmod a+rx /usr/local/bin/sgrep
```

## Rebuilding

`Actions → build-sgrep → Run workflow` with the upstream ref and the version. The workflow builds both targets on a standard runner
(public repo: no billed minutes), writes `SHA256SUMS`, and publishes the release. Nothing runs on a schedule and nothing runs on push;
a build happens only when a human dispatches it. The first release (`sgrep-0.3.0`, 2026-10-18) was built on a Vast box from the
default branch at the pinned commit and uploaded by hand with the same asset names.

## Backlog (tools considered for this store, not yet released)

| tool | why | how it would ship |
|---|---|---|
| `fd` ([sharkdp/fd](https://github.com/sharkdp/fd), MIT/Apache-2.0) | file-NAME finder to pair with ripgrep's content search; today the harness relies on `rg --files -g PATTERN` and `find`, which work but are slower to write and to run on big trees | no build needed: upstream publishes `fd-vX.Y.Z-x86_64-unknown-linux-{gnu,musl}.tar.gz`; a release here would mirror the pinned asset's binary under our naming + `SHA256SUMS`, then one row in the nexus-cortex `TOOLS_MANIFEST` (`cortex tools install`) and the sandbox Dockerfiles. Added 2026-10-18 while the sgrep steering A/B was running; decide after that readout says whether search steering moves outcomes at all. |
| ~~sgrep model cache: add `potion-base-8M`~~ DONE 2026-10-18: `sgrep-model-cache-v2.tgz` (both models) is on the `sgrep-0.3.0` release; the workflow packs both from now on | sgrep's `--model auto` picks the TEXT embedding model (`minishlab/potion-base-8M`) for prose-only candidates and downloads it on first use; the vendored `sgrep-model-cache-potion-code-16M-v2.tgz` carries only the code model, so a prose corpus inside an offline container stalls on that download | extend the `Model cache` step of `build-sgrep.yml` to run one prose query too, then re-tar `~/.cache/huggingface` (both models) and re-pin the asset's sha in the bench bootstrap / adapter. Found 2026-10-18 measuring index reuse. |
