---
created: 2025-04-09T14:52:34+09:00
modified: 2025-12-26T15:28:22+09:00
---

# Model

[[Model Theory]]

https://www.williamjbowman.com/blog/2023/06/15/what-is-a-model/


**Definition 0** In science and engineering, a model is “an abstract description of a concrete system using mathematical concepts and language”. See [Wikipedia](https://en.wikipedia.org/wiki/Mathematical_model "Mathematical model") provides a nice introduction to this kind of model, and the [Standard Encylopedia of Philosophy](https://plato.stanford.edu/entries/model-theory/#Modelling "Models and Modelling") provides a nice explanation in the context of model theory, which will be relevant later in this post.

**Definition 1** A _syntactic model_ (of a type theory) is defined by [Boulier, Pédrot, and Tabareau](https://doi.org/10.1145/3018610.3018620 "The Next 700 Syntactical Models of Type Theory") as a translation from one type theory into another that preserves typing, the definition of false, and definitional equivalence. This syntactic model enables the source type theory to inherit properties of the target type theory—such as consistency.

**Definition 2** A _model_ (of a _vocabulary_ also called a _language_ ) in the sense of model theory (as defined by [Elements of Finite Model Theory](https://doi.org/10.1007/978-3-662-07003-1 "Elements of Finite Model Theory")) is a _-structure_ (“also called a _model_”) defining a set _A_ along with 3 sets providing interpretations of that vocabulary. These sets are , which interprets each constant in  as an element of , , which interprets each n-ary predicate symbol or relation symbol from  as an n-ary (set-theoretic) relation between elements of , and , which interprets each n-ary function symbol in  as a (set-theoretic) function from n elements of  to an element of .

**Definition 3** The above definition is confusing, since it conflates _structure_ and _model_, which the text later distinguishes with the following separate definition. A _model_ (of a _theory_ (over a vocabulary )) is a _structure_ (“also called a _model_”) of _vocabulary_  such that every sentence in the theory is interpreted in the structure to make the sentence true. (A _theory_ is a set of sentences drawn from a vocabulary.) My rephrasing of the definition of model is intentionally confusing and difficult to parse, to make apparent the inherit confusingness created by the several layers of definitions and one definition that defines “model” using a second definition of “model”.

**Definition 4** [Nlab hosts an article](https://ncatlab.org/nlab/show/structure+in+model+theory#Definition "Definition of 'structure' in model theory") with a much clarified definition, which distinguishes _language_, _theory_, _structure_, and _model_ carefully. In particular, it is careful to only call _structure_ the interpretation of the _language_ (call _vocabulary_ above), and only call _model_ an interpretation that makes true the _axioms_ composing the _theory_ of the _language_.

**Definition 5** [Carlo Anguli once gave me the following definition of model](https://twitter.com/carloangiuli/status/1640421574733078528?s=20):

>