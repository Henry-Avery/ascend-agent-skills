---
name: platform-agent-launch-config
description: "对齐推理平台与 Agent 的启动配置，生成平台填写说明、节点启动脚本和差异检查。用于把已有平台任务转为 Agent 启动材料，或把 Agent 启动方案转回平台；不负责实际部署。"
---

# 平台与 Agent 启动配置

从同一份已确认的配置，生成平台和 Agent 两边可检查的启动材料。先读平台实际启动逻辑，不预设任一方的命令就是标准答案。

## 怎么做

1. **拿到输入。** 优先读取用户给的已有任务配置、启动脚本和必要日志；同时定位平台源码并固定版本，有本地改动时记录实际读取文件的指纹。已有材料能确定的参数不重复询问。只有 PR 链接不足以确定环境与启动参数，先列缺项，不猜机器、镜像或卡数。
2. **对齐流程。** 沿“页面字段 → 配置保存 → 参数计算/模板生成 → 环境加载与执行包装 → 最终命令”追踪。区分用户填写值、平台生成值和日志/进程证明的生效值。有正常任务材料时绑定同一次执行核对；没有则明确只做静态分析。已有映射可复用，但要核对所读源码版本，并单独说明它与目标部署版本是否一致，未知就保留未知。
3. **生成并检查。** 尽量复用平台已有的纯参数计算和脚本生成函数，不另写一套 rank、端口、设备分配规则。缺运行时信息时先交付已知部分和待补项，含占位符的脚本标为草稿；不借创建任务或 SSH 探测补齐信息。合成示例所补的绑定与假设单独列出，不混为用户已确认配置。

## 交付什么

- **平台材料**：逐项填什么、对应哪个字段，以及可粘贴的完整启动模板。
- **Agent 材料**：各节点 `.sh`、必要 launcher/配置文件和调用命令；说明环境加载、工作目录、变量来源、日志/PID及依赖。如果需外部文件，指出来源，不把孤立一条 serve 命令当作完整复现。标明就绪判断与停止由哪个已有流程负责；生成器附带的 `stop.sh` 不一定是平台实际停止入口。
- **核对结果**：用一张小表列出两边对应值、必要差异、缺失或冲突，并标明是输入、静态派生、运行观察还是未知。已有基线时只改用户要求或证据支持的差异，并解释原因。

输出放到当前任务的独立目录，不覆盖原始配置。简单任务可直接在回复中展示填写说明和核对表，脚本保存为文件。

## 重点核对

- 模型路径、TP/DP/PP/PCP/DCP、KV connector、graph/eager、speculative decoding：只检查当前方案适用项；同一参数在 CLI、JSON 和页面重复出现时核对一致性，冲突不静默选值。Agent 脚本转回平台时，区分共享参数与节点展开值，不把某节点的 rank/IP 固化为通用模板默认值。
- 节点角色/rank、可见设备、通信与管理地址、端口、Proxy、环境加载、cwd、后台进程和日志：说明平台原来负责什么，Agent 侧如何保留这些条件。
- Shell 引号、变量、JSON 和续行：对生成文件做 `bash -n` 等静态检查，不执行脚本。动态参数追到 launcher 传参和 Shell 算式，计算器的中间值不能直接当最终值；标清静态推导或未知，语法通过不代表运行时展开正确。
- 将“材料已生成”“静态检查通过”“真实服务已验证”分别报告。健康检查不能证明模型正确性或两种启动方式完全等价。

本 skill 默认只生成和核对，不提交平台表单、不分配设备、不安装环境、不启停服务。用户另行要求执行时交给现有部署流程，并按当次授权处理。本地页面、MCP、会话同步不是本流程的前置条件。

## 极光源码从哪里看

使用 AiClusterHub 时，在用户提供的源码根下优先定位这些入口；路径和行为随版本核实：

| 关心的内容 | 源码入口 |
| --- | --- |
| 页面值与请求体 | `inference/frontend/src/pages/HomePage/index.tsx`，`createTaskWithWorkspaceBinding` |
| 合并配置和执行阶段 | `inference/backend/app/application/services/task_service.py` |
| rank、实例、端口计算 | `inference/backend/app/domain/services/parameter_calculator.py` |
| 节点模板与 launcher | `inference/backend/app/domain/services/script_generator.py` |
| 环境包装、cwd、启动与日志 | `inference/backend/app/infrastructure/tasks/handlers/start_service_handler.py` |

完整 `script_template` 与结构化参数生成是不同分支。不要假设修改 `global_config.model_path` 就会改写贴入脚本，或页面脚本就是最终脚本。先核对 `/tasks/{id}/scripts` 的实现，接口存在不代表返回真实产物。纯前端 Mock 原型不能提供后端启动依据。

例：`用 $platform-agent-launch-config，把这份已跑通的配置整理成极光填写材料和 Agent 节点脚本，只生成，不执行。`

需求来源：[MindIE Skills Issue #5](https://github.com/mindie-agent/awesome-mindie-skills/issues/5)。此技能提供方法，不打包平台实现；引用或分发其代码前核对平台许可。
