# PPT Image Redraw

`ppt-image-redraw` 用于把参考图片、截图、渲染后的 PPT 页面或学术汇报页重新画进 PowerPoint。它强调两件事：视觉尽量贴近参考图，同时明确哪些内容是真正可编辑的 PPT 对象。

## 适用场景

- 用户要求“一比一复刻”“图片重新画到 PPT 里”“截图转可编辑 PPTX”
- 参考图是答辩 PPT、论文汇报页、流程图、架构图、图表页
- 页面包含公式、下标、希腊字母、分段函数，需要用 PowerPoint 插入公式
- 页面包含流程箭头，需要避免箭头头尾画反
- 页面包含靶心、菱形、盾牌、眼睛等语义图标，需要作为独立图标对象插入

## 交付策略

优先交付两类文件：

- **图片版 PPTX**：整张参考图铺满一页，视觉最接近原图，但不可对象级编辑。
- **可编辑重绘版 PPTX**：用 PPT 原生对象、OfficeMath 公式和独立图标资产重建，方便后续修改。

不要把图片版说成可编辑版。可编辑版也不要用整页截图当背景来冒充重绘。

## 目录结构

```text
ppt-image-redraw/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── arrows-icons-checklist.md
    └── officemath-checklist.md
```

## 核心要求

1. **先识别对象类型**
   文字、形状、图表、箭头、公式、图标分别处理，不要一股脑用普通文本或形状堆叠。

2. **公式必须用 OfficeMath**
   `f(x)`、`μ_c`、`s(y_t,k)`、`π_t(k)`、`c_t`、`q_b(k)`、分段函数等都应作为 PowerPoint 公式对象插入。

3. **公式需要 `a14:m` 包装**
   仅写 `m:oMathPara` 可能导致 PowerPoint 中出现空公式框。详见 `references/officemath-checklist.md`。

4. **箭头方向必须核对**
   每个箭头要记录 `from`、`to`、`direction`、`head_at`、`bbox`，生成后复查头部是否朝向目标对象。

5. **图标作为单一语义对象**
   靶心、菱形、盾牌、眼睛等图标应插入为一个独立对象，不要用多个圆、线、框叠加后当作生产级图标。

## 使用示例

```text
Use $ppt-image-redraw to redraw image/xxx.png into an editable PowerPoint page.
```

中文任务示例：

```text
使用 $ppt-image-redraw 将这张答辩 PPT 图片一比一重画到 PPT 中，输出图片版和可编辑版。公式必须用插入公式，箭头不要画反，图标单独插入。
```

## 验收清单

生成后至少检查：

- PPTX 包结构可打开，所有 XML / rels 可解析
- 图片版只包含 1 张全页图片，嵌入图片哈希与参考图一致
- 可编辑版包含合理数量的形状、文本、公式和图标对象
- 公式对象包含 `a14:m`、`m:oMathPara`、`m:oMath`
- 下标包含 `m:sSub`，分段函数包含 `m:eqArr` 和 `m:d`
- 旧的普通文本公式残留为 0，例如 `<a:t>f(x)</a:t>`、`<a:t>qb(k)</a:t>`
- 箭头方向与参考图一致，没有头尾反向
- 每个语义图标可作为一个对象移动或替换

## 已知限制

- 如果本机没有 PowerPoint、LibreOffice 或 `soffice`，只能做 PPTX 结构校验，不能声称完成视觉渲染验收。
- 如果图标没有用户提供或合规来源，只能交付结构预览、内置图标近似或明确标注的降级版本。
- 真实 Logo、校徽、印章等不应由生成模型伪造；优先使用用户提供素材或简化占位。
