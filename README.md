# Custom Rounded Rectangle for Adobe After Effects (v1.1)

[![Live Browser Simulator](https://img.shields.io/badge/Live%20Demo-AE%20ScriptUI%20Simulator-2c8cff?style=for-the-badge&logo=javascript)](https://work2survive.github.io/AE_JS_builder/?jsx=custom_rounded_rect.jsx)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Adobe After Effects](https://img.shields.io/badge/Adobe%20After%20Effects-CS6%20--%20CC%202026%2B-9999FF?style=for-the-badge&logo=adobeaftereffects)](https://www.adobe.com/products/aftereffects.html)
[![Built with Antigravity](https://img.shields.io/badge/Built%20With-Antigravity-4285F4?style=for-the-badge&logo=google)](https://deepmind.google/)

**Live Demo**: [https://work2survive.github.io/AE_JS_builder/?jsx=custom_rounded_rect.jsx](https://work2survive.github.io/AE_JS_builder/?jsx=custom_rounded_rect.jsx)

A complete, production-grade tool, CEP extension, and expression engine for creating parametric vector rectangles with **individually customizable corner radii** in Adobe After Effects.

> **Live Browser Simulator**: Test and preview this tool interactively in the [AE JS & ScriptUI Simulator](https://work2survive.github.io/AE_JS_builder/?jsx=custom_rounded_rect.jsx) directly in your browser without launching After Effects!

Unlike After Effects' built-in `Rectangle Path` (which only supports a single global roundness value), this tool enables full, independent control of each corner (**Top-Left, Top-Right, Bottom-Right, Bottom-Left**) with mathematical **proportional scaling** to prevent overlapping curves, adhering to the **W3C CSS `border-radius` standard**.

---

## ✨ What's New in v1.1

- **Two Distinct ScriptUI Tabs**:
  - **Geometry Tab**:
    - Scrubbable sliders + numeric text inputs for Width & Height ($10\text{ to }2000\text{ px}$).
    - **Quick Dimension Presets**: Instant sizing for *Card (400×240)*, *Square (300×300)*, *HD (1920×1080)*, and *Banner (728×90)*.
    - **Link All Corners**: Master Radius slider driving all 4 corners simultaneously.
    - **2×2 Corner Radii Grid**: Independent sliders and fields for Top-Left, Top-Right, Bottom-Left, and Bottom-Right ($0\text{ to }500\text{ px}$).
    - **Quick Corner Presets**: *Capsule / Pill*, *Top Tab*, *Bottom Tab*, *Diagonal*, *Card 30px*, and *Sharp 0px*.
    - **W3C Prevent Overlap Toggle**: Mathematical proportional solver preventing corner collisions.
  - **Appearance Tab**:
    - **Interactive Color Swatches**: Real-time color swatches paired with direct hex inputs for both Fill and Stroke.
    - **Opacity & Stroke Controls**: Sliders for Fill Opacity ($0\text{--}100\%$), Stroke Width ($0\text{--}50\text{ px}$), and Stroke Opacity ($0\text{--}100\%$).
    - **0-Stroke Suppression**: Dynamic expression automatically suppresses stroke rendering when stroke width is 0.
- **14 Master Effect Controls**: Builds a clean Shape Layer driven by native Slider, Checkbox, and Color Effect Controls for full timeline keyframing and Essential Graphics / MOGRT export.
- **Legacy Version Preservation**: Previous version v1.0.0 is preserved in the `old/` directory.

---

## 📦 What's Included

```
AECustomRoundedRect/
├── CustomRoundedRect.jsx               # Standalone ScriptUI panel v1.1 (for direct AE usage)
├── custom_rounded_rect.jsx             # Matching alias for simulator integration
├── CustomRoundedRect.zxp               # Signed package ready for Adobe Exchange submission
│
├── old/                                # Legacy version archive
│   ├── CustomRoundedRect_v1.0.jsx      # Preserved legacy v1.0.0 ScriptUI script
│   └── CustomRoundedRect.jsx           # Legacy v1.0.0
│
├── extension/                          # Adobe CEP Extension source directory
│   ├── CSXS/
│   │   └── manifest.xml                # Adobe CEP Manifest (v9.0, After Effects CC 2018-2026+)
│   ├── .debug                          # Remote debugging configuration (port 8088)
│   ├── index.html                      # HTML5 panel UI matching AE dark theme
│   ├── css/
│   │   └── style.css                   # Responsive panel styling
│   ├── js/
│   │   ├── CSInterface.js              # Adobe CEP host communication library
│   │   └── main.js                     # Panel UI event handling and ExtendScript bridge
│   ├── jsx/
│   │   └── hostscript.jsx              # ExtendScript backend running inside After Effects
│   └── icons/                          # Menu & panel icons (iconNormal, iconDark, rollovers)
│
├── expressions/
│   ├── CustomRoundedRect_Expression.js # Standalone path expression (Effect Controls & hardcoded)
│   └── ResponsiveTextBox_Expression.js # Auto-sizing text box expression with sourceRectAtTime()
│
├── screenshot.png                      # Panel UI preview
├── README.md                           # Documentation & mathematical specification
└── LICENSE                             # MIT License
```

---

## 📐 Mathematical Specification

### 1. Proportional Overlap Prevention (W3C CSS Algorithm)

When corner radii are large relative to the rectangle's dimensions, curves along the same edge would collide and cross over each other. 

The tool implements the official [W3C CSS Backgrounds and Borders Module Level 3 (§5.5)](https://www.w3.org/TR/css-backgrounds-3/#corner-overlap) scaling algorithm:

$$\begin{aligned}
S_{\text{top}}    &= r_{TL} + r_{TR} \\
S_{\text{bottom}} &= r_{BL} + r_{BR} \\
S_{\text{left}}   &= r_{TL} + r_{BL} \\
S_{\text{right}}  &= r_{TR} + r_{BR}
\end{aligned}$$

A scaling factor $f$ is calculated:

$$f = \min\left(1,\; \frac{W}{S_{\text{top}}},\; \frac{W}{S_{\text{bottom}}},\; \frac{H}{S_{\text{left}}},\; \frac{H}{S_{\text{right}}}\right)$$

If $f < 1$, all radii are scaled by $f$:

$$r'_{TL} = r_{TL} \cdot f, \quad r'_{TR} = r_{TR} \cdot f, \quad r'_{BR} = r_{BR} \cdot f, \quad r'_{BL} = r_{BL} \cdot f$$

---

### 2. Cubic Bézier Circle Approximation

A 90-degree quadrant of a circle of radius $r$ is modeled using a cubic Bézier curve. The optimal control tangent length factor $\kappa$ that minimizes radial error is:

$$\kappa = \frac{4}{3}\left(\sqrt{2} - 1\right) \approx 0.5522847498307936$$

For each corner of radius $r$, the tangent handle length is $L = r \times \kappa$.

```
           P0 (xMin + rTL, yMin)        P1 (xMax - rTR, yMin)
               *----------------------------*
             /                                \
   P7       /                                  \      P2
(xMin,     *                                    *   (xMax,
 yMin+rTL) |                                    |    yMin+rTR)
           |                                    |
           |                                    |
   P6      *                                    *     P3
(xMin,      \                                  /    (xMax,
 yMax-rBL)   \                                /      yMax-rBR)
               *----------------------------*
           P5 (xMin + rBL, yMax)        P4 (xMax - rBR, yMax)
```

The path consists of **8 vertices** and tangent vectors (relative to each vertex):

| Vertex | Description | Position $[x, y]$ | In-Tangent | Out-Tangent |
| :--- | :--- | :--- | :--- | :--- |
| **$P_0$** | Top edge, TL curve end | $[-W/2 + r_{TL},\; -H/2]$ | $[-r_{TL} \cdot \kappa,\; 0]$ | $[0,\; 0]$ |
| **$P_1$** | Top edge, TR curve start | $[W/2 - r_{TR},\; -H/2]$ | $[0,\; 0]$ | $[r_{TR} \cdot \kappa,\; 0]$ |
| **$P_2$** | Right edge, TR curve end | $[W/2,\; -H/2 + r_{TR}]$ | $[0,\; -r_{TR} \cdot \kappa]$ | $[0,\; 0]$ |
| **$P_3$** | Right edge, BR curve start | $[W/2,\; H/2 - r_{BR}]$ | $[0,\; 0]$ | $[0,\; r_{BR} \cdot \kappa]$ |
| **$P_4$** | Bottom edge, BR curve end | $[W/2 - r_{BR},\; H/2]$ | $[r_{BR} \cdot \kappa,\; 0]$ | $[0,\; 0]$ |
| **$P_5$** | Bottom edge, BL curve start | $[-W/2 + r_{BL},\; H/2]$ | $[0,\; 0]$ | $[-r_{BL} \cdot \kappa,\; 0]$ |
| **$P_6$** | Left edge, BL curve end | $[-W/2,\; H/2 - r_{BL}]$ | $[0,\; r_{BL} \cdot \kappa]$ | $[0,\; 0]$ |
| **$P_7$** | Left edge, TL curve start | $[-W/2,\; -H/2 + r_{TL}]$ | $[0,\; 0]$ | $[0,\; -r_{TL} \cdot \kappa]$ |

The curve is closed with `createPath(points, inTangents, outTangents, true)`.

---

## 🚀 Installation

### Option 1: Dockable ScriptUI Panel
1. Copy `CustomRoundedRect.jsx` to:
   - **Windows**: `C:\Program Files\Adobe\Adobe After Effects <version>\Support Files\Scripts\ScriptUI Panels\`
   - **macOS**: `/Applications/Adobe After Effects <version>/Scripts/ScriptUI Panels/`
2. Restart After Effects.
3. Access from **Window > CustomRoundedRect.jsx**.

### Option 2: Run Without Installing
In After Effects, go to **File > Scripts > Run Script File...** and select `CustomRoundedRect.jsx`.

---

## 🔗 Related Projects

- **[AE JS & ScriptUI Simulator](https://github.com/work2survive/AE_JS_builder)**: Browser-based emulator for developing and testing After Effects ScriptUI panels and 2D/3D shape layers.
- **[AE Message Bubble Generator](https://github.com/work2survive/AE_message_bubble)**: Authentic, animated chat and text message bubbles for After Effects.
- **[AE Pie Slice Tool](https://github.com/work2survive/AE-Pie-Slice-Tool)**: Procedural circular sectors, arcs, and pie slices for After Effects.

---

---

## 🤖 Acknowledgments

Engineered, architected, and co-developed using **[Google DeepMind Antigravity](https://deepmind.google/)** agentic coding assistant.

## 📄 License

MIT License. Free for personal and commercial use in any motion design, broadcast, or VFX production.