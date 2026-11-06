# 统一嵌入层字库使用指南

## 文件说明

| 文件名 | 说明 |
|--------|------|
| `汉字向量字库.json` | 原始汉字向量 (6000字, 18维) |
| `符号向量字库.json` | 原始符号向量 (168个, 18维) |
| `merged_vocab.json` | 合并后的词汇表 |
| `embedding_matrix.npy` | 嵌入矩阵 (需运行脚本生成) |
| `generate_embedding_matrix.py` | 生成嵌入矩阵的脚本 |
| `unified_embedding_pytorch.py` | PyTorch嵌入层 |
| `unified_embedding_tensorflow.py` | TensorFlow嵌入层 |

## 快速开始

### 1. 生成嵌入矩阵

```bash
python generate_embedding_matrix.py
```

这将生成 `embedding_matrix.npy` 文件 (约 440KB)。

### 2. PyTorch 使用方法

```python
from unified_embedding_pytorch import UnifiedEmbedding
import torch

# 创建嵌入层
embedding = UnifiedEmbedding(
    vocab_file='merged_vocab.json',
    embedding_file='embedding_matrix.npy',
    padding_idx=0
)

# 编码文本
text = "你好世界"
indices = embedding.encode(text)
print(f"编码: {indices}")  # tensor([4, 5, 6, 7])

# 获取嵌入
embeddings = embedding(indices.unsqueeze(0))
print(f"嵌入形状: {embeddings.shape}")  # torch.Size([1, 4, 18])

# 解码
decoded = embedding.decode(indices)
print(f"解码: {decoded}")  # 你好世界
```

### 3. TensorFlow 使用方法

```python
from unified_embedding_tensorflow import UnifiedEmbedding
import tensorflow as tf

# 创建嵌入层
embedding = UnifiedEmbedding(
    vocab_file='merged_vocab.json',
    embedding_matrix='embedding_matrix.npy',
    mask_zero=True
)

# 编码文本
text = "你好世界"
indices = embedding.encode(text)
print(f"编码: {indices.numpy()}")  # [4 5 6 7]

# 获取嵌入
embeddings = embedding(tf.expand_dims(indices, 0))
print(f"嵌入形状: {embeddings.shape}")  # (1, 4, 18)

# 解码
decoded = embedding.decode(indices.numpy())
print(f"解码: {decoded}")  # 你好世界
```

### 4. 直接使用NumPy矩阵

```python
import numpy as np

# 加载嵌入矩阵
embeddings = np.load('embedding_matrix.npy')
print(f"矩阵形状: {embeddings.shape}")  # (6172, 18)

# 手动查找嵌入
char = "中"
# 需要先查词汇表获取索引
# 然后直接索引
```

## 词汇表索引分布

| 索引范围 | 内容 | 数量 |
|---------|------|------|
| 0 | `<PAD>` (填充符) | 1 |
| 1 | `<UNK>` (未知符) | 1 |
| 2 | `<SOS>` (句首) | 1 |
| 3 | `<EOS>` (句尾) | 1 |
| 4 - 6003 | 汉字 | 6000 |
| 6004 - 6171 | 符号 | 168 |

**总计: 6172 个token**

## 嵌入向量格式

- **维度**: 18维
- **数据类型**: float32
- **值范围**: 0-25 (对应字母a-z的编码)

### 向量构成

**汉字向量 (18维)**:
- 位置 0-3: 四角号码 (4位数字)
- 位置 4-7: 五笔字型编码 (a=1, b=2, ... z=26)
- 位置 8-13: 拼音编码 (a=1, b=2, ... z=26)
- 位置 14: 声调 (1=平声, 2=上声, 3=去声, 4=入声, 5=轻声)
- 位置 15: 字符类型 (0=汉字)
- 位置 16-17: 保留

**符号向量 (18维)**:
- 位置 0-15: 全为0
- 位置 16-17: 16×16矩阵坐标 (可表示256个符号)

## 在模型中使用

### PyTorch nn.Embedding

```python
import torch
import torch.nn as nn
import numpy as np

# 加载嵌入矩阵
embeddings = np.load('embedding_matrix.npy')

# 创建嵌入层
embedding_layer = nn.Embedding.from_pretrained(
    torch.tensor(embeddings, dtype=torch.float32),
    padding_idx=0
)

# 使用
input_ids = torch.tensor([[4, 5, 6, 7]])  # 示例
output = embedding_layer(input_ids)  # shape: (1, 4, 18)
```

### TensorFlow keras.layers.Embedding

```python
import tensorflow as tf
import numpy as np

# 加载嵌入矩阵
embeddings = np.load('embedding_matrix.npy')

# 创建嵌入层
embedding_layer = tf.keras.layers.Embedding(
    input_dim=6172,
    output_dim=18,
    weights=[embeddings.astype(np.float32)],
    mask_zero=True
)

# 使用
input_ids = tf.constant([[4, 5, 6, 7]])  # 示例
output = embedding_layer(input_ids)  # shape: (1, 4, 18)
```

## 注意事项

1. **首次使用需运行生成脚本**: `python generate_embedding_matrix.py`
2. **词汇表索引从0开始**: `<PAD>` 固定为索引0
3. **UNK处理**: 未知字符会自动映射到 `<UNK>` (索引1)
4. **兼容性好**: 生成的 `.npy` 文件可在任何支持NumPy的框架中使用
