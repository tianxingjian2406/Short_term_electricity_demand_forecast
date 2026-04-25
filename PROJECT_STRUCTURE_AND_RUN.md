# 项目文件说明、环境依赖与运行方式

本文档用于说明电力负荷预测项目上传 GitHub 时的文件组成、运行环境、依赖安装方式和推荐项目结构。

---

## 1. 文件说明

| 文件/目录 | 是否必须 | 说明 |
|---|---|---|
| `README.md` | 必须 | GitHub 项目主页说明文件，介绍项目背景、方法流程、模型、指标和结果说明。 |
| `electricityD_final_annotated.ipynb` | 必须 | 最终版 Jupyter Notebook，包含完整的数据读取、特征工程、模型训练、融合预测、评价指标和可视化分析流程。 |
| `requirements.txt` | 必须 | Python 环境依赖文件，用于一键安装项目所需第三方库。 |
| `continuous dataset.csv` | 视情况 | 原始电力负荷数据文件。若数据不能公开上传，可不放入仓库，但需要在 README 中说明数据来源、字段格式和获取方式。 |
| `model_comparison_results.csv` | 运行后生成 | 各模型在验证集和测试集上的 MAE、RMSE、MAPE 结果汇总。 |
| `test_predictions.csv` | 运行后生成 | 测试集真实值以及各模型预测值，用于后续画图、误差分析或复核结果。 |
| `.gitignore` | 推荐 | 用于排除虚拟环境、缓存文件、大型数据文件和运行中间文件。 |

---

## 2. 环境依赖

建议使用 Python 3.9 或以上版本。推荐在独立虚拟环境中运行，避免与本机其他项目依赖冲突。

### 2.1 核心依赖

项目核心依赖包括：

```text
numpy
pandas
matplotlib
scikit-learn
```

这些库用于数据处理、特征工程、模型训练、评价指标计算和基础可视化。

### 2.2 可选模型依赖

notebook 中还包含以下模型依赖：

```text
xgboost
lightgbm
catboost
tensorflow
```

其中：

- `xgboost` 用于训练 XGBoost 回归模型；
- `lightgbm` 用于训练 LightGBM 回归模型；
- `catboost` 用于训练 CatBoost 回归模型；
- `tensorflow` 用于构建和训练 LSTM 模型。

如果本地没有安装某些可选依赖，notebook 会自动跳过对应模型。但如果希望完整复现实验结果，建议安装 `requirements.txt` 中的全部依赖。

---

## 3. 安装方式

### 3.1 克隆项目

```bash
git clone <your-repository-url>
cd <your-repository-name>
```

### 3.2 创建虚拟环境

使用 `venv`：

```bash
python -m venv .venv
```

激活虚拟环境：

Windows：

```bash
.venv\Scripts\activate
```

macOS / Linux：

```bash
source .venv/bin/activate
```

### 3.3 安装依赖

```bash
pip install -r requirements.txt
```

如果 TensorFlow 在本机安装失败，可以先安装除 TensorFlow 外的依赖，并在 notebook 中跳过 LSTM 部分：

```bash
pip install numpy pandas matplotlib scikit-learn xgboost lightgbm catboost jupyter ipykernel
```

---

## 4. 运行方式

### 4.1 准备数据文件

默认情况下，notebook 会读取项目根目录下的：

```text
continuous dataset.csv
```

数据文件至少应包含：

| 字段名 | 说明 |
|---|---|
| `datetime` | 时间字段，用于时间解析、排序、划分训练集/验证集/测试集和构造时间特征。 |
| `nat_demand` | 电力负荷需求字段，作为预测目标变量。 |

如果数据文件名或字段名不同，需要在 notebook 的全局配置部分修改：

```python
DATA_PATH = Path("continuous dataset.csv")
DATETIME_COL = "datetime"
TARGET_COL = "nat_demand"
```

### 4.2 启动 Jupyter Notebook

```bash
jupyter notebook electricityD_final_annotated.ipynb
```

或者使用 JupyterLab：

```bash
jupyter lab
```

打开 notebook 后，按照单元格顺序从上到下运行即可。

### 4.3 运行完成后查看结果

运行完成后，项目会生成：

```text
model_comparison_results.csv
test_predictions.csv
```

其中：

- `model_comparison_results.csv` 用于比较 Baseline、Ridge、XGBoost、LightGBM、CatBoost、LSTM 和 Weighted Ensemble 的验证集/测试集表现；
- `test_predictions.csv` 用于保存测试集真实值和各模型预测值，可用于进一步画图或误差复核。

---

## 5. 运行注意事项

1. 时间序列预测不能随机打乱数据后再划分训练集和测试集，否则会产生数据泄漏。
2. 所有由目标变量构造的滞后、滚动统计和差分特征，都必须只使用历史信息。
3. 验证集用于模型筛选和融合权重确定，测试集只用于最终泛化能力评估。
4. 如果 TensorFlow 或 LSTM 运行较慢，可以只运行传统机器学习模型，不影响整体项目流程。
5. 最终模型选择应优先参考测试集 RMSE，同时结合 MAE 和 MAPE 综合判断。
