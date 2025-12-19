# Design Aesthetics Guide

Avoid distributional convergence - the tendency toward generic, predictable design choices. Make unexpected, distinctive choices across all four dimensions.

## Typography

**Avoid:** Inter, Roboto, Open Sans, Lato, system-ui, default system fonts

**Prefer:**
- Distinctive typefaces: Playfair Display, Space Grotesk, IBM Plex, Outfit, Sora, Clash Display
- Extreme weight contrasts: pair 100 with 900, not 400 with 600
- Dramatic size jumps: 3x+ differences between hierarchy levels (e.g., 14px body, 48px heading)
- Mixed families: serif headings with sans-serif body, or vice versa

```css
/* Generic (avoid) */
font-family: Inter, system-ui, sans-serif;
font-weight: 500;

/* Distinctive (prefer) */
font-family: 'Space Grotesk', sans-serif;
font-weight: 800;
letter-spacing: -0.02em;
```

## Color & Theme

**Avoid:** Purple gradients, blue-to-purple, safe corporate palettes, too many colors

**Prefer:**
- Commit to a cohesive aesthetic via CSS variables
- Draw from cultural or domain aesthetics (brutalist, editorial, retro-computing, analog warmth)
- IDE themes as inspiration (Dracula, Nord, Catppuccin, Tokyo Night)
- Dominant color with sharp accent - not multiple competing colors
- High contrast: very dark backgrounds with bright accents, or light with bold color

```css
/* Generic (avoid) */
--primary: #6366f1;
--background: #f9fafb;

/* Distinctive (prefer) - Tokyo Night inspired */
--bg-primary: #1a1b26;
--bg-secondary: #24283b;
--accent: #7aa2f7;
--accent-warm: #ff9e64;
--text: #c0caf5;
```

## Motion

**Avoid:** Static pages, no transitions, jarring instant changes

**Prefer:**
- CSS-only for HTML; Motion (framer-motion) for React
- Orchestrated page loads with staggered reveals
- `animation-delay` for cascading effects
- Micro-interactions on hover/focus states
- Ease curves with personality: `cubic-bezier(0.34, 1.56, 0.64, 1)` for bounce

```tsx
/* Staggered reveal pattern */
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: { staggerChildren: 0.1 }
  }
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};
```

```css
/* CSS cascade */
.list-item {
  opacity: 0;
  animation: fadeSlideIn 0.4s ease-out forwards;
}
.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 50ms; }
.list-item:nth-child(3) { animation-delay: 100ms; }

@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}
```

## Backgrounds

**Avoid:** Solid colors (#ffffff, #f5f5f5), plain white/gray backgrounds

**Prefer:**
- Subtle gradients that create depth
- Geometric patterns (dots, grids, noise textures)
- Contextual atmospheric effects
- Layered backgrounds with multiple elements
- Grain/noise overlays for texture

```css
/* Generic (avoid) */
background: #ffffff;

/* Distinctive (prefer) */
background:
  radial-gradient(ellipse at top, rgba(120, 119, 198, 0.15), transparent),
  radial-gradient(ellipse at bottom, rgba(255, 158, 100, 0.1), transparent),
  #0f0f14;

/* Noise texture overlay */
.textured::before {
  content: '';
  position: absolute;
  inset: 0;
  background: url("data:image/svg+xml,...") repeat;
  opacity: 0.03;
  pointer-events: none;
}
```

## Quick Reference

| Dimension | Red Flag | Green Flag |
|-----------|----------|------------|
| Typography | Inter, Roboto, default weights | Distinctive font, extreme weight contrast |
| Color | Purple gradient, corporate blue | Cohesive theme, cultural reference |
| Motion | No animations, instant changes | Staggered reveals, micro-interactions |
| Background | Solid white/gray | Gradient layers, texture, depth |

## UX Principles

While being visually distinctive, ensure usability:

1. **Contrast ratios** - WCAG AA minimum (4.5:1 for text)
2. **Touch targets** - 44x44px minimum for interactive elements
3. **Loading states** - Skeleton screens over spinners
4. **Error states** - Clear, actionable feedback
5. **Reduced motion** - Respect `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```
