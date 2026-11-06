# 点阵字形实验项目 (dotmatrix_glyph)

基于 HZK32 32×32 点阵字库的字形相似度实验项目。通过将汉字转为 1024 维 0/1 二值向量，计算余弦相似度来探索形近字的聚类效果。

## 项目背景

本项目是 NeuroGlyph34 系列的子实验，旨在对比不同字形表示维度（34 维结构特征 vs 1024 维点阵特征）在形近字聚类任务上的表现。NeuroGlyph34 使用 34 维结构化特征（四角号码、五笔、笔画、笔顺、拼音、部首等），而本实验使用 HZK32 的 32×32 原始点阵（1024 维）作为对照。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `HZK32` | ~846 KB | GB2312 点阵字库（B0-F7 区，6768 字，每字 128 字节） |
| `hzk32_reader.py` | - | HZK32 字库读取器，按 GB2312 区位码解析点阵 |
| `shape_sim_test.py` | - | 形近字聚类快速验证脚本 |
| `char_to_idx.json` | ~100 KB | 词表（字符→索引映射），来自 neuroglyph34_cloud_v10 |
| `idx_to_char.json` | ~116 KB | 词表（索引→字符映射），来自 neuroglyph34_project |
| `model_v10.py` | ~4.3 KB | NeuroGlyph34V10 模型架构参考 |
| `vector_info.txt` | ~1.7 KB | 34 维结构化特征说明（对照参考） |
| `README.md` | - | 本文件 |

## 语料资源

语料文件路径（因文件过大仅记录路径）：

- **corpus_v10_article.txt**: `D:\Desktop\学习结果小文件\neuroglyph34_cloud_v10\corpus_v10_article.txt` (~2.0 GB)
- **corpus_v10_article.txt.gz**: `D:\Desktop\学习结果小文件\neuroglyph34_cloud_v10\corpus_v10_article.txt.gz` (~291 MB)

## 技术细节

### HZK32 格式

- 编码：GB2312
- 汉字范围：区码 B0-F7（72 区 × 94 字 = 6768 字）
- 点阵规格：32×32 = 1024 位 = 128 字节/字
- 文件大小：866,304 字节
- 偏移公式：`offset = ((区码 - 0xA1) × 94 + (位码 - 0xA1)) × 128`

### GB2312 区位码

```
区码 = GB2312 编码第一字节
位码 = GB2312 编码第二字节
汉字区: B0A1 (啊) ~ F7FE
```

## 使用方法

### 查看单个汉字点阵

```bash
python hzk32_reader.py
```

### 运行形近字聚类验证

```bash
python shape_sim_test.py
```

### 在代码中使用

```python
from hzk32_reader import HZK32Reader

reader = HZK32Reader("HZK32")
vector = reader.get_glyph_vector("汉")  # (1024,) numpy array
matrix = reader.get_glyph_matrix("汉")  # (32, 32) numpy array
reader.print_glyph("汉")                # 终端打印
```

## 预期结果

- 形近字对（己/已/巳、人/入/八、未/末 等）的余弦相似度应显著高于随机基线
- 全局相似度分布应呈现长尾特征
- 可对比 34 维结构化特征的结果，评估不同表示维度的优劣

## 相关项目

- `neuroglyph34_cloud_v10` ~ `v12`: 34 维字形嵌入训练项目
- `neuroglyph34_project`: 项目根目录，含原始数据和工具脚本

---

**创建日期**: 2026-06-28
**HZK32 状态**: 合成占位文件（网络限制，无法下载真实点阵数据）
