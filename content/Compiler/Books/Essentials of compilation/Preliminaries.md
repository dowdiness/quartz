---
created: 2025-04-12T01:09:37+09:00
modified: 2025-12-22T18:48:34+09:00
publish: true
tags: [compiler]
---

# Preliminaries

## AST

ASTを構成する一つ一つの箱をNodeと呼びます。ArrowによってNodeが結ばれた先のNodeがChildrenです。一番上のNodeはRootと呼びます。Root以外のNodeはParentを持っています。ChildrenのないNodeはLeafであり、そうでなければInternal nodeになる。

## Grammar

Context free grammarのルールにはそれぞれ左側と右側がある。右側にマッチしたASTは左側のルールにより分類できる。

## Recursive Functions

match節により文法に対応したそれぞれのchild nodeで再起呼び出しをする関数を `structual recursion` と呼ぶ。

```racket
(define (is_exp ast)
  (match ast
    [(Int n) #t]
    [(Prim 'read '()) #t]
    [(Prim '- (list e)) (is_exp e)]
    [(Prim '+ (list e1 e2))
      (and (is_exp e1) (is_exp e2))]
    [(Prim '- (list e1 e2))
      (and (is_exp e1) (is_exp e2))]
    [else #f]))

(define (is_Lint ast)
  (match ast
    [(Program '() e) (is_exp e)]
    [else #f]))
```

非終端記号を再起呼び出しで処理することにより再起関数を書ける。

詳しくは [how to design Programs](https://htdp.org/) を参照。

## Interpreter

*definitional interpreter*

[Definitional interpreters for higher-order programming languages](https://dl.acm.org/doi/10.1145/800194.805852)

*trapped-error*

*unspecified behavior*

**コンパイラの仕事とはプログラム言語のプログラムを、同じようにふるまう異なるプログラム言語のプログラムへと変換すること。**

## A Partial Evaluator

