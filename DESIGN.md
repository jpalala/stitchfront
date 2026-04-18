# StitchFront Shopify Engine

## 1. Vision
An AI-native design system bridge that converts high-level mood boards into 
functional Shopify storefront assets using FRUI (React) and Global CSS.

## 2. Visual Personas (Mood Boards)
The AI must strictly adhere to one of these three variable sets:

### A. Gadgets & Tech
- **Primary:** #00FF41 (Matrix Green) | **Surface:** #0A0A0A
- **Corner Radius:** 0px (Hard edges)
- **Typography:** 'Geist Mono', monospace
- **Vibe:** High-density, technical, raw data.

### B. Artsy & Boutique
- **Primary:** #D4A373 (Terracotta) | **Surface:** #FEFAE0
- **Corner Radius:** 16px (Organic)
- **Typography:** 'Fraunces', serif
- **Vibe:** Soft, warm, human-centric.

### C. Fashion & Editorial
- **Primary:** #000000 | **Surface:** #FFFFFF
- **Corner Radius:** 4px (Minimal)
- **Typography:** 'Inter', sans-serif (Extra Bold headings)
- **Vibe:** High-contrast, spacious, "Vogue" style.

## 3. Technical Constraints (Zero Trust)
- **CSS Delivery:** All styles must be delivered via CSS Variables (`:root`).
- **JS Delivery:** Use FRUI components wrapped in standard Web Components 
  to ensure compatibility with Shopify Liquid or Hydrogen.
- **Security:** Every request for a theme asset MUST be verified via 
  SHA-256 HMAC. Do not serve assets to unverified domains.

## 4. Component Rules (FRUI)
- Use `--brand-primary` for all `frui.Button` backgrounds.
- Use `--radius-ui` for all `frui.Input` and `frui.Card` borders.
- Spacing must follow a 4px scale (e.g., gap: var(--space-4)).
