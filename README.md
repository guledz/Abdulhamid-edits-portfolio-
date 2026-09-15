# Abdulhamid Hassen — Portfolio

**Edits that keep viewers watching.**

A conversion-focused portfolio for a freelance video editor and graphics
designer. The site's job is narrow: prove the work is good, make the results
visible, and make it effortless to start a conversation.

Live: [abdulhamidedits.com](https://abdulhamidedits.com) · Built with Next.js 14 (App Router), TypeScript, Tailwind CSS, Framer Motion.

---

## The problem

Video editors sell on proof, not copy. A portfolio has roughly three seconds
to answer two questions: *does this person's work look like what I want*, and
*what do I do next*.

Most editor portfolios fail at both. They open with a wall of text, bury the
reel behind a scroll-jacked landing page, and end with a contact form that
looks like a tax return.

**Target visitors:** YouTubers, short-form creators, brands, agencies — people
who have footage sitting on a drive and a deadline approaching.

**Primary goal:** book a call.
**Secondary goal:** get a quote request.

---

## Approach

### 1. Lead with the outcome, not the service list

The hero headline is the value proposition, not a job title:

> Edits that keep **viewers watching.**

Retention is the thing creators actually buy. Naming it in the first line
qualifies the visitor immediately — if they don't care about watch time, this
isn't the right editor, and the bounce is a *good* outcome.

The stats row sits directly under the CTAs. It currently mixes verified facts
(5+ years, Harari Broadcasting Network) with clearly-marked placeholders
(`[ADD NUMBER]`) rather than invented figures. Fabricated view counts are the
fastest way to lose a serious client who checks.

### 2. Two paths, two intents

| CTA | Intent | Destination |
| --- | --- | --- |
| **View work** | "Show me you can do this" | Work library |
| **Get a quote** | "I already want to hire you" | Contact form |

The header carries a phone CTA (`0922 094 236`) instead of a generic "Book a
call". For a freelance editor working locally and internationally, a tappable
phone number converts better on mobile than a scheduling link behind another
click.

### 3. Case studies in modals, not pages

Each project card opens a native `<dialog>` containing the video embed, the
brief, the approach, and — when available — real result metrics.

Native `<dialog>` with `showModal()` gives focus trapping, Escape-to-close,
and a real backdrop for free. No dialog library, no focus-management code,
no bundle cost. The tradeoff is browser support, which is fine in 2025 and
has a graceful fallback (`setAttribute("open", "")`).

### 4. Hover previews that don't break on touch

Preview clips play on hover *and* keyboard focus on desktop only, gated behind
`(hover: hover) and (pointer: fine)`. On touch devices, or when
`prefers-reduced-motion: reduce` is set, the preview never fires and the tap
goes straight to the modal.

Hover is an enhancement here, never the only way to reach content.

---

## Design system

| Token | Value | Role |
| --- | --- | --- |
| `--ink` | `#0E1112` | Page background |
| `--ink-soft` | `#15191B` | Cards, panels |
| `--ink-line` | `#242A2D` | Borders, dividers |
| `--fg` | `#F2F5F6` | Primary text |
| `--muted` | `#9BA5A9` | Body text, labels |
| `--accent` | `#FF6B2C` | CTAs, hover states, highlights |

**Type stack**

- **Anton** — headlines. Condensed, heavy, reads as broadcast titling.
- **Inter** — body copy. Neutral, high legibility at 14–18px.
- **JetBrains Mono** — labels, stats, metadata. Gives the UI an editor's
  timeline feel and visually separates data from prose.

**Spacing** runs on an 8-point grid. Section rhythm is `80px → 112px` vertical
padding, which is generous enough that the dark theme never feels cramped.

**One accent colour, used consistently.** Orange marks: primary buttons,
active nav state, active filter, project client names, result numbers, focus
rings. Nothing else. A second accent would dilute the signal.

### Contrast audit

| Pair | Ratio | Result |
| --- | --- | --- |
| `#9BA5A9` body on `#0E1112` | 7.3:1 | AAA |
| `#FF6B2C` accent on `#0E1112` | 6.6:1 | AA+ |
| `#0E1112` on `#FF6B2C` (button) | 6.6:1 | AA+ |
| `#F2F5F6` heading on `#0E1112` | 15.8:1 | AAA |

All text passes WCAG AA. The accent was chosen for contrast at this specific
background rather than picked from a palette and checked afterwards.

---

## Build notes

### Project structure

```
app/
  layout.tsx        Fonts, metadata, skip link
  page.tsx          Composition + JSON-LD
  globals.css       Tokens, base, reduced-motion
  sitemap.ts
  robots.ts
  api/contact/      Resend handler
components/
  Header.tsx        Sticky nav, scroll spy, mobile menu
  Hero.tsx
  FeaturedWork.tsx
  WorkLibrary.tsx   Client — filter state
  ProjectGallery.tsx
  ProjectCard.tsx
  ProjectModal.tsx
  Modal.tsx         Native <dialog> wrapper
  Services.tsx
  Testimonials.tsx
  About.tsx
  Contact.tsx
  ContactForm.tsx
  Reveal.tsx        Framer Motion + reduced-motion escape
lib/
  site.ts           All editable copy and config
  projects.ts       Work library data
  utils.ts
```

### Content lives in two files

Everything a non-developer needs to edit is in `lib/site.ts` and
`lib/projects.ts`. No CMS, no MDX pipeline, no database. For a portfolio with
a dozen projects this is the correct amount of infrastructure — adding Sanity
here would be resume-driven development.

The `Project` type is already shaped for a CMS migration if the list ever
grows past ~30 entries.

### Server components by default

Only four components ship JavaScript: `Header` (scroll spy, mobile menu),
`WorkLibrary`/`ProjectGallery` (filter + modal state), `ContactForm`, and
`Reveal`.

`Reveal` short-circuits entirely when `prefers-reduced-motion: reduce` is set
— it returns a plain `<div>` instead of a motion component, so the animation
library never initialises.

### Contact form

`app/api/contact/route.ts` posts to Resend. It validates server-side (email
shape, required fields, length caps), includes a honeypot that silently
accepts bot submissions so they don't retry, and fails loudly with a direct
email fallback if `RESEND_API_KEY` is missing.

The front end handles `sending` / `sent` / `error` states with an
`aria-live="polite"` region so screen readers announce the result.

---

## Accessibility

Built in, not bolted on.

- Skip link to `#main`, visible on focus
- Semantic landmarks: `<header>`, `<nav aria-label>`, `<main>`, `<footer>`
- Filter buttons use `aria-pressed` and are fully keyboard operable
- Modal uses native `<dialog>` → focus trap and Escape for free
- Preview videos are `aria-hidden` and `tabindex="-1"` — decorative only
- `:focus-visible` outlines at 2px accent with 3px offset
- `prefers-reduced-motion` disables all transitions, animations, and smooth
  scrolling
- Every image has an `alt`; icon-only buttons have `aria-label`

---

## Performance

- `next/image` with AVIF/WebP, explicit `sizes`, and `priority` only on
  above-the-fold media
- Videos are `preload="none"` with poster images — nothing downloads until
  the user interacts
- No dialog library, no animation library in the critical path
- `poweredByHeader` disabled, security headers set in `next.config.mjs`
- Target: Lighthouse 90+ across all four categories

---

## SEO

- Title template and meta description targeting "freelance video editor"
- Open Graph + Twitter cards with a 1200×630 cover
- `Person` + `WebSite` + `CreativeWork` JSON-LD graph
- Generated `sitemap.xml` and `robots.txt`
- `metadataBase` set so OG image URLs resolve absolutely

---

## What I'd do next

- **Wire the contact form** to a live Resend key (currently simulated so the
  success state is previewable)
- **Replace all `[ADD …]` placeholders** — real projects, thumbnails, embeds,
  result numbers, testimonials
- **Add a showreel** to the hero card and make it play inline
- **Client logo strip** as a trust signal between hero and featured work
- **Analytics** — track CTA clicks and modal opens to see which projects
  actually drive enquiries
- **Move to a CMS** only if the project count passes ~30

---

## Placeholder policy

This repo ships with real, verified facts where available and explicit
`[ADD NUMBER]` / `[ADD PROJECT TITLE]` markers everywhere else.

**No invented statistics. No fake testimonials. No fictional client names.**
Empty sections hide themselves automatically — the testimonials block
disappears entirely if the array is empty, and the featured grid collapses if
nothing is marked `featured: true`.

A portfolio that lies is worse than a portfolio with gaps. Gaps read as
"still building"; invented numbers read as "will lie to me later".

---

## Run locally

```bash
npm install
npm run dev
```

Create `.env.local`:

```bash
CONTACT_TO_EMAIL="your-email@example.com"
CONTACT_FROM_EMAIL="Portfolio <hello@yourdomain.com>"
RESEND_API_KEY="re_xxxxxxxxxxxx"
```

---

## License

Content and imagery © Abdulhamid Hassen. Code released under the MIT License.
