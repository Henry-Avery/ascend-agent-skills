# Ascend Agent Skills

A small, independently published collection of agent skills for Ascend inference engineering and its development workflow. Each directory under `skills/` is a standalone skill. The repository contains only selected reusable methods.

| Skill | Use it for | Boundary |
| --- | --- | --- |
| [ascend-profile-code-review](skills/ascend-profile-code-review/SKILL.md) | Connect an existing Ascend profile to the code and configuration that actually ran; assess observed cost and evidence for added latency. | It does not collect profiles or run benchmarks. Raw trace parsing and controlled before/after measurement can use separate skills when available. |
| [operator-interface-migration](skills/operator-interface-migration/SKILL.md) | Trace producer, tensor layout, dynamic parameters, operator outputs, and graph lifecycle when migrating an operator API. | It does not replace kernel development or prove device compatibility from source inspection. |
| [platform-agent-launch-config](skills/platform-agent-launch-config/SKILL.md) | Trace a platform's launch path and produce matching platform fill-in materials, per-node Agent scripts, and a configuration difference report. | Generates and statically checks materials only; it does not submit tasks, provision devices, or start services. |
| [vllm-dev-chat-organization](skills/vllm-dev-chat-organization/SKILL.md) | Name and organize AI coding chats for vLLM and vLLM Ascend development; prepare an explicit archive request without losing task context. | It requires a chat management tool for actual changes and does not archive chats unless requested. |

## 使用

将需要的单个 `skills/<name>/` 目录安装到支持 Agent Skills 的客户端技能目录。先阅读对应 `SKILL.md`；其中提到的其他技能是可选衔接，缺失时按文中证据边界继续。设备测试、远程操作和发布动作仍需按具体任务授权。

这些技能描述的是工作方法，不附带业务代码、现场日志、服务地址或运行结果。目录中的文件可单独审阅；`ascend-profile-code-review` 有两份检查与报告参考文档。

## Scope and evidence

These skills are instructions for analysis, review, and configuration preparation. They do not claim runtime reproduction or performance gains. Reports produced with them should identify the exact source and execution evidence that supports each finding.

The skills are available under the [MIT License](LICENSE).
