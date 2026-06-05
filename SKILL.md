---
name: ppt-image-redraw
description: Use when Codex needs to redraw a reference image, screenshot, rendered slide, or academic PPT page into PowerPoint, especially when the user asks for 一比一复刻, 可编辑PPTX, 图片重新画进PPT, 截图转PPT, correct arrow directions, separate icon insertion, or formulas/math symbols as PowerPoint equations.
---

# PPT Image Redraw

## Overview

把参考图片页重建为 PowerPoint。默认同时考虑视觉还原和可编辑性：图片版负责像素级观感，可编辑重绘版负责后续修改。

核心原则：先识别对象类型，再选择正确的 PPT 表达方式。普通文本、容器、线条、流程箭头、图表用 PPT 原生对象；数学符号和公式用 PowerPoint OfficeMath；图标作为单一语义资产插入，不要用多个小形状叠加冒充完整图标。

## Workflow

1. 读取参考图，确认尺寸、比例、输出目录和用户是否要求可编辑。
2. 建立视觉清单：标题、Logo/校名、分区框、标签页、图表、流程卡片、箭头、图标、说明文字、公式。
3. 先画拓扑草图：记录每个箭头的起点、终点、方向、是否虚线、是否仅为分隔线。箭头方向不确定时回看参考图，不要凭习惯默认从左到右。
4. 若用户要求“一比一完美复刻”，先交付全图铺满的图片版 PPTX，作为视觉基准。
5. 构建可编辑重绘版：按参考图坐标重建 PPT 原生对象，避免把整页截图作为最终可编辑页背景。
6. 对所有公式和数学符号单独处理：`f(x)`、`μ_c`、`s(y_t,k)`、`π_t(k)`、`c_t`、`q_b(k)`、分段函数等都应插入为 OfficeMath 公式对象，而不是普通正文文本。
7. 对所有图标单独处理：目标靶心、菱形网络、盾牌/方框、隐蔽性眼睛等应作为一个独立图标对象插入；不要拆成圆、线、框逐个叠加。
8. 生成后运行结构校验：PPTX 包可解析、对象数量合理、图片版哈希匹配、箭头方向清单匹配、图标对象清单匹配、公式节点存在、旧的普通文本公式残留为 0。
9. 告知用户文件路径、可编辑范围、已知限制和验证结果。

## Object Rules

- **图片版**：允许整张参考图作为唯一全页图片；只承诺视觉一比一，不承诺对象可编辑。
- **可编辑重绘版**：不得使用整页截图作为背景来冒充可编辑；文本和形状应为 PPT 原生对象。
- **图表**：优先用轴线、网格线、柱形、数据标签、刻度文本重建。若数据可从图中读出，保留读出的值，不编造额外数据。
- **流程卡片**：卡片、编号圆点、中文步骤为形状/文本；数学符号拆成独立公式对象。
- **箭头**：流程箭头必须记录方向和连接关系。箭头头部必须朝向参考图中的目标对象；不要把 `leftArrow`、`rightArrow`、旋转角或起终点写反。
- **学校标识/真实 Logo**：可用简化几何占位或用户提供素材；不要用生成模型伪造真实 Logo。
- **图标**：把图标当作单一语义资产。优先插入用户提供、合规素材库、PowerPoint 内置图标或单独生成的透明 PNG/SVG/EMF；不要用多个几何形状逐个叠加成最终图标，除非明确标注为“可编辑草稿/结构预览”。

## Arrows And Icons

读取 [references/arrows-icons-checklist.md](references/arrows-icons-checklist.md) 当页面包含流程箭头、循环箭头、虚线分隔、底部预期作用图标、功能图标、徽章或任何容易被误画成普通形状堆叠的视觉对象时。

最低要求：

- 每个箭头有 `from`、`to`、`direction`、`head_at`、`bbox`
- 生成前人工核对箭头方向；生成后检查 PPT XML 中箭头类型、位置和翻转/旋转
- 每个图标有 `semantic_id`、`meaning`、`asset_source`、`inserted_as_single_object`
- 生产级可编辑重绘中，图标应是一语义单元一对象；不能用多个红色圆/线/方框叠加后当作图标完成

## OfficeMath Rules

PowerPoint 对公式渲染很挑剔。仅写 `m:oMathPara` 可能出现“选中公式框但内部空白”。用 OpenXML 生成 PPTX 时，公式段落必须使用 `a14:m` 包装，并声明数学和 Office 2010 drawing 命名空间。

读取 [references/officemath-checklist.md](references/officemath-checklist.md) 当任务包含公式、下标、希腊字母、分段函数、softmax、概率符号或用户明确要求“插入公式”时。

最低要求：

```xml
<p:sld
  xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math"
  xmlns:a14="http://schemas.microsoft.com/office/drawing/2010/main">
...
<a14:m>
  <m:oMathPara>
    <m:oMath>...</m:oMath>
  </m:oMathPara>
</a14:m>
```

校验时检查：

- `slide.xml` 中有 `xmlns:a14` 和 `xmlns:m`
- 每个公式对象都有 `<a14:m>`、`<m:oMathPara>`、`<m:oMath>`
- 下标使用 `<m:sSub>`
- 分段公式使用 `<m:eqArr>` 和 `<m:d>`
- 旧普通文本公式如 `<a:t>f(x)</a:t>`、`<a:t>qb(k)</a:t>` 不应残留

## Build Guidance

优先使用项目已有 PPTX/OpenXML 构建脚本和样式习惯。若缺少 `python-pptx`，可直接写 PPTX ZIP 包：

- `ppt/presentation.xml` 设置 16:9 页面尺寸
- `ppt/slides/slide1.xml` 写形状、文本、图片和公式
- `ppt/media/` 只用于图片版或合规资产
- 所有 `.xml` 和 `.rels` 生成后用 XML parser 校验

当已有旧脚本时，先复用其坐标系、颜色、字体和封装函数；只在必要处添加公式对象、图片版打包和验证逻辑。

## Validation

必须新跑验证后再声称完成：

```powershell
python path\to\build_script.py
python path\to\formula_test.py
```

验证报告至少包含：

- 输出 PPTX 路径
- 图片版：图片数量为 1，嵌入图片哈希与参考图一致
- 可编辑版：形状数、文本数、图片数
- 箭头：方向清单和参考图一致，没有头尾反向
- 图标：语义图标数量、来源和单对象插入结果
- 公式：`a14:m` 数量、`m:oMathPara` 数量、`m:sSub` 数量、分段公式节点数量
- 旧普通文本公式残留检查结果

如果本机没有 PowerPoint、LibreOffice 或 `soffice`，明确说明无法做渲染预览，只能做 PPTX 结构校验；不要把结构校验说成视觉渲染验收。

## Common Mistakes

- 把整页截图铺底后叠几个透明文本框，说成可编辑复刻。
- 公式只用 Cambria Math 字体写普通文本。
- 只写 `m:oMathPara`，没有 `a14:m` 包装，导致 PowerPoint 中公式框空白。
- 漏识别流程图里的小公式，把 `μc`、`πt(k)`、`ct`、`qb(k)` 留在正文里。
- 箭头方向画反，尤其是复用 `leftArrow/rightArrow` 或坐标镜像时没有复查头部方向。
- 把靶心、菱形、盾牌、眼睛等图标拆成多个形状叠加，导致后续无法作为一个图标移动/替换。
- 文件正在 PowerPoint 中打开时覆盖失败，却没有另存新版本。
- 只报告“生成成功”，没有说明图片版和可编辑版的差异。
