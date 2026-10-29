# KarlsenMiner v3.2.0

[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20HiveOS-blue.svg)](https://github.com/karlsencoin/karlsen-miner/releases)
[![GPU](https://img.shields.io/badge/GPU-AMD%20%2B%20NVIDIA%20%2B%20Intel%20Arc-green.svg)]()
[![Join the Karlsen Discord Server](https://img.shields.io/discord/1169939685280337930.svg?label=&logo=discord&logoColor=ffffff)](https://discord.gg/QyrvshRBJV)

A high-performance GPU miner for **Karlsen (KLS)** using the **KarlsenHashV2** algorithm (FishHashPlus).
AMD and Intel Arc through OpenCL, NVIDIA through CUDA, in a single binary.

This repository publishes the release binaries and documentation.

## Features

- **One binary for every GPU** — AMD (OpenCL), Intel Arc (OpenCL) and NVIDIA (CUDA) are detected automatically
- **One thread per GPU** — batches are pipelined so a GPU never waits for the host; workload is tuned automatically (~100 ms per batch)
- **CPU-verified shares** — every share is re-hashed on the CPU before it is sent, so a faulty GPU shows up as HW errors instead of pool rejects
- **Setup wizard** — first start asks for wallet, pool, GPUs, thermal limits and (optionally) NVIDIA tuning; saved to `karlsenminer_config.json`
- **NVIDIA tuning** — power limit, core / memory clock lock, core / memory clock offset and fan speed per card, restored when the miner exits
- **Multi-pool failover** — up to 4 pools with automatic switchover
- **SSL/TLS** — built-in, certificates verified the same way on Windows and Linux
- **GPU monitoring** — temperature, junction, power, voltage, clocks, fan, efficiency (MH/s per W)
- **Thermal protection** — pauses a hot GPU and resumes it when it cools down
- **Fault tolerance** — a failing GPU is dropped, the others keep mining
- **Ready for 10 BPS** — shares stay submittable for 8 s regardless of block rate

## Download

Pre-built binaries are on the [Releases](https://github.com/karlsencoin/karlsen-miner/releases) page:

| File | Platform |
|------|----------|
| `karlsenminer-v3.2.0-windows.zip` | Windows 10/11 x64 |
| `karlsenminer-3.2.0.tar.gz` | HiveOS custom miner |
| `karlsenminer-v3.2.0-linux.tar.gz` | Linux x64 (glibc 2.25+) |
| `SHA256SUMS.txt` | Checksums of the files above |

Requirements: GPU with 6 GB+ VRAM (the DAG is ~4.6 GB); NVIDIA driver R525 or newer.

## Quick Start

### Windows — setup wizard
Unzip and run `karlsenminer.exe`. The wizard asks for:
1. Your Karlsen wallet address and rig name, and a pool
2. Which GPUs to mine on and the thermal limits
3. NVIDIA tuning (only when an NVIDIA card is present; optional)

Run `karlsenminer.exe --wizard` any time to change the settings.

### Command line
```bat
karlsenminer.exe -o stratum+ssl://pool.woolypooly.com:3132 -u YOUR_WALLET.rig1
```

### Example: AMD GPUs with thermal protection
```bat
karlsenminer.exe ^
  -o stratum+ssl://pool.woolypooly.com:3132 ^
  -u karlsen:qz...abc.rig1 ^
  -d amd ^
  --gpu-off-temp 80
```

### Example: NVIDIA GPUs with tuning
```bat
karlsenminer.exe ^
  -o stratum+ssl://pool.woolypooly.com:3132 ^
  -u karlsen:qz...abc.rig2 ^
  -d nvidia ^
  --pl 190,150 ^
  --cclk 1200 ^
  --moff 250 ^
  --tstop 85
```
`--pl 190,150` gives the first NVIDIA card 190 W and the second 150 W; `--cclk 1200` applies to every NVIDIA card.

### HiveOS
Flight sheet: Miner = **Custom**, Installation URL = the `karlsenminer-3.2.0.tar.gz` link from the release, Hash algorithm = `karlsenhashv2`, Wallet and worker template = `%WAL%.%WORKER_NAME%`, Pool URL = e.g. `stratum+ssl://pool.woolypooly.com:3132`, Extra config arguments (optional) = e.g. `-d amd --tstop 85`.

### Linux
```bash
tar xzf karlsenminer-v3.2.0-linux.tar.gz
cd karlsenminer-v3.2.0-linux
./karlsenminer -o stratum+ssl://pool.woolypooly.com:3132 -u YOUR_WALLET.rig1
```

## NVIDIA Tuning

Set it in the wizard (per card, or the same values on every card) or on the command line. Command-line values override the saved ones.

| Setting | Option | Unit | Notes |
|---------|--------|------|-------|
| Power limit | `--pl` | W | Clamped to the card's allowed range |
| Core clock lock | `--cclk` | MHz | Best for efficiency |
| Core clock offset | `--coff` | MHz | e.g. `-200` or `100` |
| Memory clock lock | `--mclk` | MHz | RTX 30 / 40 and newer |
| Memory clock offset | `--moff` | MHz | Use this on Turing (RTX 20) and CMP cards |
| Fan speed | `--fan` | % | `0` = driver fan curve |

- **One value or a list:** a single value applies to every selected NVIDIA card; a comma list sets cards in the order the miner lists them (a shorter list repeats its last value). Only cards selected with `-d` are touched; AMD cards are never changed.
- **Administrator rights:** tuning needs them. On Windows the miner asks once through UAC and restarts itself; if you decline, it keeps mining without tuning. On Linux run it as root (HiveOS already does).
- **Restore on exit:** power limit, clock locks, offsets and fan speed go back to their previous values when the miner exits with Ctrl+C. Closing the window with ✕ or killing the process skips this; reboot or reset the card in that case.
- **Turing and CMP:** these cards drop to a lower memory state under CUDA, so a memory lock has no effect. Use a memory offset.
- Offsets need NVIDIA driver R510 or newer.

## Status Display

```
   KarlsenMiner v3.2.0 | Pool: pool.woolypooly.com:3132 | Uptime: 8h 36m 1s | Diff: 4.0358
  ===========================================================================================================
   GPU  Name              Speed         A/R         HW    Eff.   Power   Volt   CCLK   MCLK   Core   Junc   Fan
     0  RX 6700 XT        29.27 MH/s    51             0   0.353     83W    700   1083   1990    43C    46C   77%
     1  RX 6700 XT        29.10 MH/s    55/2           0   0.401     73W    737   1077   1988    39C    40C   67%
  ===========================================================================================================
   Total: 58.37 MH/s (avg 58.12 MH/s) | A:106/R:2 | 0.375 | 156W | Best: 4.21
  ===========================================================================================================
```

Speed column states: a yellow speed means the card is noticeably slower than the rest; `THERMAL` means it is paused by thermal protection; `FAILED` means it was dropped after an error.

## All Options

### Pool & Authentication
| Option | Description |
|--------|-------------|
| `-o, --pool URL` | Pool address; repeat up to 4 times for failover. Formats: `host:port`, `stratum+tcp://`, `stratum+ssl://` |
| `-u, --user WALLET[.RIG]` | Wallet address and rig (worker) name |
| `-p, --pass PASSWORD` | Pool password (default: `x`) |
| `--tls on\|off` | Force TLS on every pool |

### Device Selection
| Option | Description |
|--------|-------------|
| `-d all` | All GPUs (default) |
| `-d amd` / `-d nvidia` / `-d intel` | One vendor only |
| `-d amd:0,1` / `-d nvidia:0` | Specific cards of one vendor (indexes count only that vendor) |
| `-d 0,1,3` | Indexes as printed at start-up |

### NVIDIA Tuning
| Option | Description |
|--------|-------------|
| `--pl W[,W,...]` | Power limit |
| `--cclk MHz[,MHz,...]` | Core clock lock |
| `--coff MHz[,MHz,...]` | Core clock offset |
| `--mclk MHz[,MHz,...]` | Memory clock lock |
| `--moff MHz[,MHz,...]` | Memory clock offset |
| `--fan %[,%,...]` | Fan speed (`0` = driver curve) |

### Thermal Protection
| Option | Description |
|--------|-------------|
| `--gpu-off-temp T` | Pause a GPU at T °C core temperature (alias: `--tstop`) |
| `--gpu-on-temp T` | Resume at T °C (default: off temperature − 20) |

### Mining & Misc
| Option | Description |
|--------|-------------|
| `-w, --workload N` | Nonces per batch per GPU (default: automatic) |
| `--benchmark` | Mine a synthetic job without a pool and report hashrate |
| `--log on\|off` | Log to file |
| `--logfile PATH` | Log file path (default: `karlsenminer.log`) |
| `--config PATH` | Config file (default: `karlsenminer_config.json`) |
| `--wizard` | Run the setup wizard even if a config exists |
| `-V, --version` | Print the version |

## Supported Hardware

### NVIDIA (CUDA)
6 GB+ VRAM, Pascal (GTX 10) through Blackwell (RTX 50); the kernel is prebuilt for sm_61 / 75 / 86 / 89 / 120:
- GeForce GTX 1070 / 1080, RTX 20, 30, 40 and 50 series
- CMP 30HX–170HX
- Quadro / RTX professional and data-centre cards

### AMD (OpenCL)
6 GB+ VRAM:
- RX 6600 / 6700 / 6800 / 6900 (RDNA2)
- RX 7600 / 7700 / 7800 / 7900 (RDNA3)
- RX 9060 / 9070 (RDNA4)

### Intel (OpenCL)
- Arc A750 / A770 and newer with 6 GB+ VRAM

## Algorithm

KarlsenHashV2 is based on [FishHashPlus](https://github.com/karlsen-network/karlsend/blob/mainnet_karlsenhashv2/domain/consensus/utils/pow/fishhashplus_kernel.go), a memory-hard proof-of-work. Each GPU holds a ~4.6 GB DAG in VRAM, which keeps mining GPU-friendly and resistant to specialised hardware.

Pipeline:
1. **Blake3** — header hash
2. **FishHash lookups** — random 128-byte reads from the DAG
3. **Blake3** — final hash and difficulty check

## Related Projects

- [rusty-karlsen](https://github.com/karlsen-network/rusty-karlsen) — Karlsen full node (Rust)
- [karlsen-cpu-miner](https://github.com/karlsencoin/karlsen-cpu-miner) — Karlsen CPU miner
- [KarlsenAI](https://ai.karlsencoin.org/) — decentralised AI inference on Karlsen GPUs

## License

See [LICENSE](LICENSE).

## Community

- [Discord](https://discord.gg/QyrvshRBJV)
- [Telegram](https://t.me/KarlsenCoin)
- [Reddit](https://www.reddit.com/r/KarlsenNetwork/)
- [Website](https://karlsencoin.org/)
