# AGENTS.md - Guide for Agentic Coding Agents

This document provides guidelines for AI coding agents working in this ZMK keyboard firmware configuration repository.

## Repository Overview

This is a ZMK (Zephyr Mechanical Keyboard) firmware configuration repository for a Corne keyboard with Nice!View displays, using nice!nano v2 controllers. The keyboard uses Brazilian ABNT2 layout mappings.

## Build System

### Build Commands

**Main build process:**
- Builds are triggered automatically via GitHub Actions on push/PR
- The build uses ZMK's official build workflow: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
- Build configuration is defined in `build.yaml`
- Build artifacts are generated as `firmware.zip` containing left/right half firmware files

**Local validation:**
- This project does not have local build/test commands
- All builds happen through GitHub Actions
- Check `.github/workflows/build.yml` for CI configuration
- Build logs are stored in `logs/` directory

**Keymap visualization:**
- Keymap diagrams are auto-generated on push via `draw-keymaps.yml` workflow
- Uses `keymap-drawer` tool to generate SVG diagrams
- Configuration: `keymap_drawer.config.yaml`
- Output: `keymap-drawer/corne.svg`

### Testing/Validation

- No automated tests exist for this project
- Validation is done through successful firmware builds
- Testing is manual: flash firmware and verify keyboard behavior
- Syntax validation happens during ZMK build process

## File Structure

```
config/
  ├── corne.keymap       # Main keymap configuration (C/devicetree)
  ├── corne.conf         # Keyboard-specific configuration
  ├── keys_abnt2.h       # Brazilian ABNT2 key mappings
  └── west.yml           # West manifest for ZMK dependencies
build.yaml               # Build matrix configuration
keymap_drawer.config.yaml # Keymap visualization config
.github/workflows/       # CI/CD automation
```

## Code Style Guidelines

### Language: C/Devicetree for ZMK

This project uses ZMK's devicetree-based configuration format, which is C preprocessor syntax with devicetree overlays.

### File Naming

- Keymap files: `<keyboard_name>.keymap`
- Config files: `<keyboard_name>.conf`
- Header files: snake_case with `.h` extension (e.g., `keys_abnt2.h`)

### Formatting

- **Indentation:** Use spaces, NOT tabs (except in devicetree nodes where tabs may be used)
- **Line length:** No strict limit, but keep reasonable (~120 chars)
- **Comments:** Use C-style comments `/* */` for blocks, `//` for single lines
- **Whitespace:** Maintain alignment in keymap bindings for readability

### Keymap Structure

**Layer definitions:**
```c
#define ALPHAS 0
#define EDIT 1
#define NUMBERS 2
#define SYMBOLS 3
```

**Macro definitions:**
- Use descriptive names for shortcuts (e.g., `L_CUT`, `L_COPY`, `PASTE`)
- Prefix local defines with `L_` to avoid redefinition warnings
- Group related macros together with comments

**Keymap bindings:**
- Align bindings vertically for visual representation of physical layout
- Use 3-column thumb cluster format (as per Corne layout)
- Include `display-name` property for each layer

**Example formatting:**
```c
alphas_layer {
    display-name = "alphas";
    bindings = <
&kp Q  &kp W  &kp E      &kp R       &kp T                    &kp Y            &kp U      &kp I      &kp O   &kp P         
&kp A  &kp S  &kp D      &kp F       &kp G                    &kp H            &kp J      &kp K      &kp L   &kp SEMI  
&kp Z  &kp X  &kp C      &kp V       &kp B                    &kp N            &kp M      &kp COMMA  &kp DOT &kp FSLH       
              &kp LCTRL  &kp LSHIFT  &lt EDIT SPACE           &lt NUMBERS RET  &kp LGUI  &kp LALT
    >;
};
```

### Naming Conventions

- **Layers:** SCREAMING_SNAKE_CASE (e.g., `ALPHAS`, `EDIT`, `NUMBERS`)
- **Macros/Defines:** SCREAMING_SNAKE_CASE (e.g., `L_CUT`, `BR_ACUT`)
- **Behaviors:** snake_case (e.g., `paste`, `home`, `bspc`)
- **Combos:** snake_case descriptive names (e.g., `combo_acute`, `lbrc`, `esc`)

### Comments

- Add section headers with comments for major sections
- Document non-obvious behavior customizations
- Include key position diagrams for combos
- Reference external resources when applicable

**Example:**
```c
/* Layers */
#define ALPHAS 0

/* Shortcuts */
//L_ stands for LOCAL, this is to avoid redefinition warnings during the build
#define L_CUT LS(DEL)

/* KEY POSITIONS
     0   1   2   3   4      5   6   7   8   9
    10  11  12  13  14     15  16  17  18  19
    20  21  22  23  24     25  26  27  28  29
            30  31  32     33  34  35          */
```

### Brazilian ABNT2 Localization

- Use `keys_abnt2.h` for all BR-specific key mappings
- Prefix Brazilian keys with `BR_` (e.g., `BR_ACUT`, `BR_TILD`)
- Include conditional compilation guard for keymap-drawer:
```c
#ifndef KEYMAP_DRAWER
#include "keys_abnt2.h"
#endif
```

### ZMK-Specific Patterns

**Behaviors (custom key actions):**
- Define in `behaviors` node
- Use `compatible = "zmk,behavior-*"` property
- Common types: `mod-morph` for shift/modifier behavior changes

**Combos (simultaneous key presses):**
- Define in `combos` node with `compatible = "zmk,combos"`
- Specify `key-positions` array (0-indexed physical positions)
- Set reasonable `timeout-ms` (typically 50ms)

**Conditional layers:**
- Define in `conditional_layers` node
- Use `if-layers` to specify trigger layers
- Use `then-layer` for activated layer

## Configuration Files

### corne.conf

- Sets keyboard-specific settings (name, display, sleep)
- Use `CONFIG_ZMK_*` prefix for all settings
- Boolean values: `y` for yes, `n` for no
- Numeric values: milliseconds for timeouts

### build.yaml

- Defines build matrix for GitHub Actions
- Each entry specifies board + shield combination
- Format:
```yaml
include:
  - board: nice_nano_v2
    shield: corne_left nice_view_adapter nice_view
```

### west.yml

- West manifest for dependency management
- Points to ZMK firmware repository
- Typically no changes needed unless using custom ZMK fork

## Git Workflow

### Commit Messages

- Write in Portuguese (Brazilian)
- Be descriptive and concise
- Focus on "why" rather than "what"
- Examples:
  - "Adiciona combo para parênteses na camada base"
  - "Ajusta timeout dos combos de acentuação"
  - "Corrige mapeamento da tecla cedilha"

### Branches

- Use Portuguese for branch names
- Use kebab-case: `ajusta-combos`, `adiciona-layer-numeros`

### Pull Requests

- Write PR titles and descriptions in Portuguese
- Include summary of changes and motivation
- Reference related issues if applicable

## Common Tasks

### Adding a New Key Binding

1. Locate the appropriate layer in `config/corne.keymap`
2. Find the physical position in the bindings array
3. Use ZMK binding syntax: `&kp KEY` or `&lt LAYER KEY`
4. Maintain alignment for readability
5. Update keymap diagram will auto-generate on push

### Adding a Combo

1. Add to `combos` node in `config/corne.keymap`
2. Determine key positions (see diagram in file)
3. Define with: name, key-positions, bindings, optional timeout-ms
4. Test by pushing and building firmware

### Modifying ABNT2 Mappings

1. Edit `config/keys_abnt2.h`
2. Follow existing pattern: `#define BR_KEY ZMK_KEY`
3. For shifted keys: use `LS(BR_KEY)` pattern
4. For AltGr keys: use `RALT(BR_KEY)` pattern

### Adjusting Display/Power Settings

1. Edit `config/corne.conf`
2. Modify `CONFIG_ZMK_*` settings
3. Common settings:
   - `CONFIG_ZMK_IDLE_TIMEOUT` (milliseconds)
   - `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` (milliseconds)
   - `CONFIG_ZMK_DISPLAY` (y/n)

## Flashing Firmware

While agents don't flash firmware, understanding the process helps:

1. Push changes to trigger GitHub Actions build
2. Download `firmware.zip` artifact from Actions tab
3. Extract to get `corne_left-*.uf2` and `corne_right-*.uf2`
4. Double-press reset button on keyboard half (enters bootloader)
5. Copy appropriate `.uf2` file to USB drive that appears
6. For keymap-only changes, flashing left half is sufficient

## References

- [ZMK Documentation](https://zmk.dev/docs)
- [ZMK Keycodes](https://zmk.dev/docs/codes)
- [Devicetree Bindings](https://zmk.dev/docs/config)
- [Keymap Drawer](https://github.com/caksoylar/keymap-drawer)
- [ZMK Locales](https://github.com/joelspadin/zmk-locales)
- [ABNT2 Layout Reference](https://github.com/zmkfirmware/zmk/issues/177#issuecomment-1306247821)

## Important Notes

- Always validate changes by pushing and checking GitHub Actions build status
- Keymap changes only affect left half (for split keyboards)
- Changes to `build.yaml` or `west.yml` affect both halves
- Display/hardware config changes require flashing respective half
- Use keymap-drawer conditional compilation guard when including locale headers
