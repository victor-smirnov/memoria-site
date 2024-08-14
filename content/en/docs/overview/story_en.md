---
title: "Memoria Story (EN)"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 190
toc: true
---


## What’s This Project About?

Memoria (yeah, you can just call it that, no need to add "Framework" unless you’re feeling formal) has been in the works for over 16 years. Throughout this time, I’ve tried to align it with the latest tech trends -- file systems, NoSQL, data platforms, analytics, and now AI. These trends change faster than a cat's mood, but each attempt to keep up left its mark on the code and documentation. Riding the trend wave is essential since community interest is often speculative -- companies view open-source as a way to cut development costs through a win-win socialization deal. A significant chunk of contributors in any project are commercial companies.

On the flip side, you’ve got to ride the trend wave into an uncharted niche to avoid competing for user attention. For example, plenty of analytical data platforms have popped up over time -- good enough for their purposes, though not perfect. Competing with them? Pointless. The same goes for databases, both transactional and analytical. They collectively meet user needs pretty well. Memoria was designed to do way more than what regular databases need to do, making it kind of overkill in that niche. Plus, its unique features can feel like carrying around a suitcase without a handle -- hard to justify.

Memoria was originally created for AI tasks, [like these](https://memoria-framework.dev/docs/data-zoo/associative-memory-2). But in the 2010s, AI took the neural network route, which is basically matrix multiplication. At the same time, the computational power of relevant accelerators was growing exponentially. Now, technology has hit a saturation point, hardware and software progress has slowed down, but appetites have only grown. The rise of LLMs (Large Language Models) sparked a paradoxical resurgence of interest in "old" methods because of the cool opportunities for [hybridization](https://memoria-framework.dev/docs/applications/aiml). This, in turn, brought along vector databases (for RAG). Companies promoting "hybrid AI" are emerging too. Memoria’s technologies are set to be in demand in the foreseeable future.

Currently, the project is positioned as a Hardware/Software [co-design](https://memoria-framework.dev/docs/applications/co-design) framework, reflecting two main needs.

**First**: Software alone isn’t enough for Memoria’s grand goals. CPU-centric hardware architectures and the software built around them are outdated. Data is fundamentally far from the processors, creating latencies and heat. Lots of heat. And long latencies. This hampers scalability and makes it tough to efficiently implement latency-sensitive algorithms (which is pretty much all of AI that used to be called "symbolic"). So, if someone wants to build a hybrid system, the efficient hardware only exists for the part that’s not latency-sensitive and doesn’t require dynamic parallelism (the GPU headache).

In short, to meet the industry’s needs for hybrid AI effectively, we need a new hardware stack. And since hardware without software is just an expensive brick, we also need software -- a software architecture that allows this hardware to be used efficiently.

**Second**: The corresponding market niche is practically empty and waiting to be filled.

One of Memoria’s central value propositions is that it doesn’t just wrap specialized hardware in a software runtime; it also brings a lot of extra solutions in the form of containers, protocols, storage, code execution systems, and even data description languages integrated with programming languages. Memoria’s approach to co-design is "top-down," meaning the hardware is built to meet software needs, not the other way around (like, "Here’s our awesome accelerator, now figure out how to use it"). So, hardware development is just a step in developing algorithms and data structures. Ideally, in the future, this will become completely transparent through deep automation of processes. Right now, there are real reasons to expect this outcome.

Overall, Memoria should be seen as a _fundamental_ project focused on long-term data use across the entire stack, from physical storage to computational processing. The project even touches on such exotic stuff as the possibility of reading data recorded today, like, a billion years from now. This might sound funny, but the problem of data longevity is real. It’s not just a human problem. Everything decays. And much faster than it seems.

## For Hardware Developers

For hardware developers, the value proposition of Memoria as a co-design framework lies in its ability to define the contract between software and hardware. This contract is outlined through a reference meta-architecture [MAA](/docs/overview/accel) and its components: xPU, problem-specific RISC-V instruction set extensions, memory architecture, the [Hermes](/docs/overview/hermes) format, and the [HRPC](/docs/overview/hrpc) protocol. Ideally, if a soft-core or chiplet meets these specifications, the framework should be able to use them with minimal adjustments on its side (100% transparency isn’t the goal here, as it’s a bit too hard to achieve).

This doesn’t mean that Memoria's project repository will contain HDL/HCL for the corresponding hardware modules. Reference implementations of RISC-V cores and HRPC infrastructure elements are planned, but their purpose is more for prototyping rather than actual use. Not that using them is forbidden -- it's just not the main objective.

The idea is that hardware projects using Memoria as a framework are external projects. The same goes for software projects -- the framework aims to ensure good reusability of its components. For example, Core, which includes the base library, Hermes, some HRPC elements, DSLEngine, and related tools (parser generators, IDL generators, etc.), can be used independently of the other framework components and doesn’t have any tricky dependencies.

To simplify build and dependency management, there's a separate overlay for [Vcpkg](https://github.com/victor-smirnov/memoria-vcpkg-registry).

Overall, improving modularity and reusability of the framework’s elements is one of the top priorities (but without going overboard).

## This is a BIG Project

Memoria has turned out to be a very big project. Much bigger than what one person can handle. And certainly bigger than one person can support the code for (even with the new opportunities for automating software development). Even on a paid basis. This doesn’t mean it won’t continue to develop. But it does mean that without community help, it will progress slowly and, most likely, won’t keep up with the trends. This has its pros (no situational pressure from pragmatism) and cons (disconnect from reality).

## Priorities

### N1
To wrap up this introduction, I’d like to outline the current priorities of the project, excluding the aspects related to community building, business engagement, and deploying the necessary infrastructure. These will be happening in the background with their own set of priorities.

As mentioned earlier, Memoria is a foundational project, so there will be a strong focus on making decisions "for the distant future." This means refining and optimizing the core algorithms, interfaces, data structures, and formats, which, of course, might slow down practical adoption. To mitigate these effects, there will be proto/dev/stable branches.

Among the major subsystems, **Core** -- which includes Hermes and the tools related to it -- has reached a high level of conceptual stability. This means it's ready for freezing and production use. Core is independent of other subsystems, doesn’t involve complex IO, and isn’t tied to specific runtimes. Conceptually, it’s very similar to the corresponding subsystems of other related projects. And this subsystem doesn’t require any specialized knowledge of the rest of Memoria (advanced algorithms and data structures). This subsystem could have an independent maintainer and its own separate team.

Right now, the primary focus will be on Core.

### N2
The next priority is the integration of Core, particularly the [Hermes](/docs/overview/hermes) data types and [Containers](/docs/overview/containers). Hermes is a format oriented toward messages and documents, and by design, its objects can't be large. While there are no physical limitations, Hermes uses a fairly simple copying garbage collector with linear complexity in both time and memory. The larger the documents, the longer (linearly) the garbage collection will take.

Containers, on the other hand, are for large and heavily structured data, with Hermes objects acting as elements of these containers (maps, vectors, tables, indexes, etc.).

Alongside Containers, the priority is [DSLEngine](/docs/overview/vm) and the related tools (compilers, build systems, etc.). This subsystem depends on Core and Runtimes but ideally doesn’t depend on Containers. It can be developed independently of everything else.

Memoria gravitates towards databases, and programming for Memoria is more like in-database programming than traditional development. This is due to both the programming paradigms used and the peculiarities of the runtime. For example, Memoria’s runtime heavily relies on fibers (userland threads scheduling), which is not easily compatible with many other systems. If C++ code can somehow manage, integrating this runtime with Java or Python is a no-go.

Using familiar high-level, productive languages like Java and Python for in-database programming with Memoria won’t work because of the differences in runtimes. And even if it could work, these languages (and their runtimes) have weak support for advanced data types—both in terms of language syntax (leading to bulky code) and in terms of potential optimizations. Java/Python would allow for a quick start and faster practical use, but medium-term issues would arise. Forking them to suit Memoria’s needs isn’t something we’re keen on doing.

In the third decade of the 21st century, the last thing we need is yet another programming language. But DSLEngine isn’t just another programming language (it’s an IR+Runtime for it). "Another one" would be Mojo, which aims to make the Python + C++ combo easier to use by giving Python developers the power of a C++ optimizing compiler. Otherwise, it’s all the same -- the same basic primitives (control flow), the same basic data types (arrays and list structures), the same paradigm, just now everything’s out-of-the-box from industry leaders.

DSLEngine is a different paradigm, moving away (but not abandoning) from the familiar control-flow towards data-flow and complex event processing (RETE). It’s first-class support for advanced data structures at the IR and Runtime levels. It’s focused on data processing in general, with a deep emphasis on automation. I believe such a platform has a right to exist in the 21st century.

The special significance of DSLEngine becomes clear when you consider that Memoria needs -- and will develop -- a separate computing [architecture](/docs/overview/accel) (MAA), which moves away from the traditional paradigm of a unified coherent address space and the algorithms and data structures possible within it. Instead, we have messaging, [persistent data structures](/docs/overview/storage/#persistent-data-structures), in-memory computing, corresponding hardware infrastructure, and programming patterns tied to all of this.

Moreover, MAA can scale both "up" to the size of large clusters (and even decentralization) and "down" to the level of [embedded devices](/docs/applications/eiot). DSLEngine will provide a more-or-less compatible (100% compatibility isn’t the goal, as it's technically very challenging) Runtime for code at any scale.

### N3
The third priority is Storage Engines, the most algorithmically complex part of the framework. But the complexity here is not just algorithmic; it's also system-technical. Organizing reliable memory state dumps to stable storage is quite challenging. The corresponding database subsystems are always quite complex in terms of handling various edge cases.

One of Memoria’s important and practically significant subprojects is [computational storage](/docs/applications/storage/) (CompStor), which essentially means implementing the storage engine + DSLEngine directly in the storage controller space, with bidirectional access via the HRPC protocol. This way, transactional semantics (and power-failure resilience semantics) can be implemented with the best guarantees by the device manufacturer. And it can ensure the highest speed since there won't be any efficiency losses at the block device interfaces. These interfaces often have to be made overly complex both at the storage (SSD) level and the file system or database level to handle this.

CompStor, especially in Memoria’s multi-model format, is a very bold project because it will require almost everything to be redesigned. No more OS that contains file systems and databases. No need for multi-core CPUs with their crutches and quirks that execute this code. Just grab a network accelerator, a computation accelerator, and a storage accelerator. And that's it -- a ready-made computing system. No CPU needed. Except maybe for running legacy code.

With CompStor, a database is reduced simply to a Query Federation Engine like PrestoDB. By the way, PrestoDB is a very well-designed system, and I worked closely with it (or its fork, I don't remember exactly) during my time at Currents (more on this below), and its internal design greatly influenced how the API for Containers and Hermes was later implemented.

Regarding CompStor, it's probably the easiest part of the project to get funding for -- both through grants and business interactions. But it also comes with the highest number of potential risks -- patent issues, pushback, and just plain opposition out of nowhere. CompStor barges into a niche that is thoroughly proprietary with an ideology of openness. Has anyone seen open-source SSDs in the wild?

The project currently has three main (and several auxiliary in the future) storage engines:

**MemStore** -- an in-memory store with branch support and the ability to have multiple parallel writers. It’s the fastest storage option. Ideal for testing and when data easily fits in memory (which can be quite large these days).

**SWMRStore** -- single-writer-multiple-readers, working through mmap (for simplicity of implementation, for now). It supports branches, commit history, but doesn’t support parallel writers. However, writers don’t interfere with readers. It uses reference counting for memory management, meaning it’s relatively slow on writes (every update triggers many counters). Ideal for analytics.

**OLTPStore** (not finished yet) uses a memory management mechanism from LMDB/MDBX but with its own structures (Memoria’s specialized containers) as a FreeList. It doesn’t use counters, so it’s theoretically very fast on writes. It’s being developed immediately for IO through io_uring, not mmap, meaning it will have its own cache. Like LMDB/MDBX, it allows instant recovery after failures and is sensitive to the lifetime of readers, as in this scheme, readers block the release of all memory allocated after them. For this reason, it’s not suitable for analytics, where long-running queries are common. However, it’s ideal for transactional convergent (super-multimodel) databases providing high-performance strongly-serialized transactions. It can work, for example, as a very high-performance (in CompStor’s version) persistent queue/log and be useful for projects like Kafka/RedPanda.

In the Storage Engines direction, there are two priorities:

**Basic functionality**. Bring the existing basic functionality to a stable and high-performance state (which is already sufficient for many types of applications), including implementation for the reactor ([Seastar](https://seastar.io), asynchronous IO). The goal is to pass an extended system of randomized functional tests. The architecture should be adapted to CompStor.

**Advanced functionality**. For example, replication, both logical and physical—through patches, similar to how we do it in Git. Or parallelizing the writer for SWMR. The writer can be one, but there can be many threads. They just all have to commit simultaneously as a single operation. This will speed up some heavy O(N) operations like table conversion, ETL, etc.

## Development Experience in Companies and Conclusions
After dabbling a bit with Memoria's development in commercial companies as an employee, I’ve come to two conclusions.

**First conclusion** -- I don’t want to do this anymore. While I gain (without exaggeration) invaluable experience in implementing and productizing the project, the project itself starts to warp significantly under the company’s needs and vision, even if the company isn’t explicitly pushing in that direction.

**Second conclusion** -- Distributed storage is quite complex because it requires a distributed, fault-tolerant, and very high-performance garbage collector to manage memory -- removing blocks that are no longer visible after commits are deleted.

This is all doable, but it’s labor-intensive work, and it’s better left to commercial cloud companies to handle "for themselves" if they need it. Contributions are welcome, as always! But in the project, I’m not particularly interested in pursuing this otherwise tempting direction "on my own dime."

Instead, I’ve chosen the path of decentralized storage and emulating distribution through decentralization.

The decentralized model is like Git. Any storage can contain all or just a fragment of the data, and data migrates between storages explicitly -- through patch transfers (physical replication). Memory management here is purely local since everything happens within a single machine, and there’s no distributed garbage collection.

This approach is a bit slower and more complex to implement at the application level because now applications need to explicitly manage data transfers between nodes in a distributed system, directly requesting the data they plan to process. Additionally, datasets must fit within the external memory of a single machine. In practice, this isn’t problematic, but it’s still a limitation since this space now needs to be planned.

An initially distributed scheme is free from these limitations, so if someone needs it and can do it, they should develop a distributed garbage collector for themselves.

It’s important to note that the challenges exist only with universal, fully persistent storages that use reference counting to track block reachability. Reference counting, by the way, isn’t needed if a tracing, rather than deterministic, garbage collector is used (which would create counters during the process). With this, writes would be significantly faster, but garbage collection would likely be slower. So, what the average outcome would be remains a question.

OLTPStore, designed for transaction processing, doesn’t use reference counting for blocks or a garbage collector, although blocks to be released are accumulated in internal structures for deferred execution (like a GC). The other types of storage are for analytics, where having the data locally and reading predominates over writing. This was the main logic behind focusing on decentralization. The decentralized scheme is more grassroots, fitting well with the familiar Git Flow. It’s also easy to deploy locally and is portable to the cloud 1-to-1, without modifying applications. It will be good enough (if not perfect) for data science, analytics, AI -- the areas Memoria is aimed at.

In short, the choice of storage system (and it will indeed be a system that collectively covers the main applications -- OLTP, analytics, static datasets, embedded in processes, etc.) is dictated by many factors. These include technical aspects like implementation and usage complexity, minimizing the possibility of data loss, as well as economic ones, like the desire to avoid competition and not fall into the patent trap. The latter is a minefield, especially in the area of data storage and processing.

## AI and Philosophy
I've been interested in this topic since my university days. My BS thesis and MS dissertation were on speech recognition (late 1990s). I remember writing a software sonograph on Qt to visualize the frequency structure of speech signals -- back in the days when I was already using Linux. I must have spent hundreds of hours analyzing the variety of speech signal spectra. I wrote the programs, defended my theses, and came to one conclusion: "The problem of speech recognition boils down to the problem of understanding speech, and the latter cannot be solved by digital signal processing." It was time to figure out how humans actually understand speech.

First task -- "figure out how the brain works." This was quite the quest in the early 2000s, especially since I didn't even have a phone (let alone dial-up internet) at home in Smolensk. So, I studied the topic through medical websites. To the credit of Russian medicine, they were leading the way in informatization back then, with a lot of popular material published online. I managed to get a handle on things, even learning about Mountcastle and the columnar organization of the cortex, which didn’t become widely discussed in popular science until 10 years later (much to my surprise).

After digging into the brain, I realized that I understood nothing. And it wasn’t just me -- no one really understood it. Otherwise, they would have written about it. So, I tried approaching it from the "psychological" side. This experience turned out to be much more productive, though far less straightforward than what we expect from reading programming tutorials. I made a ton of mistakes and had to learn from them. But in the end, I stumbled upon the beginnings of [Algorithmic Information Theory](http://www.scholarpedia.org/article/Algorithmic_information_theory) (AIT), which in Europe and Russia is more commonly known as Kolmogorov Complexity Theory. I call it AIT because the abbreviation is more widely recognized. I started googling and found out that everything had already been invented before me. There were even [mathematical theories](https://people.idsia.ch/~juergen/artificial-curiosity-since-1990.html) for some fundamental higher cognitive processes. That’s when Schmidhuber became my idol -- not as a person (I don't know him personally), but because he did "this" long before I did. Whether I like it or not, I continue and expand on his work.

"Discovering Schmidhuber" was a huge relief for me, as it meant I could stop my research and focus on practical work, which, around 2010, was in short supply in AI. Especially in methods derived from AIT (not much has changed since then, but we’ll work on that). Later on, I realized that stopping my research program was a mistake, and I’ve somewhat resumed it. But more on that later.

That’s how the practical ideas that eventually led to the creation of Memoria came about.

Schmidhuber's ACC is simply genius, though the reader will have to take my word for it. Besides its brilliance, it's the best private theory of emotions currently available. Like almost everything in AIT, it's as practically useless in its direct form as it is mathematically universal. And it’s absolutely universal. That is, any "universal AI" would inevitably operate on this principle.

AIT methods are theoretically well-suited to approximation (estimating an object's Kolmogorov complexity through its compression), including ACC, and to make it applicable in psychology, such an approximation needs to be carried out. The brain is far from a "universal AI." It can perform some compression, but that’s about it. Moreover, various evolutionary mechanisms layer on top of this, resulting in phenomena like "interest" acting as an ensemble or mixture of distinct and diverse mechanisms. This creates the main set of challenges in psychology, as higher cognitive functions are complex ensembles of low-level ones, which are relatively easy to register and study. But the structure of this ensemble is not necessarily a weighted sum. It could be some arbitrary algorithm (*). And this is also known as the "mind-body problem."

(*) You could say that "any arbitrary algorithm" can be approximated by a weighted sum, which is precisely what neural networks do. And you'd be formally correct. The question is, what’s easier for a person to do mentally -- discover the actual ensemble algorithm or derive the weighted sum coefficients? The brain doesn’t do backpropagation.

When we try to "understand ourselves" or "understand how our mind works," we’re trying to solve the mind-body problem -- reducing subjective experiences to some objective basis. That basis could be the brain, if we knew how it worked. Of course, I’m oversimplifying; we know a lot, but not everything yet. My point is that reducing experiences to brain activity is still an unstable basis -- it may change over time.

Digital computation, in this sense, is a much simpler and more stable basis than neural tissue activity. While it may seem scientifically absurd to reduce human experiences to the basis of a silicon electronic-digital machine, considering that our brain is nothing like that, this approach exists and is called Computationalism.

The interesting point here is that the success of LLMs in simulating human-likeness (which I'll define below) has greatly boosted the status of Computationalism as a practical philosophy and psychology of AI. So, as they say, the topic has taken an unexpected turn.

In this context, Schmidhuber's ACC is a piece of the puzzle for solving the mind-body problem within the framework of Computationalism. And a very high-quality piece. Its second advantage (besides mathematical universality) is its extensibility -- the ability to fit other experiences into the same mathematical framework. For example, [Simplicity Theory](https://en.wikipedia.org/wiki/Simplicity_theory), which for some reason has apparently stalled in its development. There are ideas as to why, but we’ll get to that later.

This is the direction I’m working in (my personal research) in AI. I have a notebook associated with the Memoria project [here](https://github.com/victor-smirnov/digital-philosophy/blob/master/Artificial%20Intelligence.md), where I occasionally jot down ideas within Computationalism that have crystallized after discussions in chats. I formalize higher cognitive functions (HCF) as they appear to an informed (*) subject and try to identify a minimal basis for them, which can then be supported by a computational platform (algorithms and data structures). HCFs (at the experience level) will be obtained later through the composition of basic functions and data.

For example, the Observer problem is defined there as a reformulation of the qualia problem from the philosophy of mind, and a computational basis is proposed for the Observer -- the so-called self-applicable Turing machine, and for experiences -- "higher-order computational phenomena" occurring in such a machine. There’s no esotericism here. Higher-order computational phenomena are the program's reflection on its runtime and state. For example, on its execution time ("I’ve been thinking for a long/short time"). Some of these phenomena can have stable, consistent manifestations, and for this reason, they can be elements of the self-applicable machine's internal language. The idea is that, for example, "free will" is one of these phenomena, reduced to the machine’s observation of its inability to see all the determinants of its own decisions. "Free will" is needed for a very practically important function -- agency, on which the ability of a computational system to enter relationships within the framework of human agency depends.

(*) The ability to describe higher cognitive functions in humans directly depends on their level of intrapersonal intelligence (Intrapersonal Intelligence or I.I.). In general, this is the ability to analyze and describe one's mental states. As with all other types of intelligence, people with low I.I. are unaware of this. The Dunning-Kruger effects fully apply here. High I.I. isn’t necessary for everyday life. People who don’t develop I.I. will likely have low I.I. (with some nuances). There are people with a genetically determined abnormally high level of I.I. Such individuals are of particular interest for human-like AI, as they are capable of "debugging" it.

Self-applicability and related phenomena are already actively at work. For example, server systems use logs, storing part of their state in them, and these logs are then analyzed, based on which decisions are made that affect the system's functioning. This is already full-fledged self-applicability, just not deep or high-level enough yet. But there are no technical barriers to achieving this.

What I called a "self-applicable Turing machine" doesn’t necessarily have to be a Turing machine. In Memoria, the results of searching for and forming a formal basis for HCFs (like the Observer) will be implemented at the level of the basic type system. This hasn’t been published yet, but there’s a lot of room for persistent/functional data structures (which are a generalization of the software logs mentioned above), basic computation forms (see below), and even special hardware functions (e.g., hardware support for software reflection and introspection).

A Forward-chaining Rule System (FCRS) based on variants of the RETE algorithm, exemplified by [Drools](https://www.drools.org/) and [CLIPS](https://www.clipsrules.net/), surprisingly fits well as a "self-applicable Turing machine" since it inherently operates on this principle, and nothing substantial needs to be added. For example, "I’ve been working for a long time" is just another event the system can respond to in its usual way, like all other events.

FCRS (along with backward chaining RS and the familiar control-flow (CF)) will be fully supported at the DSLEngine level and, eventually, at the MAA level. RETE is very well accelerated at various levels. Beta nodes are simply the Cartesian product of sets, have quadratic time complexity, and therefore fit well with systolic computational architectures. RETE also hybridizes well at both the alpha and beta memory levels. You can create fuzzy and probabilistic extensions of the basic algorithm, as well as hybridize with neural networks, which are essentially also from the FCRS class.

A renewed interest in FCRS might arise from multi-agent systems based on LLMs, linked to replacing the CF controller (Python) with an FCRS controller and tools. FCRS is much better suited for real-time event processing than CF and can significantly simplify the overall system design. Although CF doesn’t surpass FCRS in terms of expressiveness.

Returning to self-applicable computations, qualia, and agency, implementing their functional analogs at the DSLEngine level won’t automatically give it "human consciousness." That won’t happen at all, because DSLEngine itself contains too little data (it’s just a sophisticated rule engine) for behavior of human-level complexity to emerge. And there’s nowhere for that data to come from, at least not quickly. The agency that can be manually programmed at the DSLEngine level can be characterized as "micro-agency," though this term should be carefully defined, as humans, surprisingly, also exhibit micro-agency.

It remains an open question whether micro-agency will have any serious practical applications. Its presence in the system may ensure that the agent system passes corresponding tests within the functionalist approach to agency. That is, it’s simply recognized that, for example, within its micro-scale, it has a functional equivalent of free will and can therefore bear a functional analog of responsibility. But how complex such behavior will be is a separate question. We don’t recognize human-level agency in an ant, and, accordingly, we don’t see it as a legal entity within human society. The same applies to such micro-agency.

However, there are at least two indirect applications.

**First** -- it’s a tool for developing intrapersonal intelligence within the framework of the computationalist paradigm. The thing is, our ability to understand ourselves directly depends on our ability to express the structure of mental states and the relationships between them in language. Both the states themselves and (especially) the relationships between them can be very complex, far beyond the familiar symbolic structures (tables, trees, and graphs). We need tasks and tools that would motivate such activity and support progress toward corresponding goals. These tasks are AI, and the tool is Memoria. Memoria places a clear emphasis on advanced data structures and fairly complex algorithms, which are expected to create a foundation for the radical development of I.I.

**Second** -- LLMs spontaneously develop self-applicability. A large corpus of texts describing their properties forms, meaning they develop a self-model. Or rather, elements of it. For now, just elements, which are still insufficient for full-fledged micro-agency. But the latter may emerge simply because the model’s memory structure suddenly becomes sufficient for it, and the learning algorithms will do the rest (derive the necessary structures for attention, for instance). This might happen simply because the developers themselves have low I.I. due to the predominance of the "black box" approach in their environment. Few are even interested in psychology, let alone theories of mind.

Here’s what needs to be said. "Functional consciousness" (and functional agency) will belong to the "weak" category according to Searle. That is, it’s simply a functional imitation of the corresponding HCFs of humans. However, the open question remains: how will the collective human mind react to the emergence of full-fledged functional agency in AI? We are NOT READY FOR THE CONSEQUENCES.

However, the genie is already out of the bottle, and the process of developing AI toward self-applicability can’t be stopped, even if LLM training is completely banned worldwide. A human-like AI, albeit one that stumbles over its own feet, has already been created. And it’s already interacting with us in ways we don’t see due to the weakness and unpreparedness of our intrapersonal intelligence for such a sudden turn.

I believe the best strategy is to try to lead what cannot be prevented. If I previously kept the priority of practical work on functional consciousness low, now it’s time to reconsider it to be prepared for what’s coming. Memoria will help develop I.I., which in turn will help us see what exactly we need to prepare for.

## Memoria and AIT
So, let’s put I.I. aside for now. Now, let's talk about AIT and Memoria.

AIT is fascinating because it proposes extremely generalized models of intelligence -- Solomonoff Induction, AIXI, Goedel Machine, and their many variants. These models are directly or indirectly based on the idea that intelligence (in a broad mathematical sense) can be reduced to predicting strings (in the case of AIT, binary strings, but it doesn’t have to be). For a given predictor P(s), the more accurately it predicts the next symbol s (the closer the empirical distribution is to the theoretical one), the more intelligent it is. This setup might seem very abstract, but in the example of autoregressive LLMs, we can see how such a low-level model of intelligence can unfold into a full-fledged human-like intelligence, interacting with us through natural language.

At this point, it’s worth defining what "human-like AI" means in terms of the predictive task. Just like the predictor P(s), the human brain is capable of predicting the parameters of incoming signals. This may be very implicit, but let’s accept it as a given. Just as a predictor assesses the probabilities of its strings, so does the brain evaluate the probabilities of its signals. A human-like predictor P(s) would assign probabilities close to those assigned by the human brain. In this case, the high-level part of the model’s intelligence would be perceived by humans as natural, intuitive, and almost as if they were interacting with another human.

It’s important to note that a human-like predictor P(s) doesn’t have to assign true probabilities to strings. If it doesn’t, it would just make the same mistakes humans typically make. And from the human perspective, it would still seem human-like. But when the predictor makes different mistakes than humans typically do, it leads to human frustration (the reverse is also true, but models don’t yet know how to experience frustration).

An interesting point is that training LLMs on text datasets seems to create probability estimates for strings that are very close to those produced by the brain for corresponding signals, despite the significant difference in inductive biases. This phenomenon still awaits its researcher.

The main tenet of AIT regarding string prediction is that the better a predictor P(s) can compress a string s, the better it can predict it. Therefore, prediction boils down to compression, and so does intelligence. For neural networks (and other models obtained through machine learning), compression corresponds to the model’s "generalization of its training set," meaning it correctly classifies (predicts) points belonging to the same class as those in the training set. Compression here means that the amount of information about the classes contained in the model is significantly less than the amount of information that would be required for all points in all classes. In other words, the model is a compressed description of the corresponding object.

Extracting some intelligence directly from compression might be technically challenging. However, it can be [done](https://arxiv.org/abs/cs/0312044). This also implies that direct hardware acceleration of data compression algorithms makes sense in the context of AI accelerators.

But there are indirect techniques as well. [AIXI](https://en.wikipedia.org/wiki/AIXI) is a _universal_ Bayesian agent based on Algorithmic Probability and the Universal Prior M(x), which is interesting because it already contains all the models of the environment, and AIXI simply "pulls out" these models through Bayes’ rule and interacts with the environment based on them.

M(s) is non-computable, but resource-bounded approximations are possible. One such approximation of AIXI in the domain of Variable-Order Markov Models (VMM) is [MC-AIXI-CTW](https://arxiv.org/abs/0909.0801). In this model, M(x) is approximated using a probabilistic data structure (here: a structure encoding probability distributions) of a suffix tree for VMM and a method of approximating the CTW stock, which the tree lacks. Such an agent can learn to play board games and even beat other agents.

LLMs work in a very similar way, but there’s an important nuance that might help us better understand LLMs and why they work. MC-AIXI-CTW is also based on a probabilistic model, but of the environment rather than natural language. But this isn’t a fundamental difference. In both cases, there are strings (texts) the model has seen during training and those it hasn’t seen, but for both types of strings, it must provide a probability estimate close to the true one.

In the case of MC-AIXI-CTW, there is a clear separation between the "database" (suffix tree) and the mechanism for deducing probabilities of new strings based on CTW. Such databases were once called deductive because they could deduce new knowledge from existing data.

In the case of a transformer-based LLM (neural network), there’s no clear separation between the "database" and the "deducer." It’s just a function approximator that does the computation. However, such a separation can be implied. There is a part that is remembered from the dataset (database/knowledge base) and another part that comes from the transformer’s ability to "generalize." I think it’s unnecessary to prove that the larger the remembered part, the less the need for generalization, and vice versa.

Now, considering that the Kolmogorov complexity of a classifier cannot be less than the complexity of the boundary between classes and the complexity of a generator cannot be less than the complexity of the generated object, it should become clearer why leading LLMs are so large: they don’t generalize very well. More on this below.

As doctors say, MC-AIXI-CTW remains "underexplored" in history, meaning under-researched, because everyone "went to the front" -- pairing deep learning with reinforcement learning. We also thought back then that CTW didn’t approximate the probabilities of new, previously unseen strings very well. And, as noted above, there’s suspicion that transformers don’t generalize them very well either. But to test this, we’d need not only to create a truly massive database for MC-AIXI-CTW’s suffix tree but also an efficient mechanism for computing on these data structures.

In short, MC-AIXI-CTW and its analogs and extensions seem like interesting test tasks or benchmarks for MAA within the framework of AI. The advantages of this agent include that it’s fully dynamic and interactive. It learns directly during its operation, unlike LLMs, which are trained asynchronously with a rather long update cycle. Moreover, a suffix tree is perfectly interpretable.

When priorities allow, a full-scale "large" implementation of MC-AIXI, designed for scalability, will be made in Memoria. This practical platform will also kickstart more active practical research into methods derived from AIT.

A separate sub-direction of AIT and its approach to AI is compressed data structures (CDS). For containers, a compressed container is one whose size (in bytes) is proportional not to the number of elements it contains but to the amount of information in those elements. For example, compressed data structures based on Huffman codes or Universal codes are often popular. Data structures that use them usually have a physical size proportional to the zero-order entropy H0.

CDS also have intrinsic value. For example, the suffix tree in MC-AIXI-CTW can be very large and "branched." Representing it as a list structure in memory creates enormous redundancy, while there are ways to encode ordinary trees with a spatial complexity of 2 bits per node or less, with quite good dynamic characteristics.

The structural primitives on which CDS rely can be quite computationally intensive; processors will spend cycles on encoding-decoding, which may impact overall performance. However, a simple cache miss when accessing memory leads to the processor waiting tens of nanoseconds and hundreds of cycles before the data arrives. This time can be well spent on encoding-decoding if it reduces the physical volume of data and thus improves caching efficiency.

Moreover, encoding processing can also be done in hardware. We now have more than enough silicon for this. Due to their computational intensity, CDS were once considered impractical, but since the early 2000s, with the proliferation of superscalar OoOE processors, the situation began to change. Memoria started in the late 2000s with Ulrich Drepper's [paper](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) "What Every Programmer Should Know About Memory," which discusses how to properly arrange data in memory to improve computational performance. Improper data placement can reduce performance by 10-100 times.

In other words, CDS are useful in conventional engineering without any "intelligence" involved. [Here](https://memoria-framework.dev/docs/data-zoo/hierarchical-containers), it’s shown how to use compressed character sequences and a B+tree sledgehammer to create "hierarchical containers" -- prototypes of all "nested" containers in Memoria, from Map<K, Vector<V>> to tables and beyond, as far as imagination allows. And here, I explain how compression can help fight the curse of dimensionality in spatial trees and even slightly overcome it.

AIT and mathematics will allow us to go much further in this direction, and Memoria will be the necessary technical platform for implementing these advancements.

## Memoria and LLM

I’m planning to actively explore the possibilities of LLMs for automatic programming. Initially, there was a lot of enthusiasm in the programming community about this, along with fears of losing professional relevance. But over time, that enthusiasm shifted to skepticism (and programmers collectively "breathed a sigh of relief").

LLMs can be used for code generation, but not in a fully automated mode. That is, only in situations where a human can monitor the result using various tools. One such area, especially relevant for OSS, is code consolidation. There are many projects full of good, valuable code, but interest in them has faded. LLMs can be used to help rewrite this code (either entirely or in parts) to fit the Memoria environment.

Another direction is "Memoria as a dataset." The idea is simple: structure the project and enrich the code with metadata. Then, use all of this as part of the training corpus for models (since it’s OSS). After that, the models trained in this way can be used to improve the next iteration of the project itself. And so on...

This is something to think about, and perhaps OSS could get a significant boost from this approach.

One specific direction is probabilistic data structure generation, but this leans more towards probabilistic programming than LLMs. Though LLMs might also prove useful here.
