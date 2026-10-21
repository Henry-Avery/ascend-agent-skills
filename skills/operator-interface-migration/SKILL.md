---
name: operator-interface-migration
description: Analyze or implement an operator package/API migration by tracing producers, tensor views, parameter updates, output processing, and graph dependencies. Use for 算子接口替换、两段式调用迁移、上游输入合同变化. Not kernel development, allocator redesign, or standalone diagnosis of an already isolated failure.
---

# 算子接口迁移

保留用户指定的计算语义、缓存所有权和执行模式；把“换入口”与“改变算法或输入生产方式”分开。

## 建立事实边界

读取项目已有的任务说明、参考版本和接口记录；没有统一文档布局时，从代码与调用方建立最小输入合同。有歧义再询问。固定实际代码版本，不将旧记录当成已验证现状。

区分证据：文档约定、参考源码行为、目标包实际 schema/Meta、运行结果。没有安装包时可分析和编写条件性适配方案，不能宣称二进制兼容。同名算子、相同 shape、相同 dtype 都不足以证明 ABI 或语义相同。

## 追踪调用链

从真实入口检查到输出消费方，而不只搜索函数名：

`输入生产者 → Tensor/view 交接 → metadata/动态参数更新 → 主算子 → 输出布局/投影 → 图捕获与回放`

构造一张够用的参数表：

| 参数/输出 | 参考来源与消费位置 | 目标类型/布局/语义 | 更新时机与生命周期 | 证据/状态 |
|---|---|---|---|---|

状态用“已确认（说明确认范围）/推断/未知”。不把特定算子的维度、布局字符串、mask 数值或页大小固化成通用规则。

按实际算子检查：

- Tensor 的 shape、dtype、device、stride、storage offset、共享 storage、写入方和只读/原位约定。连续性合法不等于物理内容符合 ABI。
- 标量与长度是 actual、capacity 还是 cumulative；请求顺序、逻辑页与物理页编号、padding/live token 约定必须一致。
- 两段式调用共享哪些输入和属性，哪些变化要求重建调度结果；metadata 必须来自对应批次的真实 producer，不伪造调度表。
- 返回 tuple、输出 layout、值维度、LSE 和后处理的实际消费方式；不凭变量名判断维度。
- 跨设备/CPU 的长度来源：不要读过期 mirror，也不要全局删除其他 backend 仍需要的 CPU 更新。

上游合同未交付时先列出最小交接字段，允许用明确标为“候选合同”的 mock 验证消费者；不得把 mock 的选择记录成真实上游承诺。

## 图模式与动态参数

定位谁在捕获前、每次回放前、回放中更新输入与调度。检查稳定地址、bucket 身份、容量、跨 stream producer/consumer 依赖及复用栅栏；防止下一批覆盖尚在被读取的 buffer。

旧 `update_params` 路径提前返回可能表示职责迁移，也可能遗漏更新，要追到新执行者证明；不能只因看到 return 就删除或恢复旧逻辑。区分 host 属性计算与把设备长度读回 CPU 的同步。

区分 eager、直接设备图、编译/GE、动态 fallback，以及目标和 draft 的图。一个模式通过不推出其他模式通过。

## 修改与验证

- 用户只要求分析时只交付表格、缺口、最小改动建议，不修改代码。
- 请求实现时保留用户改动，先确认所选路径再替换调用；接口不支持时明确失败或提请选择，不静默 cast、压紧整份缓存、改 mask 或回退旧算子来制造通过。
- 小规模必要转换应说明语义、成本和是否授权；allocator、COW、生命周期或算法变化不因接口迁移自动纳入范围。
- 注释说明参考来源和非显然约束，不堆积已过期提交号或重复代码含义。
- 先定义独立 reference、明确用例和误差标准，再验证；不事后放宽阈值。至少区分真实算子数值、集成入口、动态更新、图回放和整模型证据。
- 文档和 mock 成功只能支持其覆盖范围，不能宣布上游/完整模型或分布式模式已经通过。

## 按故障层级交接，而不重复造工具

vLLM/Ascend 改动使用可用的 `vllm-ascend-change-validation` 规划和汇总证据。只在需要时调用后续技能，并先读取其说明：

- 单算子已定位：`ascend-operator-debug`。
- eager 正常、整图异常：`vllm-ascend-graph-debug`，先定位阶段，不直接宣布内核有错。
- 多 rank/通信特有问题：`vllm-ascend-distributed-debug`。
- 模型结果比较：`vllm-ascend-correctness-validation`。
- 同步或服务操作：使用目标项目已有的远程开发与服务管理工具，仅在用户请求覆盖操作时执行。

没有这些技能时，明确缺少执行能力，交付可移交的输入合同与验证方案。不自动安装工具或扩展权限。收尾给出参数对应、改动范围、实际验证证据、未确认输入和下一步责任方。
