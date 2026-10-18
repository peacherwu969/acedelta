# AceDelta

**简体中文** | [English](README.en.md)

**高效的Android EROFS分区镜像 OTA 差分方案**

相比最优配置的安卓开源代码(AOSP 17) ，AceDelta 的差分包平均减小 **超三分之一**，差分速度是AOSP的 **2.5 倍**。

## 评测结果（对照AOSP）

下面每组测试均使用同一手机两个版本的官方镜像，主要是常规版本更新（间隔2~3个月)，也包含了小米从Android 15到16 (HyperOS 2到3)的大版本升级。

**五部手机、七组测试的总差分包减小 35.7%，总差分加速比为 2.5×。** 差分包大小单位为 MB；加速比为 AOSP 耗时 ÷ AceDelta 耗时：


| 设备 / 升级版本 | AOSP 包大小 | AceDelta 包大小 | 包大小缩减 | 生成加速比 |
| :--- | ---: | ---: | ---: | ---: |
| OnePlus 15 · A.22 → A.27 | 1152.6 | 683.7 | **40.7%** | 2.5x |
| OPPO Find X8 Pro · C.76 → C.79 | 1059.1 | 733.1 | **30.8%** | 2.5x |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 1038.6 | 655.6 | **36.9%** | 2.7x |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 833.4 | 490.4 | **41.2%** | 2.6x |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 2657.9 | 1685.8 | **36.6%** | 2.3x |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 767.9 | 514.7 | **33.0%** | 3.0x |
| Realme GT8 Pro · 16.0.7 → 16.0.9 | 1268.9 | 878.1 | **30.8%** | 2.3x |

下面是PC上模拟的升级时间与峰值内存（7组数据上的均值）。大致上，单线程比AOSP略慢5%；而多线程比AOSP快20%，代价是更多的内存使用：

| | AOSP | AceDelta (单线程) | AceDelta (多线程) | 
| :--- | ---: | ---: | ---: |
| 升级时间(秒)| 381|400|306|
| RSS (MB) |739|544|1402|


### EROFS 重压缩性能比较

和AOSP一样，AceDelta也需要对EROFS镜像的LZ4 数据进行重压缩。如果压缩结果无法逐字节还原目标数据，则需要额外的小补丁修正。这对于差分包大小，差分时间和升级时间都有一定的负面影响。自AceDelta 版本1.1起，在全部测试样本上重压缩结果均完全一致，不需要任何修补。

| 设备 / 升级版本 | AOSP 小补丁总数 | 
| :--- | ---: |
| OnePlus 15 · A.22 → A.27 | 310 |
| OPPO Find X8 Pro · C.76 → C.79 | 275 |
| 小米 15 Pro · 2.0.105 → 2.0.214 | 0   |
| 小米 15 Pro · 3.0.3 → 3.0.7 | 141 |
| 小米 15 Pro · 2.0.214 → 3.0.3 | 241 |
| 努比亚 Z80 Ultra · 16.0.12 → 16.0.16 | 145 |
| Realme GT8 Pro · 16.0.7 → 16.0.9 | 273|


<details>
<summary><strong>展开：评测镜像详情</strong></summary>

| 设备  | 源版本 | 目标版本 | 新镜像总大小（GB） |
| :--- | :--- | :--- | ---: |
| OnePlus 15 | `CPH2745_11.A.22_0220_202511110029` | `CPH2745_11.A.27_0270_202601280008` | 11.44 |
| OPPO Find X8 Pro | `PKC110_11.C.76_1760_202606011922` | `PKC110_11.C.79_1790_202607311812` | 11.31 |
| 小米 15 Pro | `OS2.0.105.0.VOBCNXM_15.0` | `OS2.0.214.0.VOBCNXM_15.0` | 9.56 |
| 小米 15 Pro | `OS3.0.3.0.WOBCNXM_16.0` | `OS3.0.7.0.WOBCNXM_16.0` | 10.40 |
| 小米 15 Pro | `OS2.0.214.0.VOBCNXM_15.0` | `OS3.0.3.0.WOBCNXM_16.0` | 10.17 |
| 努比亚 Z80 Ultra | `MyOS16.0.12` | `MyOS16.0.16` | 10.14 |
| Realme GT8 Pro | `RMX5210_11.A.68_0680_20260512`|`RMX5210_11.A.71_0710_20260724`|12.04|

镜像提取自第三方提供的完整 OTA 包或 payload：
	- https://firmwarefile.com/
	- https://danielspringer.at/

</details>


<details>
<summary><strong>展开：测试方法/流程</strong></summary>


| 项目  | 配置  |
| :--- | :--- |
| 主机  | 普通火山云节点；8 核 CPU，32 GiB 内存 |
| AOSP 基线 | 标签 `android-17.0.0_r1`，`delta_generator` PC/x64版本，minor version 10，启用 `lz4diff` |
| AOSP 可执行文件 | `out/host/linux-x86/bin/delta_generator` |
| 差分线程 | AOSP 8 线程；AceDelta 8 线程 |
| AceDelta Verity重建工具 | 独立实现的重建程序 |
| 测试范围 | 主机端逐镜像生成、应用与输出校验；未进行设备端完整 OTA 验证 |

AOSP PC测试流程：

**生成差分包：**

```bash
delta_generator \
  -out_file=PATCH.BIN \
  -partition_names=0 \
  -new_partitions="$new" \
  -old_partitions="$old" \
  -minor_version=10 \
  -enable_lz4diff
```

**应用差分包，使用内置 DeltaPerformer：**

```bash
delta_generator \
  -in_file=PATCH.BIN \
  -partition_names=0 \
  -new_partitions=NEW.IMG \
  -old_partitions="$old"
```

**AceDelta PC测试流程：**

详细请参考[评估程序](https://github.com/peacherwu969/acedelta/releases/latest) 。

</details>


## 集成设计

AceDelta 面向 Android OTA 链路中的差分生成与补丁应用环节。

- **块设备接口：** 设备应用时，AceDelta以块为单位写入数据，可以方便的和主流VAB的`update_engine` 中 COW writer 集成对接。
- **Verity：** 目前是调用独立实现的程序重建Verity数据。设备端可考虑集成相应重建功能，或对接 `update_engine` 原生的 `VerityWriter`。

## 评估与商业授权

本项目免费提供Linux [评估版差分生成、补丁应用程序](https://github.com/peacherwu969/acedelta/releases/latest) 。

商业差分生成程序与应用端 SDK 按 OEM / 设备型号授权。应用端 SDK 包括静态库与头文件、源代码托管及 NDA 下的源码交付安排可单独沟通。

联系：[peacherwu969@gmail.com](mailto:peacherwu969@gmail.com)
