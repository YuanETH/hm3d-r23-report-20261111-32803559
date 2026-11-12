# HM3D R23 OSS 数据接入说明

本批是尚未划分训练/验证集的 `generation_pool`，默认编码分支为 K64。不要把池名或 K64 默认值解释为 train，也不要将本批离线量化误差称为独立验证/测试集性能。

发布前缀：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/`

统一清单：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/DATASETS.json`

完成记录：`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/RELEASE_COMPLETE.json`。读取器默认要求完成记录存在且状态为 complete；没有完成记录时不得投入默认训练。

来源：原始 `oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/`，补采 `oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/supplement-20261111/`。本次使用 OSS QC_HANDOFF SHA256 `9611c09a33a0a899406696511b7c8b8bb9437890a4d435a06a681241990249cb`。逐来源对账和未知项见 RECONCILIATION.json。

本次未拟合，使用固定参考码本。三份码本已复制到交付前缀，训练端无需挂载参考 CPFS。

|分支|训练索引|码本 SHA256|
|---|---|---|
|k32|`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/k32/TRAINING_INDEX.json`|`2cd130dc5ae806c3334e5780fb4d529c5cacb6aa08cffc403321cd5b5e36d66a`|
|k64|`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/k64/TRAINING_INDEX.json`|`0cfa304c662a598c32bd2e3f655d3b72ebb28ce79040aee6d6557c64dc780ecc`|
|rvq|`oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/rvq/TRAINING_INDEX.json`|`a77f0577ac9b63b9d652f5482b250a9c434fabcb2184e8ec44a291fde94b8d3f`|

获取小型读取器代码包（不是原始数据集归档）：

```bash
ossutil cp oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/tools/reader.tar.gz ./hm3d-reader.tar.gz
mkdir -p hm3d-reader
tar -xzf hm3d-reader.tar.gz -C hm3d-reader
cd hm3d-reader
python -m pip install alibabacloud-oss-v2 requests numpy av torch
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --workers 0 --batch-size 4
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --workers 2 --batch-size 4
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch k32 --workers 2 --batch-size 4
python reader.py oss://ss-oss1/data/user/yuanxu/vln_data/hm3d-r23-k3-bin1-20261105/_qc_releases/hm3d-qc-20261111T175811-32803559/ --branch rvq --workers 2 --batch-size 4
```

默认 `NavigationDataset(prefix)` 返回 K64；显式 `branch='k32'` 或 `'rvq'` 切换。`partition='generation_pool'`，split 为 null。Dataset 长度、索引和分片来自 TRAINING_INDEX.json，读取器不扫描计划清单或原始全集。

视频保留在授权 OSS tar 内。每条标注提供已验证的 archive_uri、member_path、video_sha256、video_size、member_offset。仅未压缩 tar 使用 OSS Range 取单个 MP4，并核对该 MP4 的完整 SHA256。它不是独立 MP4 OSS URI。视频在有界内存文件中顺序解码到精确原始帧序号；不使用近似 seek 冒充指定观测。

世界位姿 poses 为 NWU（北、西、上）的 [x,y,z,yaw]；局部动作相对当前机身前、左及逆时针转角。图像为 uint8 RGB H×W×3。codec_ids 为 uint16；K32/K64 单个 ID，RVQ 三个 ID。机身局部动作单位为米、逆时针弧度。历史按此前保留节点的时间顺序取值，跳过帧不再加入历史。首末帧均为真实原始观测，只有原始末帧 STOP。各宏节点记录时间差不构成固定控制截止时间。

K32/K64 每次执行一个解码宏目标。RVQ 五点都是相对当前观测的绝对局部位姿；future_valid_mask 排除重复真实终点的 padding。每次只执行 decoded[0] 后重新观测。完整三元组 000 与 <apos_1299> 只在真实原始终点同时出现；单独某一级 0 不是 STOP。

Object 为目标标注框几何中心的相机视锥投影，区域目标用 <opos_0>。Pointing 采用路径 5 cm 格点、4 m 前瞻的首个连续视野内区段最远点；不声明证明无遮挡。

真实模型接线：安装 transformers、Pillow，使用 `register_tokens(tokenizer, model, branch)` 注册原子 token 并调整 embedding；用 `ProcessorCollator(processor, token_map)` 对保留图像序列调用模型 chat template/processor，仅监督 assistant 动作 token。请勿把 codec ID 直接当词表 ID。多 worker 使用 spawn，避免在 CUDA 初始化后 fork 导致继承锁状态。

已实际测试真实 Qwen3-VL tokenizer/processor、原子 token 注册、tiny Qwen3 测试模型 embedding 调整以及 0/2 worker、batch size 4 的精确像素与历史一致性；测试记录见 VALIDATION.json。这不是对外部训练作业的启动或完整策略训练声明。

OSS 访问使用训练用户现有 ~/.ossutilconfig [default] 的地域、endpoint 和权限。当前客户端已直接连接配置的内网 endpoint，并验证交付引用可读取。未提供另一个训练账户的凭据，因此不宣称验证了其他主体权限。不要公开桶 ACL、密钥或带凭据临时签名 URL。

质量保留、unresolved_gt、unresolved_rvq、分支数量与完整性未知项分别见 DATASETS.json、RECONCILIATION.json、ENCODING.json、EXCLUSIONS_INDEX.json。逐 canonical episode 的最终分数、指标、规则、视觉修订与来源依据见 qc/FINAL_QC_INDEX.json。目标、指令、相机 active profile 与冻结归档的独立交叉核对见 qc/METADATA_AUDIT_INDEX.json。初次扫描与逐视频 checkpoint 为不可变的过程证据；经过冻结目标／相机／地形高度核对与显式视觉修订后的最终结论以 qc/FINAL_QC_INDEX.json 为准。地形对应误报仅在归档修正上限及 XY／航向／时刻吻合时清除，真实观测不变，不构成黑区规则放行。qc/AI_REVIEWS.json 仅覆盖明确记录的抽帧范围，不能外推为全量 AI 视频审核。
