# HM3D R23 数据接入（2026-11-18 联合码本）

K32/K64 基于深圳与 HM3D 的质检保留轨迹联合拟合，每条轨迹等权基础上将 |左向位移|≥2.5 cm 的真实动作权重提高到 5 倍；RVQ 保持原来的逐轨迹等权联合码本。HM3D 是未划分训练、验证或测试集的 `generation_pool`，默认 `k64`。

K64 于2026-11-19增加多样性：保留56个码字，将前向0.2–0.4m密集直线段的8个码字移到真实左右侧向动作，左右各4个；密集区18码减至10码，侧向至少1cm的码字由10增至18。K64仍为每步1个动作token，K32和RVQ沿用当前版本。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

先读 `DATASETS.json` 与 `RELEASE_COMPLETE.json`，再读取以下索引并核对其码本 SHA256：

| 分支 | 码本 SHA256 | 索引 SHA256 |
|---|---|---|
| K32 | `589c78646229561178e0fb0251a228dc0e4b2ee68f96bd7f3deee3710d4acac6` | `ec1f7c32d2d978efdc73b7f1b2ff4bb2ba54a85dfc578bbec427f585b6451a92` |
| K64 | `b03891724cfab35bc174fab31a20c6e1161df704c0ae65be6961547882f18276` | `df7ef5ef93edde4582c1f7a47f1ec784c237dec18bfed07aa45d0f8087d20951` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

训练索引依次为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；码本在各分支 `codebook.npz`。在授权 OSS 下载 `tools/reader.tar.gz` 并解压，使用例如：

```bash
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k64 --workers 0
```

分支也可选 `k32`、`rvq`。K32/K64 使用局部 [前向、左向、偏航]，0 为 STOP；RVQ 三层 ID 的完整 `[0,0,0]` 才是 STOP，单层 0 为普通码字。RVQ 以 `future_valid_mask` 排除终点填充，仅执行 `decoded[0]` 然后重新观测。历史仅使用此前真实保留节点，时间差不是控制截止时间。视频从原始 OSS tar 通过 Range 恢复并校验。

编码 ID 为 uint16，训练模型需自己注册原子 token、映射词表并调整 embedding。独立训练账户的 OSS 权限、策略训练及真机表现尚未验证。新码本全量开环误差和码频见编码报告及 report/downloads/JOINT_FULL_METRICS.json；旧误差统计不适用于本版。
