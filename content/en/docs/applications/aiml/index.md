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

TBC ...

