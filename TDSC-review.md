总结
Novelty——和编译时收集工作的区别(与CCR在跨编译器跨语言上的区别，与OracleGT在解决方案上的区别)
Reviewer 1.

How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.
Reviewer 2.

Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.
Reviewer 3.

Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.
Justification. (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)

(2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.


随机化粒度
Reviewer 1.

The limitations, such as, no-support for basic-block level randomization should be mentioned.
Reviewer 2.

Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization
回复
函数级代码随机化通常也可以视为细粒度随机化的一种，尤其是许多基于二进制重写的随机化工作也都采用了这一粒度。本文从两个方面说明了函数级随机化能够和基本块随机化一样提供足够的熵值：
1. 表2统计了数据集中构成可执行程序的平均函数数量，可以看到平均每个可执行程序由2623个函数组成，带来了2^26012的随机化熵值，这足以对抗攻击者的暴力猜解。
2. Section 6.3-B测试了每个函数内部的代码量，以观测当攻击者可以泄露一个函数指针时可以通过相对偏移推断出多少可用信息。实验结果可以看到大部分函数的长度是小于20KB的，这低于先前研究所报告的构建ROP Chain通常所需的最少值。而对于长度大于20KB的函数，我们会在在无条件跳转指令位置对函数内部进行二次切割。
实际上，汇编代码中同样含有基本块边界信息，额外收集这些边界信息作为元数据可以支持MLARandom进一步实现基本块粒度的随机化。考虑到两位审稿人关注到了这一点，是否我们需要在下一版中实现这一额外设计以支持基本块级随机化？

重汇编器评估实验
Reviewer 3.

It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.
回复
Section 6.4介绍了关于重汇编器的可靠性评估实验，这部分实验一是为了证明“用重汇编器随机化跨语言程序并不可靠，还是得像本文一样建立在编译时信息收集的基础上并致力于实现跨语言收集才行”这一观点，二是因为本文的原型工具能够在编译时准确收集所有指针，这刚好可以作为重汇编器评估的Ground Truth，以填补目前ARM64重汇编器评估的空白。
论文目前评估了两项知名重汇编工具，分别是S&P'20的RetroWrite以及USENIX'20的Ddisasm。Review3提到本文没有对比另外两个state-of-art的重汇编器，分别是USENIX'23的ARMore以及ASPLOS'20的Egalito，说明如下：
ARMore. 
Egalito. 

表达问题
Reviewer 1.

The paper should have a section about the deployment and usage of the scheme. Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.
Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.
How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.
What aspects about a code affects the entropy of the randomization?
Review 2.

Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process
Review 3.

The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?
Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.
In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).




Reviewer: 1
Recommendation: Author Should Prepare A Major Revision For A Second Review

Comments: Summary. The papers proposes a framework for function-level randomization of ARM64 binaries. While prior works to provide such fine-grained randomization is proposed for x86_64 platforms, the authors claim that achieving the same for ARM64 is significantly more complex due to multiple instructions to jointly dereference a single target address. ARM64 has significantly more memory addressing variants as well. Another contribution of the framework is the it can handle multi-languages, such applications that use C/C++ and RUST, thus preventing Cross-Language Attacks. To achieve multi-language support, the authors standardise compilers across different languages.

Comments. In general, this is a good engineering effort. The paper was also considerably well written, and in sufficient detail for a journal. I do, however, have certain concerns, if addressed, would improve the quality of the work.

The paper should have a section about the deployment and usage of the scheme. Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.
Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.
The limitations, such as, no-support for basic-block level randomization should be mentioned.
How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.
How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.
What aspects about a code affects the entropy of the randomization?
Reviewer: 2
Recommendation: Revise and resubmit as “new”

Comments: The paper presents MLARandom, a function-level randomization tool designed to protect multi-language ARM64 applications against both traditional Code Reuse Attacks (CRA) and advanced Cross-Language Attacks (CLA). While the paper highlights the effectiveness of MLARandom in providing egalitarian information during compilation and its compatibility with various high-level languages, there are several negative points and potential areas for improvement that can be discussed:

Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization
Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.
Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process
Reviewer: 3
Recommendation: Revise and resubmit as “new”

Comments: ### Summary This paper proposes MLARandom, a fine-grained randomization technique that is applicable to multi-language applications (e.g., those written in Rust and C++) for randomizing the code layout at a fine-grained level. The authors argue that multi-language applications running on ARM64-based systems need protection against code reuse attacks and cross-language attacks. However, existing compiler-based solutions primarily focus on x86-64 and lack universal applicability for multi-language applications. To address this problem, this paper proposes a compilation standardization technique that performs metadata collection at the assembly level, making it applicable to multi-language applications compiled with different compilers and languages. In addition, this paper systematically analyzes the ARM64 specifications and categorizes pointer encoding variants, which helps the binary rewriter reliably patch various types of code pointers and instructions. The evaluation demonstrates that MLARandom imposes negligible performance overhead.

### Strengths 

+ Timely topic 

+ Engineering efforts 

+ Negligible performance overhead 

+ Opensource

### Weaknesses 

- Limited novelty 

- Design decision and some arguments are not properly justified 

- Lack of comparison with previous work

### Detailed Comment 

This work is timely since there is an urgent need for protection against emerging threats to multi-language programs, such as cross-language attacks. However, I have a few concerns as follows.

Design Decision & Novelty. The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?

Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.

Justification. (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)

(2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.

Clarification. Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.

In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).

Evaluation. It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.

### Reference [1] CCR: Compiler-assisted Code Randomization, S&P 18 [2] DDisasm: Datalog Disassembly, USENIX Security 20 [3] Egalito: Layout-agnostic Binary Recompilation, ASPLOS 20 [4] ARMore: Pushing Love Back Into Binaries, USENIX Security 23
