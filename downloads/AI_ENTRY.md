# HM3D R23 数据接入（2026-11-20 K64 短窗口FDE更新）

深圳与 HM3D 质检保留轨迹共享码本。K64 本版主要优化未来5步和10步FDE；K32保持5cm间距版，RVQ保持原联合码本。HM3D 是未划分训练、验证或测试集的 `generation_pool`，默认 `k64`。

K64 以未来5步和10步重叠窗口的终点FDE为主要目标，联合优化码本与整条离线标签序列；5/10步使用同一条序列，每步仍为1个运动token、64个运动码字加STOP=0，近直行前进距离间隔至少5cm；K32、RVQ和深圳K256沿用当前版本。

K64 编码约定（2026-11-20）：标签使用真实未来路径，联合搜索同一条码字序列来减小5步和10步FDE，不能用单步最近码字复现当前标签。每步1个运动token、64个运动码字及STOP=0，解码仍为码字的局部[前向、左向、偏航]。离线编码器、源码和参数在 k64/encoder/。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

先读 `DATASETS.json` 与 `RELEASE_COMPLETE.json`，再读取以下索引并核对其码本 SHA256：

| 分支 | 码本 SHA256 | 索引 SHA256 |
|---|---|---|
| K32 | `0429002c179e4223b21a5faf5fd00ae18326f7cd5b2384be22dea9799cc0c16b` | `2577875b67beed1854d607707d40283d59862485dca2754169f878cc5ccd12ef` |
| K64 | `f0cc68c7892d8b43b1c1a9aa5a309db0b9053b6864b3fd73f96aa94d8249950b` | `f33306080bbe07987bb3d5abdb15797b02a34d3be4272b663a20f4f0b22c105e` |
| RVQ | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` | `15c9b95cb83866a9fe9c796c863124178fa9439b7fc40e83b50bb51befa75e13` |

训练索引依次为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；码本在各分支 `codebook.npz`。在授权 OSS 下载 `tools/reader.tar.gz` 并解压，使用例如：

```bash
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k64 --workers 0
```

分支也可选 `k32`、`rvq`。K32/K64 使用局部 [前向、左向、偏航]，0 为 STOP；RVQ 三层 ID 的完整 `[0,0,0]` 才是 STOP，单层 0 为普通码字。RVQ 以 `future_valid_mask` 排除终点填充，仅执行 `decoded[0]` 然后重新观测。历史仅使用此前真实保留节点，时间差不是控制截止时间。视频从原始 OSS tar 通过 Range 恢复并校验。

编码 ID 为 uint16，训练模型需自己注册原子 token、映射词表并调整 embedding。独立训练账户的 OSS 权限、策略训练及真机表现尚未验证。主要5/10步FDE见 `report/downloads/K64_WINDOW_COMPARISON.json`；辅助整轨迹诊断和码频见编码报告及 report/downloads/JOINT_FULL_METRICS.json；旧误差统计不适用于本版。
