# AceDelta

[简体中文](README.md) | **English**

**Efficient OTA delta generation for Android EROFS partition images**

Compared with the Android Open Source Project (AOSP 17) in its best-performing configuration, AceDelta produces patches that are **more than one third smaller** on average and generates them **2.5× as fast**.

## Benchmark results against AOSP

Each test uses official images from two firmware versions for the same phone. Most cases are routine updates with a gap of 2–3 months; the tests also include a major Xiaomi upgrade from Android 15 to Android 16 (HyperOS 2 to HyperOS 3).

**Across seven test cases on five phones, AceDelta reduces the total patch size by 35.7% and achieves an overall patch-generation speedup of 2.5×.** Patch sizes are in MB. Generation speedup is calculated as AOSP time ÷ AceDelta time.

| Device / Update | AOSP patch size | AceDelta patch size | Size reduction | Generation speedup |
| :--- | ---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 1152.6 | 683.7 | **40.7%** | 2.5x |
| OPPO Find X8 Pro · C.76 → C.79 | 1059.1 | 733.1 | **30.8%** | 2.5x |
| Xiaomi 15 Pro · 2.0.105 → 2.0.214 | 1038.6 | 655.6 | **36.9%** | 2.7x |
| Xiaomi 15 Pro · 3.0.3 → 3.0.7 | 833.4 | 490.4 | **41.2%** | 2.6x |
| Xiaomi 15 Pro · 2.0.214 → 3.0.3 | 2657.9 | 1685.8 | **36.6%** | 2.3x |
| Nubia Z80 Ultra · 16.0.12 → 16.0.16 | 767.9 | 514.7 | **33.0%** | 3.0x |
| Realme GT8 Pro · 16.0.7 → 16.0.9 | 1268.9 | 878.1 | **30.8%** | 2.3x |

Patch application performance in PC-based tests is comparable to or slightly faster than AOSP. The following table shows application time in seconds and peak resident memory usage (RSS) in MB.

| Device / Update | AOSP | AceDelta (1 thread) | AceDelta (multiple threads) | &nbsp; | AOSP RSS | AceDelta RSS (1 thread) | AceDelta RSS (multiple threads) |
| :--- | ---: | ---: | ---: | :---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 385.3 | 501.6 | 394.8 | | 886 | 530 | 1107 |
| OPPO Find X8 Pro · C.76 → C.79 | 331.9 | 426.4 | 334.7 | | 714 | 530 | 1375 |
| Xiaomi 15 Pro · 2.0.105 → 2.0.214 | 380.3 | 369.9 | 270.2 | | 654 | 530 | 1438 |
| Xiaomi 15 Pro · 3.0.3 → 3.0.7 | 355.4 | 337.3 | 252.8 | | 607 | 530 | 1482 |
| Xiaomi 15 Pro · 2.0.214 → 3.0.3 | 440.0 | 447.4 | 334.1 | | 647 | 530 | 1571 |
| Nubia Z80 Ultra · 16.0.12 → 16.0.16 | 342.5 | 316.9 | 242.2 | | 788 | 530 | 1444 |
| Realme GT8 Pro · 16.0.7 → 16.0.9 | 428.8 | 490.1 | 402.9 | | 876 | 630 | 1398 |
| **Average** | **381** | **413** | **319** | | **739** | **544** | **1402** |

With one thread, AceDelta applies patches roughly 10% slower than AOSP. With multiple threads, it is roughly 20% faster, at the cost of higher memory usage.

### EROFS recompression performance

Like AOSP, AceDelta needs to recompress LZ4 data in EROFS images. If recompression does not reproduce the target data byte for byte, a small corrective patch is required. These corrective patches are typically no more than a few hundred bytes each, so they have little impact on total patch size, but they do affect overall patch application time. AceDelta requires significantly fewer corrective patches than AOSP:

| Device / Update | AOSP corrective patches | AceDelta corrective patches |
| :--- | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 310 | 103 |
| OPPO Find X8 Pro · C.76 → C.79 | 275 | 96 |
| Xiaomi 15 Pro · 2.0.105 → 2.0.214 | 0 | 0 |
| Xiaomi 15 Pro · 3.0.3 → 3.0.7 | 141 | 37 |
| Xiaomi 15 Pro · 2.0.214 → 3.0.3 | 241 | 48 |
| Nubia Z80 Ultra · 16.0.12 → 16.0.16 | 145 | 43 |
| Realme GT8 Pro · 16.0.7 → 16.0.9 | 273 | 96 |

<details>
<summary><strong>Expand: benchmark image details</strong></summary>

| Device | Source version | Target version | Total target image size (GB) |
| :--- | :--- | :--- | ---: |
| OnePlus 15 | `CPH2745_11.A.22_0220_202511110029` | `CPH2745_11.A.27_0270_202601280008` | 11.44 |
| OPPO Find X8 Pro | `PKC110_11.C.76_1760_202606011922` | `PKC110_11.C.79_1790_202607311812` | 11.31 |
| Xiaomi 15 Pro | `OS2.0.105.0.VOBCNXM_15.0` | `OS2.0.214.0.VOBCNXM_15.0` | 9.56 |
| Xiaomi 15 Pro | `OS3.0.3.0.WOBCNXM_16.0` | `OS3.0.7.0.WOBCNXM_16.0` | 10.40 |
| Xiaomi 15 Pro | `OS2.0.214.0.VOBCNXM_15.0` | `OS3.0.3.0.WOBCNXM_16.0` | 10.17 |
| Nubia Z80 Ultra | `MyOS16.0.12` | `MyOS16.0.16` | 10.14 |
| Realme GT8 Pro | `RMX5210_11.A.68_0680_20260512` | `RMX5210_11.A.71_0710_20260724` | 12.04 |

The images were extracted from full OTA packages or payloads provided by third-party sources:

- [firmwarefile.com](https://firmwarefile.com/)
- [danielspringer.at](https://danielspringer.at/)

</details>

<details>
<summary><strong>Expand: test methodology and workflow</strong></summary>

| Item | Configuration |
| :--- | :--- |
| Host | Standard Volcengine cloud instance; 8 CPU cores, 32 GiB RAM |
| AOSP baseline | Tag `android-17.0.0_r1`; x86_64 host build of `delta_generator`; minor version 10; `lz4diff` enabled |
| AOSP executable | `out/host/linux-x86/bin/delta_generator` |
| Patch-generation threads | AOSP: 8; AceDelta: 8 |
| AceDelta verity reconstruction | Independently implemented reconstruction utility |
| Test scope | Host-side patch generation, application, and output verification for individual images; complete on-device OTA updates have not been validated |

AOSP host-side test workflow:

**Generate a patch:**

```bash
delta_generator \
  -out_file=PATCH.BIN \
  -partition_names=0 \
  -new_partitions="$new" \
  -old_partitions="$old" \
  -minor_version=10 \
  -enable_lz4diff
```

**Apply the patch using the built-in DeltaPerformer:**

```bash
delta_generator \
  -in_file=PATCH.BIN \
  -partition_names=0 \
  -new_partitions=NEW.IMG \
  -old_partitions="$old"
```

**AceDelta host-side test workflow:**

See the [evaluation tools](https://github.com/peacherwu969/acedelta/releases/latest) for details.

</details>

## Integration design

AceDelta targets patch generation and application within the Android OTA workflow.

- **Block-device interface:** For device-side application, AceDelta uses block-based writes to facilitate integration with the COW writer in `update_engine` for Virtual A/B (VAB) updates.
- **Verity:** Verity data is currently rebuilt using an independently implemented utility. For device-side deployment, this reconstruction functionality could be integrated, or handled through the native `VerityWriter` in `update_engine`.

## Evaluation and commercial licensing

This project provides free Linux x86_64 [evaluation tools](https://github.com/peacherwu969/acedelta/releases/latest) for patch generation and application.

The commercial patch generator and device-side SDK are licensed per OEM / device model. The device-side SDK includes static libraries and headers. Source code escrow and source code delivery under an NDA can be discussed separately.

Contact: [peacherwu969@gmail.com](mailto:peacherwu969@gmail.com)
