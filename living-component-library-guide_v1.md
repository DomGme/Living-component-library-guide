# Building with a Living Component Library
### How to use Claude Code to auto-generate, preview, edit, and propagate a design system from day one

---

## What This Is

A complete workflow where every time you build with Claude, a shared component library grows automatically. You can browse every component visually, edit any one of them, and watch the change update everywhere across your product — the same way editing a Figma library component updates every instance across your files.

No retrofitting. No separate design system project. The library IS the product.

---

## How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   "Build me a settings page"                                        │
│                                                                     │
│   Claude checks the library:                                        │
│                                                                     │
│   ┌──────────────────────────────┐                                  │
│   │  COMPONENT LIBRARY           │                                  │
│   │                              │                                  │
│   │  ✓ Button (exists)           │ ──→ reuse                        │
│   │  ✓ Card (exists)             │ ──→ reuse                        │
│   │  ✓ Input (exists)            │ ──→ reuse                        │
│   │  ✗ Toggle (doesn't exist)    │ ──→ CREATE in library, then use  │
│   │  ✗ SettingsRow (new pattern) │ ──→ CREATE in library, then use  │
│   │                              │                                  │
│   └──────────────────────────────┘                                  │
│                                                                     │
│   The library grows with every feature you build.                   │
│   Next time anyone needs a Toggle, it already exists.               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Glossary

**React** — A JavaScript library for building user interfaces out of reusable components. It handles the view layer: rendering, state, interaction.

**Next.js** — A framework built on top of React. It adds routing (files become URLs), server-side rendering, and deployment tooling. Think of React as the building blocks, Next.js as the house structure.

**Component** — A reusable piece of UI. A Button, a Card, a Header. Write it once, import it everywhere.

**Tokens** — The raw design values: colors, spacing, font sizes, shadows. Components reference tokens instead of hardcoding values like `#6187FA`. Change a token, every component using it updates.

**Storybook** — A tool that renders every component in isolation so you can browse, preview, and test them. Like a visual catalogue of your library.

**Onlook** — An open-source visual editor. It runs on top of your actual app and lets you click on any component, change its styles with sliders and color pickers, and it writes those changes directly back to the source code.

**CLAUDE.md** — A file in your project root that Claude Code reads automatically on every session. It contains the rules and architecture Claude must follow. This is the governance layer.

**MCP (Model Context Protocol)** — A standard that lets AI tools connect to external data sources. Storybook MCP gives Claude awareness of your full component library.

**Hot reload** — When you save a file, the running app and Storybook update instantly in the browser without a manual refresh.

---

## The Three Layers

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  LAYER 1: THE LIBRARY (source of truth)                             │
│                                                                     │
│  /components/ — Claude builds and maintains this.                   │
│  Every UI element lives here. CLAUDE.md enforces the rules.         │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  LAYER 2: THE VISUAL BROWSER + EDITOR                               │
│                                                                     │
│  Storybook — browse every component, toggle props, see every state  │
│  Onlook — click on anything, edit visually, changes write to code   │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  LAYER 3: THE PRODUCT                                               │
│                                                                     │
│  /app/ — Every screen imports from /components/.                    │
│  Change a component → every screen using it updates.                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Folder Structure

This is what Claude creates on day one and maintains forever:

```
my-product/
│
├── CLAUDE.md                    ← The brain. Claude reads this every session.
│
├── components/                  ← THE LIBRARY. Single source of truth.
│   ├── index.ts                 ← Barrel file. Every component exported here.
│   │
│   ├── primitives/              ← Lowest-level building blocks
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx   ← Storybook story (auto-generated)
│   │   ├── Input.tsx
│   │   ├── Input.stories.tsx
│   │   ├── Text.tsx
│   │   ├── Icon.tsx
│   │   └── ...
│   │
│   ├── patterns/                ← Compositions of primitives
│   │   ├── Card.tsx
│   │   ├── Card.stories.tsx
│   │   ├── FormField.tsx
│   │   ├── ListItem.tsx
│   │   ├── Modal.tsx
│   │   └── ...
│   │
│   └── blocks/                  ← Larger, reusable sections
│       ├── Header.tsx
│       ├── Header.stories.tsx
│       ├── SettingsRow.tsx
│       ├── PricingCard.tsx
│       └── ...
│
├── tokens/                      ← Design tokens. Components reference these.
│   ├── colors.ts
│   ├── spacing.ts
│   ├── typography.ts
│   ├── shadows.ts
│   └── index.ts
│
├── app/                         ← The actual product screens (Next.js)
│   ├── dashboard/
│   │   └── page.tsx             ← imports from components/ only
│   ├── settings/
│   │   └── page.tsx             ← imports from components/ only
│   └── ...
│
└── library-manifest.json        ← Auto-generated inventory of every component
```

---

## The CLAUDE.md File

This is the most important file in the project. Drop it in your project root. Claude Code reads it automatically every session.

```markdown
# CLAUDE.md

## Project Architecture

This is a Next.js project using React. All UI is built from a shared
component library at `/components/`. There is no other source of UI elements.

## THE SINGLE MOST IMPORTANT RULE

**Every piece of UI must come from `/components/`.**

When building any screen, page, or feature:

1. FIRST check what already exists in `/components/`
2. Reuse existing components wherever possible
3. If a component doesn't exist, CREATE it in `/components/` first
4. Then import it into the screen

NEVER write inline UI elements in page files. NEVER create one-off styled
divs in `/app/`. If you're writing JSX that isn't importing from
`/components/`, you're doing it wrong.

## Component Creation Rules

When creating a new component:

1. Decide the tier:
   - `primitives/` — single-purpose, no child components (Button, Input, Badge)
   - `patterns/` — compositions of primitives (FormField, Card, ListItem)
   - `blocks/` — larger sections made of patterns (Header, SettingsPanel)

2. Every component MUST:
   - Accept props with TypeScript types
   - Use tokens from `/tokens/` for all colors, spacing, typography, shadows
   - Have a default export
   - Be added to the barrel file at `/components/index.ts`
   - Have a `.stories.tsx` file created alongside it

3. Every component MUST NOT:
   - Hardcode color values (use tokens)
   - Hardcode spacing values (use tokens)
   - Contain business logic (components are visual only)
   - Import from `/app/` (one-way dependency: app → components)

## Tokens

All visual values live in `/tokens/`:
- Colors: `/tokens/colors.ts`
- Spacing: `/tokens/spacing.ts`
- Typography: `/tokens/typography.ts`
- Shadows: `/tokens/shadows.ts`

When creating or modifying components, ALWAYS reference tokens.
Example: `color: tokens.colors.primary` not `color: '#6187FA'`

## Storybook Stories

Every component MUST have a `.stories.tsx` file next to it.
Example: `Button.tsx` → `Button.stories.tsx`

Story requirements:
- Default story showing the most common usage
- One story per meaningful variant (Primary, Secondary, Ghost)
- One story per state (Disabled, Loading, Error)
- Use Controls for all props so they're editable in the Storybook UI
- Story file must be created AT THE SAME TIME as the component

## Library Manifest

After creating or modifying any component, update `/library-manifest.json`.

The manifest structure:
{
  "lastUpdated": "ISO timestamp",
  "components": {
    "Button": {
      "tier": "primitive",
      "path": "components/primitives/Button.tsx",
      "description": "Primary action trigger.",
      "props": ["variant", "size", "disabled", "onClick", "children"],
      "usedIn": ["app/dashboard/page.tsx", "app/settings/page.tsx"],
      "tokens": ["colors.primary", "spacing.sm", "typography.body"]
    }
  }
}

## When Asked to Build a New Screen

Follow this exact sequence:

1. Read the request and identify all UI elements needed
2. Read `/library-manifest.json` to see what exists
3. List which existing components you'll reuse
4. List which new components you need to create
5. Tell me the plan before building
6. Create new components in `/components/` first (with stories)
7. Update the barrel file
8. Build the screen importing from `/components/`
9. Update `/library-manifest.json`

## When Asked to Change a Component

1. Make the change in `/components/[tier]/[Component].tsx`
2. That's it. Every screen importing this component gets the update.
3. If the change affects props, check all `usedIn` entries in the manifest
   and update those screens if needed.
4. Update the story if props changed.
5. Update `/library-manifest.json` if props or tokens changed.

## Common Commands

- `npm run dev` — start dev server (your app)
- `npm run storybook` — start component library browser
```

---

## How It Plays Out

### Day 1: "Build me a dashboard"

```
YOU:    Build me a dashboard with user stats and a sidebar nav.

CLAUDE: (reads CLAUDE.md, checks /components/ — it's empty)

        I need to create these components first:

        Primitives: Button, Text, Icon, Badge
        Patterns:   Card, StatCard, NavItem
        Blocks:     Sidebar, StatsRow

        Creating 9 components with stories, then assembling the dashboard.

        (creates everything in /components/, updates manifest,
         builds dashboard importing from /components/)

LIBRARY: 9 components
```

### Day 5: "Build me a settings page"

```
YOU:    Build me a settings page.

CLAUDE: (reads manifest — finds 9 existing components)

        I can reuse: Sidebar, Button, Card, Text, Icon
        I need to create: Toggle, FormField, SettingsRow, SettingsSection

        Creating 4 new components, reusing 5 existing ones.

LIBRARY: 9 → 13 components
```

### Day 12: "Make all buttons rounder"

```
YOU:    Make all buttons have more rounded corners.

CLAUDE: (reads manifest — Button is used in 8 screens)

        Changing Button.tsx border-radius.
        This affects: dashboard, settings, profile, onboarding,
        pricing, checkout, admin, login.

        All 8 screens updated automatically because they all
        import the same Button.tsx.

        (one file change — done)
```

This is the Figma analogy:

| Figma concept | Code equivalent |
|---|---|
| Component in your library | `.tsx` file in `/components/` |
| Instance on a design screen | `import` in a page file |
| Edit the main component | Edit the source `.tsx` file |
| All instances update | All imports hot-reload |
| Detach instance (bad practice) | Inline UI in a page file (CLAUDE.md prevents this) |
| Publish library changes | Save the file (hot reload) or `git push` (deploy) |

---

## Browsing the Library: Storybook

Once you run `npm run storybook`, you get this:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  STORYBOOK                                                http://lo... │
├──────────────┬──────────────────────────────────────────────────────────┤
│              │                                                          │
│  Primitives  │   ┌─────────────────────────────────────────────┐       │
│  ├─ Button   │   │                                             │       │
│  ├─ Input    │   │   [ Primary ]  [ Secondary ]  [ Ghost ]    │       │
│  ├─ Badge    │   │                                             │       │
│  ├─ Toggle   │   │   [ Small ]    [ Medium ]     [ Large ]    │       │
│  ├─ Text     │   │                                             │       │
│  │           │   │   [ Disabled ]                              │       │
│  Patterns    │   │                                             │       │
│  ├─ Card     │   └─────────────────────────────────────────────┘       │
│  ├─ FormFiel │                                                          │
│  ├─ ListItem │   Controls                                               │
│  │           │   ┌──────────────────────────────────────────┐          │
│  Blocks      │   │ variant   [primary ▼]                    │          │
│  ├─ Header   │   │ size      [md ▼]                         │          │
│  ├─ Sidebar  │   │ disabled  [ ] false                      │          │
│  └─ PricingC │   │ children  [Click me          ]           │          │
│              │   └──────────────────────────────────────────┘          │
│              │                                                          │
│              │   Change "variant" → component re-renders live          │
│              │   Edit the source file → hot reloads instantly          │
│              │                                                          │
└──────────────┴──────────────────────────────────────────────────────────┘
```

**What it gives you:**
- Sidebar navigation of every component, organized by tier
- Live preview of each component in isolation
- Controls panel to toggle props (variant, size, disabled)
- Hot reload: edit the source file, Storybook updates immediately
- Auto-generated docs from your TypeScript types

**With Storybook MCP addon:** Claude Code gains awareness of the full library. When you ask Claude to build something new, it knows exactly what exists and what each component's API looks like.

---

## Editing Visually: Onlook

Onlook is the piece that gives you the Figma-like editing experience on real code.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ONLOOK                                                                 │
├──────────────┬──────────────────────────────────────────────────────────┤
│              │                                                          │
│  Component   │   ┌─────────────────────────────────────────────┐       │
│  Tree        │   │                                             │       │
│              │   │   Your actual running app                   │       │
│  ▼ App       │   │                                             │       │
│    ▼ Layout  │   │   ┌──────────────────────────┐              │       │
│      Header  │   │   │  Dashboard               │              │       │
│      ▼ Main  │   │   │  ┌──────┐ ┌──────┐      │    Properties│       │
│        Stats │   │   │  │ Stat │ │ Stat │ ...   │    ┌─────────┤       │
│        Chart │   │   │  └──────┘ └──────┘      │    │ padding │       │
│        Table │   │   │                          │    │ 16px    │       │
│      Footer  │   │   │  Click any element ──────┼──▶ │         │       │
│              │   │   │  to select and edit       │    │ radius  │       │
│              │   │   │                          │    │ 8px     │       │
│              │   │   └──────────────────────────┘    │         │       │
│              │   │                                    │ color   │       │
│              │   │   Drag a slider ─────────────────▶ │ #6187FA │       │
│              │   │   Code updates instantly           │         │       │
│              │   │   App re-renders live              └─────────┘       │
│              │   │                                                      │
└──────────────┴──┴──────────────────────────────────────────────────────┘
```

**What it gives you:**
- Click on any rendered element in your running app
- Edit styles visually: sliders, color pickers, spacing controls
- Changes write DIRECTLY back to the source `.tsx` files
- AI chat built in for describing changes in natural language

**Why this matters for your library:**
Your app renders components from `/components/`. You click on a Button anywhere in the app. You change its border radius with a slider. Onlook writes the change to `/components/primitives/Button.tsx`. Every screen using Button updates live — because they all import the same source file.

This is the visual edit → propagate loop.

---

## The Propagation Model

```
  EDIT (pick one)                    SOURCE OF TRUTH              EVERYWHERE
                                                                    
  ┌──────────────────┐                                              
  │ Claude Code      │               ┌──────────────┐   ┌────────────────┐
  │ "change button   │──────────────▶│              │──▶│ Storybook      │
  │  padding to 12px"│               │  Button.tsx  │   │ preview updates│
  └──────────────────┘               │              │   └────────────────┘
                                     │  (in /compo- │          
  ┌──────────────────┐               │   nents/)    │   ┌────────────────┐
  │ Onlook           │               │              │──▶│ Dashboard      │
  │ click + drag     │──────────────▶│              │   │ page updates   │
  │ slider           │               │              │   └────────────────┘
  └──────────────────┘               │              │          
                                     │              │   ┌────────────────┐
  ┌──────────────────┐               │              │──▶│ Settings       │
  │ Direct code edit │               │              │   │ page updates   │
  │ in your editor   │──────────────▶│              │   └────────────────┘
  └──────────────────┘               └──────────────┘          
                                                        ┌────────────────┐
  Three ways in.                     One file.          │ Every other    │
  Same result.                       One truth.    ────▶│ screen using   │
                                                        │ Button updates │
                                                        └────────────────┘
```

### Changing a Token (deepest cascade)

```
  tokens/colors.ts   (change primary: '#6187FA' → '#2563EB')
       │
       ├──→ Button.tsx ──→ dashboard, settings, login, checkout...
       ├──→ Toggle.tsx ──→ settings
       ├──→ Badge.tsx ──→ dashboard, admin
       ├──→ NavItem.tsx ──→ sidebar (appears on every page)
       └──→ every component referencing tokens.colors.primary

  ONE token change. ENTIRE product updated.
```

---

## The Full Loop

```
 START A NEW PROJECT
         │
         ▼
 ┌───────────────────────────────────────────────────┐
 │ Drop CLAUDE.md into an empty project folder        │
 │ "Build me a [first screen]"                        │
 │                                                    │
 │ Claude creates:                                    │
 │ - /tokens/ (colors, spacing, type, shadows)        │
 │ - /components/primitives/ (Button, Input, Text...) │
 │ - /components/patterns/ (Card, FormField...)       │
 │ - /components/blocks/ (Header, Sidebar...)         │
 │ - .stories.tsx file for each component             │
 │ - /app/dashboard/page.tsx (using components)       │
 │ - library-manifest.json                            │
 └───────────────────────────┬───────────────────────┘
                             │
                             ▼
 ┌───────────────────────────────────────────────────┐
 │ Run two dev servers:                               │
 │                                                    │
 │ Terminal 1: npm run dev        → your app          │
 │ Terminal 2: npm run storybook  → component library │
 │                                                    │
 │ Both are live. Both hot-reload from /components/.  │
 └───────────────────────────┬───────────────────────┘
                             │
                             ▼
 ┌───────────────────────────────────────────────────┐
 │ BROWSE THE LIBRARY                                 │
 │                                                    │
 │ Open Storybook → see every component organized     │
 │ Click through primitives, patterns, blocks         │
 │ Toggle props, see every variant and state          │
 └───────────────────────────┬───────────────────────┘
                             │
                     ┌───────┴───────┐
                     │               │
                     ▼               ▼
 ┌─────────────────────┐  ┌──────────────────────────┐
 │ EDIT VIA CLAUDE      │  │ EDIT VISUALLY             │
 │                      │  │                           │
 │ "Make buttons more   │  │ Open Onlook               │
 │  rounded and change  │  │ Click on a Button         │
 │  primary to blue"    │  │ Drag the radius slider    │
 │                      │  │ Pick a new color          │
 │ Claude edits:        │  │                           │
 │ - tokens/colors.ts   │  │ Onlook writes to:         │
 │ - Button.tsx         │  │ - Button.tsx              │
 └──────────┬───────────┘  └──────────┬────────────────┘
            │                         │
            └────────────┬────────────┘
                         │
                         ▼
 ┌───────────────────────────────────────────────────┐
 │ PROPAGATION (automatic)                            │
 │                                                    │
 │ Button.tsx changed (by Claude OR by Onlook)        │
 │                                                    │
 │ Hot reload kicks in:                               │
 │                                                    │
 │ ✓ Storybook Button story → updated                 │
 │ ✓ Dashboard (uses Button) → updated                │
 │ ✓ Settings (uses Button) → updated                 │
 │ ✓ Login (uses Button) → updated                    │
 │ ✓ Every screen using Button → updated              │
 └───────────────────────────┬───────────────────────┘
                             │
                             ▼
 ┌───────────────────────────────────────────────────┐
 │ BUILD MORE                                         │
 │                                                    │
 │ "Build me a pricing page"                          │
 │                                                    │
 │ Claude checks the manifest → reuses 8 components,  │
 │ creates 3 new ones → library grows.                │
 │ Storybook auto-refreshes to show the new ones.     │
 │                                                    │
 │ The library is never a separate project.            │
 │ It IS the product. It grows with every feature.     │
 └───────────────────────────────────────────────────┘
```

---

## Performance and Consistency

**Performance:** There are no concerns. Importing a shared `Button.tsx` across 50 screens doesn't duplicate code. The bundler (Vite or Webpack) includes it once and every screen references the same module. This is how React is designed to work — and it's actually *more* performant than scattered inline UI, because the bundler can tree-shake unused components and the browser caches shared chunks.

**Consistency (the "50 different buttons" problem):** This is a governance problem, not a technical one. It's the same challenge as Figma — nothing stops someone from detaching a component instance. In code, nothing stops someone from writing an inline button. The CLAUDE.md solves this: Claude reads it every session and follows the rule — check the library first, reuse what exists, only create new when there's a genuinely new pattern. One primary button. One secondary button. One set of tokens. No duplication.

**The "publish" step:**
- During development: instant. Save the file, everything hot-reloads. No publish step.
- For production: `git push` → deploy. The updated components ship with the build.

---

## The Library Manifest

The `library-manifest.json` is the machine-readable table of contents. Claude maintains it automatically. You can ask "what components do I have?" or "what uses the Card component?" at any time.

```json
{
  "lastUpdated": "2026-02-06T14:30:00Z",
  "components": {
    "Button": {
      "tier": "primitive",
      "path": "components/primitives/Button.tsx",
      "description": "Primary action trigger. 3 variants, 3 sizes.",
      "props": {
        "variant": "'primary' | 'secondary' | 'ghost'",
        "size": "'sm' | 'md' | 'lg'",
        "disabled": "boolean",
        "onClick": "() => void",
        "children": "ReactNode"
      },
      "tokens": ["colors.primary", "colors.primaryHover", "spacing.sm",
                  "spacing.md", "radius.md", "typography.button"],
      "usedIn": [
        "app/dashboard/page.tsx",
        "app/settings/page.tsx",
        "app/login/page.tsx",
        "components/patterns/FormField.tsx",
        "components/blocks/Header.tsx"
      ]
    },
    "Card": {
      "tier": "pattern",
      "path": "components/patterns/Card.tsx",
      "description": "Container with optional header, padding, shadow.",
      "props": {
        "title": "string (optional)",
        "padding": "'sm' | 'md' | 'lg'",
        "elevated": "boolean",
        "children": "ReactNode"
      },
      "tokens": ["colors.surface", "shadows.card", "radius.lg",
                  "spacing.md", "spacing.lg"],
      "usedIn": [
        "components/patterns/StatCard.tsx",
        "components/blocks/ChartCard.tsx",
        "app/dashboard/page.tsx",
        "app/settings/page.tsx"
      ]
    }
  }
}
```

---

## Tool Stack

| Tool | What it does | Cost |
|------|-------------|------|
| **Claude Code** | Builds everything. Maintains library. Enforces rules via CLAUDE.md. | Pro/Max plan |
| **CLAUDE.md** | The governance file. The design system brain. | Free (it's a text file) |
| **Storybook** | Visual browser for the library. Organized, documented, interactive. | Free / open source |
| **Storybook MCP** | Gives Claude awareness of the full component library for future builds. | Free / open source |
| **Onlook** | Visual editor. Click any component, edit with sliders, writes to code. | Free / open source |
| **library-manifest.json** | Machine-readable inventory of every component, props, and usage. | Free (it's a JSON file) |

---

## Setup Checklist

```
Phase 1: Foundation (5 min)
[ ] Create empty project folder
[ ] Drop CLAUDE.md (from this doc) into the root
[ ] Create empty: components/primitives/, components/patterns/,
    components/blocks/, tokens/, app/
[ ] Create library-manifest.json with: { "components": {} }
[ ] Open Claude Code in that folder
[ ] First prompt: "Set up the token files and build me a [screen]"

Phase 2: Component Library Browser (10 min)
[ ] Ask Claude: "Add Storybook to this project with the MCP addon"
[ ] Run: npm run storybook
[ ] Browse your auto-generated component library
[ ] Verify: every component has stories, controls work

Phase 3: Visual Editing (10 min)
[ ] Install Onlook (onlook.dev — free, open source)
[ ] Point it at your running app (npm run dev)
[ ] Click a component → see its styles in the sidebar
[ ] Change something with a slider → verify it wrote to /components/
[ ] Check: Storybook also updated (it should — same source files)

Phase 4: Ongoing
[ ] Build new features with Claude → library grows automatically
[ ] Browse library in Storybook → organized, documented, interactive
[ ] Edit visually in Onlook → writes to source, propagates everywhere
[ ] Edit via Claude → "change button padding to 12px"
[ ] All changes propagate to every screen automatically
```

---

## Migrating an Existing Product

You don't have to start from scratch. If you already have a product built, you can introduce this system and gradually migrate toward it. The approach is different — instead of growing the library as you build, you extract the library from what already exists.

### The honest tradeoff

When you consolidate inconsistent UI into a single component library, every screen snaps to the canonical version. If your existing product has 4 slightly different button styles across 12 screens, picking one Button means 11 screens will change visually. That's both the point (consistency) and the risk (unexpected visual changes). Know this going in and choose your pace accordingly.

### Two strategies

**Gradual migration (recommended for products with users):**
Migrate the most-used components first. Leave the rest alone. Each week, extract a few more patterns. The product slowly converges toward consistency without a big-bang redesign. New features use the library from day one. Old screens get cleaned up over time.

**Full audit (good for pre-launch or internal tools):**
Claude scans the entire codebase, identifies every pattern, proposes a consolidated library, and rewires everything. Faster, but more visual change at once. Best when you can tolerate a noticeable UI refresh.

### The migration sequence

```
 EXISTING PRODUCT (messy, inconsistent)
         │
         ▼
 ┌───────────────────────────────────────────────────────────┐
 │  STEP 1: AUDIT                                            │
 │                                                           │
 │  Prompt: "Audit this project. Identify every repeated     │
 │  UI pattern. Group them by similarity. Don't change       │
 │  anything — just give me the plan."                       │
 │                                                           │
 │  Claude walks through your codebase and reports:          │
 │                                                           │
 │  BUTTONS                                                  │
 │  ├─ app/dashboard/page.tsx    → inline, blue, rounded-md  │
 │  ├─ app/settings/page.tsx     → inline, blue, rounded-lg  │
 │  ├─ app/login/page.tsx        → inline, blue, rounded-md  │
 │  ├─ app/checkout/page.tsx     → inline, navy, rounded-sm  │
 │  └─ Recommendation: consolidate into 1 Button component   │
 │     with variant and size props. 3 of 4 are nearly        │
 │     identical. Checkout button is a different variant.     │
 │                                                           │
 │  CARDS                                                    │
 │  ├─ app/dashboard/page.tsx    → div with shadow, p-6      │
 │  ├─ app/settings/page.tsx     → div with shadow, p-4      │
 │  ├─ app/pricing/page.tsx      → div with border, p-6      │
 │  └─ Recommendation: 1 Card component with padding and     │
 │     elevated props.                                       │
 │                                                           │
 │  COLORS                                                   │
 │  ├─ #6187FA used 23 times (primary)                       │
 │  ├─ #6188FB used 4 times (probably meant to be primary)   │
 │  ├─ #374151 used 31 times (text)                          │
 │  └─ Recommendation: extract into tokens/colors.ts         │
 │                                                           │
 │  ... (continues for all patterns)                         │
 │                                                           │
 └───────────────────────────────┬───────────────────────────┘
                                 │
                                 ▼
 ┌───────────────────────────────────────────────────────────┐
 │  STEP 2: APPROVE THE PLAN                                 │
 │                                                           │
 │  You review Claude's audit. You decide:                   │
 │                                                           │
 │  ✓ "Yes, consolidate buttons — use rounded-md as default" │
 │  ✓ "Yes, consolidate cards — keep the shadow version"     │
 │  ✗ "Leave the pricing card alone for now, it's special"   │
 │  ✓ "Yes, extract all colors into tokens"                  │
 │                                                           │
 │  You're choosing the canonical version of each component. │
 │  This is the design decision. Claude handles the rest.    │
 │                                                           │
 └───────────────────────────────┬───────────────────────────┘
                                 │
                                 ▼
 ┌───────────────────────────────────────────────────────────┐
 │  STEP 3: EXTRACT (one component at a time)                │
 │                                                           │
 │  Prompt: "Extract a Button component into                 │
 │  /components/primitives/Button.tsx based on the           │
 │  dashboard version. Add variant and size props.           │
 │  Create a story file. Then replace all inline             │
 │  buttons in the codebase with imports from this           │
 │  component. Update the manifest."                         │
 │                                                           │
 │  Claude:                                                  │
 │  1. Creates /components/primitives/Button.tsx              │
 │  2. Creates /components/primitives/Button.stories.tsx      │
 │  3. Replaces inline buttons in dashboard → import Button  │
 │  4. Replaces inline buttons in settings → import Button   │
 │  5. Replaces inline buttons in login → import Button      │
 │  6. Replaces inline buttons in checkout → import Button   │
 │     (maps the navy variant to variant="secondary")        │
 │  7. Adds Button to /components/index.ts                   │
 │  8. Updates library-manifest.json                         │
 │                                                           │
 │  Repeat for Card, Input, Badge, etc.                      │
 │                                                           │
 └───────────────────────────────┬───────────────────────────┘
                                 │
                                 ▼
 ┌───────────────────────────────────────────────────────────┐
 │  STEP 4: ADD GOVERNANCE                                   │
 │                                                           │
 │  Drop CLAUDE.md into the project root.                    │
 │                                                           │
 │  From this point forward, Claude follows the rules:       │
 │  - New features use the library                           │
 │  - New components get added to the library                │
 │  - No more inline UI in page files                        │
 │                                                           │
 │  Old screens that haven't been migrated yet still work.   │
 │  They just don't benefit from the library until they're   │
 │  migrated. No rush — do it as you touch those screens.    │
 │                                                           │
 └───────────────────────────────┬───────────────────────────┘
                                 │
                                 ▼
 ┌───────────────────────────────────────────────────────────┐
 │  STEP 5: ONGOING                                          │
 │                                                           │
 │  New features → built with the library (automatic)        │
 │  Old screens → migrated when you touch them (gradual)     │
 │  Library → grows with every new feature                   │
 │  Consistency → improves over time, not overnight          │
 │                                                           │
 └───────────────────────────────────────────────────────────┘
```

### The gradual migration in practice

Here's what a realistic timeline looks like for a medium-sized product:

```
WEEK 1:  Audit. Claude scans everything, reports patterns.
         You review and approve the plan.
         Extract tokens (colors, spacing, typography).
         This alone catches inconsistencies like #6187FA vs #6188FB.

WEEK 2:  Extract 5 primitives: Button, Input, Text, Badge, Icon.
         These are the most reused. Biggest consistency win.
         Rewire screens that use them.
         Set up Storybook — you now have a browsable library.

WEEK 3:  Extract 3-4 patterns: Card, FormField, ListItem, Modal.
         Add CLAUDE.md governance.
         New features from here on use the library automatically.

WEEK 4+: Extract blocks as you encounter them.
         Migrate remaining old screens as you touch them.
         Library grows. Consistency increases. No big bang.
```

### What changes visually (and what doesn't)

When you consolidate, screens that had slightly different styling will snap to the canonical version. Here's what to expect:

**Will change:**
- Inconsistent border-radius values normalize (e.g., some buttons had 8px, some had 12px → all become your chosen default)
- Color variations converge (e.g., 3 slightly different blues → 1 primary blue token)
- Spacing inconsistencies resolve (e.g., some cards had 16px padding, some had 24px → one Card component with a padding prop)

**Won't change:**
- Layout and page structure (those aren't components)
- Content, copy, images
- Business logic, data flow, routing
- Anything you choose not to migrate yet

**How to control the risk:**
- Migrate one component at a time
- Review the visual diff after each extraction (your dev server shows changes live)
- If a screen looks wrong after migration, it means the canonical component needs a new variant — add it
- Use git branches: migrate on a branch, review visually, merge when satisfied

### The audit prompt

Copy and paste this into Claude Code on your existing project:

```
Audit this entire codebase for UI patterns. For each pattern you find:

1. List every instance (file path and line numbers)
2. Note the visual differences between instances
3. Group them by similarity
4. Recommend which version should become the canonical component
5. Note which screens would change visually if consolidated

Organize your findings into three tiers:
- Primitives (single-purpose elements: buttons, inputs, badges)
- Patterns (compositions: cards, form fields, list items)
- Blocks (larger sections: headers, sidebars, footers)

Also identify:
- Hardcoded color values that should become tokens
- Hardcoded spacing values that should become tokens
- Hardcoded font sizes that should become tokens

Don't change any code. Just report the findings and recommendations.
```

---

## What This Enables

You end up with a product where:

- Every component exists once and is used everywhere
- You can browse the full catalogue visually at any time
- You can edit any component visually and see it update across the product
- You can ask Claude to make changes and it knows exactly what exists
- The library grows organically as you build — no separate "design system" workstream
- Consistency is enforced by architecture, not by discipline alone
- A new team member can open Storybook and understand the full UI vocabulary immediately

The CLAUDE.md is the design system governance. The manifest is the table of contents. Storybook is the visual catalogue. Onlook is the visual editor. And the component folder is the single source of truth that everything else reads from.
