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

Generate these raster image assets with the GPT Image 2 image-generation model, `gpt-image-2`. Do not hand-draw them, build them as SVG/HTML/CSS diagrams, render them locally in 3D software, or use another image model/provider unless the user explicitly asks for that fallback.

If `gpt-image-2` is unavailable in the current environment, stop and say so clearly instead of substituting a different production method. The reusable prompts below are the source of truth for image generation; adapt only the page title, page concept, dimensions, and any page-specific semantic objects.

When replacing existing docs assets, preserve the established output sizes unless the user asks otherwise: card images are `900 x 506` PNGs and cover images are `1990 x 480` PNGs. Save final generated files into `.gitbook/assets/` so GitBook references project-local assets, not temporary generated-image paths.

After generation, inspect the images before finalizing. Confirm that they follow the shared visual language, contain no text or labels, use the Burla cyan accent intentionally, remain readable at card size, and that cover-image outer edges match `#F8FBFC` without visible seams.

Use a minimal premium 3D product-render style. Images should feel like expensive SaaS documentation art: matte white objects, restrained geometry, soft studio lighting, subtle contact shadows, generous whitespace, and one saturated Burla cyan accent. The image should communicate the page concept quickly, even when seen at small size in a sidebar, card grid, or preview tile.

Prefer clear, iconic product-metaphor compositions over abstract decorative diagrams. Each image should have one dominant central object that represents the main concept, plus only the minimum supporting objects needed to explain the action. The viewer should understand the page concept in under a second at card size.

Use stronger visual hierarchy than a neutral workflow diagram: a large central subject, smaller supporting objects, and one or two thick Burla-cyan action arrows or connectors. Avoid spreading many small objects evenly across the frame. The image should feel like a polished 3D product icon, not a generic architecture diagram.

Objects should be semantically recognizable but still minimal: files can have subtle non-text document grooves, storage can use a simple cloud/storage mark, GPUs can use simplified chip geometry, and containers can use layered blocks. Do not use labels or real UI text. The image should communicate through shape, scale, and motion, not written explanation.

The target vibe is calm, expensive, and instantly understandable: matte white physical objects, soft bevels, compact composition, generous whitespace, subtle shadows, and a single saturated Burla-cyan accent showing the important action.

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

Cover images may include a wider left-to-right story than card images, but they should still remain iconic and sparse. Do not turn the cover into a complete process diagram. Use a few large, readable objects with clear hierarchy.

## Card Images

Card images and thumbnails should be simpler and more iconic than page covers. They need to remain understandable at small sizes.

Use one clear central concept, usually one primary object plus one simple visual action or relationship. Prefer fewer objects, larger shapes, and stronger silhouettes over detailed scenes. The goal is instant recognition, not a complete diagram.

Card images should read like a premium app icon or product object scene: one central concept, one clear action, and no more than two supporting objects. If the concept involves movement or data flow, use thick rounded cyan arrows that are short, bold, and easy to see at thumbnail size.

Card backgrounds may use clean white or near-white studio lighting when that better fits the card surface. Avoid hard rectangular seams, harsh cutouts, or compositions that look like a larger image was simply shrunk and pasted into the middle.

## Reusable Card Prompt

```text
Create a premium card image for the Burla docs page titled "<PAGE_TITLE>".

Use a minimal premium 3D product-render style for expensive SaaS documentation art. The image should communicate this concept: <PAGE_CONCEPT>. Make it extremely simple and readable at small thumbnail size.

Composition: one dominant central subject, one clear action, no more than two supporting objects, generous whitespace, strong silhouette, soft studio lighting, subtle contact shadows, matte white or near-white materials, and one saturated Burla cyan accent.

The image should feel like a polished 3D product icon, not a generic architecture diagram. Use semantically recognizable but minimal objects. Communicate through shape, scale, and motion, not text.

If the concept involves movement, transfer, routing, or data flow, use one or two thick rounded Burla-cyan arrows or connectors. Keep arrows short, bold, sculptural, and readable at thumbnail size.

Use Burla colors: cyan #7ECBDD for the main accent, muted blue-gray #3B5A64 only for subtle edges or shadows, and white/near-white for primary objects and background.

The image should feel polished, calm, reliable, expensive, and instantly understandable. It should not look like clipart, a generic stock tech image, a busy tutorial diagram, or a decorative abstract scene.

Constraints: no text, no labels, no logos, no people, no purple/lavender, no glossy glass, no glow effects, no particles, no complex network lines, no tiny UI details, no clutter, no Windows Vista styling.
```

## Reusable Cover Prompt

```text
Create a premium page cover image for the Burla docs page titled "<PAGE_TITLE>".

Use a minimal premium 3D product-render style for expensive SaaS documentation art. The image should communicate this concept: <PAGE_CONCEPT>. The composition should be wider and more spacious than a card image, while staying simple and immediately understandable.

The background must match the docs website background exactly: rgb(248, 251, 252) / #F8FBFC. All outer edges and corners must be this exact color so the cover blends into the page instead of looking like a rectangular image box. Blend shadows and studio lighting smoothly into that background with no clipped seams, pasted-in white panels, or visible image boundary.

Composition: very wide and short, one dominant central subject or focal group, no more supporting objects than needed, important objects centered vertically, generous empty space, no important detail near top or bottom edges, matte white or near-white objects, soft studio lighting, subtle contact shadows, restrained geometry, and one saturated Burla cyan accent.

The image should feel like a wide version of a polished 3D product icon, not a full workflow diagram. Use clear visual hierarchy: larger main object, smaller supporting objects, and one or two obvious action arrows or connectors when useful.

Use Burla colors: cyan #7ECBDD for the main accent, muted blue-gray #3B5A64 only for subtle edges or shadows, white/near-white for primary objects, and #F8FBFC for the page-matching background.

The image should feel polished, calm, reliable, expensive, and instantly understandable. It should look like custom product documentation art made specifically for this page, not generic AI art or stock cloud-computing imagery.

Constraints: no text, no labels, no logos, no people, no purple/lavender, no glossy glass, no glow effects, no particles, no complex network lines, no tiny UI details, no clutter, no Windows Vista gradients, no generic stock tech scene.
```
