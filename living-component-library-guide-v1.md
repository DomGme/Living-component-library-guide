# Building with a Living Component Library
### Use Claude Code to auto-generate, edit, and propagate a design system from day one

---

## What This Is

Every time you build with Claude, a shared component library grows automatically. Any change to a component propagates across your entire product — the same way editing a Figma library component updates every instance.

No retrofitting. No separate design system project. The library IS the product.

---

## The Core Idea

```
  "Build me a settings page"

  Claude checks the library:

  ┌──────────────────────────────────┐
  │  COMPONENT LIBRARY               │
  │                                  │
  │  ✓ Button (exists)    → reuse    │
  │  ✓ Card (exists)      → reuse    │
  │  ✓ Input (exists)     → reuse    │
  │  ✗ Toggle (new)       → create   │
  │  ✗ SettingsRow (new)  → create   │
  └──────────────────────────────────┘

  New components get added to the library.
  Next time anyone needs a Toggle, it already exists.
```

---

## The Foundation: shadcn/ui

Your library starts from [shadcn/ui](https://ui.shadcn.com) — 57 accessible React components you copy into your project and own. Once a component enters your library, it's yours to modify. shadcn is the well you draw from. Your library is the water tank. Your product only drinks from the tank.

```
  BASE SYSTEM          YOUR LIBRARY           YOUR PRODUCT
  (the well)           (the tank)             (drinks from tank only)

  ┌────────────┐       ┌────────────┐         ┌────────────┐
  │ shadcn/ui  │──────▶│ /components│────────▶│ your       │
  │            │ pull  │    /ui/    │ import  │ screens    │
  │ 57 comps   │ as    │            │ only    │ and        │
  │ available  │ needed│ your comps │ from    │ pages      │
  └────────────┘       │ your tokens│ here    └────────────┘
                       └────────────┘
```

**The decision tree Claude follows every time:**

1. Need a component → **check our library first**
2. It's in our library → **use ours**
3. Not in our library, but shadcn has it → **pull it in, it's ours now**
4. Nobody has it → **build it, add to library**

Once a component enters the library, Claude never goes back to shadcn for it.

### Using a different foundation

shadcn is the default, but the architecture works with any base system. To swap, change the foundation section in CLAUDE.md and the init command. Everything else stays the same.

| Alternative | Best For |
|---|---|
| Radix UI | Unstyled primitives, maximum design freedom |
| Chakra UI | Rapid prototyping, built-in theme system |
| Mantine | Feature-rich, 100+ components |
| MUI | Material Design, large ecosystem |
| Your own | Existing internal design system |

---

## Folder Structure

Following shadcn's convention — flat, alphabetical, everything in `components/ui/`:

```
my-project/
│
├── CLAUDE.md                  ← Rules Claude follows every session
│
├── components/
│   └── ui/                    ← THE LIBRARY. Everything lives here.
│       ├── accordion.tsx
│       ├── alert.tsx
│       ├── avatar.tsx
│       ├── badge.tsx
│       ├── button.tsx
│       ├── card.tsx
│       ├── dialog.tsx
│       ├── input.tsx
│       ├── settings-row.tsx       ← custom component, same folder
│       ├── stat-card.tsx          ← custom component, same folder
│       └── ...
│
├── pages/ or app/ or src/     ← Your screens (framework-dependent)
│
├── globals.css                ← Design tokens (CSS variables)
└── tailwind.config.ts         ← Maps tokens to Tailwind classes
```

**Why flat?** This is how shadcn organizes its own library. No distinction between "came from shadcn" and "we built this." A button is a button. A settings-row sits next to a select.

**No separate tokens folder.** shadcn uses CSS variables in `globals.css`. One place for all design values.

**No manifest file.** Claude reads the filesystem directly.

**No barrel file.** Import directly: `import { Button } from '@/components/ui/button'`.

### What goes in the library vs. what stays in a page

A component belongs in `/components/ui/` if it could be used on more than one screen. A `Button`, `Card`, `SettingsRow` — these are library components.

A component that's specific to one screen and will never be reused (like a one-off dashboard chart layout) can live in the page file. But if Claude finds itself building something similar on another screen, it should extract it into the library.

When in doubt: put it in the library. It's easier to have a component you don't reuse than to have duplicated UI you need to consolidate later.

---

## Design Tokens — shadcn already did the work

A proper design system starts with foundations: color scales, font families, sizing tokens, and the architecture that connects them. shadcn ships with all four. You don't need to build a token system. You need to set your values.

### What shadcn gives you out of the box

**Color scale** — CSS variables in `globals.css` with semantic names and foreground pairs:

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;           /* your main brand color */
    --primary-foreground: 210 40% 98%;       /* text on primary */
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }
}
```

**Font families** — defined in `tailwind.config.ts`:

```ts
fontFamily: {
  sans: ["var(--font-sans)", ...fontFamily.sans],  // swap your font here
  mono: ["var(--font-mono)", ...fontFamily.mono],
}
```

**Sizing / spacing scale** — Tailwind's built-in scale, which every shadcn component already uses:

```
Tailwind class → px value
p-1            → 4px
p-2            → 8px
p-3            → 12px
p-4            → 16px
p-5            → 20px
p-6            → 24px
p-7            → 28px
p-8            → 32px
p-10           → 40px
p-12           → 48px
p-16           → 64px
p-20           → 80px
```

**The token architecture** — how it all connects:

```
globals.css (CSS variables)           ← your raw values live here
       ↓
tailwind.config.ts                    ← maps variables to utility classes
       ↓
components/ui/ (Tailwind classes)     ← components use classes, never raw values
       ↓
your screens (imports components)     ← screens use components, never raw values

Change a value at the top → everything below updates.
```

### What you decide, what you don't

You **don't** need to build the architecture. It exists.

You **do** need to set your values before building your first screen:

| Foundation | What to decide | Where it lives |
|---|---|---|
| Colors | Your brand primary, secondary, accent, destructive | `globals.css` — change the HSL values |
| Fonts | Your font families (or keep the defaults for now) | `tailwind.config.ts` — swap the font variable |
| Spacing | Keep Tailwind's scale (recommended) or customize | `tailwind.config.ts` — extend if needed |
| Radius | Your preferred border radius | `globals.css` — change `--radius` |

If you're not ready to decide on colors and fonts yet, that's fine. Start with shadcn's defaults. The architecture means you can change any value later and it updates across the entire product. The important thing is that the system exists from day one — the specific values can evolve.

---

## The CLAUDE.md File

Drop this in your project root. Claude reads it automatically every session.

```markdown
# CLAUDE.md

## Architecture

React project. All UI comes from /components/ui/. No exceptions.

## Foundation

system: shadcn/ui
docs: https://ui.shadcn.com/docs/components
pull: npx shadcn@latest add [component-name]

To pull a shadcn component, run the pull command above. This copies
the component source into /components/ui/. It then becomes ours.

## Rules

1. Every piece of UI must come from /components/ui/
2. Check the library first — reuse what exists
3. If it doesn't exist, check shadcn — run `npx shadcn@latest add [name]` to pull it in
4. If shadcn doesn't have it, build it from existing components
5. Never write inline UI in page files
6. Never go back to shadcn for a component already in our library
7. New components go in /components/ui/ alongside everything else
8. Before creating a new component, verify nothing similar already exists
   (e.g., don't create ActionButton if Button with a variant would work)

## Tokens

Token architecture (do not bypass this):
- Color scale, radius: CSS variables in globals.css
- Font families: defined in tailwind.config.ts
- Spacing: Tailwind's built-in scale (p-1=4px, p-2=8px, p-4=16px, etc.)

Components use Tailwind classes that reference these variables.
Never hardcode color, spacing, font, or radius values in components.
Always use the token system: bg-primary, text-muted-foreground, rounded-md, etc.

## Library vs. Page

- If a component could be used on more than one screen → /components/ui/
- If it's truly one-off and screen-specific → can live in the page file
- When in doubt → put it in the library

## Building a Screen

1. Identify UI elements needed
2. Check what exists in /components/ui/
3. Reuse existing, pull from shadcn, or build new as needed
4. New components go in /components/ui/
5. Build the screen importing only from /components/ui/

## Changing a Component

1. Edit the file in /components/ui/
2. Every screen using it updates automatically
```

---

## How It Plays Out

### Day 1: "Build me a dashboard"

```
CLAUDE: Library is empty. Pulling from shadcn:
        npx shadcn@latest add button card badge avatar separator

        Building custom (added to same folder):
        stat-card, stats-row

        LIBRARY: 7 components
```

### Day 5: "Build me a settings page"

```
CLAUDE: Checks library — reuses button, card, badge

        Pulling from shadcn (new):
        npx shadcn@latest add switch input label select

        Building custom: settings-row, settings-section

        LIBRARY: 7 → 13 components
```

### Day 12: "Change primary to our brand blue, rounder corners"

```
CLAUDE: Two changes in globals.css:

        --primary: 221 83% 53%  →  --primary: 230 90% 45%
        --radius: 0.5rem        →  --radius: 0.75rem

        Every component. Every screen. Updated.
```

### The visual evolution

```
  DAY 1                    WEEK 3                   MONTH 2
  (shadcn defaults)        (your tokens evolving)   (fully yours)

  ┌────────────────┐       ┌────────────────┐       ┌────────────────┐
  │  [ Button ]    │       │  [ Button ]    │       │  [ Button ]    │
  │  default look  │       │  your colors   │       │  your brand    │
  │  default radius│       │  your radius   │       │  your identity │
  └────────────────┘       └────────────────┘       └────────────────┘

  Looks like shadcn.       Looks like you.          Is yours.
```

---

## Editing and Propagation

### How changes propagate

```
  EDIT                         SOURCE            EVERYWHERE
  (pick any)                   OF TRUTH

  Claude Code ────────────▶┐                 ┌──▶ Every screen
                           │  button.tsx     │    using Button
  Onlook ─────────────────▶│  (one file)     │
                           │                 │──▶ Every pattern
  Your editor ────────────▶┘                 │    containing Button
                                             │
                                             └──▶ Storybook / Ladle
                                                  (if you use one)

  Three ways in. One file. One truth.
```

### Onlook — visual editing (optional, free)

[Onlook](https://onlook.dev) runs on top of your running app. Click any component, change styles with sliders and color pickers, and it writes changes directly back to the source `.tsx` file. Because every screen imports from `/components/ui/`, changing a Button through Onlook updates that Button everywhere.

### Component browser (optional, add later)

When your library grows large enough that you want to browse and preview components visually, add one of these:

| Tool | Setup | Notes |
|---|---|---|
| **Storybook** | `npx storybook@latest init` | Full-featured, controls panel, most ecosystem support |
| **Ladle** | `npm install @ladle/react` | Lightweight, Vite-powered, used at Uber, compatible with Storybook story format |

Both are free and open source. Neither is required to use this system. The library works without a browser tool — Claude reads the filesystem directly to know what exists.

---

## Figma Comparison

| Figma | This approach |
|---|---|
| UI kit you start from | shadcn/ui |
| Component in your library | `.tsx` file in `/components/ui/` |
| Instance on a design screen | `import` in a page file |
| Edit the main component | Edit the source `.tsx` |
| All instances update | All imports hot-reload |
| Detach instance (bad practice) | Inline UI in a page (CLAUDE.md prevents) |
| Color style / variable | CSS variable in `globals.css` |
| Publish library changes | Save (hot reload) or `git push` (deploy) |

---

## Tool Stack

| Tool | Role | Cost | Required? |
|---|---|---|---|
| **Claude Code** | Builds everything, maintains library, follows CLAUDE.md | Pro/Max plan | Yes |
| **CLAUDE.md** | Governance — the rules Claude follows | Free (text file) | Yes |
| **Onlook** | Visual editor that writes to source code | Free / open source | Optional |
| **Storybook or Ladle** | Browse and preview components | Free / open source | Optional, add later |

---

## Setup Checklist

```
Phase 1: Project Setup (5 min)
[ ] Create a React project (Next.js, Vite, Remix — your choice)
[ ] Initialize shadcn: npx shadcn@latest init
[ ] Drop CLAUDE.md (from this guide) into your project root

Phase 2: Set Your Foundations (5 min)
[ ] Open globals.css — set your color values (or keep defaults for now)
[ ] Open tailwind.config.ts — set your font families (or keep defaults)
[ ] Confirm spacing scale works for you (Tailwind default is fine)
[ ] Set --radius to your preferred border radius

    Or ask Claude: "Set up my design foundations. I want [describe your
    brand colors, preferred fonts, and any spacing preferences]. Update
    globals.css and tailwind.config.ts."

Phase 3: Start Building
[ ] Open Claude Code
[ ] First prompt: "Build me a [screen]. Follow the rules in CLAUDE.md."
[ ] Claude pulls from shadcn, applies your tokens, adds to library
[ ] Library starts growing

Ongoing
[ ] Build features → library grows automatically
[ ] Edit via Claude, Onlook, or your editor
[ ] Change a token in globals.css → entire product updates
[ ] Everything propagates

Optional (add when library gets large)
[ ] Add Storybook or Ladle for visual browsing
[ ] Add Onlook for visual editing
```

---

## Migrating an Existing Product

You don't need to start from scratch. If you have an existing product, you can introduce this gradually.

### The honest tradeoff

Consolidating inconsistent UI into shared components means every screen snaps to the canonical version. If you had 4 slightly different button styles, picking one means the others change. That's the point and the risk.

### The approach

**Step 1: Audit.** Paste this into Claude Code:

```
Audit this codebase for UI patterns. For each pattern:
1. List every instance (file + line)
2. Note visual differences between instances
3. Recommend which version becomes canonical
4. Note which screens would change visually

Also identify hardcoded colors, spacing, and font sizes
that should become CSS variables.

Don't change any code. Just report.
```

**Step 2: Approve.** Claude reports what it found. You decide what to consolidate.

**Step 3: Extract gradually.** Start with the most-used components. New features use the library from day one. Old screens get cleaned up as you touch them.

```
Week 1:  Audit + extract tokens into globals.css
Week 2:  Extract 5 most-used components into /components/ui/
Week 3:  Add CLAUDE.md — new features use library automatically
Week 4+: Migrate remaining screens as you touch them
```

---

## Quick Reference: First Prompt

Copy this for your first Claude Code session on a new project:

```
I've set up a React project with shadcn and dropped in a CLAUDE.md.
Read the CLAUDE.md first.

Build me a [describe your first screen]. Follow the rules:
- Pull any components you need from shadcn using npx shadcn@latest add
- Put everything in /components/ui/
- Use tokens from globals.css, never hardcode values
- Import only from /components/ui/ in the page file

Tell me which components you're reusing, which you're pulling from
shadcn, and which you're building custom before you start.
```
