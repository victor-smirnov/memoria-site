---
title: "Hybrid AI"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2010
toc: true
---

## What is "Hybrid AI" in Memoria?

Historically, by Hybrid AI people meant something related to Khaneman's [Dual process theory](https://en.wikipedia.org/wiki/Dual_process_theory) or any combination of "intuitive reasoning" (shallow, fast and wide System 1) and "symbolic reasoning" (deep, slow and narrow System 2) that are expected to _complement_ each other. LLMs turned out to be well-hybridizable with many different technologies, not limited to symbolic reasoners and databases. So, interests in ANNs is fuelling _secondary_ interest in technologies that previously have been resting in an oblivion.

In Memoria, the meaning of this term is slightly different (but not contradicting). Memoria project follows the intuition that there is no any specific "secret" in human intelligence in particular, and in intelligence in general: at the end of the day (after all possible _optimizations_ have been applied) it's all about resources -- _compute_ and _memory_. This position should not be confused with Sutton's [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html). While both are similar in wording and final resolutions, Memoria implies that _resources are always limited_, and that makes huge difference: _problem-specific optimizations do matter_. Ultimately:

* If there is some algorithm or mathematical method, or data structure that can reduce computational complexity of AI, it's worth using.
* If there is a custom hardware architecture that can improve raw performance _and_/_or_ performance per watt, it's worth using.
* If there is some _physical process_ that we can utilize to improve performance characteristics, it's worth considering.
* Quantum supremacy? Perfect!
* If we can [improve our introspection](https://medium.com/@victorsmirnov/how-to-compensate-introspection-illusion-62f357e9326c) and get some bits about inner machinery of mind that may help us to achieve better _human-likeness_ of AI, let's do it!
* Any useful bits form other disciplines are always welcome!

Memoria is grounded in [Algorithmic Information Theory](https://en.wikipedia.org/wiki/Algorithmic_information_theory) and it's compression-based approach to AI. From this perspective, Systems 1 and 2 are just different _compressibility domains_. System 2 corresponds with highly-compressible domain, and System 1 corresponds with low-compressible domain. Traditional for programming distinction to _algorithms_ and _data structures_ has the same nature.

There may be many more compressibility domains than just two, so, potentially, we may have System 1...N in our AI architecture, where N is pretty large. Even within the same complexity domain there are many _sub-domains_, so methods like [Mixture of Experts](https://en.wikipedia.org/wiki/Mixture_of_experts) and [Ensemble learning](https://en.wikipedia.org/wiki/Ensemble_learning) are efficient. These methods work across even distant domains too, it's just a technical question how to make it working efficiently. Those technical questions are in the focus of Memoria.

In Memoria, by "Hybrid AI" it's meant an architecture spawning multiple different compression domains. 

## Probabilistic LM-based Hybridization

A probabilistic language model is simply a probability distribution over a set of strings $S$ representing texts in this language: $P(S)$, where $S = w_0w_1w_2...w_i$ -- is a sequence of text elements (usually, _tokens_). Probabilistic models are used by _sampling_ (generating elements) from them. For sampling from language models we may use _autoregressive_ schema by sampling form _conditional distribution_ $P(w_i|w_{i-1}...w_0)$ -- probability of a next element in the string given its prefix. 

Autoregressive sampling means that we generate a string in an element by element, left-to-right way, each time appending newly sampled elements to the prefix. Additional techniques, like [beam search](https://en.wikipedia.org/wiki/Beam_search), may be used to increase the probability value of the generated string. Autoregressive sampling gives us one additional important feature: we can sample strings that are _continuations_ of a given prefix, that is called a _prompt_. 

The language model has to be created somehow, and the simplest way is to learn the model inductively from the set of strings drawn from the real language. There are a lot of scientific and technical challenges here, but, basically, there are three different notable approaches: statistical [n-gram-based](https://en.wikipedia.org/wiki/Word_n-gram_language_model), [NN-based](https://en.wikipedia.org/wiki/Large_language_model) and [VMM-based](https://arxiv.org/abs/0909.0801). In all cases we feed the mode a corpus of strings and expect it to predict those (seen) strings correctly. What we do want from the model is to predict correctly the _unseen_ strings. In ML they call it _generalization_. When we are solving AI problems with ML, this is where al the 'magic' happens.

It have turned out that some very large neural language models (LLM) can generalize so well over natural language that they can solve some logical and mathematical problems, follow instructions, reason about some emotions and mental states (sic!), write program code, translate from one language to another, change style of a text, summarize/elaborate and maintain conversation -- all from the natural language (e.g.: English).

Of course LLMs aren't doing _everything_ good enough (at the expert human-level), they are making a lot of hard to recognize and fix mistakes that is seriously limiting their practical suitability. They are pretty good at tasks in _low-compressible domain_: translation, style transfer, summarization and elaboration, and some others. The reason is probably that in low-compressible domains the role of generalization isn't that high and the model size/scale [is all that ultimately matters](https://arxiv.org/abs/2001.08361).

In highly compressible domains like basic math, logic puzzles, constraint solving, board games, database query execution and logical inference -- generalization matters, but generalizability depends on many factors. The most important of them are training _data quality_, _model architecture_ and _learning algorithms_. Both model architecture and learning algorithms are fixed for neural LM. There is no way single architecture may be good for everything, one may even say that NFL theorem prohibits this. There some indirect evidence that effects behind In-Context Learning in Transformers _may_ help models to adapt to specific narrow tasks like basic arithmetic beyond what would be expected from the architecture alone. But those effects are severely limited. Basically, no amount of scaling can make a database engine out of a neural network. 

Actually, the latter isn't an issue if we want to achieve HXL-AI (Human-Expert Level AI), because humans aren't that good at symbolic tasks either. The point is that _mere scaling_ of LLMs is a wrong direction. Instead, we need to identify certain domains where scaling doesn't work but there can be different solutions, and provide custom solvers -- arithmetic, logical reasoning, constraint solving... and so on. A relatively small but fine-tuned LLM may be used here to pre-process the input, find narrow formal problems in it, invoke corresponding solvers and then post-process the result. Text in a mixture of natural language and structured formats can be seen as _Intermediate Representation_ for this type of hybrid AI.

Using LLMs as a human-like interface to various problem solvers greatly increases their exposure to the potential audience. Problem solvers, despite having great potential _value_ are pretty hard to use directly. 

## Hybrid and Approximate Reasoning

By "reasoning" we mean what is usually meant by "declarative problem solving": logic (of various types), constraint solving, SAT/SMT, theorem proving, planning and many other types of combinatorial problem solving (CPS). CPS was initially meant as a main purpose of AI because of the _value_ it creates. It literally solves problems and makes our life _better_. But there are two main obstacles:

1. CPS is rather hard to set up and use: "you need a PhD for that". 
2. CPS is _very slow_. Many important problems are out of our reach. 

Complexity of CPS methods may be addressed with specialized LLMs, translating from problem description obtained in a dialogue with a user to structured problem representation. Right now (2024) it doesn't work well in many ways, a lot of improvements of are still ahead. But at least this specific direction looks feasible and economically viable. 

_Computational complexity_ of CPS is mach harder to solve issue because, basically, there is no workaround. Logic is computationally complex. If we augment an LLM with a reasoner that will be logical parts of the query, it may take practically infinite time for even apparently simple problems. Inference time is rather hard to predict.

Approximate reasoning may be useful in _some_ cases. He, humans, aren't perfect in logic and other types of CPS either, but that's OK. Especially if there are ways to _improve_ a partial or approximate solution. There are two main ways to implement approximate CPS:

1. Heuristic (ex: greedy) methods.
2. Trading speed for memory.

Heuristic methods are some statistically and _statically_ inferred rules that we can use to reduce computational complexity of a CPS method and produce a "very good" result in many (but not most!) cases.

Trading speed for memory is a large family of computational complexity reducing methods, with dynamic programming as a famous example.

TBC ...

