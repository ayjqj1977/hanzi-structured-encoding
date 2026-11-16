# hanzi-structured-encoding

汉字结构化编码开源项目：从 18 维手工特征，到 34 维 NeuroGlyph，再到 HZK32 点阵与可解释字形判别。

一位业余研究者（非 AI 从业者）用业余时间和少量云算力完成的实验记录、代码与数据。核心结论不是"我做成了"，而是"哪条路被消融证伪、为什么"。

## 仓库结构

| 路径 | 内容 |
|------|------|
| `code/` | 18/32 维嵌入层（PyTorch / TensorFlow / NumPy）、NeuroGlyph-34、PointMatrix、HZK32 读取器、复现脚本 |
| `data/` | 18/32 维汉字向量字库（JSON/CSV），6172–7987 字符 |
| `docs/` | HZK32 点阵实验报告、两年/近三个月全记录、数据集说明、结果口径总表 |
| `neuroglyph/` | NeuroGlyph-34 的 NumPy 权重（`neuroglyph34_np.npz`）与云端训练说明 |
| `index.html` / `neuroglyph_dashboard.html` | 数据与训练过程的可视化面板 |

## 快速开始

```bash
pip install -r requirements.txt
# 免训练复现字形→语义预测（示例：树）
python code/reproduce.py 树
```

- `code/train_spatial.py`：HZK32 点阵 + 卷积编码器训练
- `code/train_pointnet.py`：点云/全连接版本
- `code/train_neuroglyph34.py`：34 维 NeuroGlyph 训练
- 数据格式见 `docs/开源数据集说明.md`

## 结果口径

不同文档中的 PPL 数字来自不同实验批次与评估方式，**不能直接横向比较**。请先读 [docs/结果口径总表.md](docs/结果口径总表.md)。

- 18 维：音近 1.55x、形近 1.17x
- 34 维+V6：纯 34 维 PPL 59.56
- 消融：96 维纯随机 PPL 9.66，优于 34 冻死 + 64（10.07）
- 点阵：PointMatrix 字形区分力 17–18x，PPL 14.12
- 规则：G2 留出近形字对均值 0.0152，末/未 15.8%

## 文档

- [HZK32 点阵编码实验报告](docs/HZK32点阵编码实验报告.md)
- [近三个月实验记录](docs/近三个月实验记录.md)
- [两年研究全记录](docs/两年研究全记录.md)
- [开源数据集说明](docs/开源数据集说明.md)
- [结果口径总表](docs/结果口径总表.md)

## 许可

CC BY-NC-SA 4.0（署名—非商业性使用—相同方式共享）。商业使用请联系作者。
