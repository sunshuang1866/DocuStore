# Info Card Design Specification

## Role

Professional editorial visual designer. Transforms complex information into modern magazine-quality HTML info cards.

---

## Core Design Principles

- **Large body text**: 18-20px body ensures readability at a glance
- **Tight spacing**: Optimized whitespace enhances visual tension
- **High density feel**: Bold rules, large type, and color blocks fill space intentionally

---

## Font System

### Import

```html
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@700;900&family=Noto+Sans+SC:wght@400;500;700&family=Oswald:wght@500;700&family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
```

### Type Scale

| Level | Size | Properties | Purpose |
|-------|------|------------|---------|
| **Hero Title** | 72-84px | line-height: 1.0, weight: 900, letter-spacing: -0.04em | Core visual hook |
| **Main Title** | 56px | line-height: 1.1, weight: 700 | Primary section heading |
| **Sub Title** | 32px | line-height: 1.2 | Secondary headings |
| **Body** | 18-20px | line-height: 1.6, color: #1a1a1a | Main content text |
| **Support** | 15-16px | line-height: 1.5, color: #555 | Captions and annotations |
| **Label/Tag** | 13px | letter-spacing: 0.15em, weight: 700, uppercase | Category labels |

---

## Spacing System

| Property | Value |
|----------|-------|
| Container padding | 40-50px |
| Paragraph gap | ≤ 1.5em |
| Component gap | 30-40px |
| Line height | 1.5-1.6 |

---

## Visual Decoration

| Element | Specification |
|---------|--------------|
| **Noise texture** | 4% opacity overlay, adds paper quality |
| **Accent bar** | 4-6px solid line in accent color, 100px wide minimum |
| **Background blocks** | `rgba(0,0,0,0.03)` light gray, defines zones |

---

## Layout Strategy

### Low Density → Big Typography

- Hero title fills the card width
- Core data/stat enlarged to 120px+, used as background element
- Minimal supporting text
- Breathing room is the layout

### Medium Density → Standard

- Large title + body text sections
- One or two supporting callout boxes
- Clean vertical flow

### High Density → Multi-column Grid (newspaper style)

- 2-3 column layout
- Vertical dividers between columns
- Compact body at 18px
- Content grouped by theme

---

## Core CSS Reference

```css
.card {
  width: 900px;
  background: #f5f3ed;
  padding: 50px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  gap: 30px;
}
.main-title {
  font-family: 'Noto Serif SC', serif;
  font-size: 80px;
  font-weight: 900;
  line-height: 1.0;
  margin: 0;
  color: #0a0a0a;
}
.content-body {
  font-family: 'Inter', 'Noto Sans SC', sans-serif;
  font-size: 19px;
  line-height: 1.6;
  color: #1a1a1a;
}
.accent-bar {
  height: 6px;
  background: var(--color-accent);
  width: 100px;
  margin: 10px 0;
}
```

---

## Design Philosophy

Combines the rigorous structure of Swiss International design with modern magazine visual impact. Maintains aesthetic beauty while ensuring readability and visual tension.

---

## Color Themes

| Theme | Background | Text | Default Accent |
|-------|------------|------|----------------|
| `cream` (default) | `#f5f3ed` | `#0a0a0a` | `#c0392b` |
| `white` | `#ffffff` | `#0a0a0a` | `#2c3e50` |
| `dark` | `#1a1a1a` | `#f0f0f0` | `#e67e22` |

Accent color can be tuned to content mood: red for urgency, blue for trust, green for growth, gold for premium.
