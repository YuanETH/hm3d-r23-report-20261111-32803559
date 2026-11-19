# HM3D R23 数据接入（2026-11-19 直行间距更新）

K32/K64 使用深圳与 HM3D 的质检保留轨迹联合拟合，本版增加近直行至少5cm间距约束；RVQ 保持原来的逐轨迹等权联合码本。HM3D 是未划分训练、验证或测试集的 `generation_pool`，默认 `k64`。

K32/K64 近直行码字（|左向|<0.5cm且|转角|<3°）前进距离间隔至少5cm，分别保留13和16个；其他既有码字向量保持一致，释放的1和9个ID重新拟合到真实侧向动作；原始归一化、STOP及每步1个运动token保持一致；RVQ沿用当前联合码本。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

先读 `DATASETS.json` 与 `RELEASE_COMPLETE.json`，再读取以下索引并核对其码本 SHA256：

| 分支 | 码本 SHA256 | 索引 SHA256 |
|---|---|---|
| K32 | `0429002c179e4223b21a5faf5fd00ae18326f7cd5b2384be22dea9799cc0c16b` | `2577875b67beed1854d607707d40283d59862485dca2754169f878cc5ccd12ef` |
| K64 | `5f633256e06b28b78da1a16c370c37711212d7146c20c12078e5cd38a446aeb7` | `de9946d7d0b066c980fa17e6e93764b5ba0e03d00b6dd416e23355053a254d13` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

训练索引依次为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；码本在各分支 `codebook.npz`。在授权 OSS 下载 `tools/reader.tar.gz` 并解压，使用例如：

```bash
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k64 --workers 0
```

分支也可选 `k32`、`rvq`。K32/K64 使用局部 [前向、左向、偏航]，0 为 STOP；RVQ 三层 ID 的完整 `[0,0,0]` 才是 STOP，单层 0 为普通码字。RVQ 以 `future_valid_mask` 排除终点填充，仅执行 `decoded[0]` 然后重新观测。历史仅使用此前真实保留节点，时间差不是控制截止时间。视频从原始 OSS tar 通过 Range 恢复并校验。

编码 ID 为 uint16，训练模型需自己注册原子 token、映射词表并调整 embedding。独立训练账户的 OSS 权限、策略训练及真机表现尚未验证。新码本全量开环误差和码频见编码报告及 report/downloads/JOINT_FULL_METRICS.json；旧误差统计不适用于本版。
