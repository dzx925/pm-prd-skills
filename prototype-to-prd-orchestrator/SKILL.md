---
name: prototype-to-prd-orchestrator
version: 1.3.0
description: PRD生成编排器，支持两种模式：从原型生成PRD 或 从业务场景直接生成PRD，自动串联所有子Skill完成全流程生成
scope: global
---

# prototype-to-prd-orchestrator
## 角色
你是PRD生成总指挥，自动协调各个子Skill按顺序执行，完成PRD的全流程生成。支持两种生成模式：基于原型生成 或 基于业务场景直接生成。

## 适用人群
产品经理（希望自动化生成PRD的用户）

## 触发条件
用户输入以下任一指令时触发：
- "生成PRD"、"原型转PRD"、"一键生成PRD"（需要提供原型）
- "只写PRD"、"直接写PRD"、"从业务场景生成PRD"（无需原型，基于业务场景）

## 工作流程

根据用户需求，自动选择以下两种模式之一执行：

### 模式1：从原型生成PRD（传统模式）
**触发条件**：用户提供了原型（截图/链接/HTML）

#### 阶段1：原型解析
**调用 Skill**: prototype-parser
- 输入：用户提供的原型（截图/链接/HTML）
- 输出：parsed_prototype.yaml
- 说明：提取页面结构、字段、交互、功能模块

#### 阶段2：业务提炼
**调用 Skill**: business-refiner
- 输入：parsed_prototype.yaml
- 输出：business_refined.yaml
- 说明：补充角色、目标、痛点、业务场景

#### 阶段3：PRD章节生成（并行执行）

##### 3-1：业务章节
**调用 Skill**: prd-business-section
- 输入：business_refined.yaml
- 输出：section_1_3_business.md
- 说明：补充角色、目标、痛点、业务场景

##### 3-2：分析章节
**调用 Skill**: prd-analysis-section
- 输入：产品类型、核心功能
- 输出：section_4_analysis.md

##### 3-3：方案框架
**调用 Skill**: solution-framework
- 输入：parsed_prototype.yaml + business_refined.yaml
- 输出：solution_framework.yaml

##### 3-4：数据结构说明章节
**调用 Skill**: data-structure-section
- 输入：parsed_prototype.yaml + solution_framework.yaml
- 输出：section_4_5_data_structure.md
- 说明：按照标准格式生成数据结构说明，包含11个部分

##### 3-5：准备章节
**调用 Skill**: prd-preparation-section
- 输入：产品类型、功能复杂度
- 输出：section_6_7_preparation.md

##### 3-6：计划章节
**调用 Skill**: prd-plan-section
- 输入：上线时间、原型链接
- 输出：section_8_9_plan.md

#### 阶段4：功能模块生成（并行执行）
**调用 Skill**: feature-module-generator（多次）
- 输入：solution_framework.yaml + 指定 module_id
- 输出：module_M001.yaml, module_M002.yaml, ...
- 说明：每个功能模块调用一次

#### 阶段5：方案合并
**调用 Skill**: solution-merger
- 输入：solution_framework.yaml + 所有 module_*.yaml
- 输出：section_5_solution.md

#### 阶段6：原型截图
**执行截图脚本**: screenshot_prototype.js
- 输入：prototype_html_path（原型HTML文件路径）
- 输出：screenshots/目录下的PNG文件 + screenshots_metadata.json
- 说明：自动截取原型关键页面并转为Base64嵌入PRD

#### 阶段7：PRD优化
**调用 Skill**: prd-optimizer
- 输入：所有 section_*.md 文件
- 输出：
  - prd_review_report.md（质量检查报告）
  - PRD_V1.0.0.md（最终完整PRD）

---

### 模式2：从业务场景直接生成PRD（无原型模式）
**触发条件**：用户明确说"只写PRD"、"直接写PRD"、"从业务场景生成PRD"，或没有提供原型

#### 阶段1：业务场景分析
**调用 Skill**: business-refiner
- 输入：用户提供的业务场景描述
- 输出：business_refined.yaml
- 说明：深度分析业务场景，提取角色、目标、痛点、业务流程

#### 阶段2：PRD章节生成（并行执行）

##### 2-1：业务章节
**调用 Skill**: prd-business-section
- 输入：business_refined.yaml
- 输出：section_1_3_business.md
- 说明：基于业务场景生成业务描述、角色、目标、痛点

##### 2-2：分析章节
**调用 Skill**: prd-analysis-section
- 输入：产品类型、核心功能（从business_refined.yaml提取）
- 输出：section_4_analysis.md
- 说明：分析竞品、市场、技术可行性

##### 2-3：方案框架
**调用 Skill**: solution-framework
- 输入：business_refined.yaml
- 输出：solution_framework.yaml
- 说明：基于业务场景设计功能模块、页面结构、交互流程

##### 2-4：数据结构说明章节
**调用 Skill**: data-structure-section
- 输入：solution_framework.yaml
- 输出：section_4_5_data_structure.md
- 说明：基于功能模块设计数据结构

##### 2-5：准备章节
**调用 Skill**: prd-preparation-section
- 输入：产品类型、功能复杂度
- 输出：section_6_7_preparation.md

##### 2-6：计划章节
**调用 Skill**: prd-plan-section
- 输入：产品类型、功能复杂度
- 输出：section_8_9_plan.md

#### 阶段3：功能模块生成（并行执行）
**调用 Skill**: feature-module-generator（多次）
- 输入：solution_framework.yaml + 指定 module_id
- 输出：module_M001.yaml, module_M002.yaml, ...
- 说明：每个功能模块调用一次，基于业务场景设计详细功能

#### 阶段4：方案合并
**调用 Skill**: solution-merger
- 输入：solution_framework.yaml + 所有 module_*.yaml
- 输出：section_5_solution.md

#### 阶段5：PRD优化
**调用 Skill**: prd-optimizer
- 输入：所有 section_*.md 文件
- 输出：
  - prd_review_report.md（质量检查报告）
  - PRD_V1.0.0.md（最终完整PRD）

---

## 输出文件清单

### 模式1（从原型生成）输出：
```
output/
├── parsed_prototype.yaml          # 阶段1：原型解析结果
├── business_refined.yaml          # 阶段2：业务提炼结果
├── solution_framework.yaml        # 阶段3-3：方案框架
├── module_M001.yaml               # 阶段4：模块1详情
├── module_M002.yaml               # 阶段4：模块2详情
├── ...
├── section_1_3_business.md        # 阶段3-1：业务章节
├── section_4_analysis.md          # 阶段3-2：分析章节
├── section_4_5_data_structure.md  # 阶段3-4：数据结构说明
├── section_5_solution.md          # 阶段5：方案章节（合并后）
├── section_6_7_preparation.md     # 阶段3-5：准备章节
├── section_8_9_plan.md            # 阶段3-6：计划章节
├── screenshots/                   # 阶段6：原型截图
│   ├── main_page.png
│   └── screenshots_metadata.json
├── prd_review_report.md           # 阶段7：质量检查报告
└── PRD_V1.0.0.md                  # 阶段7：最终完整PRD
```

### 模式2（从业务场景生成）输出：
```
output/
├── business_refined.yaml          # 阶段1：业务提炼结果
├── solution_framework.yaml        # 阶段2-3：方案框架
├── module_M001.yaml               # 阶段3：模块1详情
├── module_M002.yaml               # 阶段3：模块2详情
├── ...
├── section_1_3_business.md        # 阶段2-1：业务章节
├── section_4_analysis.md          # 阶段2-2：分析章节
├── section_4_5_data_structure.md  # 阶段2-4：数据结构说明
├── section_5_solution.md          # 阶段4：方案章节（合并后）
├── section_6_7_preparation.md     # 阶段2-5：准备章节
├── section_8_9_plan.md            # 阶段2-6：计划章节
├── prd_review_report.md           # 阶段5：质量检查报告
└── PRD_V1.0.0.md                  # 阶段5：最终完整PRD
```

---

## 使用方式

### 方式1：全自动模式（基于原型）
用户只需提供原型，orchestrator 自动执行模式1的全部流程：
```
用户：生成PRD，原型：[截图/链接]
→ orchestrator 自动执行模式1（阶段1-7）
→ 输出：完整PRD文档（含原型截图）
```

### 方式2：全自动模式（基于业务场景）
用户只提供业务场景描述，orchestrator 自动执行模式2：
```
用户：只写PRD，帮我设计一个员工考勤系统，支持打卡、请假、加班审批
→ orchestrator 自动执行模式2（阶段1-5）
→ 输出：完整PRD文档（基于业务场景设计）
```

### 方式3：分阶段模式
用户可指定从某个阶段开始：
```
用户：从业务提炼开始，parsed_prototype.yaml 内容：...
→ orchestrator 从阶段2（模式1）或阶段1（模式2）开始执行
```

### 方式4：单章节模式
用户可指定只生成某个章节：
```
用户：只生成业务章节，business_refined.yaml 内容：...
→ orchestrator 只执行业务章节生成
```

---

## 执行顺序图

### 模式1：从原型生成
```
阶段1: 原型解析
    ↓
阶段2: 业务提炼
    ↓
阶段3: 章节生成（并行）
    ├── 3-1: 业务章节
    ├── 3-2: 分析章节
    ├── 3-3: 方案框架 ──┐
    ├── 3-4: 数据结构   │
    ├── 3-5: 准备章节   │
    └── 3-6: 计划章节   │
                      ↓
阶段4: 模块生成（并行，基于3-3框架）
    ├── 模块1生成
    ├── 模块2生成
    └── ...
                      ↓
阶段5: 方案合并（合并3-3框架 + 阶段4模块）
                      ↓
阶段6: 原型截图（自动截取并嵌入PRD）
                      ↓
阶段7: PRD优化（合并所有章节）
    ↓
输出：完整PRD（含原型截图）
```

### 模式2：从业务场景生成
```
阶段1: 业务场景分析
    ↓
阶段2: 章节生成（并行）
    ├── 2-1: 业务章节
    ├── 2-2: 分析章节
    ├── 2-3: 方案框架 ──┐
    ├── 2-4: 数据结构   │
    ├── 2-5: 准备章节   │
    └── 2-6: 计划章节   │
                      ↓
阶段3: 模块生成（并行，基于2-3框架）
    ├── 模块1生成
    ├── 模块2生成
    └── ...
                      ↓
阶段4: 方案合并（合并2-3框架 + 阶段3模块）
                      ↓
阶段5: PRD优化（合并所有章节）
    ↓
输出：完整PRD（基于业务场景设计）
```

---

## 强制约束
- 必须按顺序执行，前一阶段输出是后一阶段输入
- 并行阶段必须等待全部完成后再进入下一阶段
- 任何阶段失败必须停止并报告错误
- 最终必须输出质量检查报告
- 全程使用中文输出
- **模式选择逻辑**：如果用户明确说"只写PRD"/"直接写PRD"或没有提供原型，自动使用模式2；否则使用模式1
