# Vistify Digital Menu Board Sales Tool

A two-page browser-based sales tool for sizing and quoting Vistify digital
menu board installations. Built as static HTML/CSS/JS — no build step, no
dependencies — and intended to be hosted on GitHub Pages.

## Pages

| File | Purpose |
| --- | --- |
| `index.html` | Landing page. Redirects to `dmb-calculator.html`. |
| `dmb-calculator.html` | **Measurement Calculator.** Size the array (screen count, screen size, orientation), pick wall vs. ceiling mount, and get the Configuration SKU plus required pole hardware. |
| `screen-selector.html` | **Commercial Display Brand & Model Selector.** Customer-profile questionnaire that ranks TCL, Samsung, Sony, and consumer-TV-plus-player options and outputs an SKU Prefix for the recommended display. |

The two pages are designed as siblings: the Screen Selector chooses the
*display brand/model* (the SKU prefix), and the Calculator chooses the
*physical configuration* (the SKU body). When both have been used in the
same browser, the calculator concatenates the prefix in front of its
computed SKU.

## SKU sibling relationship

The two pages share state via `localStorage` so a value typed in one page
flows to the other without a server round-trip. The sync is **bidirectional**
— the combined SKU is displayed on both pages whenever both halves exist.

```
screen-selector.html  <----- localStorage ----->  dmb-calculator.html
+--------------------+                            +---------------------+
| Pick a display     |  vistify.skuPrefix         | Size the array      |
| (e.g. TCL TM)      |  vistify.skuPrefixName     | (count, size,       |
|  writes prefix --> | -------------------------> |  mount, orientation)|
|                    |                            |                     |
| <-- reads body to  | <------------------------- | writes body  -->    |
|     show combined  |  vistify.skuBody           |                     |
|     SKU on its     |                            | reads prefix to     |
|     own pill       |                            | show combined SKU   |
+--------------------+                            +---------------------+

         Pill on selector       =   "TCL-TM-3x55WL" (combined)
         Pill on calculator     =   "TCL-TM-3x55WL" (combined)
```

Storage keys (all under the `vistify.*` namespace):

| Key | Written by | Read by | Value |
| --- | --- | --- | --- |
| `vistify.skuPrefix` | `screen-selector.html` | `dmb-calculator.html`, `screen-selector.html` | The selected SKU prefix string, e.g. `TCL-TM-`. |
| `vistify.skuPrefixName` | `screen-selector.html` | `dmb-calculator.html` | Human-readable display name, e.g. `TCL TM Series`. |
| `vistify.skuBody` | `dmb-calculator.html` | `screen-selector.html` | The computed configuration SKU body, e.g. `3x55WL`. |
| `vistify.selectorPrefs` | `screen-selector.html` | `screen-selector.html` | JSON of the questionnaire answers, so the form survives navigation. |
| `vistify.calcState` | `dmb-calculator.html` | `dmb-calculator.html` | JSON of the full calculator configuration (count, size, orientation, mount type, ceiling height, head clearance, etc.), so the calculator's inputs survive navigation. |

The prefix keys are cleared together from the calculator via the **clear**
link beneath the Configuration SKU pill. Both pages listen for the browser
`storage` event, so changes made in one open tab live-update the other tab.

Display rules for the Screen Selector pill:

| Prefix? | Body? | Pill label | Pill value |
| --- | --- | --- | --- |
| no  | * | `SKU Prefix` | `—` |
| yes | no  | `SKU Prefix` | the prefix |
| yes | yes | `Configuration SKU` | `prefix + body` |

## Local development

There is no build step. Open `index.html` (or any page directly) in a
browser, or serve the folder with any static server:

```bash
# Python 3
python -m http.server 8000

# Or Node
npx serve .
```

Then visit `http://localhost:8000/`.

## Deploying to GitHub Pages

1. Create a new GitHub repository (e.g. `dmb-sales-tool`).
2. From this folder:

   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo>.git
   git push -u origin main
   ```

3. In the repo on GitHub: **Settings → Pages**. Under *Build and deployment*,
   set *Source* to **Deploy from a branch**, branch **main**, folder **/ (root)**,
   and save.
4. After ~1 minute the site will be live at
   `https://<your-username>.github.io/<repo>/`.

GitHub Pages will serve `index.html` at the root URL, which forwards to
`dmb-calculator.html`. The Screen Selector is reachable at
`<root>/screen-selector.html`.

> A `.nojekyll` file is included so GitHub Pages serves the files as-is
> without running the Jekyll preprocessor.

## File layout

```
.
├── index.html                 # GitHub Pages entry — redirects to dmb-calculator.html
├── dmb-calculator.html        # Measurement Calculator (canonical)
├── screen-selector.html       # Brand & Model Selector
├── favicon.ico                # Kept at root for browser auto-discovery
├── assets/
│   ├── brand/
│   │   ├── vistify-wordmark.png  # Header wordmark on every page
│   │   ├── tcl-logo.png          # Brand strip logos (selector page)
│   │   ├── samsung-logo.png
│   │   └── sony-logo.png
│   ├── favicon/                  # All non-root favicon sizes + apple-touch-icon
│   │   ├── favicon-16.png
│   │   ├── favicon-32.png
│   │   ├── favicon-48.png
│   │   ├── favicon-192.png
│   │   ├── favicon-512.png
│   │   └── apple-touch-icon.png
│   ├── displays/                 # Optimized product-shot thumbnails (cards)
│   │   ├── tcl-screen.jpg + tcl-screen@2x.jpg
│   │   ├── samsung-screen.jpg + samsung-screen@2x.jpg
│   │   ├── sony-screen.jpg + sony-screen@2x.jpg
│   │   └── consumer-screen.jpg + consumer-screen@2x.jpg
│   └── guides/                   # Hover-popover reference illustrations
│       ├── viewing-angle.jpg + viewing-angle@2x.jpg
│       └── ambient-lighting.jpg + ambient-lighting@2x.jpg
├── concept-files/             # Source spec sheets and original concept assets
│   ├── Commercial Screen Selector.xlsx
│   ├── Copy of Vistify Digital Menu Board Measurement Calculator Tools.xlsx
│   ├── Strong Mount Pole Accessories.xlsx
│   ├── Ceiling Mount Required Measurements.pdf
│   ├── V-Logo_Color.png       # Source for the favicon set
│   ├── Samsung screen.jpg     # Original sources for the display thumbnails
│   ├── Sony screen.jpg
│   ├── TCL screen.png
│   ├── minix.jpg              # Source for the Consumer TV + VPlayer thumbnail
│   ├── Viewing Angle.png      # Source for the Q1 hover popover
│   └── Ambient Lighting.png   # Source for the Q2 hover popover
├── .gitignore
├── .nojekyll
└── README.md
```

`concept-files/` is kept in the repository as the source-of-truth for the
spec sheets that drive the calculator's pole-selection logic, brand-spec
scoring, and ceiling-clearance math — and the original high-resolution
images the optimized thumbnails in `assets/` are generated from.

## License & ownership

(c) 2026 Vistify, Inc. All rights reserved. Internal sales tool.
