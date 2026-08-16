---
title: "Matt Pocock skills：围绕上下文窗口的工程方法论"
date: 2026-08-16T10:03:36+08:00
draft: false
tags: [ai, agent, claude-code, skill]
categories: [tech]
---

![Matt Pocock skills](https://cdn.mahaoliang.tech/2026/20260816170332592.png)

Matt Pocock 的 [skills](https://github.com/mattpocock/skills) 是一套面向 agent 编程的工程方法论。它要解决的核心问题是：**agent 的上下文窗口有限且临时，而真实的工程工作常常超出一个窗口**。

这套方法论建立在两个事实之上：

1. **上下文窗口有限**，且内容越多推理质量越低。作者把模型仍能清晰推理的区间称为 [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)，前沿模型上约为 150k tokens。

2. **会话是临时的**，窗口一旦清空，对话中达成的共识随之丢失，只有写入仓库的工件能留存。

而工作的规模是分档的：有的一个 **session** 就能完成；有的构建要跨多个 **session**；有的连讨论本身都超出一个 **session**。整套方法按这三种规模各给一套工作方式。

三种规模之外，还有两个横切维度：阶段边界上的**上下文处理原则**，以及过程**文档的生命周期管理**。

本文先完成安装和项目准备，再介绍三种规模对应的流程，最后讨论两个横切维度。

## 一、安装与准备

进入具体流程之前，需要先安装 skills，并为使用它们的项目分别完成一次初始化。

### 1. 安装 skills

运行 **skills** 安装器：

```bash
npx skills@latest add mattpocock/skills -g
```

安装器会让你选择需要的 **skills** 以及要安装到哪些 coding agent。确保选中 `setup-matt-pocock-skills`，下一步要用它初始化项目。

### 2. 为每个项目初始化设置

安装解决的是「agent 有哪些 skills」，初始化解决的是「这些 skills 在当前项目里如何工作」。因此，每个要使用这套流程的项目都需要运行一次：

```text
/setup-matt-pocock-skills
```

它会确认项目使用的 **issue tracker**、**triage** 标签，以及过程文档的**保存位置**。这些设置写进项目，供后续 skills 读取；换一个项目，就需要重新运行一次。

### 3. 开始使用

设置完成后，拿一个真实的想法从 [/grill-with-docs](https://www.aihero.dev/skills-grill-with-docs) 开始：让 agent 调查事实、追问风险，把需要取舍的决策留给工程师。这也体现了整套 skills 的立场：**辅助工程师决策，而不是替代工程师决策**。

访谈形成共识后，再根据讨论和构建的规模选择后续路径。接下来依次介绍三种规模对应的流程。

## 二、装得下一个 session：grill 之后直接 implement

所有工作的起点都是 [/grill-with-docs](https://www.aihero.dev/skills-grill-with-docs)：agent 以编号问题逐轮访谈（❓Q1、❓Q2……，可按编号作答），把想法问到双方共享同一理解。访谈有两条分工：事实由 agent 自己查，决策留给访谈对象；敲定的术语随即写入仓库的 **CONTEXT.md**，够格的决策写成**架构决策记录**（Architecture Decision Record，**ADR**），留下决策内容及其理由。

访谈结束后先判断规模。如果后续构建装得下当前窗口，就直接 [/implement](https://www.aihero.dev/skills-implement)，不输出任何文档。这是信息损失最小的路径：对话本身是最完整的信息源，不跨越窗口边界，就不需要载体。

如果某个问题在纸面上定不下来，需要一个能跑的东西来反应（状态模型是否合理、UI 长什么样），走原型支线：[/handoff](https://www.aihero.dev/skills-handoff) 出去，新 session 里 [/prototype](https://www.aihero.dev/skills-prototype)，再 [/handoff](https://www.aihero.dev/skills-handoff)  把结论带回原线程。

![原型支线](https://cdn.mahaoliang.tech/2026/20260816100712249.png)

支线遵循的仍是第五节的上下文处理原则：需要跨 session 携带的是问题和结论，所以使用 **handoff**；原型本身留在自己的目录和 `prototype/<name>` 分支上，不进入原线程的窗口，原线程只收回一行结论。

这一档的前提是工作能够在 smart zone 内完成，因此保持同一个窗口，不 clear、不 compact。如果推进中发现构建实际需要跨越多个 session，就说明原来的规模判断失效，应在最近的阶段边界退出这条短路径，转入下一节介绍的主流程，把当前共识外化并拆分后续工作。

## 三、构建装不下：主流程

构建跨多个 session 时，对话必须外化成能跨越窗口边界的工件。这就是主流程：

![主流程](https://cdn.mahaoliang.tech/2026/20260816164631448.PNG)

[/grill-with-docs](https://www.aihero.dev/skills-grill-with-docs) → [/to-spec](https://www.aihero.dev/skills-to-spec) → [/to-tickets](https://www.aihero.dev/skills-to-tickets) → [/implement](https://www.aihero.dev/skills-implement) → [/code-review](https://www.aihero.dev/skills-code-review)

**/to-spec** 把整段对话综合成一份 **spec**，发布到 **issue tracker**。它不再访谈，只记录已经做出的决策。**spec** 存在的理由正是窗口会结束：它是这段对话的留存形式，供后续多个 session 消费。

**/to-spec** 并不是每项工作的必经步骤。作者给出的触发条件很明确：只有当构建大到需要跨越多个 agent session 时，才需要把对话外化为 **spec**；如果当前窗口装得下，就直接进入实现。

**/to-tickets** 把 **spec** 拆成可以逐个实现的 **tickets**。拆分时遵循两个标准：

1. **纵向可验证**：每张 **ticket** 都是一个穿透实现所需各层的垂直切片（tracer bullet），完成后可以独立验证。
2. **单窗口可完成**：每张 **ticket** 都能装进一个全新的上下文窗口。由于接手的 session 没有参与前面的讨论，**ticket** 还必须包含完成工作所需的上下文。

拆分完成后，**tickets** 用 blocking 关系标出依赖顺序；没有被其他 **ticket** 阻塞的工作可以立即开始。agent 会先展示拆分结果，由工程师确认粒度和依赖关系，获得批准后才发布。

**tickets 发布是主流程的上下文分界线。** 发布之前，从 **grilling** 到 **to-spec**、**to-tickets** 保持同一个会话窗口：**spec** 和 **tickets** 建立在同一段思考上，中途清空会丢掉「为什么」。发布之后，由于每张 **ticket** 都是自包含的，上一个 session 的上下文可以丢弃，**ticket** 之间使用 **/clear** 是最干净的做法。

![主流程](https://cdn.mahaoliang.tech/2026/20260816101222141.png)

如图，上方是一个不断裂的大窗口，下方是彼此隔离的小窗口，橙色 **/clear** 是 session 之间的闸门；闸门只出现在 **tickets** 发布之后。

发布之后，每张 **ticket** 都交给一个全新的 session：**/implement** 驱动 [/tdd](https://www.aihero.dev/skills-tdd) 逐片实现，期间持续运行类型检查和相关测试，最后运行完整测试套件。一个 **ticket** 完成后关闭 session，再用新 session 处理下一张。

这里存在一个流程顺序上的问题。**/implement** 的官方步骤是先调用 [/code-review](https://www.aihero.dev/skills-code-review)，再 commit；但 **/code-review** 要求指定一个 fixed point，并通过 `git diff <fixed-point>...HEAD` 审查从该点到 **HEAD** 的已提交变更。如果刚完成的代码还没有 commit，它就不在这次 diff 里，review 甚至可能因为 diff 为空而停止。

因此，按当前 skill 的定义，更可靠的顺序是：在开始 **ticket** 前记下当前 **HEAD** 作为 fixed point；实现并通过测试后先 commit；再运行 **/code-review**，明确传入这个 fixed point。它会派两个独立 subagent，分别检查代码是否符合仓库规范（Standards），以及实现是否符合原始 spec（Spec）。另开一个 session 并非硬性要求，但能把实现与评审进一步隔离，也让这次 review 的比较边界更加明确。

## 四、连讨论都装不下：wayfinder

还有更大的一档：目标大致明确，但讨论本身装不进一个 session，暂时还写不出完整的 **spec**，例如 greenfield 项目、跨越数月的大型功能，或尚未形成清晰推进路径的长期项目。这时使用 [/wayfinder](https://www.aihero.dev/skills-wayfinder)；如果问题已经足够清楚、可以直接写 **spec**，就不需要它。

**wayfinder** 把跨 session 的规划过程记录在 **issue tracker** 中。使用前需要先为项目运行 `/setup-matt-pocock-skills`，让它知道地图和 issues 应该保存在哪里。

![Wayfinder 使用流程](https://cdn.mahaoliang.tech/2026/20260816182944064.png)

### 1. 第一次运行：建立地图

在项目中输入 `/wayfinder` 并描述这个大型想法。agent 首先与你确认**目的地**：规划结束时应该得到什么，例如一份可以进入主流程的 **spec**。目的地确定了范围，也决定哪些问题需要解决。

接着，agent 围绕目的地梳理目前已经能够明确表述的问题，在 issue tracker 中创建一个 **map issue**，并为这些问题建立子 **tickets**。每张 **decision ticket** 只记录一个需要回答的问题，例如「数据迁移是否允许停机」，而不是「实现数据迁移」这样的开发任务；问题之间的先后依赖也会记录下来。暂时还说不清的问题留在地图的“尚未明确”区域，等前面的决策提供更多信息后再拆成新 ticket。

第一次运行只负责建立地图和 tickets，不直接解决它们；research 类型的 ticket 可以例外地交给 `/research` subagent 并行调查。

### 2. 后续运行：每个 session 解决一个问题

之后打开一个新 session，把 map 的 URL 或编号交给 `/wayfinder`。你可以指定某张 ticket；如果没有指定，agent 会从当前没有依赖阻塞、也尚未被领取的 tickets 中选择一张，并先领取它，避免多个 session 重复处理。

一个 session 最多解决一张 **decision ticket**。根据问题的性质，过程可能是与人讨论、制作原型或委派 research。得到答案后，agent 会把结论写回 ticket、关闭它，并在地图中增加一行结论摘要和链接；如果这个答案让新的问题变得清晰，就继续创建 ticket 并补上依赖关系。

地图只保存目的地、已完成决策的摘要和链接，不复制每张 ticket 的详细内容。因此，每个 session 先阅读地图了解全局，再只打开当前问题需要的 tickets，不必把此前所有讨论重新装进上下文窗口。

这与主流程的切分方式不同：主流程是在规划结束、用于实现的 tickets 发布之后才切换 session；**wayfinder** 连规划本身都太大，所以从一开始就把每个决策问题分配给独立 session。

### 3. 决策完成：回到主流程

当执行前需要解决的问题都已经关闭，也没有新的问题需要从“尚未明确”区域拆出时，规划结束。通常下一步是调用 **/to-spec**，把分散在各张 ticket 里的决策综合成一份可以执行的 **spec**，再接回 **/to-tickets** 和 **/implement**；如果规划后发现剩余构建其实能在一个 session 内完成，也可以直接进入 **/implement**。

因此，**wayfinder** 不是另一套实现流程，而是主流程在超大规划任务上的前置步骤：它的产出是足以指导构建的决策，而不是代码或其他交付物。

## 五、阶段边界：如何处理上下文

前面三种流程给出了默认的 session 切分方式，这一节处理更一般的问题：一项工作结束、下一项工作开始时，当前对话应该如何处理。这里的**阶段边界**不是上下文窗口已经耗尽的时刻，而是一个完整步骤刚刚结束、可以安全停下来的位置。这个判断只应发生在阶段边界；如果一项工作仍在进行，就继续推进，或者把剩余的独立任务交给 subagent，不要在中途清空或压缩上下文。

![如何处理上下文](https://cdn.mahaoliang.tech/2026/20260816184903693.jpg)

### 1. 从上到下判断，第一个“是”就是选择

这棵树的顺序很重要：越靠前的选择，信息损失越少；只有前面的条件都不成立，才继续向下。

**第一问：当前 session 还能继续吗？** 如果下一阶段需要把本轮对话作为第一手信息，或者剩余的 smart zone 足以容纳下一阶段，就直接 **continue**。即使窗口已经进入 dumb zone，只要后续任务足够简单，也可以继续完成。Continue 是唯一不把原始对话降级成摘要的选择，因此必须最先判断。

**第二问：当前上下文与下一阶段无关吗？** 如果探索过程、已做决策和走过的弯路都可以丢弃，就使用 **`/clear`**。例如一张 implementation ticket 已经完成，下一张 ticket 又是自包含的，上一轮实现上下文便没有继续保留的价值。

**第三问：上下文需要被带到别处吗？** 只有需要切换 agent 工具、移动到另一个目录或仓库、交给另一位协作者，或者在阶段中途分出一条支线任务时，才使用 **`/handoff`**。它把当前 session 写成可携带的 markdown，提供的是**可移植性**。如果没有信息需要“旅行”，仅仅因为想换一个新 session，并不足以使用 handoff。

**第四问：任务可以在无人持续参与时独立完成吗？** 如果范围已经足够明确，不需要人在过程中继续提供判断，就交给 **subagent**，完成后只取回报告。自动 code review、wayfinder 中可以并行调查的 research ticket，都是典型场景；主 session 的上下文不会因此被替换。

**第五步：其余情况使用 `/compact`。** 当前上下文仍然相关、工具和目录没有变化、后续又需要人继续参与时，在阶段边界压缩当前对话，并用摘要承接下一阶段。从用户视角仍在原来的工作线程里，从上下文角度则以一份有损摘要重新起步。因此 `/compact` 是决策树底部的默认项，而不是遇到窗口压力时首先伸手去拿的工具。

### 2. 与前面三种流程的关系

| 前文场景 | 默认处理 | 原因 |
|---|---|---|
| 一个 session 能完成的工作 | **Continue** 到完成 | 对话是最完整的信息源，没有必要主动降级成摘要 |
| 主流程：从 grilling 到 tickets 发布 | **Continue** | spec 和 tickets 需要保留前面讨论中的「为什么」 |
| 主流程：尚未发布 tickets，但已接近 smart zone 上限 | 在最近的阶段边界 **`/compact`** | 上下文仍然相关，但没有切换工具、目录或协作者，不需要可移植的 handoff 文件 |
| 主流程：两张 implementation ticket 之间 | **`/clear`**，下一张使用新 session | ticket 已经自包含，不需要继承上一张的实现上下文 |
| 原型支线的出去与返回 | **`/handoff`** | 原型位于独立目录和分支，需要把问题带出去、把验证结论带回主线 |
| wayfinder 的 decision tickets 之间 | 使用新 session，从地图和 ticket 恢复状态 | issue tracker 已经承担跨 session 的记忆，不必搬运完整对话 |

因此，这一节不是第四套流程，而是前三种流程背后的统一规则：**先尝试继续；上下文无关时 clear；信息必须移动时才 handoff；任务可以独立运行时交给 subagent；其余情况在阶段边界 compact。**

## 六、过程文档的生命周期

第五节解决的是阶段结束时如何处理对话。对话一旦被写成文档，还需要回答另一个问题：这份文档在多大范围内仍然有效？

这里的**生命周期**不是文件什么时候会被删除，而是它什么时候还能作为工作的当前依据。文件可以一直保留，但完成使命后，它就只是一份历史记录，不能再被后续 agent 当作最新事实。按有效范围从短到长，可以分成四层：

![过程文档的生命周期](https://cdn.mahaoliang.tech/2026/20260816191258691.png)

### 1. 只服务一次交接：handoff

**handoff** 文档把一个 session 的现场信息带到另一个地方，因此保存在 OS 临时目录，而不是项目仓库。接手的 agent 读完并开始工作后，它的使命就完成了。文件即使还在，也不能代表接手之后的新进展。

### 2. 只服务当前 session 或阶段：过程信息

对话中的探索过程、**grilling** 问答和 **code review** 的临时发现，主要帮助完成眼前这个阶段。阶段结束后，不需要把所有过程都保存下来；真正影响后续工作的结论，应进入下一层文档，或者写入长期维护的 **CONTEXT.md** 和 **ADR**。

### 3. 在当前这项工作中有效：research、地图、spec 和 tickets

这些文档正是为了跨越多个 session：**research** 提供当前决策需要的事实，**wayfinder** 地图承载跨 session 的规划，**spec** 保存这次构建已经达成的决策，**tickets** 把工作交给一个个新的实现 session。

它们会被多个 session 使用，但只服务当前这项工作。问题得到回答、规划汇总成 spec、ticket 被关闭，或者整个构建交付后，对应文档就不再是当前工作入口。它们可以继续留在 issue tracker 或仓库里作为历史，但不能再被当作项目现状；尤其是 **spec**，它只记录规划时的决策，不会随着实现自动更新。

### 4. 对项目后续工作持续有效：CONTEXT.md 和 ADR

**CONTEXT.md** 保存项目长期使用的术语，**ADR** 保存重要决策及其理由。它们不只服务当前这项工作，还会被后续多项工作反复读取，因此需要长期维护。

长期维护不等于永远正确。术语发生变化时要更新 **CONTEXT.md**；架构决策被替代时，要更新状态或说明新的 ADR 取代了旧决定。旧文件仍然可以保留，但必须让后来的 agent 看得出哪些内容仍然有效。

这四层背后的原则很简单：**文档服务的范围越长，越应该少写容易变化的实现细节；文档完成使命后，可以保留，但必须退出当前上下文。过期的信息如果仍被当作事实，往往比没有文档更危险。**
