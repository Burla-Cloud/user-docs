# AGENTS.md

## User-Facing Site Review

Before finalizing any user-facing change to this docs site, review it from the perspective of a person using the site.

Ask:

- What is the user trying to understand or do here?
- Does this change make that easier, clearer, or more compelling?
- Does it introduce confusion, awkward hierarchy, unnecessary friction, or misleading structure?
- Does the result match the user's mental model, not just the implementation structure?
- If I encountered this fresh, would it feel intentional and useful?

If the review reveals a user-experience issue, revise the change before calling it done. Briefly mention the user-experience review and any tradeoff considered when summarizing the work.

## Site Image Style

All cover images, card images, example thumbnails, article images, post images, and page artwork across the Burla docs site should share one visual design language.

Use a minimal premium 3D product-render style. Images should feel like expensive SaaS documentation art: matte white objects, restrained geometry, soft studio lighting, subtle contact shadows, generous whitespace, and one saturated Burla cyan accent. The image should communicate the page concept quickly, even when seen at small size in a sidebar, card grid, or preview tile.

Avoid generic tutorial clipart, stock cloud-computing scenes, busy diagrams, glossy glass, heavy gradients, glow effects, particles, purple/lavender palettes, many tiny objects, fake UI clutter, photorealistic people, or "Windows Vista" styling.

Use these colors:

- Burla cyan: `#7ECBDD`
- Muted blue-gray for only subtle edges/shadows: `#3B5A64`
- GitBook page background: `rgb(248, 251, 252)` / `#F8FBFC`
- Primary objects: white / near-white

## Cover Images

Page cover images should match the docs website background exactly: `rgb(248, 251, 252)` / `#F8FBFC`.

The cover should feel like the rendered objects are floating directly on the page, not like a rectangular image box sitting inside the page. Keep all outer edges and corners at the exact site background color, and blend any studio lighting or shadows smoothly into that color.

Do not leave visible clipped edges, rectangular seams, pasted-in white panels, or a different white/gray background around the artwork.

Cover images should be wide and short. Keep the important objects centered vertically with generous horizontal breathing room. Avoid placing important details near the top or bottom edges, since GitBook may crop or scale the image depending on viewport size.

## Card Images

Card images and thumbnails should be simpler and more iconic than page covers. They need to remain understandable at small sizes.

Use one clear central concept, usually one primary object plus one simple visual action or relationship. Prefer fewer objects, larger shapes, and stronger silhouettes over detailed scenes. The goal is instant recognition, not a complete diagram.

Card backgrounds may use clean white or near-white studio lighting when that better fits the card surface. Avoid hard rectangular seams, harsh cutouts, or compositions that look like a larger image was simply shrunk and pasted into the middle.

## Reusable Card Prompt

```text
Create a premium card image for the Burla docs page titled "<PAGE_TITLE>".

Use a minimal premium 3D product-render style for expensive SaaS documentation art. The image should communicate this concept: <PAGE_CONCEPT>. Make it extremely simple and readable at small thumbnail size.

Composition: one clear central subject, generous whitespace, strong silhouette, minimal objects, soft studio lighting, subtle contact shadows, matte white or near-white materials, and one saturated Burla cyan accent.

Use Burla colors: cyan #7ECBDD for the main accent, muted blue-gray #3B5A64 only for subtle edges or shadows, and white/near-white for primary objects and background.

The image should feel polished, calm, reliable, and expensive. It should not look like clipart, a generic stock tech image, or a busy tutorial diagram.

Constraints: no text, no labels, no logos, no people, no purple/lavender, no glossy glass, no glow effects, no particles, no complex network lines, no tiny UI details, no clutter, no Windows Vista styling.
```

## Reusable Cover Prompt

```text
Create a premium page cover image for the Burla docs page titled "<PAGE_TITLE>".

Use a minimal premium 3D product-render style for expensive SaaS documentation art. The image should communicate this concept: <PAGE_CONCEPT>. The composition should be wider and more spacious than a card image, while staying simple and immediately understandable.

The background must match the docs website background exactly: rgb(248, 251, 252) / #F8FBFC. All outer edges and corners must be this exact color so the cover blends into the page instead of looking like a rectangular image box. Blend shadows and studio lighting smoothly into that background with no clipped seams, pasted-in white panels, or visible image boundary.

Composition: very wide and short, important objects centered vertically, generous empty space, no important detail near top or bottom edges, matte white or near-white objects, soft studio lighting, subtle contact shadows, restrained geometry, and one saturated Burla cyan accent.

Use Burla colors: cyan #7ECBDD for the main accent, muted blue-gray #3B5A64 only for subtle edges or shadows, white/near-white for primary objects, and #F8FBFC for the page-matching background.

The image should feel polished, calm, reliable, and expensive. It should look like custom product documentation art made specifically for this page, not generic AI art or stock cloud-computing imagery.

Constraints: no text, no labels, no logos, no people, no purple/lavender, no glossy glass, no glow effects, no particles, no complex network lines, no tiny UI details, no clutter, no Windows Vista gradients, no generic stock tech scene.
```
