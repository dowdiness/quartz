---
aliases: ["Part 1: 機械学習モデルの分類法"]
created: 2026-01-24T16:42:37+09:00
modified: 2026-01-24T16:56:46+09:00
---

# Part 1: 機械学習モデルの分類法

## 📚 この章について

この章では、機械学習モデルを**実装の観点**から分類する方法を学びます。従来の「RNN vs CNN vs Transformer」といった表面的な分類ではなく、より本質的な軸で理解します。

**学習時間**: 2-4週間  
**前提知識**: Python, 基本的なニューラルネットワーク

## 🎯 学習目標

- [ ] Sequential vs Non-Sequentialの違いを説明できる
- [ ] Autoregressiveの意味と実装を理解する
- [ ] Local vs Global dependencyのトレードオフを説明できる
- [ ] 各分類軸が計算複雑性に与える影響を理解する
- [ ] State Space Models (Mamba) の位置づけを理解する

---

## 1. なぜ新しい分類が必要か？

### 1.1 従来の分類の問題点

```python
# 従来の分類
models = {
    "RNN": "系列データ用",
    "CNN": "画像用",
    "Transformer": "何でも使える"
}
```

**問題**:
1. タスクベースの分類で、本質が見えない
2. 新しいモデル（Mamba等）が分類できない
3. 計算複雑性やハードウェア適性が不明

### 1.2 実装視点の重要性

```python
# 実装視点での分類
model_properties = {
    "parallelizable": bool,      # 並列化可能か
    "memory_complexity": str,     # メモリ使用量
    "time_complexity": str,       # 計算時間
    "dependency_range": str,      # 依存範囲
}
```

**利点**:
1. ハードウェア選択の指針
2. 実装の最適化戦略が明確
3. モデル設計の理論的根拠

---

## 2. 分類軸1: Sequential vs Non-Sequential

### 2.1 定義

**Sequential (系列的)**:
- データに**時間的順序**または**位置的順序**がある
- 順序を変えると意味が変わる

**Non-Sequential (非系列的)**:
- データの順序が無関係
- 順序を変えても本質的に同じ

### 2.2 具体例

```python
# Sequential データ
text = ["The", "cat", "sat"]  # 順序重要
# "sat cat The" は異なる意味

# Non-Sequential データ
image = np.array([[r, g, b], ...])  # ピクセル位置は重要だが、
# 画像全体をランダム順で処理しても問題ない（適切な位置情報があれば）

# 集合データ（完全に順序無関係）
tags = {"AI", "ML", "DL"}  # 順序無関係
```

### 2.3 実装への影響

```python
# Sequential モデル (RNN)
class SequentialModel:
    def forward(self, sequence):
        state = self.initial_state
        outputs = []
        
        # 必ず順番に処理
        for token in sequence:  # for ループ必須
            state = self.update(state, token)
            outputs.append(self.output(state))
        
        return outputs
    
    # 並列化: 困難 ❌

# Non-Sequential モデル (CNN on image)
class NonSequentialModel:
    def forward(self, image):
        # 全ピクセルを同時処理可能
        features = self.conv(image)  # 完全並列
        return self.classify(features)
    
    # 並列化: 容易 ✅
```

### 2.4 計算複雑性

| 性質 | Sequential | Non-Sequential |
|------|-----------|----------------|
| **並列化** | 困難 (逐次依存) | 容易 |
| **GPU効率** | 低い | 高い |
| **訓練時間** | O(N) serial | O(1) parallel |
| **推論時間** | 累積的 | 一定 |

### 2.5 演習問題

**Q1.1**: 以下のデータは Sequential か Non-Sequential か？
1. 音声波形
2. グラフのノード集合
3. 動画フレーム
4. 点群データ (Point Cloud)

**Q1.2**: Sequential データを Non-Sequential に変換する方法を考えよ
（ヒント: 位置エンコーディング）

---

## 3. 分類軸2: Autoregressive vs Non-Autoregressive

### 3.1 定義

**Autoregressive (自己回帰的)**:
- 出力を**逐次的に生成**
- 前の出力を次の入力として使用

**Non-Autoregressive**:
- 出力を**並列に生成**
- 全出力を同時に予測

### 3.2 具体例

```python
# Autoregressive (GPT)
class AutoregressiveModel:
    def generate(self, prompt, max_length):
        tokens = [prompt]
        
        for i in range(max_length):
            # 前の出力を使って次を予測
            next_token = self.predict(tokens)  # tokens に依存
            tokens.append(next_token)
        
        return tokens
    
    # 生成時間: O(N) - 並列化不可 ❌

# Non-Autoregressive (BERT masked prediction)
class NonAutoregressiveModel:
    def predict_masked(self, tokens_with_mask):
        # 全マスクを同時予測
        predictions = self.model(tokens_with_mask)  # 並列
        return predictions
    
    # 生成時間: O(1) - 完全並列 ✅
```

### 3.3 数学的定義

```python
# Autoregressive
P(y1, y2, ..., yN) = ∏ P(yi | y1, ..., y_{i-1})
#                     連鎖律で分解

# Non-Autoregressive
P(y1, y2, ..., yN) = P(y1, y2, ..., yN | x)
#                    同時予測（条件付き独立を仮定）
```

### 3.4 トレードオフ

| 性質 | Autoregressive | Non-Autoregressive |
|------|---------------|-------------------|
| **品質** | 高い（依存関係を正確にモデル化） | 中程度 |
| **速度** | 遅い (O(N)) | 速い (O(1)) |
| **柔軟性** | 可変長出力 | 固定長が基本 |
| **訓練** | Teacher forcing可能 | 難しい |

### 3.5 ハイブリッドアプローチ

```python
# 反復精錬 (Iterative Refinement)
class HybridModel:
    def generate(self, input, iterations=5):
        # 最初は並列生成
        output = self.non_autoregressive_init(input)
        
        # 反復的に改善
        for _ in range(iterations):
            output = self.refine(output)  # 並列
        
        return output
    
    # 速度と品質のバランス ✅
```

**実例**: 
- NAT (Non-Autoregressive Translation)
- Iterative Refinement Models
- Diffusion Models (段階的生成)

### 3.6 演習問題

**Q2.1**: 機械翻訳で Non-Autoregressive が難しい理由を説明せよ

**Q2.2**: 画像生成タスクで Autoregressive が有効な理由を考えよ
（ヒント: PixelCNN）

---

## 4. 分類軸3: Local vs Global Dependency

### 4.1 定義

**Local Dependency (局所的依存)**:
- 近傍の要素のみに依存
- 固定サイズの受容野 (receptive field)

**Global Dependency (大域的依存)**:
- 全要素に依存可能
- 無制限の受容野

### 4.2 具体例

```python
# Local: CNN
class LocalModel:
    def forward(self, x):
        # kernel_size=3 の場合
        # 各出力は近傍3要素のみに依存
        return self.conv1d(x, kernel_size=3)
    
    # 受容野: 固定 (3) 🔒

# Global: Transformer
class GlobalModel:
    def forward(self, x):
        # Self-attention
        # 各出力は全入力に依存
        Q = self.query(x)
        K = self.key(x)
        V = self.value(x)
        
        attn = softmax(Q @ K.T / sqrt(d))  # 全要素間の関係
        return attn @ V
    
    # 受容野: 無制限 (全体) 🌐
```

### 4.3 計算複雑性

```python
# Local (CNN)
# 入力長: N, カーネルサイズ: k
time_complexity = O(N * k)    # 線形
space_complexity = O(N)

# Global (Transformer)
# 入力長: N
time_complexity = O(N^2)      # 二乗
space_complexity = O(N^2)     # Attention matrix
```

### 4.4 階層的アプローチ

```python
# 深いCNN = 階層的に受容野を拡大
class HierarchicalLocal:
    def __init__(self):
        # 層ごとに受容野が拡大
        self.conv1 = Conv1D(kernel_size=3)  # 受容野: 3
        self.conv2 = Conv1D(kernel_size=3)  # 受容野: 5
        self.conv3 = Conv1D(kernel_size=3)  # 受容野: 7
        # ... L層で受容野 ≈ 2L+1
    
    def forward(self, x):
        x = self.conv1(x)
        x = self.conv2(x)
        x = self.conv3(x)
        return x
```

**効果的受容野**:
```
層1: ●●●
層2:  ●●●●●
層3:   ●●●●●●●
→ 最終的に7要素を見る
```

### 4.5 ハイブリッド: Sparse Attention

```python
# Longformer, BigBird 等
class SparseAttention:
    def forward(self, x, window_size=256):
        N = len(x)
        attn_mask = self.create_sparse_mask(N, window_size)
        
        # Local attention (window内)
        local_attn = self.local_attention(x, window_size)
        
        # Global attention (特定のトークンのみ)
        global_attn = self.global_attention(x, global_tokens=[0, -1])
        
        return local_attn + global_attn
    
    # 複雑性: O(N * window_size) ≈ O(N)
```

### 4.6 演習問題

**Q3.1**: なぜ CNNは画像認識で成功したのか？Local biasの観点から説明せよ

**Q3.2**: Transformer で系列長10,000を処理する場合のメモリ使用量を計算せよ
（d=512, float32）

**Q3.3**: Sparse Attention の具体的なマスクパターンを3つ設計せよ

---

## 5. 統合的分類

### 5.1 主要モデルの分類

| モデル | Sequential | Autoregressive | Dependency | 複雑性 (訓練) | 複雑性 (推論) |
|--------|-----------|---------------|------------|------------|------------|
| **RNN** | ✅ | ✅ | Local* | O(N) serial | O(N) serial |
| **LSTM** | ✅ | ✅ | Local* | O(N) serial | O(N) serial |
| **CNN** | ❌ | ❌ | Local | O(N) parallel | O(1) parallel |
| **Transformer** | ✅ | ✅/❌** | Global | O(N²) parallel | O(N) / O(N²)** |
| **Mamba (SSM)** | ✅ | ✅ | Global | O(N log N) | O(1) |

*理論上はGlobalだが、勾配消失で実質Local  
**Encoder/Decoderで異なる

### 5.2 State Space Models (Mamba) の位置づけ

```python
# Mamba: 全ての利点を統合
class MambaBlock:
    """
    - Sequential: ✅ (時系列をモデル化)
    - Autoregressive: ✅ (逐次生成可能)
    - Dependency: Global (理論的にN全体)
    - 訓練: O(N log N) (並列スキャン)
    - 推論: O(1) per step (RNN的)
    """
    def __init__(self, d_model, d_state):
        self.d_model = d_model
        self.d_state = d_state
        
        # 選択的パラメータ（入力依存）
        self.delta_proj = nn.Linear(d_model, d_model)
        self.B_proj = nn.Linear(d_model, d_state)
        self.C_proj = nn.Linear(d_model, d_state)
        
        # 状態遷移行列
        self.A = nn.Parameter(torch.randn(d_model, d_state))
    
    def forward(self, x):
        # 選択的パラメータ
        delta = F.softplus(self.delta_proj(x))  # 時間刻み
        B = self.B_proj(x)
        C = self.C_proj(x)
        
        # SSM計算 (並列スキャンで高速化)
        y = self.selective_scan(x, delta, self.A, B, C)
        return y
    
    def selective_scan(self, x, delta, A, B, C):
        """
        並列スキャンアルゴリズム
        訓練: O(N log N)
        推論: O(1) per step
        """
        # 詳細は後述
        pass
```

**Mambaの革新性**:
1. **選択的状態空間**: パラメータが入力依存
2. **並列スキャン**: 訓練時に並列化
3. **定数メモリ**: 推論時にO(1)メモリ

### 5.3 モデル選択の指針

```python
def select_model(task_properties):
    """
    タスク特性に基づくモデル選択
    """
    if not task_properties.is_sequential:
        # 画像、グラフ、集合データ
        if task_properties.has_local_structure:
            return "CNN"
        else:
            return "SetTransformer"
    
    else:  # Sequential
        if task_properties.context_length < 512:
            # 短い系列
            if task_properties.needs_autoregressive:
                return "LSTM or small Transformer"
            else:
                return "BERT-style (masked)"
        
        elif task_properties.context_length < 8192:
            # 中程度の系列
            return "Transformer with optimizations"
        
        else:  # 長い系列
            if task_properties.memory_constraint:
                return "Mamba or other SSM"
            else:
                return "Sparse Transformer or Hybrid"
```

### 5.4 演習問題

**Q4.1**: 以下のタスクに最適なモデルを選択し、理由を説明せよ：
1. 100万トークンの文書要約
2. リアルタイム音声認識
3. タンパク質構造予測
4. 株価予測（過去1年分）

**Q4.2**: Mamba が RNN と Transformer の「良いとこ取り」である理由を、3つの分類軸で説明せよ

---

## 6. 実装例: 3つの分類軸を体験

### 6.1 Sequential Processing

```python
import torch
import torch.nn as nn

# Sequential: RNN
class SimpleRNN(nn.Module):
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.W_ih = nn.Linear(input_size, hidden_size)
        self.W_hh = nn.Linear(hidden_size, hidden_size)
    
    def forward(self, sequence):
        batch_size, seq_len, _ = sequence.shape
        h = torch.zeros(batch_size, self.hidden_size)
        
        outputs = []
        for t in range(seq_len):  # Sequential!
            x_t = sequence[:, t, :]
            h = torch.tanh(self.W_ih(x_t) + self.W_hh(h))
            outputs.append(h)
        
        return torch.stack(outputs, dim=1)

# 使用例
rnn = SimpleRNN(input_size=10, hidden_size=20)
seq = torch.randn(32, 100, 10)  # (batch, seq_len, features)
output = rnn(seq)
print(f"Sequential processing: {output.shape}")
```

### 6.2 Autoregressive Generation

```python
# Autoregressive: GPT-style
class AutoregressiveTransformer(nn.Module):
    def __init__(self, vocab_size, d_model, nhead, num_layers):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.transformer = nn.TransformerDecoder(
            nn.TransformerDecoderLayer(d_model, nhead),
            num_layers
        )
        self.output = nn.Linear(d_model, vocab_size)
    
    def generate(self, prompt, max_length=100):
        """Autoregressive generation"""
        tokens = prompt.clone()
        
        for _ in range(max_length):
            # 現在のトークン列から次を予測
            logits = self.forward(tokens)
            next_token = logits[:, -1, :].argmax(dim=-1)
            
            # 生成したトークンを追加
            tokens = torch.cat([tokens, next_token.unsqueeze(1)], dim=1)
            
            if next_token.item() == EOS_TOKEN:
                break
        
        return tokens

# 使用例
model = AutoregressiveTransformer(vocab_size=10000, d_model=512, nhead=8, num_layers=6)
prompt = torch.tensor([[1, 2, 3]])  # "The cat"
generated = model.generate(prompt, max_length=20)
print(f"Generated tokens: {generated}")
```

### 6.3 Local vs Global Dependency

```python
# Local: 1D CNN
class LocalConv(nn.Module):
    def __init__(self, in_channels, out_channels, kernel_size=3):
        super().__init__()
        self.conv = nn.Conv1d(in_channels, out_channels, kernel_size, padding=kernel_size//2)
    
    def forward(self, x):
        # x: (batch, channels, seq_len)
        return self.conv(x)
    
    def receptive_field(self):
        return "Local (kernel_size)"

# Global: Self-Attention
class GlobalAttention(nn.Module):
    def __init__(self, d_model, nhead):
        super().__init__()
        self.attention = nn.MultiheadAttention(d_model, nhead)
    
    def forward(self, x):
        # x: (seq_len, batch, d_model)
        attn_output, attn_weights = self.attention(x, x, x)
        return attn_output
    
    def receptive_field(self):
        return "Global (all positions)"

# 比較実験
def compare_dependencies():
    seq_len = 1000
    d_model = 512
    
    # Local
    local = LocalConv(d_model, d_model, kernel_size=3)
    x_local = torch.randn(32, d_model, seq_len)
    
    import time
    start = time.time()
    out_local = local(x_local)
    local_time = time.time() - start
    
    # Global
    global_attn = GlobalAttention(d_model, nhead=8)
    x_global = torch.randn(seq_len, 32, d_model)
    
    start = time.time()
    out_global = global_attn(x_global)
    global_time = time.time() - start
    
    print(f"Local (CNN): {local_time:.4f}s, Memory: O(N)")
    print(f"Global (Attention): {global_time:.4f}s, Memory: O(N^2)")

compare_dependencies()
```

### 6.4 演習問題

**Q5.1**: 上記のコードを実行し、seq_len を変えて計算時間を測定せよ（100, 1000, 10000）

**Q5.2**: Mambaの selective_scan を簡略版で実装せよ（並列スキャンは不要）

```python
# ヒント
def simple_ssm(x, A, B, C):
    """
    x: (batch, seq_len, d_model)
    A: (d_model, d_state)
    B: (batch, seq_len, d_state)
    C: (batch, seq_len, d_state)
    """
    batch, seq_len, d_model = x.shape
    d_state = A.shape[1]
    
    h = torch.zeros(batch, d_state)
    outputs = []
    
    for t in range(seq_len):
        # あなたのコード
        pass
    
    return torch.stack(outputs, dim=1)
```

---

## 7. まとめと次章への橋渡し

### 7.1 この章で学んだこと

1. **3つの分類軸**:
   - Sequential vs Non-Sequential
   - Autoregressive vs Non-Autoregressive
   - Local vs Global Dependency

2. **実装への影響**:
   - 並列化可能性
   - 計算複雑性
   - メモリ使用量

3. **最新モデル**:
   - Mamba (State Space Models)
   - Sparse Transformers
   - Hybrid architectures

### 7.2 次章への接続

次章 **02_Automata_Theory.md** では：

1. **形式的な計算モデル**として理解
   - RNN ≈ 有限オートマトン
   - Transformer ≈ Parallel RAM
   - 計算可能性の視点

2. **Chomsky階層との対応**
   - Regular, Context-Free, Context-Sensitive
   - 各モデルが認識できる言語

3. **計算複雑性理論**
   - P, NC, PSPACE
   - 並列化可能性の理論的根拠

### 7.3 発展課題

**プロジェクト1**: Mini-Mamba実装
```python
# 目標: 簡易版Mambaを実装
# 1. SSMの基本実装
# 2. 選択的パラメータ
# 3. （オプション）並列スキャン
```

**プロジェクト2**: モデル比較ベンチマーク
```python
# 異なる系列長でRNN, Transformer, Mambaを比較
# - 訓練時間
# - 推論時間  
# - メモリ使用量
# - 精度
```

---

## 📚 参考文献

### 必読論文

1. **Attention Is All You Need** (Vaswani et al., 2017)
   - Transformer の原論文
   - https://arxiv.org/abs/1706.03762

2. **Mamba: Linear-Time Sequence Modeling** (Gu & Dao, 2023)
   - State Space Models の最新版
   - https://arxiv.org/abs/2312.00752

3. **FlashAttention-2** (Dao, 2023)
   - 効率的Attention実装
   - https://arxiv.org/abs/2307.08691

### 実装リソース

- **PyTorch Tutorials**: https://pytorch.org/tutorials/
- **Mamba実装**: https://github.com/state-spaces/mamba
- **Annotated Transformer**: http://nlp.seas.harvard.edu/annotated-transformer/

### 次のステップ

→ **02_Automata_Theory.md** へ進む
