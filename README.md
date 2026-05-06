# PM PRD Skills - 产品经理PRD生成工具集

一套完整的从原型到PRD的Skill工具集，专为ToB产品经理设计。

## 包含Skill

### 核心流程（按执行顺序）

| 阶段 | Skill名称 | 功能说明 | 触发关键词 |
|-----|----------|---------|-----------|
| 1 | `prototype-parser` | 解析原型，提取结构化信息 | "解析原型" |
| 2 | `business-refiner` | 提炼业务角色、目标、痛点 | "提炼业务" |
| 3-1 | `prd-business-section` | 生成PRD第1-3章（业务章节） | "生成业务章节" |
| 3-2 | `prd-analysis-section` | 生成PRD第4章（竞品分析） | "生成分析章节" |
| 3-3 | `solution-framework` | 生成方案框架（流程、模型） | "生成方案框架" |
| 3-3-A | `feature-module-generator` | 生成功能模块详情 | "生成功能模块" |
| 3-3-Z | `solution-merger` | 合并方案，生成第5章 | "合并方案" |
| 3-4 | `prd-preparation-section` | 生成PRD第6-7章（准备、非功能） | "生成准备章节" |
| 3-5 | `prd-plan-section` | 生成PRD第8-9章（计划、附录） | "生成计划章节" |
| 4 | `prd-optimizer` | 检查优化，输出最终PRD | "优化PRD" |

### 主控Skill
| Skill名称 | 功能说明 | 触发关键词 |
|----------|---------|-----------|
| `prototype-to-prd-orchestrator` | 自动串联所有Skill，一键生成 | "生成PRD"、"原型转PRD" |

## 快速使用

### 方式1：全自动生成（推荐）
```
触发词：生成PRD
输入：原型截图/链接/HTML
输出：完整PRD文档
```

### 方式2：分步生成
按顺序执行各阶段Skill，每步可检查中间结果：
1. 解析原型 → 2. 提炼业务 → 3. 生成各章节 → 4. 优化合并

## 安装方法

### 方法1：Git Submodule（推荐）
```bash
cd 你的项目
git submodule add https://github.com/dzx925/pm-prd-skills.git .trae/skills/pm-prd-skills
```

### 方法2：直接克隆
```bash
cd 你的项目/.trae/skills/
git clone https://github.com/dzx925/pm-prd-skills.git
```

### 方法3：手动复制
下载ZIP，解压到 `.trae/skills/pm-prd-skills/`

## 目录结构

```
pm-prd-skills/
├── README.md                          # 本文件
├── prototype-parser/                  # 阶段1：原型解析
│   └── SKILL.md
├── business-refiner/                  # 阶段2：业务提炼
│   └── SKILL.md
├── prd-business-section/              # 阶段3-1：业务章节
│   └── SKILL.md
├── prd-analysis-section/              # 阶段3-2：分析章节
│   └── SKILL.md
├── solution-framework/                # 阶段3-3：方案框架
│   └── SKILL.md
├── feature-module-generator/          # 阶段3-3-A：功能模块
│   └── SKILL.md
├── solution-merger/                   # 阶段3-3-Z：方案合并
│   └── SKILL.md
├── prd-preparation-section/           # 阶段3-4：准备章节
│   └── SKILL.md
├── prd-plan-section/                  # 阶段3-5：计划章节
│   └── SKILL.md
├── prd-optimizer/                     # 阶段4：PRD优化
│   └── SKILL.md
└── prototype-to-prd-orchestrator/     # 主控：自动串联
    └── SKILL.md
```

## 输出文件说明

执行完整流程后，会生成以下文件：

```
output/
├── parsed_prototype.yaml          # 原型解析结果
├── business_refined.yaml          # 业务提炼结果
├── solution_framework.yaml        # 方案框架
├── module_*.yaml                  # 各功能模块详情
├── section_1_3_business.md        # PRD第1-3章
├── section_4_analysis.md          # PRD第4章
├── section_5_solution.md          # PRD第5章
├── section_6_7_preparation.md     # PRD第6-7章
├── section_8_9_plan.md            # PRD第8-9章
├── prd_review_report.md           # 质量检查报告
└── PRD_V1.0.0.md                  # 最终完整PRD
```

## 版本

v1.0.0

## 作者

dzx925

## License

MIT
