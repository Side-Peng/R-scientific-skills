---
name: "r-medical-research"
description: "R语言医学研究助手。自动识别研究主题并安装R包，进行数据处理、统计分析，制作发表级图表和表格，生成图注表注。适用于临床研究、流行病学、生物统计等医学数据分析场景。"
---

# R语言医学研究技能

## 概述

本技能提供R语言医学科学研究的完整工作流程指导，包括研究主题识别、依赖包自动安装、数据处理、统计分析、发表级图表制作及图注表注生成。

## 触发条件

当用户提出以下需求时，应调用此技能：
- 进行医学/临床数据分析
- 制作论文所需的统计表格（三线表）
- 进行流行病学数据分析
- 进行生存分析
- 进行meta分析
- 进行倾向性评分匹配等因果推断分析
- 询问如何用R进行医学统计
- 需要自动安装相关R包完成特定医学分析任务

---

## 第一部分：研究主题识别与包安装

### 1.1 自动识别研究类型

根据用户描述或数据特征，识别研究类型并安装相应包：

```r
# 基础包安装（通用）
install.packages(c("tidyverse", "readxl", "openxlsx", "data.table"))

# 描述性统计与数据清洗
install.packages(c("dplyr", "tidyr", "stringr", "lubridate", "Hmisc", "mice"))

# 统计分析
install.packages(c("stats", "rstatix", "car", "lmtest", "aod"))

# 高级统计分析
install.packages(c("survival", "survminer", "glm", "lme4", "nlme", "meta", "metafor"))

# 临床研究专用
install.packages(c("tableone", "autoReg", "moonBook", "rrtable", "MatchIt", "WeightIt", "cobalt"))

# 机器学习与建模
install.packages(c("caret", "randomForest", "e1071", "glmnet", "pROC", "rms"))

# 表格制作与导出
install.packages(c("gt", "gtsummary", "flextable", "officer", "ftExtra"))

# 可视化与出版
install.packages(c("ggplot2", "ggpubr", "ggsci", "patchwork", "cowplot", "survminer"))
```

### 1.2 按研究类型的包推荐

| 研究类型 | 核心R包 | 辅助包 |
|---------|---------|--------|
| 描述性分析 | dplyr, Hmisc, tableone | skimr, DataExplorer |
| 差异分析 | rstatix, stats, car | ggpubr, rcompanion |
| 回归分析 | stats, glm, lm, survival | autoReg, gtsummary |
| 生存分析 | survival, survminer | survivalROC, riskRegression |
| Meta分析 | meta, metafor, robumeta | metaBMA, netmeta |
| 因果推断 | MatchIt, WeightIt, cobalt | causalweight, causalinferencelab |
| 诊断试验 | pROC, ROCR, cutpointr | OptimalCutpoints, DiagTest |
| 列线图 | rms, nomogramEx | survival, ggplot2 |
| 机器学习 | caret, randomForest, glmnet | recipes, tune, yardstick |

---

## 第二部分：数据类型识别与处理

### 2.1 常见医学数据类型

```r
# 查看数据结构
str(data)
glimpse(data)
skimr::skim(data)

# 识别变量类型
# 连续变量: 年龄、血压、血糖、实验室指标
# 分类变量: 性别、疾病分期、治疗组
# 有序变量: 病情严重程度、教育水平
# 时间变量: 随访时间、生存时间
# 事件变量: 死亡、复发、进展
```

### 2.2 数据读取

```r
library(readxl)
library(openxlsx)

# 读取Excel
data <- read_excel("data.xlsx", sheet = 1)

# 读取CSV
data <- read.csv("data.csv", stringsAsFactors = FALSE)

# 读取SPSS/SAS/Stata
library(haven)
data <- read_spss("data.sav")
data <- read_stata("data.dta")
```

### 2.3 数据清洗与预处理

```r
library(dplyr)
library(tidyr)

# 缺失值处理
data <- data %>% drop_na()  # 删除含缺失值行
data <- data %>% na_replace(0)  # 用0填充
data <- data %>% fill(group, .direction = "down")  # 用前值填充

# 多重插补
library(mice)
imputed <- mice(data, m = 5, method = "pmm")
data <- complete(imputed, 1)

# 变量转换
data <- data %>%
  mutate(
    age_group = cut(age, breaks = c(0, 40, 60, 100), 
                    labels = c("青年", "中年", "老年")),
    bmi_cat = ifelse(bmi < 18.5, "偏瘦", 
              ifelse(bmi < 24, "正常", 
              ifelse(bmi < 28, "偏胖", "肥胖"))),
    log_alt = log(ALT + 1)
  )

# 数据标准化
data <- data %>%
  mutate(across(where(is.numeric), scale))
```

### 2.4 倾向性评分匹配

```r
library(MatchIt)
library(cobalt)

# 1:1匹配
psm_result <- matchit(treatment ~ age + bmi + hypertension + diabetes,
                      data = data,
                      method = "nearest",
                      ratio = 1)

matched_data <- match.data(psm_result)

# 检查匹配平衡性
love.plot(psm_result, binary = "std")
```

---

## 第三部分：统计分析方法

### 3.1 描述性统计

```r
library(tableone)
library(gtsummary)

# 创建Table 1 (基线特征表)
vars <- c("age", "sex", "bmi", "sbp", "dbp", "glucose", "treatment")
factorVars <- c("sex", "treatment", "hypertension", "diabetes")

table1 <- CreateTableOne(vars = vars, 
                         data = data,
                         factorVars = factorVars,
                         strata = "group",
                         test = TRUE)
print(table1, smd = TRUE)

# 使用gtsummary创建更美观的表格
data %>% 
  tbl_summary(by = "group", 
              statistic = list(all_continuous() ~ "{mean} ± {sd}",
                              all_categorical() ~ "{n} ({p})")) %>%
  add_p() %>%
  add_difference() %>%
  bold_labels()
```

### 3.2 差异分析

```r
library(rstatix)

# 连续变量：t检验
data %>% t_test(value ~ group, data = .)

# 连续变量：Wilcoxon秩和检验
data %>% wilcox_test(value ~ group, data = .)

# 连续变量：方差分析
data %>% anova_test(value ~ group)

# 分类变量：卡方检验
chisq.test(table(data$group, data$outcome))

# 配对样本t检验
data %>% t_test(value ~ time, paired = TRUE, data = .)

# 多重比较校正
p_values <- c(0.001, 0.01, 0.03, 0.04)
p.adjust(p_values, method = "bonferroni")
p.adjust(p_values, method = "fdr")
```

### 3.3 回归分析

```r
# 线性回归
model_lm <- lm(outcome ~ age + bmi + treatment, data = data)
summary(model_lm)
gtsummary::tbl_regression(model_lm)

# Logistic回归
model_logit <- glm(outcome ~ age + bmi + treatment, 
                   data = data, 
                   family = binomial())
summary(model_logit)
exp(confint(model_logit))  # OR值及置信区间

# 使用autoReg自动生成回归表格
library(autoReg)
autoReg(outcome ~ age + bmi + treatment, data = data) %>%
  myft()

# Cox比例风险模型
library(survival)
model_cox <- coxph(Surv(time, event) ~ age + bmi + treatment, data = data)
summary(model_cox)
```

### 3.4 生存分析

```r
library(survival)
library(survminer)

# Kaplan-Meier生存曲线
fit <- survfit(Surv(time, event) ~ group, data = data)
ggsurvplot(fit, data = data,
           pval = TRUE,
           conf.int = TRUE,
           risk.table = TRUE,
           ncensor.plot = TRUE,
           palette = c("#E64B35", "#4DBBD5"))

# Log-rank检验
survdiff(Surv(time, event) ~ group, data = data)

# Cox回归
cox_model <- coxph(Surv(time, event) ~ group + age + bmi, data = data)
ggforest(cox_model, data = data)

# 竞争风险模型
library(cmprsk)
crr_model <- crr(ftime, fstatus, cov1, failcode = 1)
```

---

## 第四部分：发表级图表制作

### 4.1 基础图表模板

```r
library(ggplot2)
library(ggpubr)
library(ggsci)

# 论文主题设置
theme_pub <- function() {
  theme_classic() +
    theme(
      axis.text = element_text(size = 10, color = "black"),
      axis.title = element_text(size = 12, face = "bold"),
      legend.text = element_text(size = 10),
      legend.title = element_text(size = 11, face = "bold"),
      plot.title = element_text(size = 14, face = "bold", hjust = 0.5),
      panel.grid.major = element_blank(),
      panel.grid.minor = element_blank()
    )
}
```

### 4.2 常用图表类型

#### 散点图与回归图
```r
ggplot(data, aes(x = age, y = bmi, color = group)) +
  geom_point(alpha = 0.6, size = 2) +
  geom_smooth(method = "lm", se = TRUE) +
  stat_cor(method = "pearson", aes(color = group)) +
  scale_color_npg() +
  theme_pub() +
  labs(x = "年龄 (岁)", y = "BMI (kg/m²)", 
       title = "年龄与BMI的相关性")
```

#### 箱线图/小提琴图
```r
ggplot(data, aes(x = group, y = value, fill = group)) +
  geom_violin(alpha = 0.6, trim = FALSE) +
  geom_boxplot(width = 0.15, fill = "white") +
  stat_compare_means(method = "t.test", label = "p.format") +
  scale_fill_npg() +
  theme_pub() +
  labs(x = "分组", y = "指标值")
```

#### 生存曲线
```r
ggsurvplot(survfit(Surv(time, event) ~ group, data = data),
           data = data,
           pval = TRUE,
           conf.int = TRUE,
           risk.table = TRUE,
           palette = c("#E64B35", "#4DBBD5"),
           legend.title = "治疗组",
           xlab = "随访时间 (月)",
           ylab = "生存概率")
```

#### 森林图
```r
library(forestplot)
forestplot::forestplot(
  labeltext = as.matrix(table_data[, 1:3]),
  mean = table_data$OR,
  lower = table_data$lower,
  upper = table_data$upper,
  hrzl_lines = TRUE,
  col = fpColors(box = "#4DBBD5", line = "black"),
  xlab = "Odds Ratio (95% CI)"
)
```

#### 热图
```r
library(pheatmap)
pheatmap(mat = expression_matrix,
         scale = "row",
         cluster_rows = TRUE,
         cluster_cols = TRUE,
         show_rownames = TRUE,
         show_colnames = TRUE,
         color = colorRampPalette(c("#2166AC", "#F7F7F7", "#B2182B"))(50),
         border_color = NA)
```

### 4.3 多图组合

```r
library(patchwork)

(p1 + p2 + p3) / (p4 + p5) +
  plot_annotation(tag_levels = "A") &
  theme(plot.tag = element_text(face = "bold"))
```

### 4.4 高质量导出

```r
# 导出为TIFF (期刊要求)
ggsave("Figure1.tiff", 
       plot = last_plot(),
       width = 8, height = 6, units = "cm",
       dpi = 300, compression = "lzw")

# 导出为PDF (矢量图)
ggsave("Figure1.pdf",
       plot = last_plot(),
       width = 8, height = 6, units = "cm")

# 批量导出
for (i in 1:3) {
  ggsave(paste0("Figure", i, ".tiff"),
         plot = plots[[i]],
         width = 8, height = 6, units = "cm", dpi = 300)
}
```

---

## 第五部分：发表级表格制作

### 5.1 三线表 (Table 1 - 基线特征表)

```r
library(tableone)
library(officer)
library(flextable)

# 创建基线特征表
vars <- c("age", "sex", "bmi", "sbp", "glucose", "outcome")
factorVars <- c("sex", "outcome")

table1 <- CreateTableOne(vars = vars, 
                         data = data,
                         strata = "group",
                         factorVars = factorVars,
                         test = TRUE)

# 导出为Word
print(table1, smd = TRUE, quote = FALSE, noSpaces = TRUE)

# 使用gtsummary创建更美观的表格
baseline_table <- data %>%
  tbl_summary(
    by = "group",
    statistic = list(
      all_continuous() ~ "{mean} ± {sd}",
      all_categorical() ~ "{n} ({p}%)"
    ),
    label = list(
      age ~ "年龄 (岁)",
      sex ~ "性别 (男)",
      bmi ~ "BMI (kg/m²)"
    )
  ) %>%
  add_p() %>%
  add_n() %>%
  bold_labels() %>%
  italicize_levels()

# 导出
as_flex_table(baseline_table) %>%
  save_as_docx(path = "Table1.docx")
```

### 5.2 回归分析结果表

```r
library(gtsummary)

# Logistic回归表格
data %>%
  mutate(outcome = factor(outcome)) %>%
  tbl_uvregression(
    method = glm,
    y = outcome,
    method.args = list(family = binomial),
    exponentiate = TRUE,
    pvalue_fun = function(x) style_pvalue(x, digits = 3)
  ) %>%
  add_global_p() %>%
  bold_p() %>%
  inline_text("variable", row = 1, column = "estimate")

# 导出到Word
tbl_regression(model_logit, exponentiate = TRUE) %>%
  as_flex_table() %>%
  save_as_docx(path = "Table2.docx")
```

### 5.3 亚组分析表

```r
library(metafor)

# 创建亚组分析表
subgroup_results <- data %>%
  group_by(subgroup) %>%
  summarise(
    n = n(),
    events = sum(outcome),
    or = NA,
    lower = NA,
    upper = NA
  )

for (i in 1:nrow(subgroup_results)) {
  subset_data <- data %>% filter(subgroup == subgroup_results$subgroup[i])
  model <- glm(outcome ~ treatment, data = subset_data, family = binomial)
  or <- exp(coef(model)["treatment"])
  ci <- exp(confint(model)["treatment", ])
  subgroup_results$or[i] <- or
  subgroup_results$lower[i] <- ci[1]
  subgroup_results$upper[i] <- ci[2]
}
```

---

## 第六部分：图注与表注生成

### 6.1 自动生成图注

```r
generate_figure_caption <- function(fig_num, type, description, data_info) {
  caption <- paste0(
    "图", fig_num, " ", description, "。",
    "A: ", data_info$group_a, "; B: ", data_info$group_b, "。",
    "数据以均值±标准误表示。",
    "组间比较采用", data_info$test_method, "检验。"
  )
  return(caption)
}

# 示例
fig1_caption <- generate_figure_caption(
  fig_num = "1",
  type = "boxplot",
  description = "两组患者血糖水平比较",
  data_info = list(
    group_a = "治疗组 (n=100)",
    group_b = "对照组 (n=100)",
    test_method = "t"
  )
)
cat(fig1_caption)
# 输出: 图1 两组患者血糖水平比较。 A: 治疗组 (n=100); B: 对照组 (n=100)。 数据以均值±标准误表示。 组间比较采用t检验。
```

### 6.2 自动生成表注

```r
generate_table_notes <- function(table_num, n, events, methods, software) {
  notes <- paste0(
    "表", table_num, " 注: ",
    "数据以均数±标准差或频数(%)表示。",
    "P值由", methods, "计算。",
    "统计分析使用R软件 (", software, ")完成。",
    "n=", n, "; 事件数=", events, "。"
  )
  return(notes)
}

# 示例
table1_notes <- generate_table_notes(
  table_num = "1",
  n = 200,
  events = 45,
  methods = "t检验或卡方检验",
  software = "version 4.3.1"
)
cat(table1_notes)
# 输出: 表1 注: 数据以均数±标准差或频数(%)表示。 P值由t检验或卡方检验计算。 统计分析使用R软件 (version 4.3.1)完成。 n=200; 事件数=45。
```

### 6.3 综合图注表注模板

```r
# 为不同类型图表生成对应的标准图注

# 生存曲线图注
survival_caption <- "图X 不同治疗组的Kaplan-Meier生存曲线比较。采用log-rank检验比较两组生存差异。"

# 森林图图注
forest_caption <- "图X 亚组分析的森林图。横轴表示风险比(HR)，垂直虚线代表HR=1。"

# 热图图注
heatmap_caption <- "图X 基因表达热图。行表示基因，列表示样本。颜色表示标准化后的表达量(按行Z-score)。"

# 相关性热图图注
corr_caption <- "图X 变量相关性热图。颜色表示Pearson相关系数，蓝色为正相关，红色为负相关。"
```

---

## 第七部分：自动化工作流

### 7.1 一键分析模板

```r
# 完整的临床研究分析流程
clinical_analysis <- function(data, outcome_var, group_var, covariables) {
  
  # 1. 描述性统计
  cat("=== 描述性统计 ===\n")
  baseline <- CreateTableOne(
    vars = c(covariables, outcome_var),
    data = data,
    strata = group_var,
    factorVars = outcome_var
  )
  print(baseline, smd = TRUE)
  
  # 2. 单因素分析
  cat("\n=== 单因素Logistic回归 ===\n")
  univ <- autoReg(outcome_var ~ ., data = data, uni = TRUE) %>%
    myft()
  print(univ)
  
  # 3. 多因素分析
  cat("\n=== 多因素Logistic回归 ===\n")
  multiv <- autoReg(outcome_var ~ ., data = data, multi = TRUE) %>%
    myft()
  print(multiv)
  
  # 4. 亚组分析
  cat("\n=== 亚组分析 ===\n")
  # (根据需要添加亚组代码)
  
  # 5. 图表输出
  cat("\n=== 生成图表 ===\n")
  # (添加图表生成代码)
  
  return(list(baseline = baseline, univ = univ, multiv = multiv))
}

# 使用示例
# results <- clinical_analysis(
#   data = mydata,
#   outcome_var = "outcome",
#   group_var = "treatment",
#   covariables = c("age", "sex", "bmi")
# )
```

### 7.2 批量导出报告

```r
library(rmarkdown)
library(officer)

generate_report <- function(data, output_file = "analysis_report.docx") {
  
  # 创建Word文档
  doc <- read_docx()
  
  # 添加标题
  doc <- doc %>% 
    body_add_par("临床研究统计分析报告", style = "heading 1")
  
  # 添加表格
  doc <- doc %>% 
    body_add_par("表1 基线特征", style = "heading 2") %>%
    body_add_flextable(ft_baseline)
  
  # 添加图表
  doc <- doc %>%
    body_add_par("图1 生存曲线", style = "heading 2") %>%
    body_add_gg(p_survival, width = 5, height = 4)
  
  # 保存
  print(doc, target = output_file)
}
```

---

## 第八部分：常见期刊要求

### 8.1 图表规范

| 期刊类型 | 宽度要求 | 高度比例 | 分辨率 |
|---------|---------|---------|-------|
| 单栏图 | 8-9 cm | 自由 | 300-600 dpi |
| 双栏图 | 12-17 cm | 自由 | 300-600 dpi |
| 半版图 | ≤8 cm | 自由 | 300-600 dpi |
| PDF矢量图 | 建议8-17 cm | 自由 | 无需dpi |

### 8.2 表格规范

```r
# 标准三线表格式
# - 顶线、底线1.5pt
# - 栏目线0.5pt  
# - 无竖线
# - 字体Times New Roman 8-10pt
# - 数字右对齐
# - 中文用宋体
```

---

## 参考资源

1. **R for Data Science**: https://r4ds.had.co.nz/
2. **gtsummary**: https://www.danieldsjoberg.com/gtsummary/
3. **survminer**: http://www.sthda.com/english/rsurvival-survminer
4. **tableone**: https://cran.r-project.org/web/packages/tableone/
5. **autoReg**: https://cran.r-project.org/web/packages/autoReg/
6. ** ggplot2**: https://ggplot2.tidyverse.org/
7. **STHDA**: http://www.sthda.com/english/

---

## 注意事项

1. **数据安全**: 医学数据需注意隐私保护，脱敏处理后再进行分析
2. **统计方法选择**: 根据数据分布和研究设计选择合适的统计方法
3. **多重比较**: 进行多次检验时需校正P值
4. **效应量报告**: 除P值外，应同时报告效应量及其95%置信区间
5. **结果可重复**: 记录数据处理和分析的所有参数和步骤
