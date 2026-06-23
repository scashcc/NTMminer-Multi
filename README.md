# NTMminer

Closed-source high-performance CPU miner for **NeuroMorph (nm/1)** PoW — Cereblix (**CRB**).
Clean-room implementation; this repo distributes **binaries only** (source is closed).

## Download

Get the binary for your platform from **[Releases](../../releases/latest)**.

| Platform | File |
|---|---|
| Linux x64 (incl. HiveOS) | `NTMminer-linux-x64` |
| Linux ARM64 | `NTMminer-linux-arm64` |
| Windows x64 | `NTMminer-windows-x64.exe` |
| macOS (Apple Silicon) | `NTMminer-macos-arm64` |
| Offline benchmark (Linux x64) | `nm_bench-linux-x64` |

Linux/macOS builds are **statically linked** — no dependencies.

## Usage

```
NTMminer <pool_host> <pool_port> <wallet> [run_seconds] [threads] [lanes]
```

Mine CRB on Cereblix:
```
chmod +x NTMminer-linux-x64
# auto threads (all logical cores), run forever:
./NTMminer-linux-x64 asia.cereblix.com 3333 crb1YOUR_WALLET_ADDRESS
# explicit threads (e.g. 96C/192T EPYC): run_seconds=0 = forever
./NTMminer-linux-x64 asia.cereblix.com 3333 crb1YOUR_WALLET_ADDRESS 0 192 1
```

- `run_seconds`: `0` (or omit) = run forever
- `threads`: **omit = auto (all logical cores)**; set explicitly for tuning
- `lanes`: MLP batch per thread (default `1`; try `2` on big-L3 CPUs)

### Benchmark (no pool needed)
```
chmod +x nm_bench-linux-x64
./nm_bench-linux-x64 15           # 15s, all cores
./nm_bench-linux-x64 15 192 1     # 15s, 192 threads, 1 lane
```

## Features

- **x86-64 register-pinning JIT** — consensus-verified byte-identical to the reference
- AES-NI / VAES (256/512) with runtime CPUID dispatch
- **Huge pages on by default** (zero-config THP on Linux — no manual hugepage setup)
- Cross-platform: Linux x64/arm64, Windows x64, macOS Apple Silicon
- **0% dev fee** for CRB

## Tips

- Large-core CPUs (EPYC etc.): try threads = full SMT count; reserve hugepages for max throughput:
  `sudo sysctl -w vm.nr_hugepages=1024`
- Disable huge pages for comparison: `NM_NOHUGE=1 ./nm_bench-linux-x64 15`
