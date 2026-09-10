# COMSOL MATLAB Simulation Skill

**让 AI 写仿真代码，把重复操作交给 MATLAB。**

一个面向 COMSOL + MATLAB / LiveLink for MATLAB 的 Agent Skill。指导 AI 将建模、求解、参数扫描和后处理组织成可复用脚本，减少逐步操控模型时的 Agent 工具调用往返。

> 核心思路：发挥 AI 写代码的能力，让 MATLAB 执行成组操作，Agent 根据结果和日志作出下一步判断。

## 为什么做这个 Skill

如果一个工作流让 Agent 通过 MCP 分别设置参数、创建几何、配置网格、启动求解、读取每个工况，Agent 就需要反复参与这些操作。许多确定性的步骤可以写进脚本，由 MATLAB 连续执行。

这个 Skill 将工作方式组织为：

```text
你描述物理问题与目标
          ↓
AI 编写 / 修改 MATLAB 脚本
          ↓
MATLAB 通过 LiveLink 操作 COMSOL
          ↓
建模 → 网格 → 求解 → 参数扫描 → 后处理
          ↓
保存模型、数据、图表与完整日志
          ↓
Agent 读取摘要，需要时排错并修改脚本
```

**目标是减少不必要的工具调用及上下文往返。** MCP 仍可以用来执行 MATLAB 脚本；重点是把一组操作交给代码运行。对于已经支持批量脚本执行的 MCP 工作流，差异可能较小。

本项目尚未提供调用次数、Token、费用或耗时的对比基准。实际额度消耗取决于 Agent 平台的计量方式、模型生成代码的开销、返回内容以及调试次数，不承诺固定节省比例。

## 能帮助你做什么

- 从已有 `.mph` 模型开始修改，并保护原始文件。
- 将 COMSOL 导出的 `.m` 脚本整理为可配置、可重复执行的工程。
- 根据物理目标组织几何、材料、边界条件、网格和求解设置。
- 在 MATLAB 中执行参数扫描，保存每个工况的结果和错误摘要。
- 导出 `.mph`、`.mat`、`.csv`、`.png` 等产物。
- 检查求解和输出，记录已解决的错误及预防方法。

这是供 AI 使用的工作规范。仓库目前不包含通用仿真执行器、经过验证的示例模型或 COMSOL API 封装。

## 使用前准备

1. 可用的 COMSOL Multiphysics、MATLAB 和 LiveLink for MATLAB 环境，以及目标物理场需要的许可。
2. 能读取技能文件并编写本地代码的 AI Agent。要自动运行，还需要可用的 MATLAB 执行通道。
3. 仿真所需的几何尺寸、材料参数、边界条件、研究类型和目标输出；有现成 `.mph` 或导出 `.m` 时优先提供。

该 Skill 不附带 COMSOL、MATLAB、许可证或 MCP 服务，也不负责安装这些软件。没有执行环境时，Agent 可以生成代码，但必须明确说明尚未运行验证。

## 安装

下载本仓库 ZIP 并解压，将包含 `SKILL.md` 和 `agents/` 的目录命名为 `comsol-matlab-simulation`。

对于使用 `~/.codex/skills` 作为技能目录的 Codex 环境，将整个文件夹放到：

```text
~/.codex/skills/comsol-matlab-simulation/
├── SKILL.md
└── agents/
    └── openai.yaml
```

Windows 默认用户目录下对应 `%USERPROFILE%\.codex\skills\comsol-matlab-simulation`。如果你的环境自定义了技能目录，请使用实际目录；重新开启会话后检查技能是否可见。

其他 Agent 可按其支持的技能加载方式使用 `SKILL.md`；本仓库没有验证所有客户端的兼容性。

## 使用示例

在独立的仿真项目中调用技能，并提供真实参数。以下是需求提示词示例，不代表仓库包含已验证的模型。

### 修改已有模型

```text
使用 $comsol-matlab-simulation。
我的 MATLAB 与 COMSOL LiveLink 已连接。
请读取我提供的 models/original.mph，先梳理模型中的参数、研究和结果变量。
按我提供的工况表编写可重复运行的 MATLAB 脚本，保留原始模型，
输出各工况数据、图表和生成的模型，并记录求解日志。
遇到影响物理含义的缺失信息先问我。
```

### 从简单模型开始

```text
使用 $comsol-matlab-simulation 编写二维稳态导热仿真。
矩形宽 100 mm、高 20 mm，导热系数 15 W/(m*K)，无内热源。
左边界 373.15 K，右边界 293.15 K，上下边界绝热。
先跑通最小模型，导出温度分布图和中线温度 CSV，保存 MPH 与运行日志。
请检查温度是否随横坐标近似线性变化；如果当前环境不能运行，明确标记未验证。
```

### 批量扫描

```text
使用 $comsol-matlab-simulation，将现有脚本中的导热系数
设为 10、15、20 W/(m*K) 三个工况，在 MATLAB 循环内完成求解和导出。
每个工况独立保存结果，单个工况失败时记录错误并继续其余独立工况。
最后返回成功 / 失败摘要和文件路径，需要排错时再读取完整日志。
```

## 生成的仿真工程

Skill 指导 Agent 在你的仿真项目中组织以下文件；这些不是本仓库自带的程序：

```text
simulation-project/
├── src/
│   ├── main.m              # 流程入口
│   ├── build_model.m       # 建模与修改
│   ├── run_case.m          # 单工况求解与错误捕获
│   └── postprocess.m       # 结果提取与导出
├── configs/                # 参数与工况配置
├── models/generated/       # 生成的模型
├── results/                # 数据与图表
├── logs/                   # 运行与求解日志
└── docs/                   # 检查清单、错误记录与验证过的 API 笔记
```

## 如何验证是否节省调用

针对同一模型、工况、Agent 和模型版本，分别记录原工作流与脚本工作流的工具调用数、平台报告的 Token / 额度消耗、总耗时、重试次数和结果正确性。比较时应计入脚本生成与调试成本，分别报告首次运行和复用运行。

欢迎通过 Issue 或 PR 分享带环境版本、可复现步骤和实际测量值的案例。提交模型前请确认允许公开其中的内容；本仓库默认忽略仿真产物与本地凭据文件。

## 许可证

本仓库中的技能说明与配置采用 [MIT License](LICENSE)，允许使用、修改和分发。COMSOL、MATLAB 及相关组件的许可由各自权利人规定。本项目是独立社区项目。

## 一句话介绍

> 开源一个 COMSOL + MATLAB 仿真 Skill：让 AI 编写可复用的 LiveLink 脚本，把建模、参数扫描和后处理交给 MATLAB 成组执行，减少 Agent 逐步调用工具的往返，并保留代码、结果与排错记录。
