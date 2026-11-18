# HM3D-V2 数据发版日志（2026-11-18 联合编码更新）

## 来源与版本

- 原始与补采的数据经对账、质检后已合并到下述统一发布入口；使用该入口即可读取两部分保留数据。
- QC_HANDOFF SHA256：`9611c09a33a0a899406696511b7c8b8bb9437890a4d435a06a681241990249cb`。
- K32、K64 与 HM3D 和深圳的质检保留轨迹联合拟合，以每条轨迹等权为基础，真实动作 |左向位移| ≥ 2.5 cm 的拟合权重为 5 倍；RVQ 沿用原有逐轨迹等权的联合码本。合计 86,448,973 个非 STOP 运动节点，两套数据共用码本。

## 正式 OSS 入口

`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

完成记录 `RELEASE_COMPLETE.json`，统一清单 `DATASETS.json`；默认 `k64`，`partition=generation_pool`、`split=null`，未划分 train/val/test。统一清单 SHA256：`3f798ea12ddd8634a842e86bda8f8550181f7d70cf4eb4564b46bf254efaa4e6`。

| 分支 | 码本 SHA256 | 训练索引 SHA256 |
|---|---|---|
| K32 | `589c78646229561178e0fb0251a228dc0e4b2ee68f96bd7f3deee3710d4acac6` | `ec1f7c32d2d978efdc73b7f1b2ff4bb2ba54a85dfc578bbec427f585b6451a92` |
| K64 | `a5ce5f61ccb23e5fba9d5d1191820f79e2a01b22fd4025d94c4e415e01cb5fa2` | `ab87e69524251a5e0131199de5cce1f814836c7daa8913ec087956fcf7ae8b37` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

索引分别为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；各分支的 `codebook.npz` 为当前码本。详细训练入口见 [AI_ENTRY.md](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/downloads/AI_ENTRY.md)。

## 数量与质检

计划 2,325,627 条；实际记录 2,304,696 条，重复记录次数 7；完整成功唯一轨迹 2,304,685 条，缺视频、视频损坏、成功载荷冲突均为 0。实际记录 failed 8、dropped 3；状态账另列 failed 131、pending 112。2 个 child_run 未就绪，涉及计划 20,696 条，实际采集数未知。

QC 保留 **1,940,232** 条、过滤 **364,453** 条，保留率 **84.19%**，分母为完整成功唯一轨迹。主要互斥排除原因：黑区 177,119、末段累计转向 100,054、末段抵消角 48,891。NavMesh 覆盖仅计算有匹配资源的 42 个场景。

每个联合编码分支 **1,940,232 条轨迹、83,302,260 个节点、1,940,232 个 STOP**；成功合并跳过 36,605,133 个原始观测。

## 全量开环误差与码频

覆盖全部 **1,940,232 条轨迹、83,302,260 个节点**，每条轨迹等权。以真实起点开始逐个累积动作，中途不以真实位姿纠正；RVQ 每步使用 decoded[0]。ADE 包含起点，FDE 为终点水平位置误差。属于离线重建，不代表闭环策略或真机效果。

| 分支 | ADE 均值 m | FDE 均值 m | FDE P95 m |
|---|---:|---:|---:|
| K32 | 0.556 | 1.140 | 3.633 |
| K64 | 0.359 | 0.734 | 2.358 |
| RVQ | 0.070 | 0.121 | 0.328 |

侧向覆盖改善与普通节点误差的取舍见编码报告。全量码字使用频率及其他误差统计见 [统计文件](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/downloads/JOINT_FULL_METRICS.json) 和编码报告。

## 验证与接入

全量暂存标注、标签、源索引绑定和逐分片计数校验通过；安装后的 60,633 个分片逐对象核验索引、SHA 元数据、长度和 CRC64，实际读取样本通过。联合拟合码本在两数据集各 300 个运动节点上做过语义抽样检查；未完成策略训练或真机测试。当前配置凭据可读，不代表其他训练账户已授权。

[质检报告](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/index.html) · [联合编码报告](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/tokenizer.html) · [训练入口](https://yuaneth.github.io/hm3d-r23-report-20261111-32803559/training.html)

K32/K64 以 0 为 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字。RVQ 使用有效 mask，并只执行第一个解码目标后重新观测；历史只使用真实保留节点。
