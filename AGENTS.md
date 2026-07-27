# AGENTS.md

## 1. 文件目的

本文件用于定义本仓库的 agent 执行规范。其目标是：在不破坏研究流程可复现性的前提下，保持 notebook、模型中产物、结果文档、README 展示材料与输出目录之间的一致性。

除非用户在当前任务中明确给出更高优先级指令，否则 agent MUST 将本文件视为默认执行约束。

## 2. 适用范围

本规范适用于以下工作；agent MUST 在这些场景下遵守本文件：

- notebook 修改；
- 数据准备、模型输入与模型中产物管理；
- README 与项目展示文档修改；
- 文件移动、重命名与输出路径调整；
- 诊断分析、回测汇总、研究结果表格与图片整理。

本规范 MUST NOT 被解释为对与当前研究任务无关的大范围重构的授权。

## 3. 执行优先级

当多条要求冲突时，agent MUST 按以下顺序处理：

1. 优先保护研究流程完整性与既有经济解释。
2. 优先保持 notebook 编号、输出编号与下游引用一致。
3. 优先做局部、最小化修改，避免无关重写。
4. 文档优先保持事实化、专业化、展示化。
5. 未经用户明确要求，不主动重跑高成本模型结果。

## 4. 仓库级强约束

### 4.1 编号系统必须同步

若 notebook 内存在步骤编号、markdown 标题编号与输出文件夹编号，它们 MUST 保持一致。

若在已有步骤之前插入新步骤，agent MUST 同步检查并更新以下内容：

- notebook markdown 标题编号；
- 代码中的输出目录变量；
- `outputs/pipeline_artifacts` 下对应文件夹编号；
- notebook 中用于定位输出的打印文案；
- README 或说明文档中涉及该步骤的引用。

流程型中产物目录当前使用如下格式：

```text
outputs/pipeline_artifacts/N-步骤名称
```

例如：

```text
outputs/pipeline_artifacts/1-训练期窗口诊断
```

### 4.2 中产物存放规则

agent MUST NOT 将诊断 CSV、核对表、比较图片、临时复查结果散落在 `data/processed`。

目录分工如下：

- `data/raw`：原始数据或未处理数据；
- `data/processed`：处理后的研究数据、模型输入数据、下游 notebook 需要读取的数据文件；
- `outputs/pipeline_artifacts/N-步骤名称`：流程诊断、复查、比较、验证、解释性输出；
- `outputs/final_results`：最终研究结果汇总、主表、最终版输出；
- `00_docs/readme_assets`：README 展示图片与展示型资源。

输出文件名 MUST 具备可读性，能够直接体现用途，例如：

- `training_window_candidate_summary.csv`
- `seasonality_test_results.csv`
- `nowcast_monthly_overall_plot.png`

### 4.3 路径与引用必须同步

当文件或文件夹发生以下任一情况时，agent MUST 同步检查所有下游引用：

- 新增；
- 移动；
- 重命名；
- 用途发生明显变化；
- 新增 notebook 读取关系。

至少需要检查：

- notebook 中的读写路径；
- README 中的相对路径；
- 输出提示文案；
- 研究结果汇总 notebook 是否仍能读取对应文件。

## 5. Notebook 修改规范

### 5.1 默认 notebook 纪律

agent SHOULD 默认只修改与当前任务直接相关的 notebook。

除非用户明确要求联动修改，MUST NOT 顺手扩散到多个 notebook。

### 5.2 新步骤插入规则

新增分析步骤或诊断步骤时，agent SHOULD 满足：

- 插入位置位于其逻辑依赖完成之后；
- 插入位置位于依赖该结果的后续建模步骤之前；
- 不应静默改变当前主线研究流程。

### 5.3 滚动窗口配置规则

涉及滚动窗口、样本区间或窗口标签的修改时，agent MUST 优先查找 notebook 中集中定义窗口参数的位置，再做局部修改。

若用户要求扩大、平移或重设训练窗口：

- MUST 只修改窗口集中配置处；
- SHOULD 让后续流程自动继承；
- MUST NOT 在不同 notebook 中引入互相冲突的硬编码窗口标签。

### 5.4 变量筛选规则

训练窗口变化时，agent MUST NOT 默认重新做变量筛选。

仅当用户明确要求重筛时，agent 才 MAY 执行变量重筛；否则：

- 保留当前模型变量集合；
- 基于新训练窗口重新计算缩尾阈值；
- 基于新训练窗口重新计算标准化参数；
- 基于新训练窗口重新生成缓存或估计结果（若下游确实依赖）。

### 5.5 高成本重跑纪律

若新增步骤的目的只是帮助判断模型问题，agent MUST NOT 自动重跑完整模型流程。

agent SHOULD 优先做轻量级诊断；只有在用户明确要求、或下游逻辑确实依赖时，才 MAY 执行完整重跑。

## 6. README 与展示文档规范

### 6.1 README 定位

本项目 README 是研究型项目展示文档，不是安装教程。

agent 修改 README 时，MUST 保持以下文风：

- 专业；
- 克制；
- 结果导向；
- 面向研究展示，而非面向使用教学。

README 内容优先级 SHOULD 按以下顺序组织：

1. 研究问题；
2. 方法逻辑；
3. 模型结构；
4. 实证与回测结果；
5. 经济含义；
6. notebook 与结果位置。

README MUST NOT 出现以下问题：

- 安装文档口吻；
- 教程式分步说明；
- 口语化评论；
- 流水账式改动记录；
- 过多低价值格式装饰。

### 6.2 README 公式规范

GitHub 对复杂 LaTeX 的渲染不稳定，尤其在以下场景下更容易出问题：

- 行内公式包含下划线；
- 上标或星号与 Markdown 强调语法冲突；
- 使用矩阵、分段函数、复杂宏；
- 同时混用中文、公式和列表符号。

因此，README 中的公式编辑，agent MUST / SHOULD 遵循以下原则：

- 优先保证 GitHub 可稳定显示，而不是追求最标准的论文 LaTeX；
- 避免在 README 中使用脆弱宏，例如 `\operatorname{}`；
- 若 GitHub 数学渲染不稳定，优先改为 plain-text 公式块、代码样式公式，或图片；
- 全文符号必须保持一致，尤其是观测变量、状态变量、累加器、信号定义。

对本项目而言，额外要求如下：

- 观测变量统一记为 `x`，不得在 README 中混用 `y`；
- 若复杂推导难以稳定显示，优先用图示替代；
- 替换 README 图片时，优先覆盖原路径文件，而不是修改引用路径。

### 6.3 README 结构规范

调整 README 章节时，agent MUST 同时保证：

- 编号连续；
- 各节功能边界清晰，避免重复分层；
- 二级与三级标题能够形成清晰导航；
- 若章节被合并、删除或上移下移，需同步检查目录与后续编号。

README 资源文件统一放在：

```text
00_docs/readme_assets
```

## 7. 项目研究上下文

### 7.1 项目目标

本项目研究季度 GDP 发布滞后条件下的实时 nowcasting 问题。核心目标是利用更早发布的月频宏观变量与更高频的流动性指标，对中国 GDP 同比增速进行实时预测，并进一步检验该信号在债券市场中的解释力与交易价值。

### 7.2 当前主线结构

除非用户另行说明，agent MUST 将以下目录视为当前主线研究结构：

- `01_data_preparation`：数据准备；
- `02_rolling_window_nowcasting`：滚动窗口 nowcasting 主流程与汇总结果；
- `03_bond_backtest`：债券回测与信号检验；
- `outputs/pipeline_artifacts`：流程型中产物；
- `outputs/final_results`：最终结果汇总；
- `00_docs`：README 展示资源、参考文献与说明性文档。

### 7.3 当前主线 notebook

当前仓库中的主线 notebook 主要包括：

- `01_data_preparation/1 data-processing gdp&macro data.ipynb`
- `02_rolling_window_nowcasting/2005-2014窗口.ipynb`
- `02_rolling_window_nowcasting/2006-2015窗口.ipynb` 至 `2016-2025窗口.ipynb`
- `02_rolling_window_nowcasting/滚动窗口模型效果汇总.ipynb`
- `02_rolling_window_nowcasting/滚动窗口模型效果汇总（去疫情期）.ipynb`
- `03_bond_backtest/3 GDP_Bond_Backtest.ipynb`

agent SHOULD 先确认当前任务对应的是哪一条主线，再决定修改范围。

### 7.4 当前核心变量背景

当前研究围绕 GDP、PMI、工业增加值、用电量、社零、固定资产投资、CPI、PPI、流动性指标与失业率相关变量展开。变量处理与窗口设定可能因 notebook 不同而略有差异，因此 agent MUST 以当前目标 notebook 的实际实现为准，而不是假定存在单一全局变量文件。

### 7.5 默认诊断顺序

若用户未指定其他路径，模型复查优先级 SHOULD 为：

1. 先做训练期候选窗口诊断；
2. 再做单变量描述性统计、波动与季节性检查；
3. 再做变量间相关性与变量-GDP 领先滞后相关；
4. 再检查变量处理是否损失信息；
5. 最后才考虑调整处理逻辑或重跑模型。

每一步都 SHOULD 给出经济学含义判断，而 MUST NOT 只生成表格。

## 8. 当前编号基准

当前流程型中产物目录以 `outputs/pipeline_artifacts` 为基准，例如：

```text
outputs/pipeline_artifacts/1-训练期窗口诊断
```

额外约束如下；agent MUST 遵守：

- 所有涉及窗口或样本区间的输出文件名，SHOULD 使用可追踪的窗口标签；
- 后续新增诊断步骤编号应沿现有目录编号体系延续；
- 若发生前置插入，必须重新同步整条编号链。

## 9. agent 结束前检查清单

agent 在完成任务前，MUST 至少确认以下事项：

- notebook、文件夹与文档中的编号是否一致；
- 路径变动是否同步更新了 notebook 输出提示；
- README 中公式或图片是否可能在 GitHub 上渲染失效；
- 中产物是否被写入了错误目录；
- 修改是否保留了研究叙事与经济含义；
- 当前 AGENTS 约束是否仍与仓库实际结构一致。

## 10. 非目标事项

在无明确要求时，agent MUST NOT 默认执行以下动作：

- 将仓库改写成通用软件包风格 README；
- 只因文档任务就重跑完整模型；
- 训练窗口变化时自动重做变量筛选；
- 仅出于风格偏好随意改名文件或文件夹；
- 用纯技术表达覆盖项目原有的经济学解释。