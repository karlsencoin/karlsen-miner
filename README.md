# KarlsenMiner v2.4

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)]()
[![GPU](https://img.shields.io/badge/GPU-AMD%20%2B%20NVIDIA-green.svg)]()
[![Join the Karlsen Discord Server](https://img.shields.io/discord/1169939685280337930.svg?label=&logo=discord&logoColor=ffffff)](https://discord.gg/ZPZRvgMJDT)

A high-performance GPU miner for **Karlsen (KLS)** using the **KarlsenHashV2** algorithm (FishHash + Blake3).  
Dual-backend: AMD OpenCL + NVIDIA CUDA in a single binary.

## Features

- **Dual GPU Backend** — AMD (OpenCL) and NVIDIA (CUDA) auto-detected, no separate builds
- **Single Executable** — Static linked, no DLL dependencies, no external files
- **Multi-Pool Failover** — Up to 4 pools with automatic switchover
- **SSL/TLS Support** — Native Windows SChannel, no OpenSSL dependency
- **Per-GPU Tuning** — Lock core clock, memory clock, and power limit per GPU via nvidia-smi
- **GPU Monitoring** — Temperature, power, voltage, clocks, fan speed, efficiency (MH/s/W)
- **Thermal Protection** — Auto-pause/resume individual GPUs at configurable temperature thresholds
- **GPU Fault Tolerance** — Faulty GPU is disabled automatically, remaining GPUs continue mining
- **Device Filtering** — Select `all`, `amd`, `nvidia`, or specific GPU indices

## Quick Start

```bat
karlsenminer.exe --pool stratum+ssl://pool.woolypooly.com:3132 --user YOUR_WALLET.worker1
```

### Example: 6 AMD GPUs with thermal protection
```bat
karlsenminer.exe ^
  --pool stratum+ssl://pool.woolypooly.com:3132 ^
  --user karlsen:qz...abc.rig1 ^
  -d 0,1,2,3,4,5 ^
  --gpu-off-temp 80
```

### Example: 2 NVIDIA GPUs with per-GPU tuning (run as administrator)
```bat
karlsenminer.exe ^
  --pool stratum+ssl://pool.woolypooly.com:3132 ^
  --user karlsen:qz...abc.mine5 ^
  -d 2,3 ^
  --cclk 1210,1110 ^
  --mclk 9501,6801 ^
  --pl 190,150 ^
  --tstop 85
```

## Download

Pre-built binaries are available on the [Releases](https://github.com/karlsencoin/karlsen-miner/releases) page.

Each release contains:
| File | Description |
|------|-------------|
| `karlsenminer.exe` | Miner binary (single file, no dependencies) |
| `mine_kls.bat` | Example startup script |
| `README.txt` | Full documentation with all options |

## Status Display

```
KarlsenMiner v2.4 | Pool: pool.woolypooly.com:3132 | Uptime: 20h 13m 8s | Diff: 3.9874
═══════════════════════════════════════════════════════════════════════════════════
GPU  Name              Speed         A/R       Eff.   Power   Volt   CCLK   MCLK   Core   Junc    Fan
---  ----              -----         ---       ----   -----   ----   ----   ----   ----   ----    ---
  0  gfx1031           28.60 MH/s    90/1     0.441    65W    662    976   1994    45C    48C    60%
  1  gfx1031           28.69 MH/s    97       0.441    65W    700    954   1986    42C    44C    59%
  2  gfx1031           28.70 MH/s    87       0.442    65W    687    960   1986    44C    46C    59%
  3  gfx1031           28.69 MH/s    99/4     0.435    66W    700    955   1988    47C    50C    58%
  4  gfx1031           28.68 MH/s    85       0.448    64W    687    951   1988    41C    43C    58%
  5  gfx1031           28.64 MH/s    85       0.428    67W    687    965   1990    48C    50C    61%
═══════════════════════════════════════════════════════════════════════════════════
Total: 171.00 MH/s (avg 172.21 MH/s) | A:552/R:5 | 392W | 0.438 | Diff: 3.9874
```

## All Options

### Pool & Authentication
| Option | Description |
|--------|-------------|
| `--pool URL` | Pool address (repeat for failover). Formats: `host:port`, `stratum+ssl://host:port` |
| `--user WORKER` | Wallet address and worker name |
| `--pass PASSWORD` | Pool password (default: `x`) |
| `--tls on\|off` | Force SSL/TLS on all connections |

### Device Selection
| Option | Description |
|--------|-------------|
| `-d all` | Use all GPUs (default) |
| `-d amd` | AMD GPUs only (OpenCL) |
| `-d nvidia` | NVIDIA GPUs only (CUDA) |
| `-d 0,1,2` | Specific GPU indices |

### NVIDIA GPU Tuning (requires administrator)
| Option | nvidia-smi | Description |
|--------|-----------|-------------|
| `--cclk MHz[,MHz,...]` | `-lgc` | Lock core clock per GPU |
| `--mclk MHz[,MHz,...]` | `-lmc` | Lock memory clock per GPU |
| `--pl W[,W,...]` | `-pl` | Power limit per GPU |

> **Note:** Clock offsets (`--coff`, `--moff`) and fan control (`--fan`) are not supported by nvidia-smi. Use MSI Afterburner for these features.

> **Note:** nvidia-smi GPU indices only count NVIDIA GPUs (0-based). Check the startup log to verify GPU index mapping.

### Thermal Protection
| Option | Description |
|--------|-------------|
| `--tstop T` | Pause GPU at T°C (alias: `--gpu-off-temp`) |
| `--gpu-on-temp T` | Resume GPU at T°C (default: tstop - 20) |

### Mining
| Option | Description |
|--------|-------------|
| `-w N` | Nonces per batch (default: 4194304) |
| `--benchmark` | Test hashrate without pool |
| `--log on` | Enable file logging |
| `--logfile PATH` | Log file path (default: `karlsenminer.log`) |

## Building from Source

### Requirements
- CMake 3.15+
- Visual Studio 2022 (MSVC)
- CUDA Toolkit 12.x
- OpenCL SDK (for AMD support)

### Build
```bat
git clone https://github.com/karlsencoin/karlsen-miner.git
cd karlsen-miner
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release
```

Output: `build\Release\karlsenminer.exe` — single file, no DLLs needed (static MSVC runtime).

## Supported Hardware

### NVIDIA (CUDA)
Any NVIDIA GPU with 6+ GB VRAM and CUDA compute capability 5.0+:
- GeForce RTX 20xx, 30xx, 40xx, 50xx series
- CMP 90HX, 170HX
- Tesla, Quadro (datacenter)

### AMD (OpenCL)
Any AMD GPU with 6+ GB VRAM and OpenCL 2.0 support:
- RX 6600/6700/6800/6900 (RDNA2)
- RX 7600/7700/7800/7900 (RDNA3)
- RX 9070 (RDNA4)

## Algorithm

KarlsenHashV2 is based on [FishHashPlus](https://github.com/karlsen-network/karlsend/blob/mainnet_karlsenhashv2/domain/consensus/utils/pow/fishhashplus_kernel.go) — an ASIC-resistant, memory-intensive proof-of-work algorithm. It generates a 4.5 GB DAG in GPU VRAM, making it GPU-friendly while being resistant to specialized mining hardware.

The hashing pipeline:
1. **Blake3** — Header hash
2. **FishHash lookup** — Memory-hard DAG access (random 128-byte reads)
3. **Keccak** — Final hash and difficulty check

## Related Projects

- [karlsend](https://github.com/karlsen-network/karlsend) — Karlsen full node (Go)
- [rusty-karlsen](https://github.com/karlsen-network/rusty-karlsen) — Karlsen full node (Rust)
- [karlsen-stratum-bridge](https://github.com/karlsen-network/karlsen-stratum-bridge) — Stratum bridge for pools
- [karlsenai](https://github.com/karlsencoin/karlsenai) — KarlsenAI GPU inference workers

## License

MIT License — see [LICENSE](LICENSE) for details.

## Community

- [Discord](https://discord.gg/ZPZRvgMJDT)
- [Telegram](https://t.me/KarlsenNetwork)
- [Reddit](https://www.reddit.com/r/KarlsenNetwork/)
- [Website](https://karlsencoin.com/)
