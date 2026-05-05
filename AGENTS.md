# AGENTS.md

## User-facing site review

Before finalizing any user-facing change to this docs site, review it from the perspective of a person using the site.

Ask:

- What is the user trying to understand or do here?
- Does this change make that easier, clearer, or more compelling?
- Does it introduce confusion, awkward hierarchy, unnecessary friction, or misleading structure?
- Does the result match the user's mental model, not just the implementation structure?
- If I encountered this fresh, would it feel intentional and useful?

If the review reveals a user-experience issue, revise the change before calling it done. Briefly mention the user-experience review and any tradeoff considered when summarizing the work.

## How-to guide image style

For Burla how-to guide card and cover images, use a minimal premium 3D product-render style. The images should feel like expensive SaaS documentation art: matte white objects, clean white background, soft studio light, subtle contact shadows, and one saturated Burla cyan accent. Avoid generic tutorial clipart, stock cloud-computing scenes, glossy glass, glow effects, particles, purple/lavender gradients, many tiny objects, or "Windows Vista" styling.

Use these colors:

- Burla cyan: `#7ECBDD`
- Muted blue-gray for only subtle edges/shadows: `#3B5A64`
- Background and primary objects: white / near-white

For the `Read/Write Files to Cloud Storage` guide, the approved design philosophy is one large matte white file with simple cyan read/write arrows. The card image is intentionally simple: one file tile in the center, one cyan arrow entering from the left, one cyan arrow leaving to the right. The cover image expands the same concept horizontally: file on the left, cloud-storage block in the center, file on the right, with thick cyan arrows connecting them.

Reusable card prompt:

```text
Create a premium GitBook card cover image for the Burla how-to guide "Read/Write Files to Cloud Storage". Concept: One File, Two Arrows. Use minimal premium 3D product-render style for an expensive SaaS documentation card. Show one large matte white document tile in the center. Add exactly two simple cyan arrows: one entering the file from the left and one leaving the file to the right, representing read and write. The arrows should be thick, sculptural, rounded, and clean, not clipart. Use Burla colors: cyan #7ECBDD for arrows, muted blue-gray #3B5A64 only for very subtle edge/shadow contrast, white/near-white background and file. Keep the composition extremely simple and readable at small card size. Constraints: no text, no labels, no logos, no folder, no cloud, no laptop, no code, no people, no purple/lavender, no glass, no glow, no particles, no extra documents, no complex diagram, no Windows Vista styling.
```

Reusable cover prompt:

```text
Create a premium GitBook page cover image for the Burla how-to guide "Read/Write Files to Cloud Storage". This is the wide article cover companion to a card image that shows one matte white file with simple cyan arrows. Keep the same design philosophy: minimal premium 3D product render, matte white objects, clean white background, Burla cyan #7ECBDD accents, muted dark blue-gray #3B5A64 only for subtle shadows and edges, no clutter. Composition must be very wide and short for a GitBook cover: important objects centered vertically, generous empty space, no important detail near top/bottom edges. Visual concept: one white file tile on the left, a simple matte white cloud-storage block in the center, and one white file tile on the right. A thick rounded cyan arrow flows from the left file into the cloud-storage block, and another thick rounded cyan arrow flows from the cloud-storage block to the right file. The cloud-storage block should be a simple storage volume with a cloud silhouette embossed or recessed, not a cartoon cloud. Objects should be large, clean, and readable even when the cover is cropped. Emotional target: simple, reliable, polished, expensive product documentation. Constraints: no text, no labels, no logos, no terminal text, no code symbols, no laptop unless absolutely minimal, no people, no purple/lavender, no glass transparency, no glow, no particles, no many files, no complex network lines, no Windows Vista gradients, no generic stock cloud computing scene.
```
