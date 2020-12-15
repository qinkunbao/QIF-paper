ICSE 2021 Paper #63 Reviews and Comments
===========================================================================
Paper #63 Abacus: Precise Side-Channel Analysis


Review #63A
===========================================================================
* Updated: 24 Nov 2020 9:01:46am AoE

Overall merit
-------------
3. Weak accept

Paper summary
-------------
This paper presents an approximate model counting approach to quantifying leakage in 
cryptographic libraries. Specifically, it applies symbolic execution on a single 
execution trace and generates a partitioning of the symbolic constraints that 
involve sensitive inputs. Using the Monte Carlo sampling approach, the authors find 
inputs that can reveal certain observations. The input sets found using the sampling 
approach are used to compute the amount of leakage. The tool has been applied to 
several real-world cryptographic libraries and estimated leakage in various versions 
of these libraries that is consistent with the documented fixes.

Strengths and weaknesses
------------------------
Strengths
=========
+ Quantifying side channel leakage is an important problem in software security
+ The tool, ABACUS, has been applied to popular cryptographic libraries and compared with 
two other tools

Weaknesses
=========
- The approach is limited to side channels that can be exploited by a local 
attacker that can perform cache-based attacks (does not handle timing side channels 
that can be conducted by a remote attacker).
- The technique seems incremental compared to the technique of CacheD, which also 
performs trace based symbolic execution.
-Lacks technical details such as the memory model and the secret relevant taint tracking.
- Missing some related work

Comments for authors
--------------------
1. Soundness: The approach is a heuristic approach. So if the tool can generate 
sufficient number of input values that fall into a constraint partition then the approach can provide a 
statistical bound on the error. However, the authors does not clarify the details of 
their taint tracking, which is important for the precision of the analysis. 
Assume that a variable, A, of width 16 bits has been tainted 
with secret data in its least significant 8 bits. If ABACUS decides that the 
expression A >> 8 (right shift A for 8 bits) is tainted with secret data then it would lead to imprecision as 
the secret bits get removed due to the shift operation. So a 
fundamental problem in side channel analysis and leakage quantification is 
performing precise reasoning about secrets at the constraint level. 
Since the authors do not provide these details, there is a possibility that 
ABACUS can introduce false positives.  There is a mention of losing precision due to 
the memory model but the paper does not explain what memory model ABACUS uses.

QUESTION: Would ABACUS mark the expression (A >> 8) as secret dependent if 
only the 8 least significant bits of A are tainted with some secret data?

2. Significance: The presented approach can be useful in improving security. 
However, there have been a lot of studies on leakage quantification. The authors 
missed the work by Pasareanu et al. :

Corina S. Pasareanu, Quoc-Sang Phan, Pasquale Malacaria:
Multi-run Side-Channel Analysis Using Symbolic Execution and Max-SMT. CSF 2016: 387-400

and a discussion should be included in the Related Work section.

Also, the approach seems to handle 
leakages that can be exploited via cache side channels only. Since their cache 
modeling is very simplistic, I am not sure how much practical the approach is. 
For instance, for the secret dependent branches the authors do not even consider 
whether the leakage can be actually exploited (do the different branches that 
get executed fall into the same (code) cache line or not?). 
For instance, an if statement like the following is very unlikely to be 
exploited: 
 
  if (secret > 0) 
  L1:   a = 1; 
  else 
  L2:   a = 0;

  as L1 and L2 would most likely map to the same cache line whereas

  if (secret > 0)
  L1:   foo(); 
  else 
  L2:   bar();

  there is a bigger chance that first basic blocks of foo and bar map to different 
  cache lines..
  
3. Novelty: The approach yields a new tool, ABACUS, with instruction-level 
trace based symbolic execution. The idea of partitioning the formula for model counting is 
not new. See 

	Carla P. Gomes, Ashish Sabharwal, Bart Selman:
Model Counting. Handbook of Satisfiability 2009: 633-654

The authors also do not provide sufficient 
conceptual comparison with the state-of-the-art approaches that perform leakage 
quantification. 

4. Verifiability: The authors mention that ABACUS would be released as an 
open-source tool. However, if they fail to do so, there are missing details in the 
technical approach that would hurt reproducibility.

5. Presentation: The paper has many grammatical mistakes. I have included some 
examples below.


Editorial
==========

I think you should use the phrase "secret data" instead of the phrase "key input" when 
you explain your technical approach. You may still use the term "key input" 
for specific examples.

Some examples about grammatical mistakes, typos, etc.

Page 3: IMO, the term "formulation" sounds more technical than the term "slogan".

Page 3: on the bottom of the right column you should replace H(K;o)= ...
with H(K|o)=... to be consistent with your definitions.

Page 4: In the title of Section IV, "Scalable to ..." => "Scaling to ..." 

        "Interprete .." => "Interpret .."

Page 5: "engineer efforts" => "engineering efforts"

        "Existing approach relies " => "Existing approaches rely "

        "element also contained" => "element is also contained" 

        "pick elements k" => "pick k elements "???

Page 6: "less numbers of input" => "less number of inputs" (better to use "fewer inputs")

        "rest input variables" => "rest of the input variables"

        "reduce the number of input while still ensure the accuracy" =>
           "reduce the number of inputs while still ensuring the accuracy"

        "to the calculate" => "to calculate"

Questions for authors to respond to
-----------------------------------
1. Would ABACUS mark the expression (A >> 8) as secret dependent if 
only the 8 least significant bits of A are tainted with some secret data? Please provide more details about your taint tracking.

Comments on the authors' response
---------------------------------
I thank the authors for explaining the challenges of binary analysis and providing the support material showing their commitment to release their code.



Review #63B
===========================================================================
* Updated: 4 Dec 2020 2:16:37am AoE

Overall merit
-------------
4. Accept

Paper summary
-------------
This paper presents a technique called Abacus for detecting side-channel vulnerabilities precisely. By “precisely”, it means that Abacus not only detects side-channel vulnerabilities but also provides additional information about how severe they are, which helps developers to effectively triage the found vulnerabilities. To this end, the paper first develops a theoretical framework for quantifying information leakage precisely and then provides practical techniques to implement the theoretical approach. Evaluation with diverse crypto libraries shows that Abacus is effective at finding important side-channel vulnerabilities.

Strengths and weaknesses
------------------------
Strengths
* Very well written paper 
* Important problem 
* Innovative approach with a well-founded theory
* Impressive engineering efforts to efficiently implement the theoretical approach
* Extensive evaluation of the proposed approach

Weaknesses
* No critical weaknesses

Comments for authors
--------------------
This is a very good paper. Although I’m not an expert, I found that this paper solves a challenging problem with an innovative approach. 

* Significance: The goal of this paper is challenging and ambitious. Detecting side-channel vulnerabilities itself is an important topic but this paper goes one step further to precisely quantify how severe and relevant the reported vulnerabilities are. By contrast to the existing approaches that can only quantify the “average” severity, Abacus aims to provide the “real” severity information under one program execution, which is definitely much helpful to developers. 

* Novelty: This paper makes novel contributions to both theory and practice and both are impressive. On the theoretical side, the paper cleanly formalizes the approach based on information theory. On the practical side, the paper addresses three challenges for the scalable implementation of the proposed idea. These contributions seem sufficiently novel. 

* Soundness: The implemented system, Abacus, is realistic enough and the evaluation methodologies are extensive and thorough. It is impressive that the authors implement Abacus from scratch for raw x86 binary instructions. The evaluation results are also impressive, where Abacus is shown to be scalable and precise for real-world crypto libraries. The case studies in Section VI.C show that Abacus is practical for quantifying leaked information for practical applications. 

* Presentation: I much liked the writing style of this paper. Although I’m not familiar with existing side-channel detection techniques, reading this paper was pleasant and it was easy to understand the main points of the paper as the paper motivates the problem first and then explains how to address the problem at all possible levels of the paper. 

* Verifiability: The authors promised to make publicly available the implementation. However, it would be better to open the benchmarks and metadata (i.e., vulnerability locations, leaked information, tool outputs, etc) as well for easy verification and reproduction of the reported results. 

Typos:
- Figure 2: FSb[(secret)) -> Fsb[(secret)]
- Page 3, In the first sentence of Subsection C: (I) -> ($I$)
- Page 4, In the second paragraph of Subsection D: a program (\beta) have -> a program (\beta) has

Questions for authors to respond to
-----------------------------------
1. Could you clarify and elaborate on the novelty compared to existing work [14]?

Comments on the authors' response
---------------------------------
Thank you for answering my question. Please incorporate the answer into the paper.



Review #63C
===========================================================================
* Updated: 2 Dec 2020 6:13:43pm AoE

Overall merit
-------------
2. Weak reject

Paper summary
-------------
This paper proposes Abacus, a tool used to discover the side-channel vulnerabilities along with fine-grained leakage information. Based on the proposed model of attacker’s observation of each leakage site, it applies optimized symbolic execution to generate constraints and then run Monte Carlo sampling for leaked-bits estimation.

Strengths and weaknesses
------------------------
Strengths:

+Precise side-channel analysis is both interesting and important.

+The proposed technique seems novel and the authors have a full implementation.

+Relatively comprehensive related work.

Weaknesses:

-The paper is not well written. 

-The evaluation is not well-organized.

Comments for authors
--------------------
The paper provides a novel approach for precise side-channel analysis. The idea is interesting and the results look very useful. 

However, the paper is not well-organized. For example, the example password checker seems not to be persuasive for the Real Attack scenario shown in Figure 4. It would be better to design a better motivation example for the proposed model of the attacker’s observation of each leakage site.

The evaluation is not well-designed: 1) Abacus is able to expose side-channel vulnerabilities, including secret-dependent control-flows and secret-dependent memory access vulnerabilities, but the selected existing tool CacheD for comparison can only detect secret-dependent memory access vulnerabilities. But the performance in detecting secret-dependent memory access vulnerabilities of them is not compared. Moreover, it would be better to display a detailed comparison with DATA in Section VI-B. 

2) The precision of the tool is not well evaluated. Although Abacus detected many new side-channel vulnerabilities, the precision of discovering severe vulnerabilities is not well evaluated. The authors seem to represent it by conducting experiments on different RSA implementations and display that newer versions of RSA in OpenSSL tend to have fewer leakages detected by Abacus. I think it can't be the whole story and the detailed effectiveness of its precision should be quantified. The confusing evaluation making it difficult for readers to evaluate the contribution and effectiveness of this work.

Questions for authors to respond to
-----------------------------------
1.	How many of the severe vulnerabilities exposed by Abacus are confirmed by the vendors?
2.	Is Abacus more efficient in detecting secret-dependent memory access vulnerabilities than CacheD?
3.	What's the detailed performance comparison between Abacus and DATA?

Comments on the authors' response
---------------------------------
The response didn't address my concern well. Especially for the comparison with existing tool DATA, and the comparison of CatchD did not discuss the performance in detecting secret-dependent memory access vulnerabilities of them is not compared.



Response by Qinkun Bao <qub14@psu.edu>
---------------------------------------------------------------------------
Thanks for the reviews. 

# Reviewer A:

You are correct that Abacus is an unsound tool for quantifying leakage through different memory access behaviors.
No tool as yet finds all forms of leakage, and we believe that Abacus makes its contribution through its 
use of symbolic analysis and quantification.

The answer to your question is no. Abacus is precise on your example.

We don't use traditional taint tracking as in CacheD. Our preliminary implementation indicates that symbolic 
interpretation can run nearly as fast as taint analysis.

We use symbolic execution to track secrets. When secrets are loaded into the memory, Abacus starts to interpret 
instructions symbolically. We treat secrets as symbols ($S$). For other variables, we use concrete values ($C$) 
from the execution. We do not know which instruction 
may manipulate a secret until we execute it. Let me explain with your example.


```
uin8_t key = input();   // key = S(len = 8)
uint32_t a = key, b;    // a = bvconcat(C(0, len = 24), S(len = 8))
b = a >> 8;             // b = bvlshr(bvconcat(C(0, len = 24), S(len = 8)), 8)
if(b) {foo()}           // The input key doesn't affect b. b is always equal to 0. 
```


In this example, after deriving the formula representing the value of b, we sample it ($bvlshr(bvconcat(C(0,len=24), S(len=8)),8)$) 
with various keys. The result shows that b is always equal to 0 (same at the branch). According to Definition 1, 
this is not leakage ($|K^o|=|K|$). Of course, in some cases, sampling will miss side-channel leakage with 
low probability. 

Thank you for the additional references. We will improve the paper with more details.

```
if (secret > 0) L1: a = 1; else L2: a = 0; => test edi,edi; set al
```

Most compilers (gcc, clang, icc) use CMOVcc or SETcc to avoid a branch. As Abacus works on the binary level, 
it will track values but not identify this statement as a secret-dependent control-flow.

```
if (secret > 0) L1: foo(); else L2: bar();
```
Abacus identifies this as a secret-dependent control-flow. Many real side-channel attacks can exploit even 
sub-cache line granularity, e.g., CacheBleed, CopyCat.

Model counting for logical theories is harder than #SAT.

Verifiability: We'll release the source code, benchmark, metadata, and results. We will attach the benchmark information, metadata, and results as part of the publication and open source Abacus.

# Reviewer B:

The novelty compared to [14] CacheD is that we not only detect leakages but also give a precise quantification 
of the leaked amount, using a new method.

# Reviewer C:

1.  Abacus detects ~150 severe leakages in the older versions of cryptography libraries. Most of those severe 
leakages are fixed by the vendors. The recent versions (OpenSSL 1.1.0f+ mbedTLS 2.15+) still have 3~5 severe 
leakages (big number implementations) identified by Abacus. Vendors fixed those leakages in the latest release version.

2. Abacus is much faster than CacheD (3-100x) according to the result in the CacheD paper. The extended version of CacheS (https://arxiv.org/abs/1905.13332) listed all the 
vulnerabilities it discovered and claims that CacheS can discover all vulnerabilities found by CacheD. 
Abacus identifies all vulnerabilities in the 
CacheS paper when we run it on the same library.  Moreover, Abacus quantifies and measures the leakage, 
not just discovers a leakage. 

3. It is difficult to perform a full comparison as some tools are not available. DATA is available, 
but we don't have access to the same traces in the paper. When we run DATA on our benchmark, we get too many 
reported leaks. DATA removes the false positives with some manual efforts. We use the data from the paper.



Review #63D
===========================================================================

Metareview
----------
The reviewers acknowledged the novelty of Abacus in addressing side-channel attacks, especially since it has been applied to popular cryptographic libraries and compared with existing tools. However, the reviewers would like to see the following issues to be addessed adequately before the paper can be accepted for ICSE 2021:

- Cite and discuss the missed related work 
- Fix the typos and improve the grammar.
- Provide a link to the tool and benchmarks.
- Include a discussion on the differences between Abacus and DATA and on the secret-dependent memory access vulnerabilities when comparing Abacus and CatchD.
