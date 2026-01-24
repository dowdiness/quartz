---
aliases: ["Part 2: オートマトン理論と計算複雑性"]
created: 2026-01-24T16:42:38+09:00
modified: 2026-01-24T16:56:47+09:00
---

# Part 2: オートマトン理論と計算複雑性

## 📚 この章について

機械学習モデルを**計算理論**の観点から理解します。オートマトン、形式言語、計算複雑性の視点で、各モデルの計算能力と限界を明らかにします。

**学習時間**: 3-4週間  
**前提知識**: Part 1, 基本的なアルゴリズム知識

## 🎯 学習目標

- [ ] Chomsky階層とNNの対応関係を説明できる
- [ ] RNNの計算能力をオートマトン理論で説明できる
- [ ] P, NC, PSPACEなど計算複雑性クラスを理解できる
- [ ] State Space Modelsの理論的位置づけを理解できる
- [ ] 形式言語の観点でモデルを設計できる

---

## 1. Chomsky階層とニューラルネットワーク

### 1.1 Chomsky階層の復習

```
Type 0: 再帰的可算言語 (Recursively Enumerable)
  ├─ チューリング機械
  └─ 制約なし文法
  
Type 1: 文脈依存言語 (Context-Sensitive)
  ├─ 線形拘束オートマトン (LBA)
  └─ 文脈依存文法
  
Type 2: 文脈自由言語 (Context-Free)
  ├─ プッシュダウンオートマトン (PDA)
  └─ 文脈自由文法
  
Type 3: 正規言語 (Regular)
  ├─ 有限オートマトン (FA)
  └─ 正規文法
```

### 1.2 ニューラルネットワークとの対応

```python
# 対応表
automata_nn_correspondence = {
    "Regular (Type 3)": {
        "automaton": "Finite Automaton",
        "nn_equivalent": "1-gram Markov Chain",
        "example": "a*b*"
    },
    "Context-Free (Type 2)": {
        "automaton": "Pushdown Automaton",
        "nn_equivalent": "Stack-Augmented RNN",
        "example": "a^n b^n"
    },
    "Context-Sensitive (Type 1)": {
        "automaton": "Linear Bounded Automaton",
        "nn_equivalent": "Transformer (有限文脈)",
        "example": "a^n b^n c^n"
    },
    "Recursively Enumerable (Type 0)": {
        "automaton": "Turing Machine",
        "nn_equivalent": "Neural Turing Machine",
        "example": "任意の計算可能関数"
    }
}
```

### 1.3 具体例: 形式言語の認識

#### 例1: 正規言語 {a*b*}

```python
# 有限オートマトン
class FiniteAutomaton:
    def __init__(self):
        self.state = 0  # 状態数が有限
    
    def recognize(self, string):
        state = 0
        for char in string:
            if state == 0 and char == 'a':
                state = 0  # そのまま
            elif (state == 0 or state == 1) and char == 'b':
                state = 1
            else:
                return False  # 拒否
        return True

# Markov Chain (NN版)
class MarkovChain:
    def __init__(self):
        self.transitions = {
            ('a', 'a'): 0.9,
            ('a', 'b'): 0.1,
            ('b', 'b'): 1.0,
            ('b', 'a'): 0.0  # 不可能
        }
    
    def probability(self, string):
        prob = 1.0
        for i in range(len(string) - 1):
            prob *= self.transitions.get((string[i], string[i+1]), 0)
        return prob

# テスト
fa = FiniteAutomaton()
mc = MarkovChain()

print(fa.recognize("aaabbb"))  # True
print(fa.recognize("ababab"))  # False
print(mc.probability("aaabbb"))  # > 0
print(mc.probability("ababab"))  # ≈ 0
```

#### 例2: 文脈自由言語 {a^n b^n}

```python
# プッシュダウンオートマトン
class PushdownAutomaton:
    def __init__(self):
        self.stack = []
    
    def recognize(self, string):
        self.stack = []
        
        # フェーズ1: aをスタックにプッシュ
        i = 0
        while i < len(string) and string[i] == 'a':
            self.stack.append('a')
            i += 1
        
        # フェーズ2: bでスタックをポップ
        while i < len(string) and string[i] == 'b':
            if not self.stack:
                return False
            self.stack.pop()
            i += 1
        
        return i == len(string) and len(self.stack) == 0

# Stack-Augmented RNN
class StackRNN:
    def __init__(self, hidden_size, stack_size):
        self.hidden_size = hidden_size
        self.stack = []
        self.max_stack_size = stack_size
    
    def forward(self, sequence):
        h = torch.zeros(self.hidden_size)
        
        for char in sequence:
            # RNNステップ
            h = self.rnn_cell(h, char)
            
            # スタック操作の決定
            action = self.controller(h)  # push/pop/no-op
            
            if action == 'push' and len(self.stack) < self.max_stack_size:
                self.stack.append(h.clone())
            elif action == 'pop' and self.stack:
                stack_top = self.stack.pop()
                h = h + stack_top  # スタック情報を統合
        
        return self.classify(h)

# テスト
pda = PushdownAutomaton()
print(pda.recognize("aaabbb"))  # True (n=3)
print(pda.recognize("aabb"))    # True (n=2)
print(pda.recognize("aaabb"))   # False (n不一致)
```

#### 例3: 文脈依存言語 {a^n b^n c^n}

```python
# 線形拘束オートマトン（簡略版）
class LinearBoundedAutomaton:
    def __init__(self):
        self.tape = []
        self.head = 0
    
    def recognize(self, string):
        # 入力をテープにコピー
        self.tape = list(string)
        n = len(self.tape)
        
        # カウント用のマーカーを配置
        # （簡略版: 実際は複雑な状態遷移）
        
        # a, b, c の数を数える（線形領域内）
        count_a = self.tape.count('a')
        count_b = self.tape.count('b')
        count_c = self.tape.count('c')
        
        # 順序をチェック
        a_phase = all(c == 'a' for c in self.tape[:count_a])
        b_phase = all(c == 'b' for c in self.tape[count_a:count_a+count_b])
        c_phase = all(c == 'c' for c in self.tape[count_a+count_b:])
        
        return count_a == count_b == count_c and a_phase and b_phase and c_phase

# Neural Turing Machine (簡略版)
class SimpleNTM:
    def __init__(self, memory_size, memory_dim):
        self.memory = torch.zeros(memory_size, memory_dim)
        self.controller = nn.LSTM(input_size=10, hidden_size=64)
    
    def forward(self, sequence):
        for char in sequence:
            # コントローラーで読み書き位置を決定
            read_addr = self.controller.read_head(char)
            write_addr = self.controller.write_head(char)
            
            # メモリ読み込み
            read_data = self.memory[read_addr]
            
            # メモリ書き込み
            write_data = self.controller.generate_write(char, read_data)
            self.memory[write_addr] = write_data
        
        # 最終判定
        return self.classify(self.memory)

# テスト
lba = LinearBoundedAutomaton()
print(lba.recognize("aabbcc"))   # True (n=2)
print(lba.recognize("aaabbbccc"))  # True (n=3)
print(lba.recognize("aabbc"))    # False
```

### 1.4 理論的結果

**定理 (Siegelmann, 1995)**: 
実数重みを持つRNNは、チューリング完全である。

**証明のスケッチ**:
```python
# RNNで任意のチューリング機械をシミュレート
class TuringCompleteRNN:
    def __init__(self):
        # 無限精度の実数を使用（理論的）
        self.weights = Real()  # 実数
    
    def encode_tape(self, tape):
        # テープを実数にエンコード
        # 例: [a, b, c] → 0.abc... (base-k encoding)
        encoding = 0
        for i, symbol in enumerate(tape):
            encoding += symbol * (1 / k) ** (i + 1)
        return encoding
    
    def simulate_tm(self, input_tape, tm_rules):
        state = self.encode_tape(input_tape)
        
        for step in range(max_steps):
            # RNNステップ = TM遷移
            state = self.rnn_step(state, tm_rules)
        
        return self.decode_output(state)
```

**実際の制約**:
1. 有限精度（float32/64）→ 実質的には有限状態
2. 勾配消失 → 長距離依存が困難
3. 訓練可能性 → 理論的能力 ≠ 実際に学習可能

### 1.5 演習問題

**Q1.1**: 以下の言語を認識するために必要な最小の計算モデルは？
1. {w | w は偶数個のaを含む}
2. {w | w は回文}
3. {a^p | p は素数}

**Q1.2**: RNNが {a^n b^n} を学習するのが難しい理由を、勾配消失の観点から説明せよ

---

## 2. 計算複雑性理論

### 2.1 複雑性クラスの定義

```python
# 主要な複雑性クラス
complexity_classes = {
    "P": {
        "definition": "決定的多項式時間",
        "example": "最短路、ソート",
        "parallel": "並列化困難なものも含む"
    },
    "NC": {
        "definition": "並列多項式時間（多項式個のプロセッサ）",
        "example": "行列積、FFT",
        "parallel": "効率的に並列化可能"
    },
    "P-complete": {
        "definition": "Pの中で最も難しい問題",
        "example": "回路評価",
        "parallel": "並列化が本質的に困難"
    },
    "NP": {
        "definition": "非決定的多項式時間",
        "example": "SAT、巡回セールスマン",
        "parallel": "検証は容易、発見は困難"
    },
    "PSPACE": {
        "definition": "多項式空間",
        "example": "量化ブール式 (QBF)",
        "parallel": "メモリ制約"
    }
}
```

### 2.2 ニューラルネットワークの複雑性

#### 訓練の複雑性

```python
# 定理: 深層NNの訓練は一般にNP-hard
class TrainingComplexity:
    """
    問題: 誤差をε以下にする重みを見つける
    
    証明のスケッチ:
    - 隠れ層が2層以上
    - 活性化関数がReLU
    → SATに帰着可能
    """
    
    def is_np_hard(self, network):
        if network.num_hidden_layers >= 2:
            return True  # 一般に
        elif network.num_hidden_layers == 1:
            return False  # 凸最適化可能
        else:
            return False  # 線形回帰

# しかし実用上は...
class PracticalTraining:
    """
    勾配降下法: 多項式時間（各イテレーション）
    
    問題:
    - 大域最適解の保証なし
    - 局所最適解で十分なことが多い
    """
    
    def train(self, data, model, iterations):
        # 各イテレーションはO(|data| × |params|)
        for i in range(iterations):
            loss = model.forward(data)
            grads = model.backward(loss)
            model.update(grads)
        
        # 時間複雑性: O(iterations × |data| × |params|)
        # 多項式時間だが、大域最適の保証なし
```

#### 推論の複雑性

```python
# 推論は常にP
class InferenceComplexity:
    def forward_pass(self, network, input):
        """
        順伝播: 単純な行列計算の繰り返し
        
        時間: O(layers × dim²)
        空間: O(dim)
        
        → 明らかにP
        """
        x = input
        for layer in network.layers:
            x = layer.forward(x)  # O(dim²)
        return x
```

### 2.3 並列化可能性

#### Sequential モデル (RNN)

```python
# RNNはP-completeの性質を持つ
class SequentialModel:
    """
    h_t = f(h_{t-1}, x_t)
    
    問題: h_t は h_{t-1} に依存
    → 並列化不可能
    → NCには属さない
    """
    
    def compute_parallel_depth(self, sequence_length):
        return sequence_length  # O(N) の深さ
    
    def is_in_NC(self):
        return False  # P-complete的性質

# 実際の影響
def sequential_time(seq_length, num_processors):
    # プロセッサ数に関わらず
    return seq_length  # 並列化不可
```

#### Parallel モデル (Transformer)

```python
# TransformerはNCの性質を持つ
class ParallelModel:
    """
    attention(Q, K, V) = softmax(QK^T)V
    
    利点: 各位置が独立に計算可能
    → 効率的に並列化可能
    → NCに属する
    """
    
    def compute_parallel_depth(self, sequence_length):
        return math.log(sequence_length)  # O(log N)
    
    def is_in_NC(self):
        return True  # 並列化可能

# 実際の影響
def parallel_time(seq_length, num_processors):
    if num_processors >= seq_length:
        return 1  # 完全並列化
    else:
        return math.ceil(seq_length / num_processors)
```

### 2.4 Mamba: 両方の世界

```python
# Mambaは訓練時NC、推論時Sequential
class MambaComplexity:
    """
    訓練: 並列スキャン → O(log N) 深さ → NC
    推論: 逐次計算 → O(1) per step → 効率的
    """
    
    def training_complexity(self):
        return {
            "time": "O(N log N)",
            "depth": "O(log N)",
            "parallel_class": "NC"
        }
    
    def inference_complexity(self):
        return {
            "time_per_step": "O(1)",
            "memory": "O(d_state)",  # 定数
            "total_time": "O(N)"  # 逐次だが各ステップが高速
        }
```

### 2.5 演習問題

**Q2.1**: 以下の操作の複雑性クラスを判定せよ
1. N×N行列の行列積
2. N個の数のソート
3. N頂点グラフの最短路
4. N変数のSAT

**Q2.2**: なぜ Transformer は訓練が RNN より速いのか、NCの観点から説明せよ

---

## 3. State Space Models の理論

### 3.1 連続時間の観点

```python
# 状態空間モデルの連続時間定式化
class ContinuousSSM:
    """
    連続時間:
    dx/dt = Ax(t) + Bu(t)
    y(t) = Cx(t)
    
    離散化（Zero-Order Hold）:
    x_k = Ā x_{k-1} + B̄ u_k
    y_k = C x_k
    
    where:
    Ā = exp(AΔ)
    B̄ = (∫_0^Δ exp(Aτ)dτ)B
    """
    
    def __init__(self, A, B, C, delta):
        self.A = A
        self.B = B
        self.C = C
        self.delta = delta
        
        # 離散化
        self.A_discrete = self.discretize_A(A, delta)
        self.B_discrete = self.discretize_B(A, B, delta)
    
    def discretize_A(self, A, delta):
        # exp(AΔ) の計算
        return torch.matrix_exp(A * delta)
    
    def discretize_B(self, A, B, delta):
        # 積分の計算（簡略版）
        I = torch.eye(A.shape[0])
        A_inv = torch.inverse(A + 1e-8 * I)
        return A_inv @ (self.A_discrete - I) @ B
```

### 3.2 並列スキャンアルゴリズム

```python
# Mambaの高速化の核心
class ParallelScan:
    """
    問題: x_k = A_k x_{k-1} + B_k を全kで計算
    
    Naive: O(N) 時間（逐次）
    Parallel Scan: O(log N) 深さ
    """
    
    def scan_sequential(self, As, Bs):
        """逐次版 - O(N) 時間"""
        N = len(As)
        xs = [None] * N
        xs[0] = Bs[0]
        
        for k in range(1, N):
            xs[k] = As[k] @ xs[k-1] + Bs[k]
        
        return xs
    
    def scan_parallel(self, As, Bs):
        """並列版 - O(log N) 深さ"""
        N = len(As)
        
        # Up-sweep (reduce)
        depth = math.ceil(math.log2(N))
        for d in range(depth):
            stride = 2 ** (d + 1)
            for i in range(stride - 1, N, stride):
                left = i - stride // 2
                # 合成: (A_i, B_i) ∘ (A_left, B_left)
                As[i] = As[i] @ As[left]
                Bs[i] = As[i] @ Bs[left] + Bs[i]
        
        # Down-sweep (scan)
        xs = [None] * N
        xs[N-1] = Bs[N-1]
        
        for d in range(depth-1, -1, -1):
            stride = 2 ** (d + 1)
            for i in range(stride//2 - 1, N-1, stride):
                right = i + stride // 2
                if right < N:
                    xs[right] = As[right] @ xs[i] + Bs[right]
        
        return xs

# ベンチマーク
import time

def benchmark_scan():
    N = 1000
    d = 64
    
    As = [torch.randn(d, d) for _ in range(N)]
    Bs = [torch.randn(d) for _ in range(N)]
    
    ps = ParallelScan()
    
    # 逐次
    start = time.time()
    xs_seq = ps.scan_sequential(As, Bs)
    seq_time = time.time() - start
    
    # 並列（シミュレート）
    start = time.time()
    xs_par = ps.scan_parallel(As, Bs)
    par_time = time.time() - start
    
    print(f"Sequential: {seq_time:.4f}s")
    print(f"Parallel: {par_time:.4f}s (depth: {math.log2(N):.0f})")
```

### 3.3 選択的状態空間

```python
# Mambaの革新: パラメータが入力依存
class SelectiveSSM:
    """
    従来のSSM:
    A, B, C は固定パラメータ
    
    Mamba (Selective SSM):
    A(x), B(x), C(x) が入力依存で変化
    """
    
    def __init__(self, d_model, d_state):
        self.d_model = d_model
        self.d_state = d_state
        
        # 入力依存のパラメータ生成
        self.B_proj = nn.Linear(d_model, d_state)
        self.C_proj = nn.Linear(d_model, d_state)
        self.delta_proj = nn.Linear(d_model, d_model)
        
        # 状態遷移行列（学習可能だが入力非依存）
        self.A = nn.Parameter(torch.randn(d_model, d_state))
    
    def forward(self, x):
        """
        x: (batch, seq_len, d_model)
        """
        batch, seq_len, _ = x.shape
        
        # 入力依存パラメータ
        B = self.B_proj(x)  # (batch, seq_len, d_state)
        C = self.C_proj(x)  # (batch, seq_len, d_state)
        delta = F.softplus(self.delta_proj(x))  # (batch, seq_len, d_model)
        
        # 選択的スキャン
        y = self.selective_scan(x, delta, self.A, B, C)
        
        return y
    
    def selective_scan(self, x, delta, A, B, C):
        """
        delta: 時間刻みの調整
        → 重要な入力で細かく、そうでないと粗く
        """
        batch, seq_len, d_model = x.shape
        
        # 離散化（入力依存）
        A_bar = torch.exp(delta.unsqueeze(-1) * A)  # (batch, seq_len, d_model, d_state)
        B_bar = delta.unsqueeze(-1) * B  # (batch, seq_len, d_state)
        
        # 並列スキャン（簡略版）
        h = torch.zeros(batch, self.d_state)
        outputs = []
        
        for t in range(seq_len):
            h = A_bar[:, t] @ h.unsqueeze(-1) + B_bar[:, t].unsqueeze(-1) * x[:, t].unsqueeze(-1)
            y_t = (C[:, t].unsqueeze(1) @ h).squeeze(1)
            outputs.append(y_t)
        
        return torch.stack(outputs, dim=1)
```

### 3.4 理論的性質

**定理**: Selective SSM は Universal Approximator

**証明のスケッチ**:
```python
# 任意の関数 f: ℝ^n → ℝ^m を近似可能
class UniversalityProof:
    """
    1. 状態空間の次元を十分大きく取る
    2. 選択的パラメータで任意の動的システムを表現
    3. 並列スキャンで効率的に計算
    
    → RNNの表現力 + Transformerの効率
    """
    
    def approximate(self, f, d_state_large_enough):
        # 理論的には可能
        # 実際には学習が必要
        pass
```

### 3.5 演習問題

**Q3.1**: 並列スキャンアルゴリズムを実装し、逐次版と比較せよ

**Q3.2**: 選択的 delta パラメータが重要な理由を、具体例で説明せよ

**Q3.3**: Mamba が RNN と Transformer の「良いとこ取り」である理由を、計算複雑性の観点から説明せよ

---

## 4. 実装: Weighted Automata

### 4.1 重み付きオートマトンの定義

```python
# 一般的な重み付きオートマトン
class WeightedAutomaton:
    """
    従来のFA: 受理/拒否 (Boolean)
    Weighted FA: 各遷移に重み
    
    Semiring上で定義:
    - 乗算: 遷移の合成
    - 加算: パスの選択
    """
    
    def __init__(self, semiring):
        self.semiring = semiring  # (⊕, ⊗, 0, 1)
        self.states = []
        self.transitions = {}  # (state, symbol) -> [(next_state, weight)]
        self.initial = None
        self.final_weights = {}
    
    def add_transition(self, from_state, symbol, to_state, weight):
        key = (from_state, symbol)
        if key not in self.transitions:
            self.transitions[key] = []
        self.transitions[key].append((to_state, weight))
    
    def compute_weight(self, word):
        """単語の重みを計算"""
        # 動的計画法
        current_weights = {self.initial: self.semiring.one}
        
        for symbol in word:
            next_weights = {}
            for state, weight in current_weights.items():
                for next_state, trans_weight in self.transitions.get((state, symbol), []):
                    new_weight = self.semiring.multiply(weight, trans_weight)
                    if next_state in next_weights:
                        next_weights[next_state] = self.semiring.add(
                            next_weights[next_state], new_weight
                        )
                    else:
                        next_weights[next_state] = new_weight
            current_weights = next_weights
        
        # 最終重みを集計
        total = self.semiring.zero
        for state, weight in current_weights.items():
            if state in self.final_weights:
                final = self.semiring.multiply(weight, self.final_weights[state])
                total = self.semiring.add(total, final)
        
        return total
```

### 4.2 Semiring の例

```python
# 各種 Semiring
class TropicalSemiring:
    """最短路問題用"""
    zero = float('inf')
    one = 0
    
    @staticmethod
    def add(a, b):
        return min(a, b)  # 最小
    
    @staticmethod
    def multiply(a, b):
        return a + b  # 加算

class ProbabilitySemiring:
    """確率用"""
    zero = 0
    one = 1
    
    @staticmethod
    def add(a, b):
        return a + b  # 和
    
    @staticmethod
    def multiply(a, b):
        return a * b  # 積

class BooleanSemiring:
    """通常のFA"""
    zero = False
    one = True
    
    @staticmethod
    def add(a, b):
        return a or b
    
    @staticmethod
    def multiply(a, b):
        return a and b
```

### 4.3 RNN as Weighted Automaton

```python
# RNNを重み付きオートマトンとして表現
class RNNasWeightedAutomaton:
    """
    Semiring: ベクトル空間 ℝ^d
    - add: ベクトル加算
    - multiply: 行列ベクトル積
    """
    
    def __init__(self, d_hidden, d_input):
        self.d_hidden = d_hidden
        self.d_input = d_input
        
        # 重み = 行列
        self.W_hh = nn.Parameter(torch.randn(d_hidden, d_hidden))
        self.W_xh = nn.Parameter(torch.randn(d_input, d_hidden))
    
    def transition_weight(self, state, input):
        """
        遷移の重み: state → next_state
        = RNNの更新式
        """
        return torch.tanh(self.W_hh @ state + self.W_xh @ input)
    
    def compute(self, sequence):
        """重み付きオートマトンの計算"""
        state = torch.zeros(self.d_hidden)  # 初期状態
        
        for x in sequence:
            # 遷移の合成 = ベクトル空間の演算
            state = self.transition_weight(state, x)
        
        return state
```

### 4.4 演習問題

**Q4.1**: Tropical Semiring を使って最短路問題を解くWeighted Automatonを実装せよ

**Q4.2**: RNN が Weighted Automaton の特殊ケースであることを、具体的な Semiring を定義して示せ

---

## 5. まとめと次章への橋渡し

### 5.1 この章で学んだこと

1. **Chomsky階層とNN**:
   - Regular → Markov Chain
   - Context-Free → Stack-RNN
   - Context-Sensitive → Transformer
   - RE → Neural Turing Machine

2. **計算複雑性**:
   - 訓練: 一般にNP-hard
   - 推論: P
   - 並列性: NC (Transformer) vs P-complete (RNN)

3. **State Space Models**:
   - 並列スキャン → O(log N) 深さ
   - 選択的パラメータ → 表現力
   - 両方の世界の良いとこ取り

### 5.2 次章への接続

次章 **03_Category_Theory.md** では：

1. **圏論的抽象化**:
   - NN as Functor
   - Composition as Category
   - 合成の代数的性質

2. **Backpropagation の本質**:
   - Lens (Optics)
   - 双方向計算の形式化

3. **並列化の理論**:
   - Monoidal Categories
   - String Diagrams

### 5.3 発展課題

**プロジェクト1**: 形式言語学習
```python
# 目標: {a^n b^n} を学習できる最小のNN
# 1. Stack-RNN実装
# 2. 訓練データ生成
# 3. 汎化性能の検証
```

**プロジェクト2**: Mambaの並列スキャン実装
```python
# 目標: 効率的な並列スキャンの実装と検証
# 1. 基本的な並列スキャン
# 2. SSMへの適用
# 3. 速度比較（逐次 vs 並列）
```

---

## 📚 参考文献

### 必読論文

1. **"On the Computational Power of Neural Nets"** (Siegelmann & Sontag, 1995)
   - RNNのチューリング完全性

2. **"Thinking Like Transformers"** (Weiss et al., 2021)
   - https://arxiv.org/abs/2106.06981
   - Transformerの形式言語的解析

3. **"Automata Theory: An Algorithmic Approach"** (Blondin, 2024)
   - https://michaelblondin.com/automata/
   - 現代的なオートマトン理論

### 教科書

- **"Introduction to Automata Theory"** (Hopcroft, Ullman)
- **"Computational Complexity"** (Arora, Barak)
  - https://theory.cs.princeton.edu/complexity/

### 次のステップ

→ **03_Category_Theory.md** へ進む
