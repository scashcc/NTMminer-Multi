# NTMminer

Closed-source high-performance miner. Clean-room implementation; this repo distributes **binaries only** (source is closed).

- **NeuroMorph (nm/1)** — Cereblix (**CRB**), CPU.
- **Argon2id** — Blocknet (**BNT**), CPU.
- **midstate VDF** — midstate (**MDS**), **CPU and NVIDIA GPU (CUDA)**.

> **v1.7.1 — single binary, CPU + GPU.** The midstate GPU backend now uses the **CUDA Driver API**, so each platform ships **one** binary that does both CPU and GPU. **Windows now has GPU too** (same `--gpu` flag as Linux). No separate `-cuda` build anymore.

## Download

Get the binary for your platform from **[Releases](../../releases/latest)**.

| Platform | File | CPU | NVIDIA GPU |
|---|---|---|---|
| Linux x64 (incl. HiveOS) | `NTMminer-linux-x64` | ✅ | ✅ |
| Windows x64 | `NTMminer-windows-x64.exe` | ✅ | ✅ |
| Linux ARM64 | `NTMminer-linux-arm64` | ✅ | — |
| macOS (Apple Silicon) | `NTMminer-macos-arm64` | ✅ | — |
| GPU consensus self-test (Linux x64) | `mid_gpu_selftest-linux-x64` | | for NVIDIA GPU |
| Offline benchmark (Linux x64) | `nm_bench-linux-x64` | | |

- **GPU needs only the NVIDIA driver** at runtime — `nvcuda.dll` (Windows) / `libcuda.so.1` (Linux) is loaded dynamically. No CUDA toolkit install required.
- **Linux x64** is **dynamically linked** (needs glibc ≥ 2.35); this is required for the GPU to load the driver (same as SRBMiner/xmrig). Linux ARM64 is statically linked.

## Algorithms (`-a` / `--coin`)

| Algorithm | `-a` value (aliases) | Default port | GPU |
|---|---|---|---|
| NeuroMorph (Cereblix / CRB) | `neuromorph` (`nm`, `crb`, `cereblix`) | 3333 | CPU |
| Argon2id (Blocknet / BNT) | `argon2id-blocknet` (`bnt`, `blocknet`) | 3333 | CPU |
| MID (midstate VDF) | `midstate` (`mid`, `mds`) | 3333 | **CPU + GPU** |

## Usage

```
NTMminer -a <algo> -o <host:port> -u <wallet> [options]
```

| Option | Meaning |
|---|---|
| `-o, --url <host:port>` | Pool address (accepts a `stratum+tcp://` prefix) |
| `-u, --user, --address <wallet>` | Wallet / login address |
| `-a, --algo, --coin <name>` | Algorithm / coin (default `neuromorph`) |
| `-t, --threads <n>` | CPU thread count (default = physical cores; max 256) |
| `--smt` | Use all logical cores (SMT on). Default off — big-core CPUs are usually faster on physical cores + MLP |
| `--gpu` | Mine on **NVIDIA CUDA GPU** (midstate only; built into the binary, needs the NVIDIA driver) |
| `--lanes <n\|auto>` | MLP interleave lanes (**NeuroMorph only**). `auto` self-tunes; key speedup on big EPYC; `--lanes 1` disables |
| `--huge-pages <2m\|1g\|off>` | Huge pages (default auto `2m`) |
| `--worker, --rig-id <name>` | Worker name (optional) |
| `-p, --pass <pass>` | Login password (default `x`) |
| `--socks5 [user:pass@]host:port` | Mine through a SOCKS5 proxy |
| `--bench <secs>` | Run N seconds then exit (testing / benchmarking) |
| `-h, --help` / `-V, --version` | Help / version |

### Examples

```bash
# CPU — NeuroMorph (Cereblix / CRB)
./NTMminer-linux-x64 -a neuromorph -o asia.cereblix.com:3333 -u crb1YOUR_WALLET

# CPU — midstate (MDS)
./NTMminer-linux-x64 -a midstate -o <pool-host>:3333 -u <YOUR_MSS_ADDRESS_HEX64>

# GPU (NVIDIA) — midstate, SAME binary & command on Linux and Windows
./NTMminer-linux-x64       -a midstate --gpu -o <pool-host>:3333 -u <YOUR_MSS_ADDRESS_HEX64>
NTMminer-windows-x64.exe   -a midstate --gpu -o <pool-host>:3333 -u <YOUR_MSS_ADDRESS_HEX64>

# Big-core EPYC — NeuroMorph (MLP auto-tune + full SMT)
./NTMminer-linux-x64 -a neuromorph -o asia.cereblix.com:3333 -u crb1YOUR_WALLET --smt --lanes auto

# Through a SOCKS5 proxy
./NTMminer-linux-x64 -a neuromorph -o asia.cereblix.com:3333 -u crb1YOUR_WALLET --socks5 127.0.0.1:9050
```

- `-a midstate` selects the midstate algorithm; `--gpu` drives the CUDA engine.
- `<YOUR_MSS_ADDRESS_HEX64>` = your 64-hex (32-byte) midstate payout address.
- Native cubins for **sm_70 (V100) → sm_120 (RTX 50)** + PTX fallback; the GPU auto-tunes at startup.

**Verify GPU consensus (recommended before mining):**
```
chmod +x mid_gpu_selftest-linux-x64
./mid_gpu_selftest-linux-x64        # expects: ALL PASS — GPU VDF == CPU scalar byte-for-byte
```

### Benchmark (no pool needed)
```
chmod +x nm_bench-linux-x64
./nm_bench-linux-x64 15           # 15s, all cores
```

## Features

- **Single binary, CPU + GPU** — midstate GPU runs via the CUDA Driver API (runtime `dlopen` of the driver), so one binary per platform does both; **Windows GPU supported**.
- **GPU (CUDA) for midstate** — multi-arch fatbin (V100 → Blackwell) + PTX fallback; consensus-verified byte-identical to the CPU scalar reference on both sm_70 (V100) and sm_86 (RTX 30).
- **x86-64 register-pinning JIT** (NeuroMorph) — consensus-verified byte-identical to the reference.
- AES-NI / VAES (256/512) and AVX-512/AVX2 runtime CPUID dispatch.
- **Huge pages on by default** (zero-config THP on Linux).
- Cross-platform: Linux x64/arm64, Windows x64, macOS Apple Silicon.

## Tips

- Large-core CPUs (EPYC etc.): try `--threads` = full SMT count; reserve hugepages for max throughput:
  `sudo sysctl -w vm.nr_hugepages=1024`
- midstate is GPU-dominant by design — a mid-range GPU outpaces a top desktop CPU several-fold.
