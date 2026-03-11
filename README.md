# R-Scientific-Skills

R语言科学研究技能库，为医学研究和数据可视化提供完整的R语言解决方案。

## 项目简介

本项目提供两个核心技能，帮助研究人员在R语言环境中高效完成医学科学研究任务：

### 1. r-medical-research - 医学研究助手
R语言医学研究助手。自动识别研究主题并安装R包，进行数据处理、统计分析，制作发表级图表和表格，生成图注表注。适用于临床研究、流行病学、生物统计等医学数据分析场景。

**核心功能：**
- 研究主题自动识别与依赖包安装
- 数据清洗与预处理（缺失值处理、变量转换、倾向性评分匹配）
- 统计分析（描述性统计、差异分析、回归分析、生存分析、Meta分析）
- 发表级图表制作（散点图、箱线图、热图、森林图、生存曲线）
- 自动化表格生成（三线表、回归结果表）
- 图注与表注自动生成
- 批量报告导出

## 目录结构

```
R-scientific-skills/
├── .trae/
│   └── skills/
│       └── r-medical-research/
│           ├── SKILL.md          # 技能定义文件
│           └── SKILL.pdf          # 技能文档
├── r-scientific-plotting/         # R语言科研绘图（参考资料）
└── README.md                      # 本文件
```

## 技能使用场景

当您需要以下操作时，可调用本技能：

- 进行医学/临床数据分析
- 制作论文所需的统计表格（三线表）
- 进行流行病学数据分析
- 进行生存分析
- 进行Meta分析
- 进行倾向性评分匹配等因果推断分析
- 使用R语言进行数据可视化
- 制作符合期刊要求的发表级图表

## 技术栈

### 统计分析
- `tidyverse`, `dplyr`, `tidyr` - 数据处理
- `tableone`, `gtsummary` - 描述性统计与表格
- `rstatix`, `stats` - 差异分析
- `survival`, `survminer` - 生存分析
- `meta`, `metafor` - Meta分析
- `MatchIt`, `WeightIt` - 倾向性评分匹配

### 数据可视化
- `ggplot2` - 核心绑图
- `ggpubr`, `ggsci` - 高级绑图与配色
- `pheatmap`, `ComplexHeatmap` - 热图
- `forestplot` - 森林图
- `patchwork`, `cowplot` - 多图组合

### 报告导出
- `gt`, `flextable`, `officer` - 表格导出
- `rmarkdown` - 报告生成

## 快速开始

### 安装R包

```r
# 基础包
install.packages(c("tidyverse", "readxl", "openxlsx", "data.table"))

# 统计分析
install.packages(c("tableone", "gtsummary", "survival", "survminer", "autoReg"))

# 数据可视化
install.packages(c("ggplot2", "ggpubr", "ggsci", "pheatmap", "forestplot"))
```

### 基本使用

```r
# 1. 加载数据
library(readxl)
data <- read_excel("medical_data.xlsx")

# 2. 描述性统计
library(tableone)
table1 <- CreateTableOne(vars = c("age", "sex", "bmi"), data = data)
print(table1, smd = TRUE)

# 3. 绑图
library(ggplot2)
ggplot(data, aes(x = age, y = bmi, color = group)) +
  geom_point() +
  theme_classic()
```

## 参考资源

- [R for Data Science](https://r4ds.had.co.nz/)
- [ggplot2 官方文档](https://ggplot2.tidyverse.org/)
- [gtsummary 文档](https://www.danieldsjoberg.com/gtsummary/)
- [STHDA 统计绘图](http://www.sthda.com/english/)

## 贡献指南

欢迎提交Issue和Pull Request来改进本项目。

## 许可证

MIT License
