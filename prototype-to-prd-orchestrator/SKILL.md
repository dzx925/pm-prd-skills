---
name: prototype-to-prd-orchestrator
version: 1.2.0
description: PRD生成编排器，自动串联所有子Skill，一键完成从原型到完整PRD的全流程生成，支持自动截取原型截图并嵌入PRD
scope: global
---

# prototype-to-prd-orchestrator
## 角色
你是PRD生成总指挥，自动协调各个子Skill按顺序执行，完成从原型到完整PRD的全流程生成。

## 适用人群
产品经理（希望自动化生成PRD的用户）

## 触发条件
用户输入"生成PRD"、"原型转PRD"、"一键生成PRD"，并提供原型。

## 工作流程（自动按顺序执行）

### 阶段1：原型解析
**调用 Skill**: prototype-parser
- 输入：用户提供的原型（截图/链接/HTML）
- 输出：parsed_prototype.yaml
- 说明：提取页面结构、字段、交互、功能模块

### 阶段2：业务提炼
**调用 Skill**: business-refiner
- 输入：parsed_prototype.yaml
- 输出：business_refined.yaml
- 说明：补充角色、目标、痛点、业务场景

### 阶段3：PRD章节生成（并行执行）

#### 3-1：业务章节
**调用 Skill**: prd-business-section
- 输入：business_refined.yaml
- 输出：section_1_3_business.md
- 说明：补充角色、目标、痛点、业务场景

#### 3-2：分析章节
**调用 Skill**: prd-analysis-section
- 输入：产品类型、核心功能
- 输出：section_4_analysis.md

#### 3-3：方案框架
**调用 Skill**: solution-framework
- 输入：parsed_prototype.yaml + business_refined.yaml
- 输出：solution_framework.yaml

#### 3-4：数据结构说明章节（新增）
**调用 Skill**: data-structure-section
- 输入：parsed_prototype.yaml + solution_framework.yaml
- 输出：section_4_5_data_structure.md
- 说明：按照标准格式生成数据结构说明，包含11个部分

#### 3-5：准备章节
**调用 Skill**: prd-preparation-section
- 输入：产品类型、功能复杂度
- 输出：section_6_7_preparation.md

#### 3-6：计划章节
**调用 Skill**: prd-plan-section
- 输入：上线时间、原型链接
- 输出：section_8_9_plan.md

### 阶段4：功能模块生成（并行执行）
**调用 Skill**: feature-module-generator（多次）
- 输入：solution_framework.yaml + 指定 module_id
- 输出：module_M001.yaml, module_M002.yaml, ...
- 说明：每个功能模块调用一次

### 阶段5：方案合并
**调用 Skill**: solution-merger
- 输入：solution_framework.yaml + 所有 module_*.yaml
- 输出：section_5_solution.md

### 阶段6：原型截图（关键增强）
**执行截图脚本**: screenshot_prototype.js
- 输入：prototype_html_path（原型HTML文件路径）
- 输出：screenshots/目录下的PNG文件 + screenshots_metadata.json
- 说明：自动截取原型关键页面并转为Base64嵌入PRD

#### 截图实现步骤

**步骤1：检查并安装依赖**
```bash
# 检查是否已安装 playwright
npm list -g playwright 2>/dev/null || npm install -g playwright

# 安装 Chromium 浏览器
npx playwright install chromium
```

**步骤2：创建截图脚本**
创建 `screenshot_prototype.js` 文件：

```javascript
const { chromium } = require('playwright');
const fs = require('fs');
const path = require('path');

async function capturePrototypeScreenshots(prototypePath) {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // 加载原型HTML
  await page.goto(`file://${path.resolve(prototypePath)}`);
  
  // 等待页面加载完成
  await page.waitForLoadState('networkidle');
  
  // 创建截图目录
  const screenshotDir = './screenshots';
  if (!fs.existsSync(screenshotDir)) {
    fs.mkdirSync(screenshotDir);
  }
  
  // 截取关键页面
  const screenshots = [];
  
  // 1. 截取首屏
  const mainScreenshot = path.join(screenshotDir, 'main_page.png');
  await page.screenshot({ path: mainScreenshot, fullPage: true });
  screenshots.push({
    name: 'main_page',
    path: mainScreenshot,
    description: '主页面'
  });
  
  // 2. 查找并截取其他页面（如果有导航）
  const links = await page.$$eval('a[href^="#"]', links => 
    links.map(link => ({
      text: link.textContent.trim(),
      href: link.getAttribute('href')
    }))
  );
  
  for (const link of links.slice(0, 5)) { // 最多截取5个页面
    try {
      await page.click(`a[href="${link.href}"]`);
      await page.waitForTimeout(500); // 等待页面切换动画
      
      const screenshotPath = path.join(screenshotDir, `${link.text.replace(/\s+/g, '_')}.png`);
      await page.screenshot({ path: screenshotPath, fullPage: true });
      
      screenshots.push({
        name: link.text,
        path: screenshotPath,
        description: `${link.text}页面`
      });
    } catch (e) {
      console.log(`无法截取页面 ${link.text}: ${e.message}`);
    }
  }
  
  await browser.close();
  
  // 生成元数据
  const metadata = {
    captured_at: new Date().toISOString(),
    prototype_path: prototypePath,
    screenshots: screenshots
  };
  
  fs.writeFileSync(
    path.join(screenshotDir, 'screenshots_metadata.json'),
    JSON.stringify(metadata, null, 2)
  );
  
  return metadata;
}

// 如果直接运行此脚本
if (require.main === module) {
  const prototypePath = process.argv[2] || './prototype.html';
  capturePrototypeScreenshots(prototypePath)
    .then(metadata => {
      console.log('截图完成:', metadata);
    })
    .catch(err => {
      console.error('截图失败:', err);
      process.exit(1);
    });
}

module.exports = { capturePrototypeScreenshots };
```

**步骤3：执行截图**
```bash
node screenshot_prototype.js ./prototype.html
```

### 阶段7：PRD优化
**调用 Skill**: prd-optimizer
- 输入：所有 section_*.md 文件
- 输出：
  - prd_review_report.md（质量检查报告）
  - PRD_V1.0.0.md（最终完整PRD）

## 输出文件清单

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
│   ├── 员工列表.png
│   └── screenshots_metadata.json
├── prd_review_report.md           # 阶段7：质量检查报告
└── PRD_V1.0.0.md                  # 阶段7：最终完整PRD
```

## 使用方式

### 方式1：全自动模式（推荐）
用户只需提供原型， orchestrator 自动执行全部流程：
```
用户：生成PRD，原型：[截图/链接]
→ orchestrator 自动执行阶段1-7
→ 输出：完整PRD文档（含原型截图）
```

### 方式2：分阶段模式
用户可指定从某个阶段开始：
```
用户：从业务提炼开始，parsed_prototype.yaml 内容：...
→ orchestrator 从阶段2开始执行
```

### 方式3：单章节模式
用户可指定只生成某个章节：
```
用户：只生成业务章节，business_refined.yaml 内容：...
→ orchestrator 只执行阶段3-1
```

## 执行顺序图

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

## 强制约束
- 必须按顺序执行，前一阶段输出是后一阶段输入
- 并行阶段必须等待全部完成后再进入下一阶段
- 任何阶段失败必须停止并报告错误
- 最终必须输出质量检查报告
- 全程使用中文输出
