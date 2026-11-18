# HM3D-V2 数据发版日志（2026-11-18 联合编码更新）

## 来源与版本

- 原始与补采的数据经对账、质检后已合并到下述统一发布入口；使用该入口即可读取两部分保留数据。
- QC_HANDOFF SHA256：`9611c09a33a0a899406696511b7c8b8bb9437890a4d435a06a681241990249cb`。
- K32、K64、RVQ 与深圳质检保留轨迹共同拟合，逐轨迹等权；合计 86,448,973 个非 STOP 运动节点。两套数据使用同一组三个码本并已重新编码，旧码本及对应标签已替换。

## 正式 OSS 入口

`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

完成记录 `RELEASE_COMPLETE.json`，统一清单 `DATASETS.json`；默认 `k64`，`partition=generation_pool`、`split=null`，未划分 train/val/test。统一清单 SHA256：`6853d8e9f0848c69b7418f67af15fc600c36509f65bb1a483d7a5918bc9ecfbb`。

| 分支 | 码本 SHA256 | 训练索引 SHA256 |
|---|---|---|
| K32 | `2787d1e071033632e99be1de29b1e58e33f501bff0a2a98cfedbe712b0dcf6ce` | `249e9b334fbf2a4a2ec4900e82f15b569c831bd53e9c31e20abf8d01f39d2a8d` |
| K64 | `ca975a90f2969c36f9173a95234fdf4a799e7e332a5649712a60782025c26e1c` | `4ec0e5e14ccb41338231ae4a26270de1d920f8cdeff561b66c7bccd1dd74e39b` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

索引分别为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；各分支的 `codebook.npz` 为当前码本。详细训练入口见 [AI_ENTRY.md](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/downloads/AI_ENTRY.md)。

## 数量与质检

计划 2,325,627 条；实际记录 2,304,696 条，重复记录次数 7；完整成功唯一轨迹 2,304,685 条，缺视频、视频损坏、成功载荷冲突均为 0。实际记录 failed 8、dropped 3；状态账另列 failed 131、pending 112。2 个 child_run 未就绪，涉及计划 20,696 条，实际采集数未知。

QC 保留 **1,940,232** 条、过滤 **364,453** 条，保留率 **84.19%**，分母为完整成功唯一轨迹。主要互斥排除原因：黑区 177,119、末段累计转向 100,054、末段抵消角 48,891。NavMesh 覆盖仅计算有匹配资源的 42 个场景。

每个联合编码分支 **1,940,232 条轨迹、83,302,260 个节点、1,940,232 个 STOP**；成功合并跳过 36,605,133 个原始观测。

## 新码本离线抽样

按种子 20261118 均匀抽取 32 个分片、共 **1,024 条轨迹**。位置为局部 XY 欧氏误差，偏航为环绕绝对误差，按有效目标加权，STOP 和填充排除；本样本参与了拟合，不是独立验证集。K32/K64 与 RVQ 评估范围不同，不直接比较。

| 分支 | 口径 | 有效目标 | 平均位置误差 cm | 平均偏航误差 ° |
|---|---|---:|---:|---:|
| K32 | 下一保留节点 | 39,802 | 1.45 | 0.92 |
| K64 | 下一保留节点 | 39,802 | 0.93 | 0.57 |
| RVQ | H5 全部有效目标 | 188,770 | 1.34 | 0.68 |

旧码本的误差、FDE 与码频不适用于本版；本版全量误差尚未重新统计。

## 验证与接入

全量暂存标注、标签、源索引绑定和逐分片计数校验通过；安装后的 60,633 个分片逐对象核验索引、SHA 元数据、长度和 CRC64，实际读取样本通过。联合拟合码本在两数据集各 300 个运动节点上做过语义抽样检查；未完成策略训练或真机测试。当前配置凭据可读，不代表其他训练账户已授权。

[质检报告](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/index.html) · [联合编码报告](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/tokenizer.html) · [训练入口](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/training.html) · [抽样统计明细](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/downloads/JOINT_REPORT_SAMPLE.json)

K32/K64 以 0 为 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字。RVQ 使用有效 mask，并只执行第一个解码目标后重新观测；历史只使用真实保留节点。
