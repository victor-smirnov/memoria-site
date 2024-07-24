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

Historically, by Hybrid AI people meant something related to Khaneman's [Dual process theory](https://en.wikipedia.org/wiki/Dual_process_theory) or any combination of "intuitive reasoning" (shallow, fast and wide System 1) and "symbolic reasoning" (deep, slow and narrow System 2) that are expected to _complement_ each other. LLMs turned out to be well-hybridizable with many different technologies, not limited to symbolic reasoners and databases. So, interests in ANNs is fueling _secondary_ interest in technologies that previously have been resting in an oblivion.

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

Approximate reasoning may be useful in _some_ cases. We, humans, aren't perfect in logic and other types of CPS either, but that's OK. Especially if there are ways to _improve_ a partial or approximate solution. There are two main ways to implement approximate CPS:

1. Heuristic (ex: greedy) methods.
2. Trading speed for memory.

Heuristic methods are some statistically and _statically_ inferred rules that we can use to reduce computational complexity of a CPS method and produce a "very good" result in many (but not most!) cases.

Trading speed for memory (TSM) is a large family of computational complexity reducing methods, with dynamic programming as a famous example. TSM may also be used for saving energy, if energy costs of storing and retrieving a solution is lower than costs or recomputing it. 

TSM can also be viewed as a heuristic method with an _unbound_ number of heuristics, so we accumulate useful heuristics in memory as soon as they help reducing computational complexity (even at the expense of precision). Example: heuristic instance-based reasoning. 

The challenge is that the number of instances/heuristics/solutions stored in memory and their descriptional complexity may be pretty large. Specifically for that, Memoria provides highly-functional solutions: advanced data structures, query engines, possibility of hardware acceleration, integrated storage stack from bare metal to high-level computing, decentralisation and many other useful features. 

## Associative memory for LLM

One of the most well-known ways of augmenting an LLM with external memory is [RAG](https://arxiv.org/abs/2312.10997). Here, simply speaking, a prompt is translated into one or more queries to external data sources like web pages and databases. Retrieved result is summarized and returned to the user. Another way is to augment transformer with [_explicit_ memory](https://arxiv.org/abs/2407.01178). By doing this, authors have reported significant reduction in effective model size required for the same level of performance. 

Implicit or _parameters_ memory of LLMs is very expensive: computational costs are in the other of O(N) where N is a number of parameters. The number of parameters itself is quadratic from the number of structural units in attention and fully connected layers. This does not scale well, especially when a model generalizes poorly for objective reasons and needs to memorize more. 

From other side, database technology provides searchable spatial data structures with logarithmic (on average) lookup time complexity. Memoria has specially designed [associative memory](/docs/data-zoo/associative-memory-2) -- multiary relation complying with Paul Smolensky's [requirements for compositional neuro-symbolic representations](https://www.researchgate.net/publication/360353836_Neurocompositional_computing_From_the_Central_Paradox_of_Cognition_to_a_new_generation_of_AI_systems). Unlike traditional database technologies where relations link _points_ together, associative memory links together _sub-volumes_. And a point is a special case of unit volume. Like with neural networks splitting space with hyper-planes, associative memory splits space with volumes, and infinitely many actual data points may fit into a single volume.

Given those 'hybrid' properties of Memoria's associative memory, it's a much better candidate for using with connectionist ML than classical graphs- and relations-based data structures (like classical RDF-like knowledge graphs).

Running complex queries over classical relational and graph data is a costly process, both in terms of memory and compute. Querying 'hybrid' advanced data structures like associative may be even more costly, because we need to use _sampling_-like algorithms for that. While it's nothing special from algorithmic perspective, we do need a specialized hardware for achieving maximal efficiency. Memoria Acceleration Architecture ([MAA](/docs/overview/accel)) may use associative memory as one of its design and performance targets.

## MC-AIXI-CTW

[AIXI](https://en.wikipedia.org/wiki/AIXI) is a theoretical mathematical formalism for artificial general intelligence. It's a reinforcement learning _agent_, maximizing the expected total reward received from the environment. AIXI is a clever _mathematical trick_ that is based on so-called [Universal Prior](https://www.lesswrong.com/posts/RKfg86eKQuqLnjGxx/occam-s-razor-and-the-universal-prior) (UP). It's universal, because it already contains all possible solutions for all problems packed into a format of a _probability distribution_. AIXI is an ultimate, universal RL-based agent, but it's uncomputable. So, it's not feasible in its ultimate form. Navertheless it's a simple and elegant formalism demonstrating how very different algoritms can be get working together as a single holistic system, by reducing everything to probabilistic string prediction. Auto-regressive LLMs are also just predicting next token in a text, but a lot of 'magic' implicitly happens behind the scene. 

AIXI is infeasible to implement, but surprisingly it can be _approximated_. One of such known approximations is [MC-AIXI-CTW](https://arxiv.org/abs/0909.0801). It approximates Universal Prior with [variable-order markov models](https://en.wikipedia.org/wiki/Variable-order_Markov_model) (VMM), represented as _trees_. For unknown string probability estimations it uses [Context Tree Weighting](https://en.wikipedia.org/wiki/Context_tree_weighting) method. 

What is interesting about MC-AIXI-CTW, is that:

1. It's based on a language model, backed with VMMs. Very much like with NN-bassed LLMs, intelligence is proportional to the model's abilities to estimate probability of unknown strings correctly. This is what we call 'generalization' of a model.
2. It's an _agent_ acting in a _environment_ according to some RL _policy_. So, ulike a raw LLM, it's an almost-ready-to-use AI.
3. VMM, implemented as a tree, is **much easier interpretable and hybridizable** than a neural network.
4. VMM is a _database_, requiring latency-optimized architecture. And can be an interesting benchmark for [MAA](/docs/overview/accel/).

Unfortunately for AIXI approximations, they have been lost in the shade of DL revolution in 2010th. Now, with broad interest resurrecting to many previously forgotten AI approaches, AIXI may see its second life. What we do need here is _specialized hardware_ (and related software) to accelerate this type of probabilistic models.

## Hardware

Neural network is just a bunch of dense arrays -- pretty simple _data structures_. Matrix multiplication generates simple and predictable memory access pattern. Computations can be easily scheduled statically ahead-of-time and at the scale of an entire cluster.

In case of Hybrid AI we need full set of hardware architectures, optimized for _static_ and _dynamic_ parallelism, optimized for minimizing memory access latency and maximizing throughput. There is no way to provide a single capable architecture. Instead, we need a constructor to build an architecture, specialized for a specific problem class. [Memoria Accelerated Architecture](/docs/overview/accel/) (MAA) is addressing this issue.

## Software

NN-oriented ML frameworks are relatively simple. Neural network is a bunch of dense arrays, the only fundamental data structure we need. We also need powerful optimizing compiler converting data-flow graph of a program into sequence of computational kernels invocation, handling alos data flow in the process. Specifics of most neural networks is that computations have highly regular and predictable data flow, so there is a lot  of opportunity for static-time optimizations, even claster-wide.

Symbolic AI, explicitly or implicitly, relies on some _search_ technique is _state space_. So, _represetation_ of the state space becomes crucial. The state space may be huge and highly irrecular requiring large complex data structures and software and hardware that can leverage _dynamic parallelism_. 

Sufficiently general _reasoning engine_ is much more complex than an advanced relational DBMS and includes it. [Relational algebra](https://en.wikipedia.org/wiki/Relational_algebra) is a derivative of [relational calculus](https://en.wikipedia.org/wiki/Relational_calculus) -- that is a tractable subset of [first-order logic](https://en.wikipedia.org/wiki/First-order_logic) (FOL). RDBMS sacrifice expressiveness of FOL for predictable performance, they also lose the _deductive_ component -- ability to infer _new facts_ from _existing ones_. Reasoning engines may generate large intermediate state and/or results and operate on large datasets, so building it on top of a powerful and generic query engine is essential.

Memoria Framework provides necessary basic and advanced elements for building standalone and hybrid reasoning engines. The main elements of the stack are:

1. [Hermes](/docs/overview/hermes) data format, deeply ontegrated with the rest of the framework, including hardware acceleration at the level of MAA if/when necessary. Hermes is flexible enough to support all the necessary structured formats like knowlege graphs, no thierd-party libraries are needed.
2. Hermes-based [HRPC](/docs/overview/hrpc) accelerated communication protocol.
3. Rich set of trivial and advanced [data structures](/docs/overview/containers). No external database or storage system is ever needed.
4. Accelerated query [execution engine](/docs/overview/vm). Turing complete superset of SQL/Datalog and beyond. Backward chaining mode (query execution) and [RETE](https://en.wikipedia.org/wiki/Rete_algorithm)-based forward-chaining engine for event-driven computations: robotics, agents, embedded systems and [IoT](/docs/applications/eiot).
5. Computational [storage layer](/docs/applications/storage). Memoria fully [redefines](/docs/overview/accel) storage and processing stacks comparing to a traditional ones, based on CPUs and monolithic operating systems with integrated complex storage layers (file systems). Complex distributed and heterogeneous architectures become much simpler.

## Beyond Reasoning

LLMs are considered to be _language_ models. Language models trained on human-generated textual data are _much more_ than just that. They are capturing not only the language itself, but also _functional approximations_ of higher mental functions, including _agency_. Conversation with a sufficiently powerful and properly-tuned LLM looks and feels like a conversation with a real well-educated person with encyclopedic knowledge. Apparent human-likeness of conversations with LLMs is impressive, especially when we take into account implicit functional aspects of it (intuitive understanding), but it should not be deceiving. LLMs do not _feel_ or really _experience_ what they say _the way we do_: generative processes in LLMs are sufficiently different from ours. We should not attribute any of our _mental states_ to them (we may do it, but only with great precaution). 

Despite 'mental states' of LLMs (if any) are very different from ours, their _observable effects_ are consistent with corresponding effects of our mental states (providing that there is enough training data). This is _the_ reason why it [feels so human-like](https://www.scientificamerican.com/article/google-engineer-claims-ai-chatbot-is-sentient-why-that-matters/) in conversations. Certain higher mental functions are [known to be really hard](https://en.wikipedia.org/wiki/Hard_problem_of_consciousness) to formalize, they are nevertheless very important for human-machine integration. This entire topic has been considered largely uninteresting in AI/ML community. But recent second dawn of LLM-based [multi-agent systems](https://en.wikipedia.org/wiki/Multi-agent_system) have brought old question to the table again. There is much more to human cognition than just 'reasoning' and 'learning', that is may be just 1% of all mental activity related to the problem solving. If we want our computational agents be integrateable into human-centric interactions, or humans be integrable into societies of computational agents, both parties need to _understand_ each others at the emotional, intuitive, unconscious levels. 

Capturing mental states and higher mental functions in LLMs via machine learning has been already proven efficient, but the real problem is that internal state of black box ML models is hardly interpretable in terms of external objects those models interact with. LLMs work, until they don't. And if they don't, there is no specific way to fix it. Moreover, textual datasets are extremely [skewed](https://en.wikipedia.org/wiki/Skewness) at textual descriptions of mental states and higher mental functions, seriously limiting LLM's abilities to reason about them. Mental states describing external _objects_ are very well represented in textual data. The same is true for certain emotional state, but now their descriptions depend on some context, introducing _subjectivity_ into interpretations. 

Certain important mental states have no textual expression at all. They may be not even _reportable_. One of foundational questions that may trick and freak programmers is "How do _you_ write programs?". Only the most experienced programmers notice that the very process of program creation is not really _accessible_. Less experienced programmers may say something about methodologies and philosophy. But these things are just reflections on generalisations of the process, not how actual mechanisms of writing a program emerge in our heads. The same is true for any other types of _reasoning_ (and accessible mental activity in general): there is a lot of an inaccessible (or _intuitive_) component of the process happening in the background. Getting this access may be the key to making our thinking processes efficient and our MLhttps://github.com/victor-smirnov/digital-philosophy/blob/master/Artificial%20Intelligence.md models economically viable.

It's yet to be proven that reducing skewedness of datasets by enriching them with textual descriptions of inner mental processes (intermediate cognitive material) will improve performance of language models in general and agent-related tests. Nevertheless, it's a solid and grounded intuition because it works this way for humans. In order to get this data from our minds, we need to develop [intrapersonal intelligence](https://en.wikipedia.org/wiki/Theory_of_multiple_intelligences#Intrapersonal) -- the ability to understand yourself, including your thoughts, feelings, motivations, and fears, and to use that understanding to make decisions and communicate -- in a way that is compatible with AI. Figuratively speaking, we need to understand our 'inner machine' and describe it in terms of algorithms and data structures expressed as scripts for agents.

There is an ongoing [multidisciplinary research process](https://github.com/victor-smirnov/digital-philosophy/blob/master/Artificial%20Intelligence.md) around Memoria targeting unification of intrapersonal intelligence from Psychology, Philosophy of Mind, Physiology Mathematics and Computer Science. The goal is to build a conceptual bridge between first-person experience and computations in the form of minimalist computational models of fundamental higher mental functions like _Observer_ and _beingness_, _feelings_, _intuition_ and many others. Positive results may be used ...



TBC ...

## Memoria as a Dataset

TBC ...

