# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK (Zephyr Keyboard Firmware) configuration repository for the Sofle split ergonomic keyboard, designed for Nice Nano v2 / Pro Micro wireless NRF52840 controllers. The firmware includes ZMK Studio support for live remapping capabilities without flashing.

## Repository Structure

```
zmk-sofle/
├── config/                      # User configuration directory
│   ├── sofle.conf              # Main ZMK configuration (OLED, pointing, encoder, Studio)
│   ├── sofle.keymap            # Keymap definition with 4 layers (BASE, LOWER, RAISE, ADJUST)
│   └── west.yml                # West manifest for ZMK v0.3 submodule
├── boards/shields/sofle/        # Sofle shield definitions
│   ├── sofle.zmk.yml            # Shield metadata
│   ├── sofle.dtsi              # Shared device tree includes
│   ├── sofle_left.conf/right.conf # Half-specific configurations
│   └── sofle_left.overlay/right.overlay # Hardware-specific GPIO pin mappings
├── build.yaml                   # GitHub Actions build matrix
└── .github/workflows/build.yml  # CI/CD workflow using ZMK official build
```

## Build Commands

### Local Build with West
```bash
# Initialize west (first time only)
west init -l config
west update

# Build left half
west build -b nice_nano_v2 -s boards/shields/sofle/sofle_left -- -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

# Build right half
west build -b nice_nano_v2 -s boards/shields/sofle/sofle_right -- -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

# Build with Studio support (live remapping)
west build -b nice_nano_v2 -s boards/shields/sofle/sofle_left -- -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n -DCONFIG_ZMK_STUDIO=y

# Clean build directory
west build -b nice_nano_v2 -s boards/shields/sofle/sofle_left -- - pristine
```

### GitHub Actions Build
Push to trigger automated builds via `.github/workflows/build.yml`. Build outputs (UF2 files) are available in the Actions artifacts.

## Flashing Procedure

1. Enter bootloader mode: double-press reset button while USB connected
2. Drag & drop the appropriate UF2 file to the bootloader drive
3. Flash each half separately (left/right or dongle/peripheral)

### Resetting Split Keyboards
When experiencing connection issues or changing configurations:
1. Flash `settings_reset.uf2` to one half
2. Repeat for the other half
3. Flash actual firmware to both halves
4. Reset both halves simultaneously (or power cycle both)
5. Forget/re-pair keyboard on all host devices (Bluetooth pairing info is cleared)

## Configuration Architecture

### Keymap Structure (`config/sofle.keymap`)
The keymap uses Devicetree syntax with 4 layers:
- **BASE (0)**: Default QWERTY layer with mouse sensor (encoder for volume, scroll)
- **LOWER (1)**: Function keys, symbols, navigation
- **RAISE (2)**: Bluetooth device selection (BT_SEL 0-4), navigation
- **ADJUST (3)**: Conditional layer activated by LOWER+RAISE simultaneously for RGB/external power control

### Mouse Sensor Configuration
- `ZMK_POINTING_DEFAULT_MOVE_VAL`: 1200 (cursor movement sensitivity)
- `ZMK_POINTING_DEFAULT_SCRL_VAL`: 25 (scroll sensitivity)
- `&msc`: Scroll sensor behavior with acceleration exponent 1, 100ms time-to-max-speed
- `&mmv`: Move sensor behavior with 500ms time-to-max-speed

### Shield Definitions
Each half has separate configuration:
- `sofle_left.conf` / `sofle_right.conf`: Half-specific Kconfig settings
- `sofle_left.overlay` / `sofle_right.overlay`: GPIO pin mappings for that half

### Build Matrix (`build.yaml`)
Defines firmware variants:
- **Dongle with display**: Studio RPC support, central role
- **Left/Right halves**: Split peripheral role (no central)
- **Settings reset**: Clears Bluetooth pairing
- **Nice View variants**: Additional display support

### West Manifest (`config/west.yml`)
- Points to ZMK firmware v0.3
- Includes `zmk-dongle-display` repository for dongle OLED support
- Sets configuration directory path to `config`

## Key Configuration Options (`config/sofle.conf`)

- `CONFIG_ZMK_DISPLAY=y`: Enable OLED display
- `CONFIG_ZMK_POINTING=y`: Enable mouse pointing functionality
- `CONFIG_EC11=y`: Encoder support
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`: Maximum Bluetooth power (+8dBm)
- `CONFIG_ZMK_IDLE_TIMEOUT=180000`: 3-minute idle timeout
- `CONFIG_ZMK_SLEEP=n`: Sleep mode disabled
- `CONFIG_ZMK_STUDIO=y`: Live remapping support

## Bluetooth Management

ZMK uses 5 numbered slots (0-4) accessible via BT_SEL keys:
- Numbers on screen indicate active slot (1 = BT_SEL 0, 2 = BT_SEL 1, etc.)
- "X" icon indicates slot was used but not currently connected
- Use BT_CLR key to reset a slot for re-pairing
- Slots cannot be overwritten; must clear before pairing new device

## Layer Conventions

- Use `&mo` for momentary layer activation
- Use `&trans` for transparent/pass-through keys
- Use `&none` for blocked positions
- Conditional layers defined in `conditional_layers` node
- Sensor bindings (`sensor-bindings`) control encoder behavior per layer

## File Naming Conventions

- Configuration files: `*.conf` (Kconfig format)
- Keymap files: `*.keymap` (Devicetree syntax)
- Overlay files: `*.overlay` (Devicetree hardware pin mappings)
- Shield metadata: `*.zmk.yml` (YAML format)
