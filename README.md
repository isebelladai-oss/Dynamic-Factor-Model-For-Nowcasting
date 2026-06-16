# repo 结构说明

```text
repo/
├─ README.md
├─ AGENTS.md
├─ 00_docs/
├─ 01_data_pipeline/
├─ 02_rolling_window_nowcasting/
├─ 03_bond_backtest/
├─ archive/
├─ data/
│  ├─ raw/
│  ├─ processed/
│  └─ external_docs/
└─ outputs/
   ├─ pipeline_artifacts/
   └─ final_results/
```

## 当前约定

- `02_rolling_window_nowcasting/` 只保留主线：`01_mainline_interest19/`。
- 疫情处理、桥方程对照、因子设定、1995/2005 起点尝试，统一放到 `archive/rolling_window_nowcasting_experiments/`。
- 债券回测保留在 `03_bond_backtest/`。
- 数据输入放在 `data/`，流程中产物和最终汇总放在 `outputs/`。

## 重点对应

- 主线模型：`02_rolling_window_nowcasting/01_mainline_interest19/`
- 非主线 rolling window 对照：`archive/rolling_window_nowcasting_experiments/`
- 债券回测：`03_bond_backtest/`
- 路径迁移表：`00_docs/migration_map.csv` 与 `00_docs/migration_map.md`

## 状态说明

- 归档副本已经复制完成。
- `02_rolling_window_nowcasting/` 下非主线目录由于文件权限/占用问题，当前仍残留在原位；逻辑上以后请以 `archive/rolling_window_nowcasting_experiments/` 为准。
