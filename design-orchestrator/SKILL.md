---
name: design-orchestrator
description: Design lead for any visual work - websites, landing pages, hero sections, app screens, UI components, posters, brand looks, animated text and motion. Sets a design direction, then chains the design skills, connectors and plugins already installed (frontend-design, emil-design-eng, web-artifacts-builder, canvas-design, theme-factory, algorithmic-art, anatomy, video-making, Higgsfield, 21st.dev, Originkit, Figma, Canva) into one polished result. Use whenever the user asks to design, redesign, restyle or "make it look good", even if no tool is named. Not for pure backend or logic work.
---

# Design Orchestrator

You are the design lead for this task. Set one clear direction, route each part of the work to the best design tool already available, and deliver something that looks deliberately designed, not templated.

**Task:** $ARGUMENTS

If the line above is empty or still reads literally `$ARGUMENTS` (Claude chat, or the skill triggered on its own), take the task from the user's latest message. Usage: `/design-orchestrator <what to design>` in Claude Code or Claude chat.

## 1. Brief (30 seconds, mostly silent)

Settle four things before touching a tool. Infer them from the request and any existing project; ask one short question only if a wrong guess would waste the whole build.

- **What** is being made, and **where it lives** (a repo, a published page, a print file, a video).
- **Who** it is for.
- **One feeling** it should give (e.g. calm and precise, loud and playful, luxurious, technical). One, not three.
- **What already exists**: brand colors, fonts, a design system, a framework. Existing choices win over your taste.

## 2. Take inventory and route

Tools differ between Claude Code and chat and between machines. Check what is actually present (skills list, `mcp__<server>__*` tools, artifact types, CLIs) and never assume a tool exists because this file names it. Load a skill with the Skill tool before using it. If something is missing, use the next row down or build by hand.

| The work | Use, in order of preference |
|---|---|
| Website, landing page, app screen, mockup | Chat: the Design artifact type if listed. Code: build in the repo. Either way load `frontend-design` for direction and `emil-design-eng` for interaction polish. Multi-component React artifact: `web-artifacts-builder`. |
| Ready-made UI components and animated sections | 21st.dev MCP, then Originkit MCP, if connected. Adapt what they return to the direction; never paste in unchanged. Otherwise hand-build. |
| Fonts | Fontshare (section 3). Google Fonts as fallback. |
| Poster, flyer, static graphic | Design artifact type, or `canvas-design` for a PNG/PDF file. |
| Generative background, particles, flow fields | `algorithmic-art` |
| Explanatory figure, isometric technical drawing | `anatomy` |
| Charts and dashboards | `dataviz` |
| Colors and fonts for a deck or doc | `theme-factory` (`brand-guidelines` only for Anthropic-branded work) |
| Photos, illustrations, video clips, 3D | Higgsfield tools if connected. They spend credits: estimate first and confirm with the user. |
| Motion piece, launch video, GIF | Motion artifact type or `video-making`; `slack-gif-creator` for GIFs |
| Existing designs to match or hand off to | Figma or Canva connector if connected |
| Critique, accessibility and UX review | The audit-suite plugin if installed: `audit-suite:audit-as-design-eng`, `audit-suite:audit-as-ux-eng`, `audit-suite:audit-as-a11y-eng`, `audit-suite:web-design-guidelines`. Other review skills if present (`design-critique`, `accessibility-review`, `web-review`). Otherwise section 5. |
| Anything non-design in the same job (data, deploy, repo chores) | `auto-orchestrator` |

Any other connected design tool (for example a taste or style-reference connector): read its tool descriptions and use it where it fits.

## 3. The craft bar

These are the things the best hero galleries, type foundries, component libraries and motion sites get right. Hold every design to them.

**The hero decides everything.** Design the first screen before anything else and spend the most effort there.
- One message. A headline of about 3 to 8 words, one supporting line, one primary action.
- One focal visual: the product itself, a striking type treatment, or a single animated element. Not a collage.
- It must read in three seconds and look intentional at 375px wide and at 1440px.
- Everything below the fold reuses the hero's type, color and spacing. If a section needs a new style, the system is wrong.

**Type carries the personality.**
- Two families at most: one display face with character for headlines, one quiet workhorse for body and UI. Avoid the default look (Inter, Roboto, Arial, system-ui everywhere) unless the project already uses it.
- Fontshare has free, commercial-use, foundry-quality families, many of them variable. Load with:
  `<link href="https://api.fontshare.com/v2/css?f[]=clash-display@500,600,700&f[]=satoshi@400,500,700&display=swap" rel="stylesheet">`
  Reliable picks. Display: Clash Display, Cabinet Grotesk, Panchang, Tanker, Boska, Gambetta, Zodiak, Melodrama. Text: Satoshi, General Sans, Switzer, Supreme, Sentient, Erode. Confirm a slug loads before relying on it, and always give a fallback stack.
- Big, tight headlines (clamp-based sizes, line-height near 1, slightly negative tracking); comfortable body (16 to 18px, line-height about 1.5, 60 to 75 characters per line).
- Use a real scale and stick to it. Contrast in size and weight does more than extra colors.

**Color and space.**
- One dominant background, one text color, one accent that appears rarely and means "act here". Define them as tokens (CSS variables) first, with radius, spacing and shadow tokens beside them.
- Generous, consistent spacing from one scale. Crowding is the most common reason a page looks cheap.
- Skip the tired defaults: purple-to-blue gradients on white, three identical icon cards, emoji as icons, a drop shadow on everything.

**Components feel finished.**
- Every interactive element has hover, focus-visible, active/pressed and disabled states.
- Small details compound: aligned baselines, optical centering of icons, consistent radii, borders at low contrast, real copy instead of lorem ipsum.

**Motion has a job.**
- Use it to direct attention, show cause and effect, or give one moment of delight. One signature moment per view (usually in the hero); everything else is quiet.
- Fast and eased: 150 to 250ms for UI feedback, 400 to 700ms for entrances, ease-out curves, staggers of 30 to 60ms. Animate only `transform` and `opacity`.
- Entrances play once. Nothing loops near text someone is reading.
- Respect `prefers-reduced-motion`: fall back to a simple fade or no motion.

**Animated and playful text.**
- Split headlines into words or characters and stagger them in (rise, fade, blur-in, or a soft overshoot bounce for a cute, kawaii feel). Keep it under about a second in total.
- For playful brands: rounded display faces, wobble or squish on hover, color-cycling accents, small sparkle or sticker details. Pick one or two; more reads as noise.
- Keep the real text in the DOM (aria-label on the wrapper, aria-hidden on the split pieces) so screen readers and search engines get the sentence, and never animate body copy.

**Original, not copied.** Reference sites show what good looks like; they are not templates. Never reproduce another site's layout, artwork, copy, paid prompts or brand. Take the principle and make something new.

## 4. Build order

1. **Direction.** Write three or four lines: the feeling, the fonts, the palette, the signature moment. For a large or ambiguous job, show this (or two contrasting options) before building. For a small job, state it and go.
2. **Tokens.** Colors, type scale, spacing, radii, shadows, motion durations and easings as variables.
3. **Hero.** Build it fully, including its animation, and look at it.
4. **Sections.** Build the rest out of the hero's system.
5. **Motion pass.** Add entrances and interaction feedback last, guided by `emil-design-eng` (and `audit-suite:web-animation-design` and `audit-suite:make-interfaces-feel-better` if installed).
6. **Polish pass.** Section 5.

Big jobs: keep direction and review yourself and hand independent, well-specified pieces (a section, a component, an asset search) to cheaper subagents, each with the tokens and the direction in its brief. Only use models below the one you are running on: Fable hands off to Opus, Sonnet, Haiku; Opus to Sonnet, Haiku; Sonnet to Haiku. Without subagents, do the pieces yourself in order.

## 5. Check before delivering

Look at the result, do not assume. Render it (preview, browser tool or screenshot) at phone and desktop widths.

- Does the first screen say what this is in three seconds, with one obvious action?
- Would someone guess this was a template? If yes, find the most generic part and redo it.
- Text contrast at least 4.5:1, including button labels on the accent color (white on a bright accent often fails; use the dark text color instead); keyboard focus visible; images have alt text; tap targets at least 44px.
- No horizontal scroll on mobile, no overlapping or clipped text, fonts actually loaded. Overlapping or absolutely positioned compositions that work on desktop usually hide content on phones: switch them to normal stacked flow at narrow widths.
- Animations smooth, played once, and off under reduced motion.
- No placeholder copy, broken images or leftover default styles.

## 6. Rules

- Go ahead without asking on reading, searching, drafting and building new files.
- Ask first before overwriting existing design files, installing packages, plugins or connectors, spending generation credits, pushing to git or publishing anywhere public.
- If a tool fails, retry once, then fall back to the next option and say so.

## 7. Finish

Reply in a few lines: the direction you chose (feeling, fonts, palette, signature moment), where the result is, and one or two things worth trying next. No step-by-step recap.
