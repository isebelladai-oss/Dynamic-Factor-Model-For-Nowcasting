# Rolling Window Notebook Fix Template

本文档提炼自 `02_rolling_window_nowcasting/2005-2014_处理疫情_1-17.ipynb` 已验证跑通的修法，用于后续同步到同结构 notebook。

## 1. 已验证修法

### 1.1 项目根目录识别

- 不再依赖 `AGENTS.md` 所在层作为 `PROJECT_ROOT`。
- 改为从 `Path.cwd()` 向上搜索，优先找到同时包含 `data/` 和 `outputs/` 的目录。

模板：

```python
PROJECT_ROOT = Path.cwd().resolve()
for candidate in [PROJECT_ROOT, *PROJECT_ROOT.parents]:
    if (candidate / "data").exists() and (candidate / "outputs").exists():
        PROJECT_ROOT = candidate
        break
```

适用原因：
- 主线 notebook 运行目录是 `动态因子模型/02_rolling_window_nowcasting`。
- 外层旧项目目录也存在 `AGENTS.md`，直接按 `AGENTS.md` 回溯会把根目录识别错到外层。

### 1.2 新框架输入输出路径

- 统一使用新主线框架。
- `processed` 输入输出保留在 `data/processed/processed`。
- 流程中产物统一放到 `outputs/pipeline_artifacts`。
- 汇总结果统一放到 `outputs/final_results`。

模板：

```python
DATA_DIR = PROJECT_ROOT / "data"
RAW_DATA_DIR = DATA_DIR / "raw"
PROCESSED_DIR = DATA_DIR / "processed" / "processed"
DATA_PATH = PROCESSED_DIR / "baseline_model_input.csv"
OUTPUT_DIR = PROCESSED_DIR
INTERMEDIATE_DIR = PROJECT_ROOT / "outputs" / "pipeline_artifacts"
RAW_MACRO_DATA_PATH = OUTPUT_DIR / "宏观变量&GDP raw data.csv"
RATE_DATA_PATH = RAW_DATA_DIR / "1980年至今数据" / "利率相关数据.csv"
```

### 1.3 固投字段口径

- 不补列，不手工填数据。
- 直接按新版 `宏观变量&GDP raw data.csv` 中已经存在的字段读取。

已确认字段：

```text
中国:固定资产投资完成额:累计同比
中国:固定资产投资完成额:固投当月同比
```

模板：

```python
RAW_FAI_CUM_YOY_COL = "中国:固定资产投资完成额:累计同比"
RAW_FAI_COL = "中国:固定资产投资完成额:固投当月同比"
RAW_MACRO_MERGE_COLS = RAW_PMI_LEVEL_COLS + [RAW_FAI_CUM_YOY_COL, RAW_FAI_COL]
```

### 1.4 利率原始字段名

- raw 文件真实列名不带 `中国:` 前缀。
- 月度转换后的内部变量名仍保留原有命名，不改下游口径。

已确认 raw 列名：

```text
逆回购利率:7天
银行间质押式回购加权利率:7天
```

保留的月度派生列名：

```text
中国:逆回购利率:7天:当月19日值
中国:银行间质押式回购加权利率:7天:上月20日至当月19日均值
```

模板：

```python
RAW_RATE_REPO_7D_COL = "逆回购利率:7天"
RAW_RATE_IBO7_COL = "银行间质押式回购加权利率:7天"
RAW_RATE_REPO_7D_MONTHLY_COL = "中国:逆回购利率:7天:当月19日值"
RAW_RATE_IBO7_MONTHLY_COL = "中国:银行间质押式回购加权利率:7天:上月20日至当月19日均值"
```

### 1.5 利率文件清洗规则

- 兼容 `utf-8-sig` 与 `gb18030`。
- 去掉表头下方的元信息行。
- 以日频 forward-fill 后，构造 19 号信息集月频值。

模板：

```python
try:
    rate_raw = pd.read_csv(rate_path, encoding="utf-8-sig")
except UnicodeDecodeError:
    rate_raw = pd.read_csv(rate_path, encoding="gb18030")

date_col = rate_raw.columns[0]
metadata_labels = {"频率", "单位", "指标ID", "来源"}
rate_raw = rate_raw.loc[~rate_raw[date_col].isin(metadata_labels)].copy()
```

## 2. 已验证结论

- `2005-2014_处理疫情_1-17.ipynb` 按上述修法已实际执行通过。
- 本次修法没有手工补数据，没有伪造字段，没有改动业务口径。
- 最终依赖的仍是新框架下已有文件：
  - `data/processed/processed/宏观变量&GDP raw data.csv`
  - `data/processed/processed/baseline_model_input.csv`
  - `data/raw/1980年至今数据/利率相关数据.csv`

## 3. 是否应同步

### 3.1 应同步到其他 rolling window notebook

应同步的原因：

- 这些 notebook 结构相同。
- 使用同一套输入路径、同一套 raw 利率文件、同一套中产物目录约定。
- 只改 `2005-2014` 会导致其他窗口继续出现：
  - 根目录识别错层；
  - 利率 raw 列名不匹配；
  - 中产物仍写旧目录。

应同步的修法：

- 根目录识别逻辑。
- `data/processed/processed` 与 `outputs/pipeline_artifacts` 路径。
- 利率 raw 列名。
- 固投字段直接按新版 raw panel 标准字段读取。

### 3.2 不应整套同步到汇总 notebook

`滚动窗口模型效果汇总.ipynb` 只需要同步路径相关修法，不需要同步变量准备修法。

原因：

- 汇总 notebook 不读取 raw 宏观字段，不做 PMI/固投/利率日频转月频。
- 它读取的是各窗口已经生成好的中产物和最终回测结果。

对汇总 notebook 只需要保留：

- 正确的 `PROJECT_ROOT` 识别。
- `INTERMEDIATE_DIR = PROJECT_ROOT / "outputs" / "pipeline_artifacts"`。
- `SUMMARY_DIR = PROJECT_ROOT / "outputs" / "final_results" / ...`。

不需要同步到汇总 notebook 的内容：

- `RAW_FAI_*` 字段修法。
- `RAW_RATE_*` 原始字段修法。
- `load_interest_monthly_panel()` 逻辑。
- PMI 水平值回填逻辑。
