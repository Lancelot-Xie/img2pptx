# img2pptx

<p align="center">
  <strong>简体中文</strong> | <a href="README.en.md">English</a>
</p>

只需要一句指令，就能让 Codex 把 GPT Image 2 生成的美观图片转换为 PPTX；然后在
PowerPoint 中选择**取消组合**或**转换为形状**，即可编辑重建后的各个元素。注意，
如果要编辑比较底层的元素可能需要多次的**取消组合**，因为我们的重建逻辑涉及多层次的
元素组合。

```text
使用 $img2pptx，把这张图片转为可编辑的 PPTX。
```

## 示例：重建科研论文插图

下面的原图来自论文
[Revisiting Uncertainty: On Evidential Learning for Partially Relevant Video Retrieval](https://arxiv.org/abs/2605.06083)
中的方法图。输入文件为不可编辑的 PDF，随后被重建为可编辑的 SVG。

```text
使用 $img2pptx，把 methodv3.pdf 重建为单页可编辑 PPTX。
```

### 原始参考图

<p align="center">
  <img src="docs/assets/methodv3-original-preview.svg"
       alt="以 PDF 形式提供的原始科研方法图"
       width="100%">
</p>

### 可编辑重建结果

<p align="center">
  <a href="docs/assets/methodv3-reconstructed.svg">
    <img src="docs/assets/methodv3-reconstructed.svg"
         alt="使用 img2pptx 生成的可编辑 SVG 重建结果"
         width="100%">
  </a>
</p>

点击重建结果即可打开完整分辨率的 SVG。这个示例由可编辑的 SVG 文字和矢量图元
构成，不包含嵌入式 `<image>` 元素。在一次完整运行中，这份 SVG 会被嵌入
`final.pptx`。

## 安装

### 推荐：从 GitHub 安装

在 Codex 中调用 `$skill-installer`，并提供本仓库中的 Skill 路径：

```text
$skill-installer 请从 GitHub 安装这个 skill：
https://github.com/Lancelot-Xie/img2pptx/tree/main/skills/img2pptx
```

英文安装指令：

```text
$skill-installer Install this skill from:
https://github.com/Lancelot-Xie/img2pptx/tree/main/skills/img2pptx
```

Codex 会自动识别新安装的 Skill。如果没有显示，请重启 Codex 或新建一个任务。

### 手动安装

作为个人 Skill 使用时，将 `skills/img2pptx` 复制到：

```text
~/.agents/skills/img2pptx
```

作为仓库级 Skill 使用时，将它复制到：

```text
YOUR_REPOSITORY/.agents/skills/img2pptx
```

## 更多使用示例

中文：

```text
使用 $img2pptx，把这张图片转为可编辑的 PPTX。
```

需要更精确地控制重建结果时：

```text
Use $img2pptx to reconstruct diagram.png as a one-slide editable PPTX.
Preserve the original aspect ratio, text, colors, arrows, and scientific symbols.
```

当请求中明确要求把栅格参考图转换、重建、描摹或矢量化为可编辑的 PowerPoint
幻灯片时，这个 Skill 也可能自动触发。

## 输出内容

主要交付物是 `final.pptx`。一次完整运行还会生成：

- `full.svg` —— 嵌入 PPTX 的完整矢量重建图；
- `component_manifest.json` —— 语义组件、层级、几何信息和约束；
- `modules/*.svg` —— 可独立复用的语义模块；
- `qa/*` —— 布局、包含关系、边框、语义、视觉和 PPTX 嵌入审计结果。

## 为什么不直接把图片放进幻灯片？

`img2pptx` 将输入图视为一个重建任务，而不是简单地把截图放进幻灯片。它致力于
保留：

- 可编辑的 SVG 文字和矢量图元；
- 有意义的嵌套分组和可复用模块；
- 箭头、包含关系、顺序、颜色语义和科学符号；
- 原始图片、SVG 与 PPTX 之间可审计的对应关系。

只有当某个元素无法在不丢失其视觉身份的情况下可靠地重建为矢量时，才会使用
局部栅格裁切。

## 可编辑性与 PowerPoint 兼容性

PPTX 中包含完整的 `full.svg`，而不是只有一张铺满幻灯片的位图。较新的
PowerPoint 版本通常可以对 SVG 使用**转换为形状**和**取消组合**。文字转换、
嵌套分组保留情况和最终外观会受到 PowerPoint 版本、操作系统、字体及 SVG
导入器的影响。

Skill 会分别报告：

1. 是否完整嵌入 SVG；
2. SVG 的结构是否已经做好转换准备；
3. 是否实际执行并检查了 PowerPoint 转换。

除非已经实际完成并检查转换，否则不会声称转换后的结果能够完美保真。

## 支持的参考图

常见输入包括：

- AI 生成的图示和信息图；
- 科研论文中的架构图；
- 科学工作流和符号图；
- 流程图、过程图和系统概览图；
- 截图以及 PNG、JPG/JPEG 和 WebP 文件。

请只使用你有权重建的图片。

## 仓库结构

```text
docs/assets/
├── methodv3-original-preview.svg
└── methodv3-reconstructed.svg

skills/img2pptx/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── semantic-constraint-audit.md
    └── visual-optimization.md
```

## 项目状态

这是一个早期开源版本。随着工作流在更多图示风格和运行环境中得到测试，后续会
继续补充真实案例和回归测试。

## 许可证

本项目采用 Apache License 2.0，详见 [LICENSE](LICENSE)。
