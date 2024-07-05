### 总结

#### Novelty——和编译时收集工作的区别(与CCR在跨编译器跨语言上的区别，与OracleGT在解决方案上的区别)

**Reviewer 1.**

- How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.

**回复.** MLARandom和CCR, PointerScope, OracleGT的区别，最好在Background上体现出来。之前介绍了，但介绍的不够。

**Reviewer 2.**

- Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.

**回复.** OracleGT分别在LLVM-6.0.0和GCC-8.1.0上实现了元数据收集策略，如我们前面所述，这一元数据收集方案首先面临跨语言问题——OracleGT只能用于纯C/C++语言编写的程序，而在OracleGT的dataset——Specpu 2006中，29个应用中的10个是全部或者部分由Fortran语言编写的。OracleGT其次面临的是跨版本问题，作为反汇编器评估的工作，它所有用于测试的二进制程序都是使用特定的编译器版本生成的，因此可能忽视不同版本间的输出差异，例如编译优化的不同以及对一些新特性指令的使用。

**Reviewer 3.**

- Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.
  
  **Justification.** (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)
  
  (2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.
  

回复. Reviewer3认为MLARandom和三类工作的区别不大，但单单将MLARandom看做基于GAS的收集方案而不考虑baseline所面临的跨语言、跨版本挑战是不公平的。本文识别到了在如今多语言程序的愈发流行的现状下，先前的编译时随机化工作由于特定于编译器实现而无法有效应用。而MLARandom建立在汇编语言被多数编译器支持这一关键观察下，通过编译标准化和汇编级元数据收集，将原有的信息收集能力从编译器中剥离出来，因而能够灵活兼容不同高级语言以及不同编译器版本。

Reviewer3还有两个论点，一是LLVM支持各种语言作为前端来argue我们的跨语言特性，二是CCR根本是一种原型验证其理论上是支持所有编译器来argue我们的跨版本特性。针对论点二，我们确实没有讲清楚说理论上支持但面向特定版本还是需要做针对性实现。另外，这刚好是提醒到了，刚好现在有LLVM3 6 9 10这三个版本的CCR实现，我们可以看下元数据收集代码是否是多变的。

**跨语言.** 出于对语言特性、性能优化等方面的追求，编程语言往往会设计自定义的前端及后端模块，而这意味着   尽管多样前端+通用的中端优化和后端代码生成这种模型是理论上存在的，但在实践中并非   如今的实践上，这种通用的后端代码生成是     后端也许和最初设想的通用编译器不同，

GCC使用GCC RTL作为Low-level Machine Intermediate Representation

LLVM使用LLVM MC作为Low-level Machine Intermediate Representation

不同语言->不同编译器->采用不同的后端

**跨版本.** 尽管编译时元数据收集的设计方案上并不局限于特定的编译器版本，但它们的实现仍取决于编译器中的具体接口。在编译器随语言特性而快速更新的当下，原本用于采集边界信息或是指针信息的接口可能发生了改变，因而需要大量的手动分析来解决接口变化并确保功能的一致性。举例来说，CCR的原型工具最初实现在LLVM 3.9.0版本上，并在随后的一年多时间里陆续扩展到了LLVM 6.0.0和LLVM 9.0.0。在这个版本更迭过程中，原本的指针修复逻辑发生了许多改动，一些接口函数例如用于查询指针Size的getFixupKindLog2Size重命名为了getFixupKindSize。对于CCR、PointerScope这类将编译时元数据收集用于随机化的工作来说，版本迁移的复杂性意味着随机化机制难以及时适配到最新的编译器上而产生新特性和程序性能优化的滞后性。对于OracleGT这类将编译时元数据收集用于反汇编器测试的工作来说，版本迁移的复杂性意味着待分析的二进制程序都由特定版本的编译器成的(LLVM-6.0.0和GCC-8.1.0)，因此可能忽视不同版本间的二进制差异，例如编译优化的不同以及对一些新特性指令的使用。

#### 随机化粒度

**Reviewer 1.**

- The limitations, such as, no-support for basic-block level randomization should be mentioned.

**Reviewer 2.**

- Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization

**回复**

**函数级代码随机化通常也可以视为细粒度随机化的一种，尤其是许多基于二进制重写的随机化工作也都采用了这一粒度。本文从两个方面说明了函数级随机化能够和基本块随机化一样提供足够的熵值：**

1. 表2统计了数据集中构成可执行程序的平均函数数量，可以看到平均每个可执行程序由2623个函数组成，带来了2^26012的随机化熵值，这足以对抗攻击者的暴力猜解。
  
2. Section 6.3-B测试了每个函数内部的代码量，以观测当攻击者可以泄露一个函数指针时可以通过相对偏移推断出多少可用信息。实验结果可以看到大部分函数的长度是小于20KB的，这低于先前研究所报告的构建ROP Chain通常所需的最少值。而对于长度大于20KB的函数，我们会在在无条件跳转指令位置对函数内部进行二次切割。
  

**需要说明的是，汇编代码中同样含有基本块边界信息，额外收集这些边界信息作为元数据的确能够支持MLARandom进一步实现基本块粒度的随机化。考虑到两位审稿人关注到了这一点，**我们是否**需要补充实现这一设计以支持基本块级随机化？**

#### 重汇编器评估实验

**Reviewer 3.**

- It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.

**回复**

本文在Section 6.4进行了重汇编器的可靠性评估实验，做这部分实验的目的一是为了证明“用重汇编器随机化跨语言程序并不可靠，还是得像本文一样建立在编译时信息收集的基础上并致力于实现跨语言收集才行”这一观点，二是因为本文的原型工具能够在编译时准确收集所有指针，这刚好可以用作重汇编器评估的Ground Truth，以填补目前ARM64重汇编器评估的空白。

论文目前评估了两项知名重汇编工具，分别是S&P'20的RetroWrite以及USENIX'20的Ddisasm。Review3提到本文没有对比另外两个state-of-art的重汇编器，分别是USENIX'23的ARMore以及ASPLOS'20的Egalito，说明如下：

**ARMore.** 和RetroWrite以及Ddisasm不同，ARMore并不追求完整识别出二进制中的所有指针。相反，ARMore通过在进程中维护一份重写前的副本数据，来兼容/实时纠正那些未能修复的数据流和控制流。另一方面，虽然ARMore在代码插装上表现出了良好的适配性，但它并不适合用来实现代码随机化，因为ARMore会在进程中拷贝原代码段副本，而这可以被攻击者用于恢复出原有的代码布局。

**Egalito.** Egalito支持x64和ARM64下PIE程序的重汇编。我们先前的评估实验中确实遗漏了这一重汇编工具，这可以在后续的版本中补充上。但是需要指出的是，Egalito同样依赖于诸多启发式规则来识别二进制中的所有指针，因此可以预见的会存在和RetroWrite以及Ddisasm类似的识别不准确情况，其同样难以应用于细粒度随机化中。

####

表达问题

**Reviewer 1.**

- The paper should have a section about the deployment and usage of the scheme. Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.

**回复.** 本文的部署模型和CCR一致，即编译时元数据收集+离线二进制重写。因此元数据收集只会在代码编译时执行一次，而对于终端用户来说，只需要执行二进制重写器即可在任意时刻对二进制执行随机化。

- Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.

**回复.** 修改Introduction第一段，展开介绍ARM64应用下的代码复用攻击以及随机化.

- How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.

**回复.** 论文的Key Observation在于汇编语言是大多数编译器都支持的一种中间表达形式，因而基于汇编器实现的元数据收集能力能够“轻量级”的适配到各类不同的编译器前端上去。从我们的分析结果来看，GCC、Fortran、LLVM等编译器无需或只需增加额外的编译参数即可实现编译标准化，因此可以兼容编译器的版本迭代而无需任何patch。另一些编译器如Rust和LLVM LTO尽管并未对外提供类似的编译选项，但它们内部仍然支持汇编格式输出。对于这类情况，我们只需要patch其中和工作流控制相关的数十行代码即可实现编译标准化。需要指出的是，这些修改的代码与编译器特性无关，因此容易在不同的编译器版本间进行迁移。

- What aspects about a code affects the entropy of the randomization?

**回复.** 在函数级随机化下，熵值取决于二进制程序中有多少可以实施位置置换的函数：ELF由用户编写的各个模块，以及静态链接的代码库所组成，这些总函数数量决定了函数级随机化的熵值。一个有趣的观察是Rust程序会将标准库以静态库的方式链接到ELF中，这使得即使是一个简单的Rust程序也会具有很高的随机化熵值。例如Listing2所展示的示例程序中只包含了4个User Function，但最终编译出的ELF却由703个函数组成，带来了2^5640的函数级随机化熵值。

**Review 2.**

- Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process

**回复.** Review2关注的问题在于当两个函数随机化之后，可能它们之间的距离会超过跳转指令的最大寻址范围。需要首先介绍的是，ARM64架构下的函数调用大多采用R_AARCH64_CALL26重定位类型，它可以寻址±128MB的临近代码空间，这足以覆盖绝大多数程序的代码段范围。另外有三种比较少见的寻址模式也会在函数调用时采用，分别是R_AARCH64_CONDBR19、R_AARCH64_LD_PREL_LO19、R_AARCH64_ADR_PREL_LO21，他们支持±1MB的寻址范围。当识别到这类寻址范围的函数调用时，我们会在随机化的同时保证caller函数和callee函数不超过原有距离。

**Review 3.**

- The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?

**回复.** Reviewer3所提到的“The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well”属于编译时随机化的一种，其问题在于每次随机化都需要重新运行一次编译过程，缺乏灵活性并且难以在终端设备上部署闭源软件。为了兼容现有的软件分发方式，本文所采用的部署模型和CCR一致，即在编译时收集元数据，以辅助终端设备进行可靠的二进制重写。

- Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.
  
  In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).
  

**回复.** Reviewer3所提到的缺少Comparsion在论文中是以两部分呈现的，汇总表格可见TABLE1：

- 第一个Comparsion是MLARandom与同作为编译时元数据收集的CCR、PointerScope、OracleGT的比较，这三类工作都面向特定版本的编译器实现，因此难以实现跨语言、跨版本的细粒度随机化支持。相应的，MLARandom首先通过编译标准化策略将元数据收集节点从编译器中剥离出来，然后在汇编器中实现汇编级的信息收集，这是本文能够随机化多语言程序的关键。为了证明这一点，我们在Section 6.1中准备了含有C/C++/Rust/Fortran的多语言数据集，结果表明MLARandom都能够正确随机化它们。
- 第二个Comparsion是MLARanom与重汇编工具RetroWrite、Ddisasm的比较。重汇编可以将二进制程序转换为可重定位的汇编语言，因此理论上可以用于多语言程序的细粒度随机化。而我们在Section 6.4节测试了重汇编器在复杂程序下能否准确识别出所有指针，结果表明即便是最先进的重汇编工具也只能识别65%ELF程序中的所有指针，因此难以应用于细粒度随机化。此外，我们进一步溯源了这些重汇编工具识别失败的Root Cause，为它们后续的改进提供依据。

###

###

### **目前初步拟定的修改内容**

1. 修改Introduction的前三段，与IOT内容相结合
  
2. 修改Background即Section 2.2内容，从跨语言支持和跨版本支持两方面详细讨论MLARandom与现有编译时信息收集工作的区别
  
3. 修改Evaluation即Section 6.2，将标题Performance Evaluation改为Correctness Evaluation，该小节将包含两个部分：第一个部分对应Section 4.2即Design的第一部分，通过展示在多语言数据集上的随机化结果以表明MLARandom对多语言程序的支持；第二个部分对应Section 4.3即Design的第二部分，梳理MLARandom在数据集中所整理到的各种指针类型。
  

~~4. 可选. 增加对Go编译器的细粒度随机化支持. go官方编译器使用plan9汇编语法因此不支持GAS格式的汇编输出，此外链接器部分也是自己开发的但可以更换为外部链接器，所以为go语言实现编译标准化是困难的。从跨语言的角度来说是没法讲的，但是从跨版本的角度来说，gccgo是随着GCC发布的一个go前端，对其使用编译标准化可以避免在gccgo的RTL表示上再实现一次元数据收集工作。~~   

5. 可选. 增加基本块级的随机化
  
6. 可选. 增加对重汇编器Egalito的评估 
  

###

### Reviewer: 1

Recommendation: Author Should Prepare A Major Revision For A Second Review

Comments: Summary. The papers proposes a framework for function-level randomization of ARM64 binaries. While prior works to provide such fine-grained randomization is proposed for x86_64 platforms, the authors claim that achieving the same for ARM64 is significantly more complex due to multiple instructions to jointly dereference a single target address. ARM64 has significantly more memory addressing variants as well. Another contribution of the framework is the it can handle multi-languages, such applications that use C/C++ and RUST, thus preventing Cross-Language Attacks. To achieve multi-language support, the authors standardise compilers across different languages.

Comments. In general, this is a good engineering effort. The paper was also considerably well written, and in sufficient detail for a journal. I do, however, have certain concerns, if addressed, would improve the quality of the work.

1. The paper should have a section about the deployment and usage of the scheme. Currently, it is not clear if the metadata collection phase is required at every installation of the application. This would require the entire tool chain to be present in every system where the randomized application executes.
  
2. Another aspect that would make the paper easier to read is an introduction to pointer patching. What exactly is done during pointer patching? How does it protect against CRAs? Currently, the introduction is written in a way that assumes that the reader is aware of fine-grained randomization. However, this may not always be the case. Rather than starting with the ARM architecture, the paper could talk about the usefulness of randomization, the limitations of ASLR, and why fine-grained randomization is needed.
  
3. The limitations, such as, no-support for basic-block level randomization should be mentioned.
  
4. How is this the present work different from [11] [12] and [16]. This can be discussed at the end of section 2.2.
  
5. How difficult is it to achieve compiler standardisation? For instance, should a patch be introduced to every new version of the compiler to achieve the required standardization.
  
6. What aspects about a code affects the entropy of the randomization?
  

### Reviewer: 2

Recommendation: Revise and resubmit as “new”

Comments: The paper presents MLARandom, a function-level randomization tool designed to protect multi-language ARM64 applications against both traditional Code Reuse Attacks (CRA) and advanced Cross-Language Attacks (CLA). While the paper highlights the effectiveness of MLARandom in providing egalitarian information during compilation and its compatibility with various high-level languages, there are several negative points and potential areas for improvement that can be discussed:

1. Limited Granularity: MLARandom employs function-level randomization, which means that entire functions are randomized as a single unit. This fixed granularity may not be sufficient for scenarios that require finer-grained randomization
2. Less Novelty: The paper's approach to identifying functions and relocations at the assembler level may not be as novel as initially perceived. Existing work such as OracleGT has already demonstrated methods for collecting function boundaries and relocation information at a similar level of granularity.
3. Limited Handling of Branch Transformation: The paper does not extensively discuss the implications and mechanisms for handling situations where relative branches may need to be transformed into absolute branches after the randomization process

### Reviewer: 3

Recommendation: Revise and resubmit as “new”

Comments: ### Summary This paper proposes MLARandom, a fine-grained randomization technique that is applicable to multi-language applications (e.g., those written in Rust and C++) for randomizing the code layout at a fine-grained level. The authors argue that multi-language applications running on ARM64-based systems need protection against code reuse attacks and cross-language attacks. However, existing compiler-based solutions primarily focus on x86-64 and lack universal applicability for multi-language applications. To address this problem, this paper proposes a compilation standardization technique that performs metadata collection at the assembly level, making it applicable to multi-language applications compiled with different compilers and languages. In addition, this paper systematically analyzes the ARM64 specifications and categorizes pointer encoding variants, which helps the binary rewriter reliably patch various types of code pointers and instructions. The evaluation demonstrates that MLARandom imposes negligible performance overhead.

### Strengths

- Timely topic 
  
- Engineering efforts 
  
- Negligible performance overhead 
  
- Opensource
  

### Weaknesses

- Limited novelty 
  
- Design decision and some arguments are not properly justified 
  
- Lack of comparison with previous work
  

### Detailed Comment

This work is timely since there is an urgent need for protection against emerging threats to multi-language programs, such as cross-language attacks. However, I have a few concerns as follows.

**Design Decision & Novelty.** The most difficult one to understand is why metadata collection is performed by the assembler and linker. The assembler can shuffle functions when generating the .o file, and the linker can shuffle them as well, but MLARandom only collects the metadata even though its implementation is based on them. If the randomization process is performed during assembling and linking time, there is no need to collect metadata, not to mention pointer patching. I think this design choice should be clarified: to maximize random entropy? Or to be compatible with software distribution systems as in CCR?

Moreover, this paper seems somewhat incremental to me as MLARandom mainly extends CCR [1] to support ARM64 platform. As far as I can understand, it seems to me that MLARandom's contribution is to move the implementation of CCR from the LLVM backend to the assembler. Other than re-collecting function and basic block information, which are lost after the LLVM MC instruction lowering process, it seems to me there is not much difference.

**Justification.** (1) The paper says that CCR [1] only supports C/C++. In my humble opinion, it seems to me that CCR [1] is possible to support Rust as well. The implementation of CCR is on the top of the LLVM MC stage, i.e., x86 backend. So, if the front-end supports Rust, shouldn't CCR also support multi-language such as Rust and C/C++, and thus can provide protection against CLA attacks? (LLVM even supported Go, although deprecated now)

(2) The sentences such as “existing compiler-assisted randomization is typically coupled with specific compiler: CCR [11] and PointerScope [12] extend LLVM v3.9 and v10.0 to collect metadata,” and “it is difficult for them to maintain compatibility with compiler version updates and lack portability across various high-level languages” seems to be justified. The claim that CCR is dependent on a specific version of LLVM is fundamentally incorrect. The design of CCR is not tied to particular LLVM versions. Moreover, It proposes a general principle that can be applied across different compilers, not to mention different versions. The implementation of a PoC on a specific LLVM version is meant to validate the concept and does not imply that the design is bound to that version.

**Clarification.** Metadata collection and pointer patching is the main contribution of this work. However, it is difficult to extract the tangible benefit of these techniques because there is no comparison with existing work. In the case of metadata collection, I would like to see an explanation of how it differs from the code pointer identification technique used by existing binary rewriting tools such as DDisasm [2] and Egalito [3]. In Section 6 (E1), it is stated that existing data flow analysis techniques do not properly recognize code pointers in cases with complex data dependencies, such as large functions. However, I would like to see a detailed explanation of how MLARandom does overcome this problem.

In the case of pointer patching, it would be nice to have a comparison if there is existing work. For example, how many more relocation types or how many more ARM ISA encodings are supported compared to the existing ones (if existing work fails to patch pointer encoding correctly).

**Evaluation.** It would be nice to see comparisons with state-of-the-art binary rewriting solutions, especially Egalito [3] and ARMore [4]. These papers demonstrate robustness by applying their solutions to Debian Packages and seeing if they pass /autopgktests/. It would be nice if the authors could apply their solution to Debian Packages and add a comparison with them.

### Reference [1] CCR: Compiler-assisted Code Randomization, S&P 18 [2] DDisasm: Datalog Disassembly, USENIX Security 20 [3] Egalito: Layout-agnostic Binary Recompilation, ASPLOS 20 [4] ARMore: Pushing Love Back Into Binaries, USENIX Security 23
