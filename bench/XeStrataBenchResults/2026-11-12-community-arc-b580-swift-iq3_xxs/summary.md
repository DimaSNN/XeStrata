<!-- 中身を確認してから貼ってください。パスとホスト名は伏せてあります。 -->

## xestrata --bench (2026-11-12 00:41)

- XeStrata: 0.2.8-1 (DPC++ compiler 7.2.0)
- GPU: Intel Arc B580 (12 GB, PCIe link before the runs 16.0 GT/s PCIe x8 (the card supports 16.0 GT/s PCIe, the board 32.0 GT/s PCIe))
- CPU: 12th Gen Intel(R) Core(TM) i5-12600KF (avx2, avx_vnni, f16c, fma), RAM 61.4 GiB
- OS: Ubuntu 26.04.1 LTS, Linux 7.0.0-42-generic; data folder on NVMe
- Model: swift-1.5-iq3_xxs, context 204800, KV int8
- Measured: RAM 43.5 GB/s (10 threads), 29.6 GB/s one thread; PCIe 14.0 GB/s to the GPU, 14.3 GB/s back; SSD 3.54 GB/s

### どこに時間がかかっているか

- 64k 出力: 1 ラウンド 60.7 ms のうち CPU のエキスパート 46%、GPU 待ち 38%、下書き（MTP） 9%。CPU のエキスパートの読み出しは 37.7 GB/s（測った RAM 帯域 43.5 GB/s の 87%）
- 64k プロンプト: 1,271 tok/s、エキスパートの転送 187976 個（ホストの時間 831 ms）、PLE 112 ms
- 32k 出力: 1 ラウンド 57.3 ms のうち CPU のエキスパート 44%、GPU 待ち 39%、下書き（MTP） 9%。CPU のエキスパートの読み出しは 37.8 GB/s（測った RAM 帯域 43.5 GB/s の 87%）
- 32k プロンプト: 1,277 tok/s、エキスパートの転送 92852 個（ホストの時間 403 ms）、PLE 101 ms
- 1k 出力: 1 ラウンド 56.5 ms のうち CPU のエキスパート 47%、GPU 待ち 38%、下書き（MTP） 7%。CPU のエキスパートの読み出しは 31.6 GB/s（測った RAM 帯域 43.5 GB/s の 72%）
- 1k プロンプト: 443 tok/s、エキスパートの転送 9591 個（ホストの時間 33 ms）、PLE 0 ms、CPU に 62%
- generate 1k: GPU の使用率の中央値 0%（GPU 以外が律速）
- serve 1k: GPU の使用率の中央値 0%（GPU 以外が律速）
- usage: GPU の使用率の中央値 0%（GPU 以外が律速）

### generate（追従・リリースと同じ条件、256 トークン greedy、2 回の中央値）

| | XeStrata 0.2.8-1 |
| --- | ---: |
| 64K プロンプト (tok/s) | 1,271 |
| 32K プロンプト (tok/s) | 1,277 |
| 1K プロンプト (tok/s) | 443 |
| 64K 出力 (tok/s) | 52.1 |
| 32K 出力 (tok/s) | 50.2 |
| 1K 出力 (tok/s) | 51.9 |

### serve greedy（4 ターンのチャット、256 トークン）

| | XeStrata 0.2.8-1 |
| --- | ---: |
| 64K 最初のプロンプト (tok/s) | 1,265 |
| 32K 最初のプロンプト (tok/s) | 1,270 |
| 1K 最初のプロンプト (tok/s) | 425 |
| 64K 会話の続きの待ち (ms) | 274 |
| 32K 会話の続きの待ち (ms) | 236 |
| 1K 会話の続きの待ち (ms) | 301 |
| 64K 出力 (tok/s) | 59.5 |
| 32K 出力 (tok/s) | 55.0 |
| 1K 出力 (tok/s) | 65.8 |

### serve 温度 0.7（4 ターンのチャット、256 トークン）

| | XeStrata 0.2.8-1 |
| --- | ---: |
| 64K 最初のプロンプト (tok/s) | 1,265 |
| 32K 最初のプロンプト (tok/s) | 1,269 |
| 1K 最初のプロンプト (tok/s) | 426 |
| 64K 会話の続きの待ち (ms) | 276 |
| 32K 会話の続きの待ち (ms) | 275 |
| 1K 会話の続きの待ち (ms) | 296 |
| 64K 出力 (tok/s) | 57.5 |
| 32K 出力 (tok/s) | 52.9 |
| 1K 出力 (tok/s) | 51.8 |

### カーネル単体（合成のデータ、µs）

| 経路の設定 | カーネル | 経路 | 形 | µs |
| --- | --- | --- | --- | ---: |
| default | expert-gate-up | int8-matrix | 16x64x1280x2560:t17 | 324.2 |
| default | expert-down | int8-matrix | 16x64x2560x640:t17 | 183.4 |
| no-xmx | expert-gate-up | off | - | - |
| no-xmx | expert-down | off | - | - |
| no-bf16-mma | expert-gate-up | int8-matrix | 16x64x1280x2560:t17 | 324.5 |
| no-bf16-mma | expert-down | int8-matrix | 16x64x2560x640:t17 | 182.5 |

### 計測中の GPU と CPU（中央値 / 最大）

| | GPU 使用率 % | 電力 W | 温度 °C | CPU % |
| --- | ---: | ---: | ---: | ---: |
| generate 64k | 99.9 / 100 | 161.4 / 167.2 | 59 / 65 | 6.4 / 62.7 |
| generate 32k | 73.5 / 100 | 97.3 / 165.4 | 57 / 64 | 6.4 / 62.7 |
| generate 1k | 0 / 100 | 37.2 / 137.4 | 51 / 58 | 12.8 / 62.8 |
| serve 64k | 99.9 / 100 | 161.6 / 168.8 | 62 / 67 | 6.4 / 62.8 |
| serve 32k | 96.8 / 100 | 119 / 166.8 | 61 / 66 | 7 / 62.7 |
| serve 1k | 0.4 / 100 | 38.5 / 150.2 | 54 / 62 | 13.4 / 62.7 |
| usage | 0 / 0 | 36.8 / 36.8 | 54 / 54 | 1.3 / 2.2 |

### 答えの指紋（generate の出力の先頭 64 トークン）

- 64k: 9f657922443f1040, 9f657922443f1040
- 32k: 19a0ec7e5f0acc3b, 19a0ec7e5f0acc3b
- 1k: a083f849b5915045, 5434c663d59c7acf

