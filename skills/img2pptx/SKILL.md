---
name: img2pptx
description: Reconstruct raster reference images as one-slide editable, modular, auditable PPTX files containing a complete SVG. Use when the user asks to convert, reproduce, trace, vectorize, or rebuild a PNG, JPG/JPEG, WebP, TIFF, HEIC, screenshot, diagram, infographic, scientific figure, architecture figure, or workflow image into an editable PowerPoint slide.
---

# Img2Pptx

将输入栅格参考图重建为一页可编辑、可拆分、可复用、可审计的 PPTX。把本 Skill 视为执行协议，不要把任务误解为简单地绘制一张看起来相似的图片。

## 运行约定

对每个任务执行以下约定：

1. 检查输入文件、工作目录和本地可用工具。
2. 在当前任务工作区按需生成绘图、渲染、裁切、比较、PPTX 生成和审计脚本。
3. 不要假定前一次任务的脚本、组件名、画布尺寸、布局或阈值适用于当前任务。
4. 优先使用环境中已经可用的工具；不要将 Skill 绑定到某个固定语言、浏览器路径或操作系统。
5. 将任务特有实现保留在当前任务目录中，使结果可复现、可修改、可审计。
6. 持续完成重建、渲染、QA、修正、PPTX 生成和最终审计；不要在第一版 SVG 生成后停止。
7. 不要修改原始输入图。

## 输入标准化

接受 PNG、JPG/JPEG、WebP 以及本地工具能够解码的其他常见栅格格式。

对于 TIFF、HEIC、多帧图片、带颜色配置文件的图片或解码支持不稳定的格式：

1. 保留原始文件，将目标帧无损或高质量转换为标准 PNG；
2. 统一颜色空间和方向，使用标准化 PNG 作为后续像素比较基准；
3. 在 manifest 中记录原始文件和标准化文件。

使用输入图的真实尺寸和纵横比。除非用户明确要求标准幻灯片比例，否则让 PPTX 页面比例与输入图一致。

## 最终目标

必须满足：

- 生成一页 PPTX，并嵌入完整 `full.svg`，不能仅嵌入整页位图；
- 可同时包含 PNG fallback，但 SVG 必须是主矢量表示，并尽可能支持 PowerPoint“转换为形状 / 取消组合”；
- 文字尽量保留为可编辑 SVG `<text>`，图形尽量保留为 vector shape、path 或矢量组合；
- 每个要求导出的模块能够独立成立，并完成结构审计、视觉对比、PPTX 审计和自动修正；
- 明确区分“SVG 已完整嵌入”和“PowerPoint 转换后仍完全保留层级”：后者受 PowerPoint 版本和 SVG importer 影响，不得虚假保证。

## 1. 按实际编辑粒度建模

按照语义完整性和实际编辑需求拆分内容，不要机械地按几何图元拆分。

一个可编辑单元应尽量满足：

- 独立拿出来仍然完整，具有明确边界、独立含义或视觉功能；
- 可以单独移动、替换或修改；
- 取消组合后不会产生大量无意义碎片。

不要把半截边框、半个图标、孤立装饰或缺少上下文的线段作为语义模块。

### 语义模块 `semantic-unit`

将能够独立表达完整概念或承担明确功能的视觉单元定义为语义模块，例如面板、流程子模块、报告框、流程节点、结构模块、关系箭头、输入框、输出框或说明框。根据实际输入自主识别，不要套用固定类型清单。

### 组合子模块 `combination-submodule`

将多个元素共同构成、具有局部完整含义的组合定义为组合子模块，例如：

- icon + label；
- 标题 + 分割线；
- 图标行；
- 报告框内部一行；
- 结构式 + 名称；
- 箭头 + 说明文字；
- 状态图标 + 数值。

### 原子元素 `atomic-element`

将在目标编辑粒度下无需继续拆分的最小元素定义为原子元素，例如：

- 可编辑文字；
- line、rect、circle、ellipse、polygon、polyline；
- SVG path；
- 基础图标；
- 复杂矢量小图案；
- logo、品牌符号；
- 分子结构片段、细胞等等科学插图；
- 小型插画或专业符号。

“原子”表示继续拆分没有实际编辑价值，不表示几何上不可继续分解。复杂图标可以使用带独立 `id` 的 `<g>` 或 `<symbol>`；manifest 可将其视为原子元素，同时允许 PowerPoint 转换后继续编辑内部 path。

## 2. 处理复杂图案

对于 logo、复杂 icon、分子结构、小型插画等内容，依次采用：

1. 输入材料中已有的矢量资产，或可获得的可靠官方 SVG / 规范资产；
2. 根据原图进行矢量重绘、路径追踪或矢量化；
3. 必要时先生成图案再转为矢量；
4. 仅在无法合理矢量化时使用嵌入式位图。

为保持与原图的视觉一致性，让原图中可辨识的图标、科学符号和小型图案保留
其图形身份与基本外观；不要仅因实现便利或可编辑性要求，用文字、emoji、
Unicode 字符、其他图标或通用占位符替代，除非原图本身就是这种表示。优先
进行矢量重绘；只有在无法可靠重绘且替代会造成明显视觉失真时，才将原图局部
裁切作为独立原子位图保留，并在 manifest 中记录来源区域、使用原因和后续
可替换状态。

不要凭印象随意绘制品牌 logo。优先保持输入图中的形状，或使用可靠规范资产。

使用位图时，在 manifest 中记录：

- `representation: raster-image`；
- 使用原因；
- 原始来源；
- 是否可以后续替换为矢量。

## 源图语义约束提取（强制）

绘图前识别所有依赖颜色、位置、方向、顺序、数量、连接、包含、正负、
排名或符号才能表达含义的组件。对这些组件：

1. 从标准化原图裁切并放大检查；
2. 记录直接观察到的 `source_observations`，将推断与直接观察分开；
3. 建立可执行的 `visual_invariants` 和 `negative_constraints`；
4. 为每条硬约束指定 `audit_method` 和唯一 `audit_check`；
5. 将不确定观察标记为需要人工复核，不得凭常识补全或伪造数据。

图表、表格、排名、流程图、架构图、科学图、地图、时间线或标注序列必须
读取并遵循 [Semantic Constraint and Audit Protocol](references/semantic-constraint-audit.md)。
只启用与当前组件类型相关的规则，不要将图表规则机械应用于普通插画。

语义正确性是硬门禁。低 MAE、结构完整或 PPTX 忠实复现生成 SVG，都不能
覆盖颜色含义、轴侧、方向、拓扑、顺序或排他关系错误。

## 3. 建立 Component Manifest

绘图前必须创建 `component_manifest.json`，并让后续绘图、模块导出、父子包含检查、对齐检查和组件 crop diff 依赖该 manifest。

每个组件至少记录：

```json
{
  "id": "evidence_a_report",
  "label": "Structure report card",
  "parent": "panel_evidence",
  "bbox": {
    "x": 0,
    "y": 0,
    "width": 100,
    "height": 80
  },
  "level": "semantic-unit",
  "export": true,
  "editable_parts": ["title", "divider", "rows", "icons", "labels"],
  "representation": "svg-group",
  "render_strategy": "vector-reconstruction",
  "semantic_role": "structure report",
  "source_observations": [],
  "visual_invariants": [],
  "negative_constraints": [],
  "notes": "Keep outline as final child"
}
```

优先使用以下字段：

- 身份与层级：`id`、`label`、`parent`、`level`；
- 几何与导出：`bbox`、`export`；
- 编辑与表示：`editable_parts`、`representation`、`render_strategy`；
- 来源与约束：`asset_source`、`constraints`、`notes`。
- 语义与审计：`semantic_role`、`source_observations`、`visual_invariants`、
  `negative_constraints`、`audit_mapping`。

同时记录画布、标准化输入文件、默认 padding、outline 参数和视觉优化参数。
每条硬语义约束必须映射到实际执行的审计检查；未映射、未执行或缺少证据时，
约束覆盖审计必须失败。

## 4. 建立布局骨架

先重建：

- 画布尺寸、纵横比和大标题位置；
- 大面板、主要 card 的 bbox 和主箭头走向；
- 模块间距、列宽、行高和整体视觉重心。

此阶段只建立结构和大尺寸关系，不处理复杂图标与细节。

执行 layout skeleton audit，检查：

- 大面板顶部和底部是否齐平，宽度和高度是否接近原图；
- 主要列间距是否合理，子模块是否明显越界；
- 主箭头是否连接正确，整体视觉重心是否接近原图。

骨架审计通过后再进入细节绘制。不得将 layout audit 硬编码为通过。

## 5. 逐个绘制独立模块

根据输入图和 manifest 逐个绘制语义模块。示例可能包括 query card、retrieval card、report card、decision bucket、LLM input、final prediction、箭头和说明框，但不得将这些示例当成固定组件清单。

每个独立导出模块必须：

- 具有完整语义并保留安全 padding；
- 使用独立 `<g id="...">`，文字尽量使用 SVG `<text>`，图形尽量使用 SVG 矢量元素；
- 复杂图标使用内部完整的 `<g>` 或 `<symbol>`；
- 不依赖模块外部元素才能正常显示。

避免影响 PowerPoint 转换的关键 SVG 特性。优先使用 text、rect、line、circle、ellipse、polygon、polyline 和 path。避免 `foreignObject`、不必要的 filter、外部资源依赖和复杂 CSS。必须使用 `<defs>`、gradient、clipPath 或 symbol 时，保证 standalone 模块完整携带依赖，并验证 PowerPoint preview。

## 6. 使用 PowerPoint-Friendly Border Layering

重要容器边框必须采用：

```text
fill-only background + top-layer outline
```

将此规则应用于 panel、card、report box、bucket、input card、final prediction card 等容器：

- 背景 shape 只负责 fill，并使用 `stroke="none"`；
- 使用单独的 outline shape，让它位于对应语义模块 group 的最后并使用 `fill="none"`；
- `stroke-width` 建议为 `1.4px–1.8px`；
- outline 坐标向内缩进 `0.5px–1px`；
- outline 不使用 shadow 或 filter，且背景层和 outline 层不得重复绘制同一条边框；
- outline 属于容器的原子元素，不作为独立语义模块导出。

## 7. 执行 Standalone Module Integrity Audit

为每个 `export: true` 的模块生成独立 SVG，并逐一检查：

- 是否被裁切、是否有内容贴近边缘 2px；
- 文字是否溢出，图标是否缺边，边框是否完整；
- 阴影或装饰是否被截断；
- 模块独立拿出来是否仍然语义完整；
- `<defs>`、gradient、clipPath、symbol 等依赖是否完整包含。

发现缺字、缺边、裁切、贴边或依赖缺失时必须修正。

## 8. 组装完整 SVG

所有独立模块通过后，根据 manifest 的 bbox 装回 `full.svg`。

保留真实父子层级，例如：

```xml
<g id="panel_evidence">
  <g id="evidence_a_unit">
    <g id="capsule_a">...</g>
    <g id="arrow_a">...</g>
    <g id="structure_report_card">...</g>
  </g>
</g>
```

不要将所有元素平铺为同一级 SVG 节点。让层级结构服务于：

- PowerPoint 取消组合；
- 模块整体移动；
- 子模块继续拆分；
- 原子元素单独编辑；
- QA 定位和自动修正。

## 9. 执行 Parent-Child Containment Audit

检查每个子模块 bbox 是否完整位于父模块 bbox 内。默认保留 6px–10px 内边距；特殊紧凑布局必须在 manifest 中说明。

检查：

- 子模块不能越过父框；
- 子模块不能无理由贴边；
- 子模块不能跨入相邻面板；
- 父子关系必须正确；
- card、capsule、bucket 等不能跑出所属 panel；
- 模块真实渲染边界应与 manifest bbox 基本一致。

将失败视为结构错误并修正。

## 10. 执行 Alignment and Spacing Audit

检查：

- 面板顶部和底部是否齐平；
- 同类 card 的左右边界是否一致；
- 标题是否正确对齐；
- 箭头是否连接到正确模块；
- 同列模块间距是否均匀；
- 同类元素尺寸是否一致；
- 子模块是否过挤或过空；
- 边框粗细是否一致；
- 图标与文字的视觉中心是否对齐；
- outline 是否没有产生双线或错位。

## 11. 执行 Border Layering Audit

专门检查：

- 重要容器是否使用 fill-only background；
- 是否存在独立 top-layer outline；
- 是否存在重复 stroke；
- outline 是否是对应 group 的最后一个视觉元素；
- outline 是否向内缩进；
- outline 是否不含 filter 或 shadow；
- 组合状态下边框是否稳定可见；
- 取消组合后边框是否仍然存在；
- 是否出现双线、变粗、错位或裁切。

## 执行 Semantic Constraint and Coverage Audit

对所有意义依赖视觉编码的组件执行语义审计，并输出：

```text
qa/semantic_constraint_audit.json
qa/constraint_coverage_audit.json
qa/semantic_review_sheet.png
```

按组件类型检查适用的不变量。图表重点检查系列颜色、图例、轴侧、正负方向、
峰值/分组、顺序、排他区域和禁止重叠；流程图重点检查节点、边、箭头方向、
端口、分支合并和拓扑；表格、排名和序列重点检查顺序、关联、跨度、长度关系和
重复/缺失；科学图重点检查符号、标签、连接、方向和颜色语义。

同时检查“不应出现”的元素、颜色、连接、重叠、顺序或区域。优先直接审计
SVG 的 group、id、属性和几何关系，再使用颜色 mask、前景比较或人工复核补充。

生成约束覆盖表，确认每条硬源图观察都有约束、每条硬约束都有审计映射、检查
已实际执行且证据相关。任何硬语义约束失败、未覆盖或需要复核却未复核时，
汇总硬门禁必须失败。完整规则见
[Semantic Constraint and Audit Protocol](references/semantic-constraint-audit.md)。

## 12. 执行 Original-Render Visual Similarity Audit

在结构审计通过后对比原图和重建图，输出：

- 标准化 original PNG；
- rendered SVG；
- 50/50 overlay；
- amplified diff；
- 每个组件的 original crop；
- 每个组件的 rendered crop；
- 每个组件的 diff crop；
- `component_diff_sheet.png`。

计算全图 MAE 和组件级 MAE。不要让大面积纯色背景掩盖局部错误。MAE 是定位和优化依据，不能替代 containment、standalone integrity、border layering 或语义视觉检查。

对高语义密度组件同时计算适用的前景加权、按颜色、按区域、按轴侧或 mask
指标。放大检查所有带硬语义约束、存在不确定观察或组件误差较高的区域。
全图 MAE 降低不得覆盖语义约束失败。

不得在未计算或未判断指标时写死 `passed: true`。视觉优化结果必须记录实际数值、阈值、尝试次数和是否达到目标。

## 13. 生成并审计 PPTX

生成一页 PPTX，并嵌入完整 `full.svg`。

检查：

- PPTX 包内是否包含 SVG；
- 是否不存在仅用整页 PNG 代替 SVG 的情况；
- 嵌入 SVG 是否与 `full.svg` 一致，优先比较内容 hash；
- PPTX preview 是否与 rendered SVG 一致；
- 组合状态下边框是否清晰；
- 文字是否乱码或错位；
- 是否方便在 PowerPoint 中转换为形状或取消组合；
- 嵌入 SVG 字节中是否保留嵌套 group；
- 是否存在 PowerPoint 不支持的关键 SVG 特性；
- SVG fallback 和 relationship 是否正确；
- PPTX 是否确实只有一页。

将“嵌入 SVG 完整性”“转换准备度”和“实际转换后的保真度”分开报告。只有在真实执行 PowerPoint 转换并检查后，才能声称转换后的层级完全保留。

## 14. 完成强制修正循环

不得只交付第一版。发现硬性问题后执行：

```text
发现问题
→ 定位 component id
→ 判断问题类型
→ 修改 bbox / 坐标 / 尺寸 / padding / 字号 / path / 箭头 / outline
→ 重新导出模块
→ 重新组装 full.svg
→ 重新渲染
→ 重新运行 QA
→ 更新审计结果
```

持续修正，直到以下硬约束没有未解决问题：

- layout skeleton；
- standalone integrity；
- parent-child containment；
- alignment and spacing；
- border layering；
- semantic constraints and negative constraints；
- constraint coverage and required human review；
- SVG/PPTX package embedding；
- PPTX preview 的结构和可读性。

不要因为生成了最终文件就忽略失败的硬审计。建立汇总门禁；任何硬审计失败时不得声称最终结果全部通过。

## 15. 执行有限、可回滚、非退化的 MAE 优化

只有结构、语义、约束覆盖和 PPTX 兼容性硬审计全部通过后，才进入视觉优化。
读取并遵循 [Visual Similarity Optimization](references/visual-optimization.md)。

默认最多尝试 3 次，以第一份有效 baseline 为基准争取 10% 相对 MAE 改善。
每次从当前 `best` 的完整副本开始，只做局部、可解释修改，并重建 modules、
`full.svg`、PPTX、preview、semantic masks 和全部 QA。

只有候选仍通过所有硬门禁、没有语义或局部退化且指标改善时才接受；否则回滚。
经验证的语义修正优先于全局 MAE，但必须记录失败约束、修正证据和指标取舍。
10% 是软目标；结构、语义、覆盖和嵌入完整性始终是硬门禁。

## 16. 最终交付

必须交付：

```text
final.pptx
full.svg
component_manifest.json
modules/*.svg

qa/layout_skeleton_audit.json
qa/standalone_integrity_audit.json
qa/containment_audit.json
qa/alignment_audit.json
qa/border_layering_audit.json
qa/semantic_constraint_audit.json
qa/constraint_coverage_audit.json
qa/semantic_review_sheet.png
qa/semantic_masks/*
qa/visual_similarity_audit.json
qa/full_overlay_diff.png
qa/amplified_diff.png
qa/component_diff_sheet.png
qa/pptx_slide_preview.png
qa/pptx_preview_audit.json
```

语义审计文件和 masks 仅在源图包含意义依赖视觉编码的组件时强制生成；其余任务
在汇总中记录 `not_applicable`，不得伪造空通过结果。

最终实际使用文件是 `final.pptx`。QA 文件用于验证重建结果是否可靠、完整、可拆分、可编辑和可审计。

交付前确认：

- 最终文件来自 `best` 完整状态；
- `final.pptx`、`full.svg`、manifest、modules 和 QA 相互一致；
- 所有结构、语义、约束覆盖和 PPTX 硬审计通过；
- 所有需要人工语义复核的项目有真实检查状态；
- 视觉优化状态被如实报告；
- 未达到 10% 软目标时，明确说明实际停止原因并交付最佳有效版本。
