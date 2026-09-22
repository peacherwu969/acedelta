# AceDelta

**面向 Android EROFS分区镜像的 OTA 差分方案。**

AceDelta 提供完整分区EROFS镜像的差分生成与补丁应用功能。在下列 4 款手机、6 组版本升级测试中，相比最优配置的安卓开源代码(AOSP 17) ，AceDelta 的差分包缩小 **19.4%–32.8%**，生成速度平均为AOSP的 **3.2 倍**，补丁应用速度为AOSP的 **0.99–1.5 倍**。


## 基准结果

每组测试均使用同一设备两个版本的官方镜像，覆盖完整的分区集合，在PC端按镜像分别执行并汇总。测试包括常规版本更新（间隔2~3个月)，以及小米从Android 15 (HyperOS 2)到Android 16 (HyperOS 3)的大版本升级。

**六组测试的平均差分包缩减为 26.7%，平均生成加速比为 3.2×。** 下表中的差分包大小单位为 MB（10⁶ 字节）。加速比为 AOSP 耗时 ÷ AceDelta 耗时，大于 1 表示 AceDelta 更快。

| 设备 / 升级版本 | AOSP 包大小 | AceDelta 包大小 | 包大小缩减 | 生成加速比 | 应用加速比 |
| :--- | ---: | ---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 1152.6 | 835.2 | **27.5%** | 3.36× | 1.04× |
| OPPO Find X8 Pro · C.76 → C.79 | 1059.1 | 853.6 | **19.4%** | 3.40× | 0.99× |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 1038.6 | 759.8 | **26.8%** | 2.54× | 1.30× |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 2657.9 | 1786.0 | **32.8%** | 2.69× | 1.27× |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 833.4 | 589.6 | **29.3%** | 3.30× | 1.40× |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 767.9 | 580.0 | **24.5%** | 3.59× | 1.47× |


<details>
<summary><strong>展开：生成耗时与内存</strong></summary>

耗时单位为秒，RSS 为进程峰值驻留内存。

| 设备 / 升级版本 | AOSP 耗时 | AceDelta 耗时 | AOSP RSS（GB） | AceDelta RSS（GB） |
| :--- | ---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 3386 | 1009 | 17.6 | 10.4 |
| OPPO Find X8 Pro · C.76 → C.79 | 3738 | 1098 | 15.7 | 19.4 |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 2493 | 982 | 18.3 | 18.6 |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 3230 | 1200 | 15.8 | 12.8 |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 2143 | 650 | 18.5 | 21.9 |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 2555 | 712 | 17.0 | 13.2 |

均使用 8 线程。

</details>

<details>
<summary><strong>展开：补丁应用耗时与内存</strong></summary>

AceDelta 应用耗时覆盖主机端脚本的完整流程，包括源镜像读取与哈希计算、块级补丁应用、适用时的哈希树与 FEC 重建、尾部还原，以及输出镜像校验。哈希树和 FEC 由独立工具生成，不包含在原始块级补丁应用器中。

| 设备 / 升级版本 | AOSP 耗时 | AceDelta 耗时 | AOSP RSS (MB) | AceDelta RSS (MB) |
| :--- | ---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 385 | 371 | 866 | 518 |
| OPPO Find X8 Pro · C.76 → C.79 | 332 | 334 | 698 | 518 |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 380 | 293 | 639 | 518 |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 440 | 346 | 633 | 518 |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 355 | 253 | 593 | 518 |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 343 | 234 | 770 | 518 |


</details>

<details>
<summary><strong>展开：完整版本号与镜像规模</strong></summary>

| 设备 | 源版本 | 目标版本 | 新镜像总大小（GB） |
| :--- | :--- | :--- | ---: |
| OnePlus 15 | `CPH2745_11.A.22_0220_202511110029` | `CPH2745_11.A.27_0270_202601280008` | 11.44 |
| OPPO Find X8 Pro | `PKC110_11.C.76_1760_202606011922` | `PKC110_11.C.79_1790_202607311812` | 11.31 |
| 小米 15 Pro | `OS2.0.105.0.VOBCNXM_15.0` | `OS2.0.214.0.VOBCNXM_15.0` | 9.56 |
| 小米 15 Pro | `OS2.0.214.0.VOBCNXM_15.0` | `OS3.0.3.0.WOBCNXM_16.0` | 10.17 |
| 小米 15 Pro | `OS3.0.3.0.WOBCNXM_16.0` | `OS3.0.7.0.WOBCNXM_16.0` | 10.40 |
| 努比亚 Z80 Ultra | `MyOS16.0.12` | `MyOS16.0.16` | 10.14 |

镜像提取自厂商官方完整 OTA 包或 fastboot 包。新镜像总大小为该组全部受测目标分区镜像的大小之和。

</details>

## 统计口径与内存配置

- **差分包大小：** AOSP 统计各受测分区完整 `update_engine` payload 的大小之和；AceDelta 统计各受测分区补丁目录中 `d`、`meta.txt` 和 `footer.az` 的文件大小之和，覆盖块级差分数据、元数据与压缩后的尾部信息。适用时，两者均在应用阶段重建哈希树和FEC。
- **应用耗时：** 两者均统计到输出完整镜像并完成校验。AceDelta 的优势主要来自更快的哈希树与 FEC 重建。
- **应用端内存：** 本次 AceDelta 使用 512 MB LZMA 窗口，各组峰值 RSS 为 518 MB。改用 256 MB 窗口时，同一测试集的峰值 RSS 为 270–330 MB，差分包增大 0.4%–3%，具体取决于数据。

<details>
<summary><strong>展开：EROFS LZ4 重压缩修补统计</strong></summary>

两种工具均对 EROFS LZ4 数据进行重压缩。如果压缩结果无法逐字节还原目标压缩簇，则需要额外的小补丁修正;小补丁一般不超过几百字节，对补丁总大小影响不大，但是比较影响应用时间。AceDelta所需的小补丁总数显著小于AOSP：

| 设备 / 升级版本 | AOSP 小补丁总数 | AceDelta 小补丁总数 |
| :--- | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 310 | 103 |
| OPPO Find X8 Pro · C.76 → C.79 | 275 | 96 |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 0 | 0 |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 241 | 48 |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 141 | 37 |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 145 | 43 |


</details>

## 测试方法

| 项目 | 配置 |
| :--- | :--- |
| 主机 | 8 核 CPU，32 GiB 内存 |
| AOSP 基线 | 标签 `android-17.0.0_r1`，`delta_generator` 主机版本，minor version 10，启用 `lz4diff` |
| AOSP 可执行文件 | `out/host/linux-x86/bin/delta_generator` |
| AceDelta 算法 | `acedelta.erofs` |
| 生成线程 | AOSP 8 线程，AceDelta 8 线程 |
| AceDelta 块级应用器 | `blockpatch.erofs`，单线程，LZMA 窗口 512 MB |
| AceDelta 重建工具 | 独立的 `gen_hash` 与 `gen_fec` 二进制 |
| 测试范围 | 主机端逐镜像生成、应用与输出校验；未进行设备端完整 OTA 验证 |

AOSP 按分区分别调用。以下命令中的 `old`、`new` 分别为源镜像和目标镜像路径。

**生成差分包：**

```bash
delta_generator \
  -out_file=dd \
  -partition_names=0 \
  -new_partitions="$new" \
  -old_partitions="$old" \
  -minor_version=10 \
  -enable_lz4diff
```

**应用差分包，使用内置 DeltaPerformer：**

```bash
delta_generator \
  -in_file=dd \
  -partition_names=0 \
  -new_partitions=nn \
  -old_partitions="$old"
```

**AceDelta PC端差分应用流程：**

详细请参考评估程序脚本。

## 集成设计

AceDelta 面向 Android OTA 链路中的差分生成与补丁应用环节。目前仅完成主机端逐镜像测试；`update_engine` 与 Virtual A/B 的设备端集成及完整 OTA 流程尚未验证。以下为面向设备端的集成设计。

- **块设备接口：** 设计上由应用端读取源 slot 的块设备及 operation 对应的 extents，按目标块顺序输出，每个目标块写入一次，不依赖设备端文件系统访问。设备端避免额外补丁中间文件是集成目标；当前主机测试脚本使用中间镜像及独立的哈希树、FEC 文件。
- **update_engine / Virtual A/B：** 集成方式是在 `update_engine` 中增加 operation 类型，并通过 COW writer 对接 Virtual A/B。该方式需要相应的集成改动。
- **流式与分段：** 差分包支持流式读取。生成端可将其划分为独立解码的 operation，以适配逐 operation 缓冲和恢复；64 MB 分段配置下，测得的包大小代价约为 5%。设备端恢复流程仍需在集成后验证。
- **Verity：** 当前主机流程根据补丁元数据调用独立的 `gen_hash` 和 `gen_fec` 工具。设备端可考虑集成相应重建功能，或对接 `update_engine` 的 `VerityWriter`；具体实现仍需集成与验证。

## 评估与商业授权

Linux x86_64 评估版差分生成器、补丁应用器与测试脚本计划通过 **Releases** 提供，评估二进制计划免费提供。当前尚无可下载的评估版本。

商业生成器与应用端 SDK 按 OEM / 设备型号授权。应用端 SDK 包括静态库与头文件；源代码托管及 NDA 下的源码交付安排可单独沟通。

联系：[peacherwu969@gmail.com](mailto:peacherwu969@gmail.com)
