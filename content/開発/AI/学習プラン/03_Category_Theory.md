# Part 3: 圏論的視点

## 📚 この章について

機械学習を**圏論 (Category Theory)** の観点から理解します。合成、関手、自然変換など、圏論の概念がニューラルネットワークの本質的な構造を明らかにします。

**学習時間**: 4-6週間  
**前提知識**: Part 1-2, 関数型プログラミング（推奨）

## 🎯 学習目標

- [ ] 圏、関手、自然変換の基本概念を理解できる
- [ ] ニューラルネットワークを圏論的に定式化できる
- [ ] Backpropagation を Lens として理解できる
- [ ] String Diagrams でモデルを設計できる
- [ ] モノイダル圏で並列化を形式化できる

---

## 1. 圏論の基礎

### 1.1 圏の定義

```haskell
-- 圏の定義
class Category cat where
    -- 対象 (Objects) は型レベルで表現
    
    -- 恒等射
    id :: cat a a
    
    -- 合成
    (.) :: cat b c -> cat a b -> cat a c
    
    -- 法則:
    -- 1. 結合律: (h . g) . f = h . (g . f)
    -- 2. 単位律: id . f = f = f . id

-- 例: Haskの圏（Haskell型の圏）
instance Category (->) where
    id x = x
    (g . f) x = g (f x)
```

### 1.2 Neural Network as Category

```haskell
-- Layer を射として定式化
data Layer a b = Layer (a -> b)

instance Category Layer where
    id = Layer (\x -> x)
    
    (Layer g) . (Layer f) = Layer (g . f)

-- 具体例
linearLayer :: Int -> Int -> Layer (Vector Double) (Vector Double)
linearLayer input output = Layer $ \x ->
    W <> x + b  -- 行列積 + バイアス
  where
    W = randomMatrix output input
    b = randomVector output

-- ネットワーク = 層の合成
network :: Layer Input Output
network = layer3 . layer2 . layer1
  where
    layer1 :: Layer Input Hidden1
    layer2 :: Layer Hidden1 Hidden2
    layer3 :: Layer Hidden2 Output
```

### 1.3 圏の法則の意味

```haskell
-- 結合律: 合成の順序は結果に影響しない
-- (layer3 . layer2) . layer1 = layer3 . (layer2 . layer1)

-- これは実装上重要:
-- どのようにグループ化しても同じ結果
pipeline1 = layer3 . (layer2 . layer1)
pipeline2 = (layer3 . layer2) . layer1
-- pipeline1 と pipeline2 は同一

-- 単位律: 恒等層は影響しない
-- id . layer = layer = layer . id

identityLayer :: Layer a a
identityLayer = Layer id

-- これは ResNet の skip connection に関連
```

### 1.4 演習問題

**Q1.1**: 以下が圏であることを確認せよ（結合律と単位律）
1. 行列の圏（対象=次元、射=行列）
2. グラフの圏（対象=グラフ、射=グラフ準同型）

**Q1.2**: ReLU層が圏の射として適切でない理由を考えよ（ヒント: 合成の型）

---

## 2. 関手 (Functors)

### 2.1 関手の定義

```haskell
-- 圏間の写像
class (Category c, Category d) => Functor f c d where
    -- 対象の写像: a → f a (型レベル)
    
    -- 射の写像
    fmap :: c a b -> d (f a) (f b)
    
    -- 法則:
    -- 1. fmap id = id
    -- 2. fmap (g . f) = fmap g . fmap f

-- 例: List関手
instance Functor [] Hask Hask where
    fmap f [] = []
    fmap f (x:xs) = f x : fmap f xs
```

### 2.2 Neural Network as Functor

```haskell
-- データ変換を関手として
data DataTransform = DataTransform

instance Functor DataTransform DataCat LearnedCat where
    -- 生データ → 学習済み表現
    fmap :: Layer a b -> Layer (Data a) (Learned b)
    fmap (Layer f) = Layer $ \dataA ->
        let rawB = f (extract dataA)
        in embed rawB

-- 具体例: Embedding層
embeddingFunctor :: Functor EmbedF DiscreteCategory VectorCategory
embeddingFunctor = Functor $ \tokenId ->
    embeddingMatrix ! tokenId  -- テーブル参照

-- Word2Vec, BERT等は関手的な構造を持つ
```

### 2.3 エンコーダー・デコーダーと関手

```python
# Pythonで概念を説明
class EncoderFunctor:
    """
    入力圏 → 潜在圏 への関手
    """
    def fmap(self, input_morphism):
        # 入力空間の射 → 潜在空間の射
        def encoded_morphism(latent):
            input_data = self.decoder(latent)
            transformed = input_morphism(input_data)
            return self.encoder(transformed)
        
        return encoded_morphism

# 例: Autoencoder
class Autoencoder(EncoderFunctor):
    def __init__(self):
        self.encoder = NeuralNet([784, 256, 64])
        self.decoder = NeuralNet([64, 256, 784])
    
    # 関手の法則を満たすように設計
    def fmap(self, f):
        return lambda z: self.encoder(f(self.decoder(z)))
```

### 2.4 演習問題

**Q2.1**: CNNの畳み込み演算が関手であることを示せ

**Q2.2**: Attention機構が関手でない理由を考えよ

---

## 3. 自然変換 (Natural Transformations)

### 3.1 定義

```haskell
-- 関手間の射
class (Functor f c d, Functor g c d) => NaturalTransformation η f g c d where
    component :: c a b -> d (f a) (g b)
    
    -- 自然性条件 (Naturality Square):
    --   f a --f h--> f b
    --    |            |
    --   η_a          η_b
    --    ↓            ↓
    --   g a --g h--> g b
    --
    -- η_b . f h = g h . η_a

-- 例: リストの長さ（List → Const Int）
length :: NaturalTransformation Length [] (Const Int)
length = NaturalTransformation $ \list ->
    Const (listLength list)
```

### 3.2 Attention as Natural Transformation

```python
# Attention は自然変換
class AttentionTransformation:
    """
    Encoder関手 → Decoder関手 への自然変換
    """
    def __init__(self, d_model, n_heads):
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
    
    def component(self, encoder_output, decoder_state):
        """
        自然変換の成分
        
        Encoder(X) → Decoder(X)
        """
        # Query from decoder
        Q = self.W_q(decoder_state)
        
        # Key, Value from encoder  
        K = self.W_k(encoder_output)
        V = self.W_v(encoder_output)
        
        # Attention
        scores = Q @ K.T / sqrt(d_model)
        weights = softmax(scores)
        
        return weights @ V
    
    def verify_naturality(self, f, encoder_out, decoder_state):
        """
        自然性条件の検証（概念的）
        
        component(f(encoder), decoder) 
        = f'(component(encoder, decoder))
        """
        # 左辺
        lhs = self.component(f(encoder_out), decoder_state)
        
        # 右辺
        rhs = f_transformed(self.component(encoder_out, decoder_state))
        
        # 自然性: lhs ≈ rhs
        return torch.allclose(lhs, rhs)
```

### 3.3 Self-Attention の自然変換的解釈

```haskell
-- Self-Attention = 恒等関手上の自然変換
data SelfAttention = SelfAttention
  { wq :: Matrix
  , wk :: Matrix
  , wv :: Matrix
  }

-- Id → Id への自然変換
selfAttentionComponent :: SelfAttention -> Sequence -> Sequence
selfAttentionComponent attn seq =
    let q = wq attn <> seq
        k = wk attn <> seq
        v = wv attn <> seq
        scores = softmax (q <> transpose k / sqrt d)
    in scores <> v

-- 自然性: 系列の変換に対して整合的
-- f: Seq A → Seq B に対して
-- attn(f(seqA)) と f(attn(seqA)) が関連
```

### 3.4 演習問題

**Q3.1**: Cross-Attention が自然変換である理由を、図式で説明せよ

**Q3.2**: Multi-Head Attention を複数の自然変換の組として定式化せよ

---

## 4. モノイダル圏 (Monoidal Categories)

### 4.1 定義

```haskell
-- テンソル積を持つ圏
class Category c => Monoidal c where
    -- テンソル積
    (⊗) :: c a b -> c x y -> c (a, x) (b, y)
    
    -- 単位対象
    unit :: c () ()
    
    -- 法則:
    -- 1. 結合律: (f ⊗ g) ⊗ h ≅ f ⊗ (g ⊗ h)
    -- 2. 単位律: unit ⊗ f ≅ f ≅ f ⊗ unit

-- 例: 関数のモノイダル圏
instance Monoidal (->) where
    (f ⊗ g) (x, y) = (f x, g y)
    unit () = ()
```

### 4.2 並列計算とモノイダル圏

```haskell
-- 並列層の合成
parallelLayers :: Layer a b -> Layer c d -> Layer (a, c) (b, d)
parallelLayers layerF layerG = Layer $ \(x, y) ->
    (runLayer layerF x, runLayer layerG y)
  where
    runLayer (Layer f) = f

-- Multi-Head Attention の定式化
multiHeadAttention :: Int -> [Layer a a] -> Layer a a
multiHeadAttention n heads = 
    let parallelHeads = foldl (⊗) unit heads
    in concat . parallelHeads . split
  where
    split :: a -> [a]
    concat :: [a] -> a
```

### 4.3 String Diagrams (紐図式)

```
# Sequential (合成)
Input → [Layer1] → [Layer2] → Output

# Parallel (テンソル積)
Input1 → [Layer1] → Output1
Input2 → [Layer2] → Output2

# Transformer Block (複雑な構造)
    Input
     │
  ┌──┴──┐
  │     │
 QKV  Identity  (Residual)
  │     │
Attn   │
  │     │
  └──┬──┘
    Add
     │
   Norm
     │
  ┌──┴──┐
  │     │
 FFN  Identity
  │     │
  └──┬──┘
    Add
     │
   Output
```

### 4.4 Python での実装例

```python
# モノイダル構造を持つNN
class MonoidalLayer:
    """
    モノイダル圏の射として層を実装
    """
    def __init__(self, f):
        self.f = f
    
    def __call__(self, x):
        return self.f(x)
    
    def compose(self, other):
        """合成 (categorical composition)"""
        return MonoidalLayer(lambda x: self(other(x)))
    
    def tensor(self, other):
        """テンソル積 (parallel composition)"""
        def parallel(inputs):
            x, y = inputs
            return (self(x), other(y))
        return MonoidalLayer(parallel)

# 使用例
layer1 = MonoidalLayer(lambda x: x * 2)
layer2 = MonoidalLayer(lambda x: x + 1)

# Sequential
sequential = layer2.compose(layer1)
print(sequential(5))  # (5 * 2) + 1 = 11

# Parallel
parallel = layer1.tensor(layer2)
print(parallel((5, 3)))  # (10, 4)
```

### 4.5 演習問題

**Q4.1**: ResNet の Skip Connection をモノイダル圏で定式化せよ

**Q4.2**: Transformer の String Diagram を描け（Self-Attention + FFN）

---

## 5. Backpropagation as Lens

### 5.1 Lens (光学系)

```haskell
-- Lens: 双方向の計算構造
data Lens s t a b = Lens
    { view :: s -> a              -- Getter (forward)
    , update :: s -> b -> t       -- Setter (backward)
    }

-- 例: タプルのLens
fstLens :: Lens (a, c) (b, c) a b
fstLens = Lens
    { view = fst
    , update = \(_, c) b -> (b, c)
    }
```

### 5.2 Neural Layer as Lens

```haskell
-- NN層をLensとして
neuralLens :: Weights -> Lens Input Output Gradient Gradient
neuralLens weights = Lens
    { view = \input -> forward weights input
    , update = \input grad_output ->
        -- Backpropagation!
        backward weights input grad_output
    }

-- Forward pass
forward :: Weights -> Input -> Output
forward w x = w <> x + b

-- Backward pass
backward :: Weights -> Input -> Gradient -> Gradient
backward w x grad_out = 
    transpose w <> grad_out  -- 勾配を入力側に伝播
```

### 5.3 Lens の合成 = Backpropagation の連鎖

```haskell
-- Lensの合成
composeLens :: Lens a b c d -> Lens b e d f -> Lens a e c f
composeLens l1 l2 = Lens
    { view = view l2 . view l1
    , update = \a f ->
        let b = view l1 a
            d = update l2 b f
        in update l1 a d
    }

-- 深層NNのBackprop
deepNetwork :: Lens Input Output Gradient Gradient
deepNetwork = composeLens layer1 
            $ composeLens layer2 
            $ layer3

-- Forward
output = view deepNetwork input

-- Backward (自動的に合成される！)
grad_input = update deepNetwork input grad_output
```

### 5.4 Python 実装

```python
# Lens を Python で
class NeuralLens:
    """
    NN層を双方向計算（Lens）として実装
    """
    def __init__(self, weights, bias):
        self.weights = weights
        self.bias = bias
        self.cache = {}
    
    def view(self, input):
        """Forward pass (Getter)"""
        output = input @ self.weights + self.bias
        self.cache['input'] = input  # 後で使用
        self.cache['output'] = output
        return output
    
    def update(self, grad_output):
        """Backward pass (Setter)"""
        input = self.cache['input']
        
        # 勾配計算
        grad_input = grad_output @ self.weights.T
        grad_weights = input.T @ grad_output
        grad_bias = grad_output.sum(axis=0)
        
        # 重み更新（簡略版）
        self.weights -= 0.01 * grad_weights
        self.bias -= 0.01 * grad_bias
        
        return grad_input

# Lensの合成
class ComposedLens:
    def __init__(self, lens1, lens2):
        self.lens1 = lens1
        self.lens2 = lens2
    
    def view(self, input):
        intermediate = self.lens1.view(input)
        output = self.lens2.view(intermediate)
        return output
    
    def update(self, grad_output):
        grad_intermediate = self.lens2.update(grad_output)
        grad_input = self.lens1.update(grad_intermediate)
        return grad_input

# 使用例
layer1 = NeuralLens(np.random.randn(10, 20), np.zeros(20))
layer2 = NeuralLens(np.random.randn(20, 5), np.zeros(5))

network = ComposedLens(layer1, layer2)

# Forward
x = np.random.randn(32, 10)
y = network.view(x)

# Backward
grad_out = np.random.randn(32, 5)
grad_in = network.update(grad_out)

print(f"Input shape: {x.shape}")
print(f"Output shape: {y.shape}")
print(f"Gradient shape: {grad_in.shape}")
```

### 5.5 演習問題

**Q5.1**: ReLU層を Lens として実装せよ（backward で勾配を正しく処理）

**Q5.2**: ResNet の Skip Connection を Lens で実装せよ

---

## 6. 実践: Catlab.jl で圏論的プログラミング

### 6.1 セットアップ

```julia
using Catlab
using Catlab.Theories
using Catlab.Graphics

# ニューラルネットワークの圏を定義
@present NeuralCat(FreeCategory) begin
    # 対象（型）
    Input::Ob
    Hidden1::Ob
    Hidden2::Ob
    Output::Ob
    
    # 射（層）
    layer1::Hom(Input, Hidden1)
    layer2::Hom(Hidden1, Hidden2)
    layer3::Hom(Hidden2, Output)
end
```

### 6.2 合成と可視化

```julia
# 層の合成
network = compose(layer1, compose(layer2, layer3))

# String Diagramとして可視化
draw(network)

# モノイダル構造
parallel_network = otimes(layer1, layer2)
draw(parallel_network)
```

### 6.3 演習問題

**Q6.1**: U-Net アーキテクチャを Catlab.jl で定義し、可視化せよ

**Q6.2**: Attention Block を圏論的に定義せよ

---

## 7. まとめと次章への橋渡し

### 7.1 この章で学んだこと

1. **圏論の基礎**:
   - 圏、関手、自然変換
   - NN を圏論的に定式化

2. **並列化の形式化**:
   - モノイダル圏
   - String Diagrams

3. **Backpropagation の本質**:
   - Lens (双方向計算)
   - 合成の自然な構造

### 7.2 次章への接続

次章 **04_Logic_And_Types.md** では：

1. **Curry-Howard対応**:
   - 論理 ↔ 型 ↔ 圏

2. **型理論での NN**:
   - 依存型
   - 線形型

3. **Hoare論理**:
   - プログラムの正しさ
   - 最適化の仕様

---

## 📚 参考文献

### 必読論文

1. **"Backprop as Functor"** (Fong et al., 2019)
   - https://arxiv.org/abs/1711.10455

2. **"Categorical Semantics of Neural Networks"** (Shiebler et al., 2021)
   - https://arxiv.org/abs/2101.05084

### 教科書

- **"Category Theory for Programmers"** (Milewski)
  - https://github.com/hmemcpy/milewski-ctfp-pdf

- **"Seven Sketches in Compositionality"** (Fong & Spivak)
  - https://arxiv.org/abs/1803.05316

### 次のステップ

→ **04_Logic_And_Types.md** へ進む
