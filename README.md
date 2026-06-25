# NTMminer

Closed-source high-performance miner. Clean-room implementation; this repo distributes **binaries only** (source is closed).

- **NeuroMorph (nm/1)** — Cereblix (**CRB**), CPU.
- **midstate VDF** — midstate (**MDS**), **CPU and NVIDIA GPU (CUDA)**.

## Download

Get the binary for your platform from **[Releases](../../releases/latest)**.

| Platform | File |
|---|---|
| Linux x64 (incl. HiveOS) | `NTMminer-linux-x64` |
| Linux ARM64 | `NTMminer-linux-arm64` |
| Windows x64 | `NTMminer-windows-x64.exe` |
| macOS (Apple Silicon) | `NTMminer-macos-arm64` |
| **Linux x64 + NVIDIA GPU (CUDA)** | **`NTMminer-cuda-linux-x64`** |
| GPU consensus self-test (Linux x64) | `mid_gpu_selftest-linux-x64` |
| Offline benchmark (Linux x64) | `nm_bench-linux-x64` |

CPU Linux/macOS builds are **statically linked** — no dependencies. The CUDA build needs only the **NVIDIA driver** at runtime (CUDA runtime is linked statically; no toolkit install required).

## Usage

### CPU — NeuroMorph (CRB)
```
NTMminer -o <host:port> -u <wallet> [--threads N] [--lanes N]
```
```
chmod +x NTMminer-linux-x64
./NTMminer-linux-x64 -o asia.cereblix.com:3333 -u crb1YOUR_WALLET_ADDRESS
```

### GPU — midstate (MDS) on NVIDIA / CUDA
The midstate VDF is a GPU-friendly PoW (zero memory-hardness). Use the CUDA build with `--gpu`:
```
chmod +x NTMminer-cuda-linux-x64
./NTMminer-cuda-linux-x64 -a midstate --gpu -o 103.80.18.140:3333 -u <YOUR_MSS_ADDRESS_HEX64>
```
- `-a midstate` selects the midstate algorithm; `--gpu` drives the CUDA engine.
- `<YOUR_MSS_ADDRESS_HEX64>` = your 64-hex (32-byte) midstate payout address.
- Native cubins for **sm_70 (V100) → sm_120 (RTX 50)** + PTX fallback; runtime auto-tunes the GPU.
- The same `NTMminer-cuda-linux-x64` also mines on CPU (drop `--gpu`).

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

- **GPU (CUDA) backend for midstate** — multi-arch fatbin (V100 → Blackwell) + PTX fallback; consensus-verified byte-identical to the CPU scalar reference.
- **x86-64 register-pinning JIT** (NeuroMorph) — consensus-verified byte-identical to the reference.
- AES-NI / VAES (256/512) and AVX-512/AVX2 runtime CPUID dispatch.
- **Huge pages on by default** (zero-config THP on Linux).
- Cross-platform: Linux x64/arm64, Windows x64, macOS Apple Silicon (CPU); Linux x64 (GPU).

## Tips

- Large-core CPUs (EPYC etc.): try `--threads` = full SMT count; reserve hugepages for max throughput:
  `sudo sysctl -w vm.nr_hugepages=1024`
- midstate is GPU-dominant by design — a mid-range GPU outpaces a top desktop CPU several-fold.
