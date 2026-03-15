What This Is
Every time you build with Claude, a shared component library grows automatically. Any change to a component propagates across your entire product — the same way editing a Figma library component updates every instance.

No retrofitting. No separate design system project. The library IS the product.

The Core Idea
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
The Foundation: shadcn/ui
Your library starts from shadcn/ui — 57 accessible React components you copy into your project and own. Once a component enters your library, it's yours to modify. shadcn is the well you draw from. Your library is the water tank. Your product only drinks from the tank.

  BASE SYSTEM          YOUR LIBRARY           YOUR PRODUCT
  (the well)           (the tank)             (drinks from tank only)

  ┌────────────┐       ┌────────────┐         ┌────────────┐
  │ shadcn/ui  │──────▶│ /components│────────▶│ your       │
  │            │ pull  │    /ui/    │ import  │ screens    │
  │ 57 comps   │ as    │            │ only    │ and        │
  │ available  │ needed│ your comps │ from    │ pages      │
  └────────────┘       │ your tokens│ here    └────────────┘
                       └────────────┘
The decision tree Claude follows every time:

Need a component → check our library first
It's in our library → use ours
Not in our library, but shadcn has it → pull it in, it's ours now
Nobody has it → build it, add to library
Once a component enters the library, Claude never goes back to shadcn for it.
