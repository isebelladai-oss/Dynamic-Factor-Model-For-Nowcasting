# Dynamic Factor Model for China GDP Nowcasting and Bond Backtest

本项目围绕两条主线展开：

1. 用月频宏观变量构建中国 GDP nowcasting 动态因子模型。
2. 将 GDP nowcasting 信号映射到中债指数的下月收益，检验其在债券交易中的可用性。

当前仓库已经整理为可直接运行的 notebook 流程，主线结果集中在滚动窗口 nowcasting 和债券回测两部分。

## 1. 项目要回答的问题

本项目的核心问题有三个：

1. 只依赖月频宏观变量，能否在季度 GDP 公布前稳定地给出当期 GDP 同比预测？
2. 这种预测在样本内、样本外和不同信息时点下的精度如何？
3. GDP nowcasting 信号能否转化为可解释、可回测的债券收益预测或交易规则？

## 2. 主线 notebook

主线 notebook 及其作用如下：

| 阶段 | notebook | 作用 |
| --- | --- | --- |
| 数据准备 | `01_data_preparation/1 data-processing gdp&macro data.ipynb` | 处理宏观原始数据，生成统一月频底表 `宏观变量&GDP raw data.csv` |
| 单窗建模 | `02_rolling_window_nowcasting/2005-2014窗口.ipynb` | 以 `2005-01` 到 `2014-12` 为训练窗，完成变量准备、标准化、EM/QN 估计、训练期拟合和未来一年滚动 nowcast |
| 多窗滚动 | `02_rolling_window_nowcasting/*.ipynb` | 对不同 10 年训练窗口重复同样流程，形成滚动窗口结果 |
| 结果汇总 | `02_rolling_window_nowcasting/滚动窗口模型效果汇总.ipynb` | 汇总训练期拟合、smoother 重构和样本外 monthly nowcast，输出总表、年度表和图片 |
| 交易检验 | `03_bond_backtest/3 GDP_Bond_Backtest.ipynb` | 读取 nowcasting 结果，构造 GDP 信号并对中债指数下月收益做拟合、IC、回归和阈值交易分析 |

## 3. 模型与数据设计

### 3.1 宏观数据

主线 monthly panel 来自宏观原始数据、GDP 数据和利率数据，统一处理为月频序列。当前 notebook 中实际覆盖的候选变量包括：

- GDP 同比
- 制造业 PMI 水平值
- 服务业 PMI 水平值
- 工业增加值同比
- 社零同比
- 固投累计同比 / 固投当月同比
- CPI 同比
- PPI 同比
- 失业率差分
- 利率与流动性变量

不同训练窗口会按训练期覆盖率和缺失率重新筛选变量。当前 `2005-2014` 执行版训练窗中，最终保留的主线变量包含 GDP、PMI、工业增加值、社零、固投、CPI、PPI 和 `19` 日逆回购利率变量。

### 3.2 动态因子模型

主线模型是三因子动态因子模型：

- 因子数：3
- 因子含义：`macro`、`real`、`nominal`
- 因子动态：每个因子使用 AR(6) 块结构
- 估计流程：先用 EM 拿到稳定初值，再用 QN 对 AR 参数做细化
- 目标输出：
  - 训练期 monthly fit
  - smoother 重构的月度 GDP
  - 样本外未来一年 monthly nowcast

### 3.3 滚动窗口设计

主线采用固定长度 10 年训练窗的滚动做法。仓库中保留了 `2005-2014` 至 `2015-2024` 的逐窗 notebook。当前已经汇总进结果表的窗口为：

- `2005-2014`
- `2006-2015`
- `2008-2017`
- `2009-2018`
- `2010-2019`
- `2011-2020`
- `2012-2021`
- `2013-2022`
- `2014-2023`

对应样本外 monthly nowcast 覆盖 `2015-01` 到 `2024-12`，共 `108` 个月。

## 4. 流程总览

完整主线可以概括为四步：

1. 用 `01_data_preparation` notebook 把原始 Excel / CSV 清洗成统一月频底表。
2. 对每个 10 年训练窗口执行变量准备、缩尾、季调、标准化、EM/QN 估计和 nowcast。
3. 用 `滚动窗口模型效果汇总.ipynb` 把不同窗口的训练期拟合、smoother 和样本外预测拼接成统一结果。
4. 用 `3 GDP_Bond_Backtest.ipynb` 将 GDP 预测信号映射到中债指数收益，做拟合、方向、IC、回归和阈值交易分析。

主要输出路径如下：

- `outputs/pipeline_artifacts/`：每个训练窗的中间产物
- `02_rolling_window_nowcasting/汇总结果/`：滚动窗口 nowcasting 汇总
- `outputs/final_results/Rolling Window Nowcasting GDP result.csv`：债券回测读取的 GDP 预测源
- `03_bond_backtest/汇总结果/`：债券回测所有章节输出

## 5. 现在得到的核心结果

### 5.1 Nowcasting 精度

滚动窗口汇总结果来自 `02_rolling_window_nowcasting/汇总结果/`。

#### 训练期 monthly fit

| 口径 | n_obs | MAE | RMSE | Corr |
| --- | ---: | ---: | ---: | ---: |
| Filter monthly fit | 1080 | 0.392 | 0.541 | 0.966 |
| Smoother monthly reconstruction | 1080 | 0.211 | 0.297 | 0.990 |

这说明模型在训练期内能够较好地抓住 GDP 与月频宏观变量之间的共振关系，尤其 smoother 重构表现明显好于 filter 口径。

#### 样本外 monthly nowcast

| 口径 | n_obs | MAE | RMSE | Corr | Mean Error |
| --- | ---: | ---: | ---: | ---: | ---: |
| Out-of-sample monthly nowcast | 108 | 1.440 | 3.095 | 0.452 | 0.149 |

这部分已经明显比训练期拟合困难，说明滚动窗口 nowcast 在真实样本外环境下会受到结构变化和异常期冲击。

#### 按距季度末月份拆分

| 距季度末月份 | n_obs | MAE | RMSE | Corr |
| --- | ---: | ---: | ---: | ---: |
| `0` 个月 | 36 | 0.974 | 2.038 | 0.822 |
| `1` 个月 | 36 | 1.493 | 3.254 | 0.356 |
| `2` 个月 | 36 | 1.852 | 3.740 | 0.264 |

一个很清楚的结论是：越接近季度末，预测质量越高。也就是说，这套 nowcasting 更适合作为“随着信息到来不断更新”的实时跟踪工具，而不是太早给出远距离预测。

#### 疫情前后对比

| 时段 | n_obs | MAE | RMSE | Corr | Mean Error |
| --- | ---: | ---: | ---: | ---: | ---: |
| `2015-2019` | 48 | 0.234 | 0.290 | 0.802 | 0.034 |
| `2020-2024` | 60 | 2.405 | 4.144 | 0.401 | 0.242 |

这说明样本外误差的主要来源集中在疫情及其后的结构性冲击阶段。README 里的主要判断也应据此理解：这套模型在常态宏观环境下更有效，在异常期会显著失真。

### 5.2 债券回测结果

债券回测结果来自 `03_bond_backtest/汇总结果/`，样本同样覆盖 `2015-01` 到 `2024-12`。

#### GDP 预测值对真实 GDP 的拟合

在剔除疫情季度后的样本中：

| 样本 | n_obs | MAE | RMSE | R2 | Pearson Corr |
| --- | ---: | ---: | ---: | ---: | ---: |
| 全部月份 | 96 | 0.820 | 1.608 | -0.789 | 0.715 |
| 季度第一个月 | 32 | 1.160 | 2.192 | -2.326 | 0.678 |
| 季度第二个月 | 32 | 0.783 | 1.467 | -0.489 | 0.738 |
| 季度第三个月 | 32 | 0.516 | 0.893 | 0.448 | 0.830 |

这里和 nowcasting 汇总的结论一致：季度第三个月最有信息量，越接近季度 GDP 披露时点，信号越可信。

#### 方向准确率

当 GDP 信号定义为“当前预测相对上一期真实 GDP 的变化”时：

- 全样本方向准确率：`81.1%`
- 季度第一个月：`60.0%`
- 季度第二个月：`83.3%`
- 季度第三个月：`95.0%`

如果改成“相对前两期真实 GDP 均值”的方案，全样本方向准确率是 `80.0%`。两种方案差异不大，但 `lag1` 方案在季度第三个月最强。

#### IC 结果

对 `12` 个中债指数收益序列，nowcasting 信号的 IC 全部为负，说明“增长越强，下一月债券收益越弱”的方向在样本内是一致的。

代表性区间如下：

- `3-5` 年期限桶：IC 大致在 `-0.114` 到 `-0.132`
- `5-7` 年期限桶：IC 大致在 `-0.086` 到 `-0.094`

这意味着：

1. 方向上是对的，GDP 走强通常不利于债券下月收益。
2. 短久期桶（`3-5` 年）对 GDP 信号更敏感。
3. 但从 `p-value` 看，nowcasting 信号的横截面线性解释力还不够强，更多体现为“弱但一致”的方向性信息。

#### 时间序列回归

在加入宏观与资金面控制变量后的 `Model3_gdp_plus_macro_liquidity` 回归中：

- `actual_gdp_signal` 对 `12` 条债券序列都给出显著负 beta，`p_value` 约在 `0.009` 到 `0.019`
- `nowcasting_gdp_signal` 的 beta 方向同样为负，但全部不显著，`p_value` 大多在 `0.52` 到 `0.69`

这说明一个关键事实：

1. 债券市场对“真实已实现 GDP”变化的反应是明显的。
2. nowcasting 信号在控制其他宏观和流动性变量后，仍然保留方向正确性，但强度还不足以成为稳定的线性定价因子。

#### 阈值交易

最佳阈值交易表显示，真正有效的交易规则主要是“增长明显走弱时做多债券”，而不是“增长明显走强时做空债券”。

以 `nowcasting_gdp_signal` 为例：

| 久期桶 | 交易方向 | 阈值 | 交易月数 | 胜率 | 单月交易平均收益 | 年化 Sharpe | 累计收益 | 最大回撤 |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `3-5` 年 | 增长明显走弱时做多债券 | 0.1 | 47 | 68.8% | 0.218% | 0.727 | 11.15% | -2.39% |
| `5-7` 年 | 增长明显走弱时做多债券 | 0.1 | 47 | 65.2% | 0.275% | 0.751 | 14.01% | -2.65% |
| `3-5` 年 | 增长明显走强时做空债券 | 2.3 | 15 | 26.7% | -0.080% | -0.149 | -1.20% | -3.88% |
| `5-7` 年 | 增长明显走强时做空债券 | 2.3 | 15 | 28.9% | -0.159% | -0.266 | -2.36% | -4.98% |

如果用 `actual_gdp_signal` 作为对照基准，做多弱增长债券的表现更强：

- `3-5` 年：年化 Sharpe `1.008`，累计收益 `15.03%`
- `5-7` 年：年化 Sharpe `1.088`，累计收益 `20.92%`

因此，当前 nowcasting 信号更适合做“弱增长防御”筛选，而不适合直接做“强增长做空”的单边策略。

## 6. 现阶段可以下的结论

基于当前主线 notebook 和已生成结果，可以给出四条比较稳的结论：

1. 三因子动态因子模型在训练期内能稳定拟合 GDP 与宏观变量的共同波动，smoother 重构质量很高。
2. 样本外 nowcasting 在常态时期有效，但在疫情和后疫情阶段误差显著放大。
3. nowcasting 信号在债券端具有正确的经济方向：增长强通常压制下一月债券收益，且 `3-5` 年品种敏感度更高。
4. 交易上最可用的是“增长明显走弱时做多债券”，而不是“增长明显走强时做空债券”；真实 GDP 信号强于 nowcasting 信号，说明当前 nowcasting 更适合作为实时预警或风格筛选，而不是单独替代真实基本面。

## 7. 如何复现

推荐执行顺序如下：

1. 运行 `01_data_preparation/1 data-processing gdp&macro data.ipynb`
2. 运行 `02_rolling_window_nowcasting/2005-2014窗口.ipynb` 以及其他滚动窗口 notebook
3. 运行 `02_rolling_window_nowcasting/滚动窗口模型效果汇总.ipynb`
4. 运行 `03_bond_backtest/3 GDP_Bond_Backtest.ipynb`

如果只想直接使用现成结果，优先看以下文件：

- `02_rolling_window_nowcasting/汇总结果/nowcast_monthly_overall_metrics.csv`
- `02_rolling_window_nowcasting/汇总结果/nowcast_monthly_by_horizon_0_2_metrics.csv`
- `outputs/final_results/Rolling Window Nowcasting GDP result.csv`
- `03_bond_backtest/汇总结果/02-拟合优度/02_gdp_fit_metrics.csv`
- `03_bond_backtest/汇总结果/03-方向准确率/03_gdp_direction_accuracy.csv`
- `03_bond_backtest/汇总结果/04-IC分析/04_nowcasting_lag1_actual_ic_summary_by_bond.csv`
- `03_bond_backtest/汇总结果/05-时间序列回归/05_ts_regression_results_by_bond_lag1_actual.csv`
- `03_bond_backtest/汇总结果/08-阈值交易探索/08_threshold_strategy_best_by_duration_lag1_actual.csv`

## 8. 备注

更细的文件说明、历史试验分支和路径说明见：

- `AGENTS.md`
- `00_docs/项目文件说明.md`

如果后续继续扩展 README，建议优先补两类内容：

1. 不同训练窗的参数稳定性与因子载荷稳定性。
2. 疫情期与后疫情期的分段误差归因。
