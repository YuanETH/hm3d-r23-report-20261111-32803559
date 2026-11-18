# HM3D R23 数据接入（2026-11-18 联合码本）

HM3D 与深圳质检保留轨迹共同拟合 K32、K64、RVQ，每条轨迹权重相同，所有对应标签重新编码。HM3D 是未划分训练、验证或测试集的 `generation_pool`，默认 `k64`。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

先读 `DATASETS.json` 与 `RELEASE_COMPLETE.json`，再读取以下索引并核对其码本 SHA256：

| 分支 | 码本 SHA256 | 索引 SHA256 |
|---|---|---|
| K32 | `2787d1e071033632e99be1de29b1e58e33f501bff0a2a98cfedbe712b0dcf6ce` | `249e9b334fbf2a4a2ec4900e82f15b569c831bd53e9c31e20abf8d01f39d2a8d` |
| K64 | `ca975a90f2969c36f9173a95234fdf4a799e7e332a5649712a60782025c26e1c` | `4ec0e5e14ccb41338231ae4a26270de1d920f8cdeff561b66c7bccd1dd74e39b` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

训练索引依次为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；码本在各分支 `codebook.npz`。在授权 OSS 下载 `tools/reader.tar.gz` 并解压，使用例如：

```bash
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k64 --workers 0
```

分支也可选 `k32`、`rvq`。K32/K64 使用局部 [前向、左向、偏航]，0 为 STOP；RVQ 三层 ID 的完整 `[0,0,0]` 才是 STOP，单层 0 为普通码字。RVQ 以 `future_valid_mask` 排除终点填充，仅执行 `decoded[0]` 然后重新观测。历史仅使用此前真实保留节点，时间差不是控制截止时间。视频从原始 OSS tar 通过 Range 恢复并校验。

编码 ID 为 uint16，训练模型需自己注册原子 token、映射词表并调整 embedding。独立训练账户的 OSS 权限、策略训练及真机表现尚未验证。新码本抽样指标见编码报告；旧误差统计不适用于本版。
