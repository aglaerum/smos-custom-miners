# smos-custom-miners

This repo holds miners repackaged for SimpleMining OS (SMOS) group configs that use Miner
Program **CUSTOM**.

## Why

On some rigs, SMOS fails every normal miner download with:

```
md5sum: t-rex-v0.26.8.tar.gz.md5: no properly formatted MD5 checksum lines found
ERROR: Miner checksum MD5 failed.
```

SimpleMining's files are fine. The problem is how SMOS (`update_miner.sh` in SM-i073) fetches
them: it downloads the `.md5` with `curl` and **without** `-L`, so a redirect or any other
non-file response breaks the check.

The CUSTOM route avoids this:

- SMOS downloads the ZIP with `curl -L`, which follows redirects.
- It checks the ZIP with `unzip -t`.
- It computes the md5 itself.

## Releases

| Tag | ZIP | Contents | `miner` sha256 |
|---|---|---|---|
| `trex-0.26.8` | `t-rex-0.26.8-smos.zip` | T-Rex 0.26.8 (NVIDIA) | `decd1f03573fe4b7171af8edd3c342799be4e6b4431ed8c2a6ed9c3728af5bad` |

Each binary is taken **unchanged** from SimpleMining's own package
(`https://download.simplemining.net/miners/<pkg>.tar.gz`, after checking it against their
`.md5`) and renamed to `miner`.

## Use in SMOS

Create a group config with Miner Program **CUSTOM**. The first token of the options is the
ZIP URL, and everything after it goes to the miner:

```
https://github.com/aglaerum/smos-custom-miners/releases/download/trex-0.26.8/t-rex-0.26.8-smos.zip -a kawpow -o stratum+tcp://<pool>:<port> -u $wallet -w $rigName -p x
```

## ZIP rules (enforced by SMOS)

- The URL must end in `.zip`.
- The ZIP must have exactly one top-level folder and no loose files.
- That folder must contain an executable named `miner`.

## Rebuilding

A ZIP's hash changes with every build because of timestamps, so compare the `miner` sha256
instead. The build is a few lines:

```bash
curl -fsSLO https://download.simplemining.net/miners/t-rex-v0.26.8.tar.gz
tar -xzf t-rex-v0.26.8.tar.gz
mkdir -p zip/t-rex-0.26.8 && cp t-rex-v0.26.8/t-rex zip/t-rex-0.26.8/miner
(cd zip && zip -r -X ../t-rex-0.26.8-smos.zip t-rex-0.26.8)
```

T-Rex is closed-source freeware by its own authors, and it takes a 1 % dev fee. This repo
only repackages it for SMOS.
