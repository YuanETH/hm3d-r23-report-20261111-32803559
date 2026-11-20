# HM3D R23 数据接入（2026-11-20 K32/K64/RVQ 短窗口FDE版）

深圳与HM3D的质检保留轨迹共用码本，K32/K64/RVQ均为未来5/10步FDE版。HM3D 是未划分训练、验证或测试集的 `generation_pool`，默认 `k64`。

深圳与HM3D共用联合码本。K32、K64和RVQ均以未来5步、10步执行轨迹的终点FDE为主要目标，联合拟合码本和离线标签序列，两个步长使用同一条序列；整轨迹FDE和每步ADE只作辅助诊断。K32/K64每步1个运动token，分别32/64个运动码字加STOP=0，近直行前进距离间隔至少5cm。RVQ每步仍为3个残差token、每层256个选项，保持未来5节点的加和解码与小动作检查。本次更新K32和RVQ，K64沿用已完成的窗口FDE版。

K64 编码约定（2026-11-20）：标签使用真实未来路径，联合搜索同一条码字序列来减小5步和10步FDE，不能用单步最近码字复现当前标签。每步1个运动token、64个运动码字及STOP=0，解码仍为码字的局部[前向、左向、偏航]。离线编码器、源码和参数在 k64/encoder/。

K32/RVQ 编码约定（2026-11-20）：直接使用当前码本和GT，确定性联合搜索同一条序列以减小未来5步、10步FDE；无需历史标签或码本。K32每步1个运动token，RVQ每步3个残差token。完整离线源码和参数分别在 k32/encoder/ 与 rvq/encoder/；解码接口保持一致。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

先读 `DATASETS.json` 与 `RELEASE_COMPLETE.json`，再读取以下索引并核对其码本 SHA256：

| 分支 | 码本 SHA256 | 索引 SHA256 |
|---|---|---|
| K32 | `6f94bedcd6e1330e796c3cd71ae9d1d18ffb4156e41ef01ef665ddf1938816b2` | `75d232413b09903526ff0fa78cea9d91f6dec8e2291c653f22d7c814eb4a4c8a` |
| K64 | `f0cc68c7892d8b43b1c1a9aa5a309db0b9053b6864b3fd73f96aa94d8249950b` | `f33306080bbe07987bb3d5abdb15797b02a34d3be4272b663a20f4f0b22c105e` |
| RVQ | `0966c2ba8d04e8fd868a5df92366c2693bf9f1ff1226e74d65c07ad5a562eeee` | `41df9e3c06296476b48ada6c4b93d9cbd29dd1d9b634964297dc13210fa9bcb1` |

训练索引依次为 `k32/TRAINING_INDEX.json`、`k64/TRAINING_INDEX.json`、`rvq/TRAINING_INDEX.json`；码本在各分支 `codebook.npz`。在授权 OSS 下载 `tools/reader.tar.gz` 并解压，使用例如：

```bash
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k64 --workers 0
```

分支也可选 `k32`、`rvq`。K32/K64 使用局部 [前向、左向、偏航]，0 为 STOP；RVQ 三层 ID 的完整 `[0,0,0]` 才是 STOP，单层 0 为普通码字。RVQ 以 `future_valid_mask` 排除终点填充，仅执行 `decoded[0]` 然后重新观测。历史仅使用此前真实保留节点，时间差不是控制截止时间。视频从原始 OSS tar 通过 Range 恢复并校验。

编码 ID 为 uint16，训练模型需自己注册原子 token、映射词表并调整 embedding。独立训练账户的 OSS 权限、策略训练及真机表现尚未验证。主要5/10步FDE见 `report/downloads/K32_WINDOW_COMPARISON.json`、`RVQ_WINDOW_COMPARISON.json`、`K64_WINDOW_COMPARISON.json`；辅助整轨迹诊断和码频见编码报告及 report/downloads/JOINT_FULL_METRICS.json；旧误差统计不适用于本版。
