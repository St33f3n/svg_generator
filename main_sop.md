# SVG Creation Standard Operating Procedure for LLMs

## Quick Reference Decision Trees

### SVG Use Case Selection
```
What type of SVG are you creating?
├─ Simple icon (≤24px, single color) → Use Icon Guidelines (Section 3.1)
├─ Complex illustration (multi-element, detailed) → Use Illustration Guidelines (Section 3.2)
├─ Data visualization (charts, graphs) → Use Data Viz Guidelines (Section 3.3)
└─ Animated graphic → Use Animation Guidelines (Section 3.4)
```

### Embedding Method Selection
```
How will the SVG be used?
├─ Need CSS/JS control or animations → Inline SVG
├─ Simple static graphic → <img> tag
├─ Decorative background → CSS background-image
└─ Multiple instances of same icon → SVG sprite + <use>
```

### Accessibility Classification
```
Is this SVG decorative or informative?
├─ Decorative (visual enhancement only) → Add aria-hidden="true"
└─ Informative (conveys meaning) → Add role="img" + title/desc
```

---

## Core SVG Standards and Structure

### 2.1 Document Structure Template

**Standard SVG Document Structure:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" 
     xmlns:xlink="http://www.w3.org/1999/xlink"
     viewBox="0 0 100 100"
     role="img"
     aria-labelledby="title desc">
  <title id="title">Brief Description</title>
  <desc id="desc">Detailed Description</desc>
  
  <defs>
    <!-- Reusable elements: gradients, patterns, symbols -->
  </defs>
  
  <style>
    /* Embedded styles */
  </style>
  
  <!-- Main content organized in logical groups -->
  <g id="background">
    <!-- Background elements -->
  </g>
  
  <g id="main-content">
    <!-- Primary content -->
  </g>
</svg>
```

### 2.2 Essential Attributes

**Always Include:**
- `xmlns="http://www.w3.org/2000/svg"` (required for standalone SVGs)
- `viewBox="0 0 width height"` (essential for responsive scaling)
- `role="img"` (for informative SVGs)
- `aria-hidden="true"` (for decorative SVGs)

**Include When Needed:**
- `xmlns:xlink="http://www.w3.org/1999/xlink"` (when using xlink:href)
- `width` and `height` (for IE9-11 compatibility)
- `preserveAspectRatio` (default: "xMidYMid meet")

### 2.3 ViewBox Best Practices

**Recommended ViewBox Ranges:**
- Icons: `viewBox="0 0 24 24"` (Material Design standard)
- Illustrations: `viewBox="0 0 300 200"` (3:2 aspect ratio)
- Data visualizations: `viewBox="0 0 800 600"` (4:3 aspect ratio)

**ViewBox Rules:**
1. Use whole numbers when possible
2. Start at origin (0,0) unless cropping
3. Match aspect ratio to intended display ratio
4. Design at largest intended size, then scale down

---

## Use Case-Specific Guidelines

### 3.1 SVG Icons

**Core Requirements:**
- Base size: 24×24px (Material Design standard)
- Single color using `currentColor` for CSS control
- Pixel-aligned paths to prevent blurriness
- File size target: ≤2KB optimized

**Icon Template:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" 
     viewBox="0 0 24 24" 
     role="img">
  <title>Icon Name</title>
  <path d="..." fill="currentColor"/>
</svg>
```

**Icon Optimization Checklist:**
- [ ] Align all paths to pixel grid
- [ ] Use boolean operations to create single paths
- [ ] Remove unnecessary anchor points
- [ ] Use `currentColor` for fill/stroke
- [ ] Test at multiple sizes (16px, 24px, 48px)

### 3.2 SVG Illustrations

**Structure Strategy:**
- **Layer hierarchy**: Background → Detail → Effects → Interactive
- **Grouping**: Organize by function, not appearance
- **Naming**: Use semantic IDs (`id="tree-trunk"`, not `id="shape-1"`)

**Complex Illustration Template:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" 
     viewBox="0 0 400 300" 
     role="img"
     aria-labelledby="illus-title illus-desc">
  <title id="illus-title">Illustration Title</title>
  <desc id="illus-desc">Detailed description of illustration content</desc>
  
  <defs>
    <linearGradient id="sky-gradient">
      <stop offset="0%" stop-color="#87CEEB"/>
      <stop offset="100%" stop-color="#98FB98"/>
    </linearGradient>
  </defs>
  
  <g id="background">
    <rect width="100%" height="100%" fill="url(#sky-gradient)"/>
  </g>
  
  <g id="landscape">
    <!-- Landscape elements -->
  </g>
  
  <g id="characters">
    <!-- Character elements -->
  </g>
</svg>
```

**Performance Thresholds:**
- Simple illustrations: <50KB
- Complex illustrations: <200KB
- Breaking point: >1000 path points (consider PNG fallback)

### 3.3 SVG Data Visualization

**Foundation Setup:**
```javascript
// D3.js standard setup
const margin = {top: 20, right: 30, bottom: 40, left: 40};
const width = 800 - margin.left - margin.right;
const height = 400 - margin.top - margin.bottom;

const svg = d3.select("#chart")
    .append("svg")
    .attr("viewBox", `0 0 ${800} ${400}`)
    .attr("role", "img")
    .attr("aria-labelledby", "chart-title chart-desc");
```

**Data Visualization Requirements:**
- Always include chart title and description
- Use semantic colors with sufficient contrast (3:1 minimum)
- Implement keyboard navigation for interactive elements
- Provide data table alternative for complex charts

**Performance Guidelines:**
- <100 elements: Direct SVG manipulation
- 100-1000 elements: Optimize with virtual scrolling
- >1000 elements: Consider Canvas fallback

### 3.4 SVG Animations

**Animation Method Selection:**

| Animation Type | Recommended Approach | Use Case |
|---------------|---------------------|----------|
| Simple transitions | CSS transitions | Hover effects, loading states |
| Icon animations | CSS transforms | Scale, rotate, opacity changes |
| Path drawing | SMIL animate | Logo reveals, signatures |
| Complex sequences | GSAP/JavaScript | Interactive experiences |
| Data transitions | D3.js transitions | Chart updates, data changes |

**CSS Animation Template:**
```css
.icon {
  transition: transform 300ms ease-out, fill 200ms ease-out;
}
.icon:hover {
  transform: scale(1.1);
  fill: #007acc;
}
```

**SMIL Animation Template:**
```xml
<path d="..." stroke-dasharray="100" stroke-dashoffset="100">
  <animate attributeName="stroke-dashoffset" 
           from="100" to="0" 
           dur="2s" 
           fill="freeze"/>
</path>
```

**Animation Performance Rules:**
- Target 60fps (16.67ms per frame budget)
- Animate `transform` and `opacity` for hardware acceleration
- Use `will-change` sparingly and remove after animation
- Implement `prefers-reduced-motion` support

---

## Performance and Optimization

### 4.1 SVGO Configuration

**Recommended SVGO Settings:**
```javascript
export default {
  multipass: true,
  datauri: 'base64',
  js2svg: {
    indent: 2,
    pretty: false
  },
  plugins: [
    'preset-default',
    {
      name: 'preset-default',
      params: {
        overrides: {
          removeViewBox: false,  // Critical for responsive SVGs
          cleanupIds: false,     // Keep IDs for animations
          removeUselessStrokeAndFill: false
        }
      }
    }
  ]
};
```

### 4.2 File Size Targets

**Optimization Targets:**
- Icons: 0.5-5KB optimized
- Simple illustrations: 10-50KB optimized
- Complex illustrations: 50-200KB optimized
- Data visualizations: Variable (optimize for update speed)

**Optimization Priority:**
1. Remove unnecessary elements (highest impact)
2. Simplify paths and reduce precision to 3 decimal places
3. Optimize gradients and consolidate colors
4. Configure SVGO plugins appropriately
5. Enable gzip compression (additional 60-80% reduction)

### 4.3 Path Optimization Techniques

**Path Simplification:**
- Use design tool simplification: Object > Path > Simplify (91% precision)
- Reduce anchor points by 30-70% without visual degradation
- Replace complex curves with simpler approximations
- Position elements on whole pixel boundaries

**Color Optimization:**
- Use HEX for static colors (smallest file size)
- Use CSS variables for repeated colors
- Consolidate similar colors into shared palette
- Extract colors to external CSS for better compression

---

## Accessibility Requirements

### 5.1 WCAG 2.1 Compliance Patterns

**Decorative SVGs:**
```xml
<svg aria-hidden="true">
  <!-- Decorative content -->
</svg>
```

**Informative SVGs (Simple):**
```xml
<svg role="img">
  <title>Search</title>
  <path d="..."/>
</svg>
```

**Informative SVGs (Complex):**
```xml
<svg role="img" aria-labelledby="title desc">
  <title id="title">Monthly Sales Chart</title>
  <desc id="desc">Sales increased 15% from January to March 2024</desc>
  <!-- Chart content -->
</svg>
```

### 5.2 Color and Contrast Requirements

**Accessibility Standards:**
- Minimum contrast ratio: 3:1 (WCAG 2.1 AA for graphics)
- Don't rely solely on color to convey information
- Provide patterns/textures as color alternatives
- Test with colorblindness simulators

### 5.3 Animation Accessibility

**Required Implementations:**
```css
@media (prefers-reduced-motion: reduce) {
  .animated-element {
    animation: none;
  }
}
```

**Animation Constraints:**
- No auto-play animations longer than 5 seconds
- No flashing/blinking at 3+ times per second
- Provide pause controls for longer animations

---

## Cross-Browser Compatibility

### 6.1 Browser Support Matrix

**Universal Support (Safe to Use):**
- Basic SVG elements: `<path>`, `<rect>`, `<circle>`, `<ellipse>`, `<line>`, `<polyline>`, `<polygon>`
- Basic attributes: `fill`, `stroke`, `stroke-width`, `opacity`
- Transforms: `translate`, `scale`, `rotate`
- CSS styling of SVG elements

**Limited Support (Use with Caution):**
- SMIL animation (not supported in IE/Edge Legacy)
- SVG fonts (deprecated, being removed)
- External references in `<use>` elements (IE issues)

### 6.2 Cross-Browser Implementation

**Maximum Compatibility Template:**
```xml
<svg xmlns="http://www.w3.org/2000/svg" 
     xmlns:xlink="http://www.w3.org/1999/xlink"
     viewBox="0 0 100 100" 
     width="100" 
     height="100">
  <!-- Include both href and xlink:href for <use> elements -->
  <use href="#icon" xlink:href="#icon"/>
</svg>
```

**Fallback Strategy:**
```html
<picture>
  <source type="image/svg+xml" srcset="image.svg">
  <img src="fallback.png" alt="Description">
</picture>
```

### 6.3 Server Configuration

**Required MIME Type:**
```
Content-Type: image/svg+xml
```

**Recommended Headers:**
```
Content-Encoding: gzip
Cache-Control: public, max-age=31536000
```

---

## Workflow and Tools

### 7.1 Recommended Tool Chain

**Creation Tools (Choose One):**
- **Figma**: Best SVG export, web-based collaboration
- **Sketch**: Industry standard for UI/UX with plugins
- **Adobe Illustrator**: Professional vector graphics (requires optimization)
- **Inkscape**: Open source alternative

**Optimization Tools:**
- **SVGOMG**: Web interface for visual optimization
- **SVGO CLI**: Command-line batch processing
- **ImageOptim**: Mac drag-and-drop optimization

**Quality Assurance:**
- **W3C SVG Validator**: Standards compliance
- **axe DevTools**: Accessibility testing
- **WAVE**: Web accessibility evaluation

### 7.2 Build Integration

**Webpack Configuration Example:**
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.svg$/,
        use: [
          '@svgr/webpack',
          {
            loader: 'svgo-loader',
            options: {
              configFile: './svgo.config.js'
            }
          }
        ]
      }
    ]
  }
};
```

### 7.3 Asset Management Workflow

**Naming Convention:**
```
[category]-[name]-[variant].[size].svg
Examples:
- icon-search-filled.24.svg
- illustration-onboarding-welcome.svg
- chart-sales-quarterly.svg
```

**Version Control Strategy:**
- Store optimized SVGs in repository
- Use semantic versioning for library updates
- Commit accessibility enhancements separately
- Document breaking changes

---

## Common Pitfalls and Solutions

### 8.1 Critical Issues to Avoid

**Namespace Problems:**
```xml
<!-- WRONG: Missing namespace -->
<svg viewBox="0 0 100 100">

<!-- CORRECT: Proper namespace -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
```

**ViewBox Misconfiguration:**
```xml
<!-- WRONG: Content outside viewBox -->
<svg viewBox="0 0 50 50">
  <circle cx="75" cy="75" r="20"/> <!-- Clipped -->
</svg>

<!-- CORRECT: ViewBox encompasses all content -->
<svg viewBox="0 0 100 100">
  <circle cx="75" cy="75" r="20"/>
</svg>
```

**Accessibility Violations:**
```xml
<!-- WRONG: No accessibility information -->
<svg>
  <path d="..."/>
</svg>

<!-- CORRECT: Proper accessibility -->
<svg role="img" aria-labelledby="icon-title">
  <title id="icon-title">Save Document</title>
  <path d="..."/>
</svg>
```

### 8.2 Performance Mistakes

**File Size Bloat:**
- Excessive decimal precision (limit to 3 places)
- Unused gradients and definitions
- Invisible elements and metadata
- Unnecessary grouping

**Animation Performance Issues:**
- Animating layout-triggering properties
- Complex SVGs with many animated elements
- Missing hardware acceleration hints
- No reduced-motion support

### 8.3 Browser-Specific Traps

**Safari Issues:**
- Filter effects render poorly
- Text metrics differ from other browsers
- Animation performance degrades with complex paths

**Internet Explorer/Edge Legacy:**
- Requires width/height attributes
- External references in `<use>` don't work
- Limited CSS transform support

---

## Quality Assurance Checklist

### Pre-Deployment Verification

**Technical Validation:**
- [ ] Valid SVG syntax (W3C validator passes)
- [ ] Proper namespace declarations
- [ ] ViewBox encompasses all content
- [ ] File size within target range
- [ ] Optimized with SVGO or equivalent

**Accessibility Compliance:**
- [ ] WCAG 2.1 AA compliance verified
- [ ] Screen reader testing completed (NVDA, VoiceOver)
- [ ] Color contrast ratios meet 3:1 minimum
- [ ] Alternative text provided for informative graphics
- [ ] Animations respect prefers-reduced-motion

**Cross-Browser Testing:**
- [ ] Chrome/Chromium latest
- [ ] Firefox latest
- [ ] Safari latest
- [ ] Edge latest
- [ ] Mobile browsers (iOS Safari, Chrome Mobile)

**Performance Testing:**
- [ ] Render time <100ms for initial load
- [ ] Animation maintains 60fps
- [ ] File size meets category targets
- [ ] Gzip compression enabled

**Responsive Testing:**
- [ ] Scales properly at all target sizes
- [ ] Maintains aspect ratio
- [ ] Text remains legible at minimum size
- [ ] Interactive elements remain accessible

---

## Implementation Quick Start

### Immediate Actions (Day 1)
1. Choose appropriate SVG template from Section 2.1
2. Apply use case-specific guidelines from Section 3
3. Implement accessibility pattern from Section 5.1
4. Validate with checklist from Section 9

### Standard Workflow
1. **Create** using recommended tools (Section 7.1)
2. **Optimize** with SVGO configuration (Section 4.1)
3. **Validate** accessibility and technical requirements
4. **Test** across target browsers and devices
5. **Deploy** with proper server configuration (Section 6.3)

This SOP provides comprehensive, actionable guidelines for creating accessible, performant, and cross-browser compatible SVG graphics. Follow the decision trees and templates for consistent, high-quality results across all SVG use cases.
