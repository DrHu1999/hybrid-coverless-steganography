# 全身人物图像生成与映射融合无载体隐写

本仓库依据胡博等人的论文《A hybrid generative and mapping method for coverless image steganography with full-body human images》重新编写。它是论文方法的可配置研究实现，**不是当时实验源代码的恢复版本，也不表示已复现论文中的准确率**。

## 能运行的部分

核心编码、解码、消息分段、补零处理、属性筛选和32种攻击操作可在普通CPU环境运行。图像生成、从图像中识别人脸属性和人体姿态需要另行配置真实模型和权重，详见 [模型安装说明](docs/MODELS.md)。发送端和接收端均有实际代码，没有把关键模型留成 `pass` 或使用随机结果代替识别。

论文采用MagicPoser人工构造姿态，本仓库保留这一步：根据消息得到的目标区间，在姿态编辑器中构造对应控制图，再登记到姿态库。没有论文原始姿态库时，需要自行建立所用编码的模板；不能只安装Python依赖就获得任意消息的完整图像生成能力。

## 第一次运行

在解压后的仓库根目录打开终端。建议Python 3.10或3.11。

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e .
python -m unittest discover -s tests -v
python -m hcs demo --output runs/demo
```

Linux/macOS激活命令为 `source .venv/bin/activate`。如果不使用虚拟环境，至少使用独立的Python环境，避免机器学习依赖互相冲突。

`demo`只验证公式、码本和消息协议，生成的骨架图及理想观测均明确标为数学演示。不能把它当成扩散模型生成图像、真实识别或抗攻击实验的证据。

## 配置选择

- `configs/paper28.json`：论文明确列出的15个人脸属性，加上13位姿态编码；对应容量28比特/图像。
- `configs/example24.json`：12位属性加12位姿态的示例配置。论文未完整公开该容量下最优12属性集合，因此这里的属性选择不是论文最优组合，不直接对应论文的80%以上平均准确率。

接收端须使用相同属性顺序、姿态位宽、阈值、模型和预处理。属性列表的顺序也是码本的一部分。

## 从消息到图像

先生成发送端计划，里面含秘密比特和生成条件，只保存在 `private/`，不要上传到公共仓库或放入传输包。

```powershell
python -m hcs plan --config configs/paper28.json --input examples/message.txt --output private/plan.json
```

计划会给出每段的人脸属性、正负提示词、左右肘角度区间、双腕间距比、双踝间距比和可选鼻部距离比较。根据姿态约束构造图像，导出同一画布上的COCO-17关键点，然后注册模板：

```powershell
python -m hcs add-pose --config configs/paper28.json --bank pose_bank --keypoints private/pose-keypoints.json --width 512 --height 768 --control private/control.png
```

`--control`为可选项。提供时使用该真实控制图；省略时由关键点绘制骨架。关键点格式、左右编号和控制图要求见 [POSE_BANK.md](docs/POSE_BANK.md)。登记时根据坐标计算编码，不采信人工填写的秘密标签。

配置模型路径后，执行：

```powershell
python -m hcs send --config configs/paper28.json --input examples/message.txt --pose-bank pose_bank --output runs/transmission
python -m hcs receive --config configs/paper28.json --images runs/transmission --output runs/recovered.txt
```

支持任意二进制文件；文字使用原文件字节。若输入已是01字符串，用 `--input-format bits`，恢复时可用 `--output-format bits`。每图容量很小，长文件会生成很多图像，另外还有一幅用于传递补零数量的图像。

实际生成只有在回读的完整属性和姿态序列都正确时才保存为有效载体。最多尝试10次，失败则中止，不能把部分图像当成完整消息。接收端从图像像素重新识别，没有通过文件名、PNG元数据或发送端标签恢复秘密信息。

## 论文实验对应与公开发布

[REPRODUCIBILITY.md](docs/REPRODUCIBILITY.md)逐项解释已实现的方法、缺失的原始实验材料和工程约定。[PUBLISHING.md](docs/PUBLISHING.md)说明如何上传GitHub或Gitee。

当前自动化测试覆盖的是数学、协议、重试流程及图像处理代码。真实扩散、SwinFace、RetinaFace和ViTPose推理需要权重和相应环境，发布前应运行真实图像验证，并保存实际环境、参数和实验日志。仓库已把这部分状态写清，不预填成功率。
