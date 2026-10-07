---
title: New Text Box Properties
date: 2026-10-28
tags:
  - CSS
  - layout
baseline: true
---

When working with web content, extra space around text can make precise alignment difficult. For years, achieving pixel-perfect typography on the web has required unpredictable line-height hacks and fragile negative margins.

This post will explore the new [text-box-trim](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/text-box-trim) and [text-box-edge](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/text-box-edge) CSS properties and how they solve these invisible whitespace blocks built into web fonts.

## The Problem

The leading is the difference between the declared line-height and the font's ascent-plus-descent metrics. The browser divides that difference equally above and below the text. Each half is called "half-leading".

The half-leading contributes to the visible space between the text and nearby content, in addition to the margins you set.

This distribution creates a gap between the visual top of your text and the actual edge of its DOM container.

### The Old Solution: Negative Margins

To align a heading with an adjacent image or grid line, developers traditionally guessed the height of this gap and pulled the text back into place using arbitrary negative margins, such as `margin-top: -0.4em`.

This approach is fragile. The negative margin "magic number" breaks immediately if the web font fails to load and falls back to a system font, the text wraps to a second line, or the user changes their default font size.

### The New Way: Trimming the Box

<baseline-status featureid="text-box"></baseline-status>

The `text-box` property reduces the need for manual offsets by trimming the text box to specified font metrics. When you apply `text-box: trim-both cap alphabetic`, this trims the text box at both edges to the cap-height and alphabetic baseline. This aligns the text-box edges with the selected font metrics, making spacing with CSS gaps and padding more predictable.

### Margin Collapsing vs. Half-Leading

It's important to clarify that `text-box` does not replace the need for `margin` or `margin-block` to separate stacked elements like paragraphs, nor does it override CSS margin collapsing. Instead, it makes your standard margins predictable.

Consider the following standard CSS applied to adjacent paragraphs in a block layout:

```css
p {
  margin-top: 2em;
  margin-bottom: 1em;
}
```

When two of these paragraphs are stacked vertically, their touching margins collapse into a single margin equal to the largest value. The physical space between the containers will be exactly `2em` (not `3em`).

Without `text-box-trim`, that `2em` gap isn't visually accurate. The true visual distance between the letters of Paragraph A and the letters of Paragraph B is `2em` plus the invisible half-leading baked into the bottom of the first paragraph and the top of the second.

Applying `text-box-trim: trim-both` enables trimming on both sides of the paragraphs. The amount removed depends on `text-box-edge`, which specifies the font metrics to trim to. The outer margin calculations remain unchanged: the margins still collapse to `2em` between the paragraph boxes, whose text edges are trimmed according to `text-box-edge`.

## How They Work Together

Think of these two properties as a pair where one holds the scissors and the other holds the ruler:

* **text-box-trim (The Scissors)**: Determines which logical edges of the text container to trim: the block-start edge (`trim-start`), the block-end edge (`trim-end`), both (`trim-both`), or neither (`none`). In top-to-bottom, left-to-right (tb-lr) writing modes, these correspond to the top and bottom edges.
* **text-box-edge (The Ruler)**: Determines where those edges are positioned relative to the font's metrics. For example, `cap alphabetic` places the over edge at the cap height and the under edge at the alphabetic baseline.

Because they are two halves of the exact same layout feature, they are almost always used together via the `text-box` shorthand.

### Syntax

Written separately:

```css
h1 {
  text-box-trim: trim-both;
  text-box-edge: cap alphabetic;
}
```

Written as a shorthand:

```css
h1 {
  /* [trim] [over-edge] [under-edge] */
  text-box: trim-both cap alphabetic;
}
```

## Practical Examples

Here are five real-world layout challenges where the `text-box` property can help.

### Aligning an Icon with a Heading

Aligning a scalable vector graphic (SVG) icon with a text heading can require brittle margin adjustments. Trimming the text box to font metrics can help `align-items: center` align the heading more closely with the icon.

```css
.header-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.header-icon {
  width: 2rem;
  height: 2rem;
}

.header-title {
  font-size: 2rem;
  line-height: 1.5;
  text-box: trim-both cap alphabetic;
}
```

### Vertical Centering in UI Elements

Buttons, badges, and pills can look visually off-center because the space above and below the visible text can be uneven. Trimming both edges can help flexbox or grid centering align more closely with the visible text.

Keep the badge as a flex container for layout and apply trimming to the child containing the label.

```html
<span class="status-badge">
  <span class="status-badge-label">Published</span>
</span>
```

```css
.status-badge {
  display: inline-flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background-color: var(--surface-highlight);
  border-radius: 999px;
}

.status-badge-label {
  text-box: trim-both cap alphabetic;
}
```

### Multiline Paragraph Grids

When placing paragraphs inside a CSS Grid, the leading at the top of the first line and the bottom of the last line prevents the text from sitting flush against the grid cell edges. `text-box-trim` only trims the outer edges of the container, leaving the comfortable line-height between the internal lines perfectly intact.

```css
.grid-card {
  display: grid;
  padding: 1.5rem;
  background: var(--surface-color);
}

.card-description {
  font-size: 1rem;
  line-height: 1.6;
  text-box: trim-both ex alphabetic;
}
```

The `ex alphabetic` values place the top edge at the font's x-height and the bottom edge at its alphabetic baseline. Capitals and ascenders can extend above the trimmed box, while descenders can extend below it. Trimming changes the box's edges; it does not clip the glyphs, so those characters remain visible unless a separate clipping rule hides them.

### Flush Container Edges in Card Layouts

When nesting text inside a padded container, the font's internal leading combines with the container's padding to create uneven visual spacing. Trimming the start of the first element and the end of the last element can make the text align more predictably with the declared padding.

Reset the heading and paragraph margins so the card's padding and gap control the spacing.

```html
<article class="card">
  <h2 class="card-title">Card heading</h2>
  <p class="card-description">A description of the card's content.</p>
</article>
```

```css
.card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 2rem;
}

.card-title,
.card-description {
  margin: 0;
}

.card-title {
  font-size: 1.5rem;
  font-weight: 700;
  text-box: trim-start cap alphabetic; /* Trim the start edge to the cap-height */
}

.card-description {
  font-size: 1rem;
  line-height: 1.5;
}

.card-description:last-child {
  text-box: trim-end text alphabetic; /* Trim the end edge to the alphabetic baseline */
}
```

### Precise Alignment with Adjacent Media

Aligning large hero text with the top edge of an adjacent image can require fragile negative margins. Trimming the start edge to the font's cap-height can help align the capital letters more closely with the top edge of the image.

```css
.hero-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: start;
}

.hero-image {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
}

.hero-headline {
  font-size: 5rem;
  line-height: 1.1;
  margin: 0;
  text-box: trim-start cap alphabetic; /* Trim the block-start edge to the cap-height */
}
```

## Browser Support and Fallbacks

As a Baseline Newly Available feature in 2026, the `text-box` property enjoys wide support across modern browser engines. It is fully supported in Safari 18.2+, Chrome 133+, Edge 133+, and Firefox 154+.

However, to ensure your layouts do not break in older browsers, it is a best practice to use `@supports` feature queries. You can apply standard padding or margin adjustments as a fallback, and then override those adjustments only when the browser understands `text-box-trim`.

The negative margin in this fallback is only an approximation. Its value depends on the font and layout, so test it with fallback fonts, different text sizes, and multiline headings rather than treating `-0.2em` as a reusable value.

```css
.hero-headline {
  font-size: 5rem;
  line-height: 1.1;
  margin: 0;

  /* Fallback for older browsers: arbitrary negative margin */
  margin-top: -0.2em;
}

/* Feature query for modern browsers */
@supports (text-box: trim-start cap alphabetic) {
  .hero-headline {
    /* Reset the negative margin fallback */
    margin-top: 0;

    /* Apply the precise trim */
    text-box: trim-start cap alphabetic;
  }
}
```
