---
name: Impact Clarity
colors:
  surface: '#fbf9f9'
  surface-dim: '#dbdad9'
  surface-bright: '#fbf9f9'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f5f3f3'
  surface-container: '#efeded'
  surface-container-high: '#e9e8e7'
  surface-container-highest: '#e3e2e2'
  on-surface: '#1b1c1c'
  on-surface-variant: '#444748'
  inverse-surface: '#303031'
  inverse-on-surface: '#f2f0f0'
  outline: '#747878'
  outline-variant: '#c4c7c7'
  surface-tint: '#5f5e5e'
  primary: '#000000'
  on-primary: '#ffffff'
  primary-container: '#1c1b1b'
  on-primary-container: '#858383'
  inverse-primary: '#c8c6c5'
  secondary: '#1b6d24'
  on-secondary: '#ffffff'
  secondary-container: '#a0f399'
  on-secondary-container: '#217128'
  tertiary: '#000000'
  on-tertiary: '#ffffff'
  tertiary-container: '#410003'
  on-tertiary-container: '#ee4640'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#e5e2e1'
  primary-fixed-dim: '#c8c6c5'
  on-primary-fixed: '#1c1b1b'
  on-primary-fixed-variant: '#474746'
  secondary-fixed: '#a3f69c'
  secondary-fixed-dim: '#88d982'
  on-secondary-fixed: '#002204'
  on-secondary-fixed-variant: '#005312'
  tertiary-fixed: '#ffdad6'
  tertiary-fixed-dim: '#ffb4ac'
  on-tertiary-fixed: '#410003'
  on-tertiary-fixed-variant: '#93000e'
  background: '#fbf9f9'
  on-background: '#1b1c1c'
  surface-variant: '#e3e2e2'
typography:
  display-lg:
    fontFamily: Hanken Grotesk
    fontSize: 48px
    fontWeight: '800'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Hanken Grotesk
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-lg-mobile:
    fontFamily: Hanken Grotesk
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 36px
  headline-md:
    fontFamily: Hanken Grotesk
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-lg:
    fontFamily: Hanken Grotesk
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Hanken Grotesk
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  label-sm:
    fontFamily: Hanken Grotesk
    fontSize: 12px
    fontWeight: '600'
    lineHeight: 16px
    letterSpacing: 0.05em
  data-metric:
    fontFamily: Hanken Grotesk
    fontSize: 40px
    fontWeight: '800'
    lineHeight: 40px
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  unit: 8px
  container-max: 1200px
  gutter: 24px
  margin-mobile: 16px
  margin-desktop: 48px
  section-gap: 80px
---

## Brand & Style

The design system is centered on **Minimalism** and **Modern Professionalism**. It is designed to convey transparency, urgency, and institutional reliability for NGO reporting. By stripping away non-functional aesthetics—such as shadows, gradients, and micro-animations—the system ensures that data remains the absolute protagonist. 

The emotional response should be one of "Informed Trust." High contrast and a structured grayscale environment provide a sober backdrop, allowing the specific semantic colors (Success Green and Alert Red) to function as clear indicators of progress or concern. The visual language is flat and architectural, emphasizing the weight of the information presented.

The target audience consists of stakeholders, donors, and policy-makers who require rapid, unambiguous access to activity metrics and impact summaries.

## Colors

The palette is strictly functional. 
- **Primary:** Deep Onyx (#1a1a1a) for maximum legibility in typography and primary structural elements.
- **Semantic Positive:** Forest Green (#2e7d32) used exclusively for "upward" trends, goals met, and positive impact data.
- **Semantic Negative:** Madder Red (#c62828) reserved for "downward" trends, unmet targets, or critical alerts.
- **Grayscale Base:** A range of neutrals from pure white (#ffffff) to light grey surfaces (#f5f5f5) to ensure a clean, high-contrast environment. 

Do not use decorative colors. Use the primary color for all standard icons and UI borders.

## Typography

Using **Hanken Grotesk** across all levels ensures a cohesive, clean, and highly legible experience. 
- **Hierarchy:** Use bold weights (700-800) for headlines to create a strong "scannable" structure. 
- **Data Display:** Numerical metrics should use the `data-metric` style for maximum impact. 
- **Labels:** Small labels use uppercase and increased letter spacing to differentiate them from body text without needing color changes.
- **Tone:** Keep line heights generous (1.5x for body text) to ensure readability in long-form reports.

## Layout & Spacing

The layout follows a **Fixed Grid** model for desktop to ensure data visualizations maintain their intended proportions, transitioning to a fluid single-column layout for mobile.

- **Desktop (1200px+):** 12-column grid with 24px gutters.
- **Tablet (768px - 1199px):** 8-column grid with 20px gutters.
- **Mobile (< 767px):** 4-column fluid grid with 16px margins.

Spacing follows a strict 8px base unit. Section vertical spacing is intentionally large (80px+) to provide visual breathing room between distinct report chapters.

## Elevation & Depth

This design system uses **Flat Design** principles. There are no shadows or blurs. Depth is conveyed exclusively through:
- **Tonal Layering:** Using `#f5f5f5` for container backgrounds against a `#ffffff` page background.
- **High-Contrast Outlines:** 1px solid borders using `#e0e0e0` or `#1a1a1a` define boundaries.
- **Visual Weight:** Strategic use of bold typography and solid black blocks to anchor the eye.
- **Spatial Grouping:** Using white space (negative space) as the primary tool for separating content modules.

## Shapes

The shape language is **Soft** but disciplined. 
- **Standard UI elements** (Buttons, Cards, Inputs) use a `0.25rem` (4px) corner radius. 
- **Large report sections or featured images** can use `rounded-lg` (8px). 
- **Avoid** pill-shapes or fully circular elements, as they feel too "consumer-tech" and detract from the professional reporting tone. 
- **Icons** should be stroke-based, using a 2px consistent weight to match the border language.

## Components

- **Buttons:** Solid primary (#1a1a1a) background with white text for primary actions. Ghost buttons (black border, no fill) for secondary actions. No hover shadows; use a simple 80% opacity shift for interactions.
- **Data Cards:** 1px solid border (#e0e0e0), white background, no shadow. Header of the card should use `label-sm`.
- **Trend Indicators:** Small triangles (pointing up/down) placed next to metrics. Use Success Green for positive trends and Alert Red for negative trends.
- **Input Fields:** 1px solid border (#757575), rectangular with 4px radius. Focus state uses a 2px primary (#1a1a1a) border.
- **Lists:** Clean rows separated by 1px horizontal dividers (#e0e0e0). No bullet points; use 24px left-padding for hierarchy.
- **Progress Bars:** Flat 8px height bars. Background is #f5f5f5, fill is Success Green (#2e7d32).
- **Images:** All photographs or report imagery should use a consistent 4px or 8px corner radius to match the card style.