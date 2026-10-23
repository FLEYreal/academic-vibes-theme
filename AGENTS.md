# AGENTS.md

Guidance for AI coding agents working on **Academic-Vibes**, a near-black Obsidian theme set in Noto Serif.

## Repository layout

| File | Purpose |
|---|---|
| `theme.css` | The whole theme. Readable CSS at the top, the bundled font as base64 `@font-face` rules at the end. |
| `manifest.json` | Theme name, version, `minAppVersion`, author. |
| `screenshot.png` | 512×288 thumbnail for Obsidian's theme browser. |
| `README.md` | Shown on GitHub and as the theme's page in Obsidian. |
| `LICENSE` | MIT for the theme code. |
| `OFL.txt` | SIL Open Font License 1.1 for the bundled Noto Serif. |

There is no build system, package manager or test suite. `PLAN.md` is the owner's local plan and is excluded from git via `.git/info/exclude`; never commit it.

## Theme structure

- `body { … }` sets fonts, sizes and weights for both modes. Only the note font is themed (`--font-text-theme`); the interface and code keep Obsidian's defaults.
- `.theme-dark { … }` holds all colors: the `--color-base-*` greys, the accent (`--accent-h/s/l`) and the Flexoki palette. The theme is **dark mode only**; light mode receives the fonts but no colors.
- Use Obsidian's CSS variables rather than selectors on internal class names where possible, as they survive app updates.

## The bundled font

Obsidian downloads only `theme.css` and `manifest.json`, so the font must live inside `theme.css` as base64 WOFF2. Never add separate font files and reference them by path.

To regenerate it (requires `fonttools` and `brotli`, e.g. in a throwaway venv):

```bash
# Source: google/fonts, ofl/notoserif/NotoSerif[wdth,wght].ttf and NotoSerif-Italic[wdth,wght].ttf
fonttools varLib.instancer "NotoSerif[wdth,wght].ttf" wdth=100 wght=400:700 -o NotoSerif-pinned.ttf
pyftsubset NotoSerif-pinned.ttf --layout-features='*' --flavor=woff2 --output-file=NotoSerif.woff2 \
  --unicodes="U+0000-024F,U+0259,U+02B0-02FF,U+0300-036F,U+0400-052F,U+1C80-1C88,U+1E00-1EFF,U+2000-206F,U+2070-209F,U+20A0-20CF,U+2100-214F,U+2150-218F,U+2190-21FF,U+2212,U+2215,U+2C60-2C7F,U+2DE0-2DFF,U+A640-A69F,U+A720-A7FF,U+FB00-FB06,U+FE2E-FE2F,U+FEFF,U+FFFD"
# Repeat for the italic, then base64-encode both into the @font-face rules at the end of theme.css.
```

- The weight range in `@font-face` (`400 700`) must cover every weight used in the CSS (currently 600 and 700).
- Keep the copyright and license comment above the `@font-face` rules, and keep `OFL.txt`. The OFL requires both.
- The Obsidian directory warns when `theme.css` is large. That is expected with an embedded font; keep the subset tight rather than adding scripts.

## Rules that are easy to break

- **`LICENSE` must stay the exact MIT text.** Any extra paragraph makes GitHub classify it as "Other", and Obsidian's directory then reports "no recognized license". Put licensing notes in `README.md` instead.
- **The theme name is permanent** once submitted to the directory. Do not change `name` in `manifest.json`. Names may not contain "Obsidian" or "Theme".
- **Release tags equal the manifest version exactly**, with no `v` prefix (`1.0.0`, not `v1.0.0`).
- **Versions use semantic versioning:** patch for fixes and color tweaks, minor for new features, major for breaking or milestone releases.

## Testing changes

1. Copy `theme.css` and `manifest.json` into `<vault>/.obsidian/themes/Academic-Vibes/`. The folder name must equal the manifest `name`.
2. Select the theme in **Settings → Appearance → Themes** and check headings, bold, italic, bold italic, quotes, lists, highlights, tags, callouts, code and Cyrillic text in both Reading view and Live Preview.
3. A user-chosen accent color in **Settings → Appearance** overrides the theme's accent. Clear it when judging the theme's own colors.
4. To test `minAppVersion`, run an old release from `obsidianmd/obsidian-releases` offline (`unshare --user --map-current-user --net`) with an isolated `XDG_CONFIG_HOME`. Unset `ELECTRON_RUN_AS_NODE`, which editors such as VS Code set, or Electron starts as plain Node.

## Screenshot

Capture the Obsidian window at **1024×576** and downscale 2× to **512×288** (`convert in.png -filter Lanczos -resize 512x288! -strip screenshot.png`). Use Reading view, scroll the note to the top, close unrelated tabs and keep the mouse out of the window.

## Releasing

1. Bump `version` in `manifest.json` and commit together with the changes it ships.
2. Push `main`, then create the release with both files attached:
   `gh release create <version> theme.css manifest.json --target main --title "<version>" --notes "…"`
3. In the community directory, use **Check for new releases** to trigger the automated review.

Do not commit, push or release without the owner's explicit go-ahead.
