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
