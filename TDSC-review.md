### 总结

#### Novelty——和编译时收集工作的区别(与CCR在跨编译器跨语言上的区别，与OracleGT在解决方案上的区别)

**Reviewer 1.**

- How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.

**Reviewer 2.**

- Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.

**Reviewer 3.** 

- Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.

  **Justification.** (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)

  (2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.



#### 随机化粒度

**Reviewer 1.**

- The limitations, such as, no-support for basic-block level randomization should be mentioned.

**Reviewer 2.** 

- Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization



#### 反汇编器评估实验

**Reviewer 3.** 

- It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.



#### 表达问题

**Reviewer 1.**

- The paper should have a section about the deployment and usage of the scheme.  Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.
- Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes
  that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.
- How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.
- What aspects about a code affects the entropy of the randomization?

**Review 2.** 

- Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process

**Review 3.**

- The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?

- Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.

  In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).



### Reviewer: 1

Recommendation: Author Should Prepare A Major Revision For A Second Review

Comments:
Summary. The papers proposes a framework for function-level randomization of ARM64 binaries. While prior works to provide such fine-grained randomization is proposed for x86_64 platforms, the authors claim that achieving the same for ARM64 is significantly more complex due to multiple instructions to jointly dereference a single target address. ARM64 has significantly more memory addressing variants as well. Another contribution of the framework is the it can handle multi-languages, such applications that use C/C++ and RUST, thus preventing Cross-Language Attacks. To achieve multi-language support, the authors standardise compilers across different languages.

Comments.
In general, this is a good engineering effort. The paper was also considerably well written, and in sufficient detail for a journal. I do, however, have certain concerns, if addressed, would improve the quality of the work.

1. The paper should have a section about the deployment and usage of the scheme.  Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.

2. Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes
   that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.

3. The limitations, such as, no-support for basic-block level randomization should be mentioned.

4. How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.

5. How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.

6. What aspects about a code affects the entropy of the randomization?








###  Reviewer: 2

Recommendation: Revise and resubmit as “new”

Comments:
The paper presents MLARandom, a function-level randomization tool designed to protect multi-language ARM64 applications against both traditional Code Reuse Attacks (CRA) and advanced Cross-Language Attacks (CLA). While the paper highlights the effectiveness of MLARandom in providing egalitarian information during compilation and its compatibility with various high-level languages, there are several negative points and potential areas for improvement that can be discussed:

1. Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization
2. Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.
3. Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process






### Reviewer: 3

Recommendation: Revise and resubmit as “new”

Comments:
\### Summary
This paper proposes MLARandom, a fine-grained randomization technique that is applicable to multi-language applications (e.g., those written in Rust and C++) for randomizing the code layout at a fine-grained level. The authors argue that multi-language applications running on ARM64-based systems need protection against code reuse attacks and cross-language attacks. However, existing compiler-based solutions primarily focus on x86-64 and lack universal applicability for multi-language applications. To address this problem, this paper proposes a compilation standardization technique that performs metadata collection at the assembly level, making it applicable to multi-language applications compiled with different compilers and languages. In addition, this paper systematically analyzes the ARM64 specifications and categorizes pointer encoding variants, which helps the binary rewriter reliably patch various types of code pointers and instructions. The evaluation demonstrates that MLARandom imposes negligible performance overhead.

\### Strengths
\+ Timely topic
\+ Engineering efforts
\+ Negligible performance overhead
\+ Opensource

\### Weaknesses
\- Limited novelty
\- Design decision and some arguments are not properly justified
\- Lack of comparison with previous work

\### Detailed Comment
This work is timely since there is an urgent need for protection against emerging threats to multi-language programs, such as cross-language attacks. However, I have a few concerns as follows.

**Design Decision & Novelty.** The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?

Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.

**Justification.** (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)

(2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.

**Clarification.** Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.

In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).

**Evaluation.** It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.

\### Reference
[1] CCR: Compiler-assisted Code Randomization, S&P 18
[2] DDisasm: Datalog Disassembly, USENIX Security 20
[3] Egalito: Layout-agnostic Binary Recompilation, ASPLOS 20
[4] ARMore: Pushing Love Back Into Binaries, USENIX Security 23

