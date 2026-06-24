# Lily58 + nice!nano Dongle ZMK Config

This is a [ZMK](https://zmk.dev) user config for a **Lily58** split keyboard that
connects to the host computer through a **third nice!nano acting as a USB
dongle**, instead of the left half being the Bluetooth host.

## Why a dongle?

Normally in a ZMK split keyboard, one half (by convention, the left) is the
"central" — it talks to your computer over USB/Bluetooth, and the other half
is a "peripheral" that only talks to the central. With a dongle, a third
controller takes over the central role:

- Both keyboard halves become peripherals, connecting only to the dongle over BLE.
- The dongle plugs into your computer over USB and shows up as the keyboard.
- Both halves get a battery-life boost, since being a peripheral uses
  noticeably less power than being the BLE central.
- Connecting to a new computer is just "plug in the dongle" instead of re-pairing Bluetooth.

The trade-off: you now need 3 controllers instead of 2, and the keyboard
won't work at all if the dongle is lost, forgotten, or out of battery.

Reference: [ZMK Split Keyboards docs](https://zmk.dev/docs/features/split-keyboards) ·
[ZMK Keyboard Dongle docs](https://zmk.dev/docs/hardware-integration/dongle)

## Hardware you need

- 2x nice!nano (or nice!nano v2) — one per Lily58 half (as normal)
- 1x **additional** nice!nano — this is your dongle
- A USB-C cable to connect the dongle to your computer
- (Optional) A small case/shell for the dongle, since it's a bare controller
  with no PCB of its own

## Repo layout

```
config/
  lily58_dongle.keymap   # <- Edit THIS to change your key bindings.
  lily58_dongle.conf     # Dongle-specific settings (USB, Studio, etc.)
  lily58.conf            # Shared settings for both Lily58 halves (no _left/_right suffix = applies to both)
boards/shields/lily58_dongle/
  Kconfig.shield          # Registers the "lily58_dongle" shield name
  Kconfig.defconfig       # Sets central role, peripheral count, BT connection limits
  lily58_dongle.overlay   # Mock kscan + Lily58's matrix transform/physical layout (so the dongle understands key positions)
  lily58_dongle.zmk.yml   # Hardware metadata
build.yaml                # Tells GitHub Actions which firmware files to build
```

### Why is the keymap on the dongle and not the halves?

In ZMK, **keymap logic always runs on the central**. Once you add a dongle,
the dongle *is* the central — so `config/lily58_dongle.keymap` is the only
keymap file that matters now. The left/right shields are built straight from
ZMK's upstream `lily58_left` / `lily58_right` definitions (just turned into
peripherals via `cmake-args` in `build.yaml`); you don't need your own
keymap files for them.

## Setup

### 1. Use this repo

Click **"Use this template"** (or fork it), then clone your copy locally,
or just edit files directly on GitHub — GitHub Actions will build the
firmware for you either way.

### 2. (Optional) Customize your keymap

Edit `config/lily58_dongle.keymap`. It starts as a copy of ZMK's stock
Lily58 keymap (3 layers: default, lower, raise) so you have a known-working
baseline. See the [ZMK Keymaps docs](https://zmk.dev/docs/keymaps) for syntax.

### 3. Push and download firmware

Commit and push your changes. Go to the **Actions** tab of your repo, open
the latest run, and download the `lily58-dongle-firmware` artifact once it
finishes. Unzip it — you should have:

- `lily58_dongle.uf2`
- `lily58_left.uf2`
- `lily58_right.uf2`
- `settings_reset.uf2`

### 4. Flash in this order

> [!WARNING]
> Never plug/unplug a TRRS/TRS cable while a controller is powered (by USB
> *or* battery) — this can permanently damage the controllers. This applies
> regardless of whether you use the wired or BLE split transport.

Because this dongle setup changes which side is "central," **old pairing
data on your controllers must be cleared first** — otherwise the halves may
try to pair with each other directly instead of with the dongle. This is
especially important if your left/right halves were previously paired in a
normal (no-dongle) configuration.

1. Put **all three** controllers into bootloader mode (double-tap reset).
2. Flash `settings_reset.uf2` to **each of the three controllers**, one at a
   time (re-enter bootloader mode each time).
3. Now flash the real firmware:
   - `lily58_dongle.uf2` → the dongle controller
   - `lily58_left.uf2` → the left half's controller
   - `lily58_right.uf2` → the right half's controller
4. Plug the dongle into your computer via USB. Power on both keyboard
   halves (USB or battery). Within a few seconds they should automatically
   pair with the dongle, and your computer will see the dongle as the
   keyboard.

Reference: [Connection Issues — Reset Split Keyboard Procedure](https://zmk.dev/docs/troubleshooting/connection-issues#split-keyboard-parts-unable-to-pair)

### 5. Future keymap changes

Since the dongle is now the central, you only need to reflash the **dongle**
when you change `lily58_dongle.keymap`. You only need to reflash the
left/right halves if you change settings in `lily58.conf` (or other files
that affect their hardware config).

## Troubleshooting

- **Halves won't pair with the dongle / pair with each other instead:**
  Redo the `settings_reset` procedure in step 4 above on all three controllers.
- **Weak/dropped connection:** try enabling `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y`
  in `config/lily58.conf` and `config/lily58_dongle.conf` (commented out by
  default — see those files).
- **Build fails on GitHub Actions:** check the Actions log for the specific
  error; the [Building Issues](https://zmk.dev/docs/troubleshooting/building-issues)
  page covers common causes.
- More general help: [ZMK Troubleshooting docs](https://zmk.dev/docs/troubleshooting) ·
  [ZMK Discord](https://zmk.dev/community/discord/invite)

## "Undongling"

If you ever want to go back to a normal left-central, right-peripheral
setup: remove the `cmake-args` lines from the `lily58_left` / `lily58_right`
entries in `build.yaml`, delete (or stop building) `lily58_dongle`, run the
`settings_reset` procedure again on the left and right controllers, then
reflash. The left half will become central again automatically (per ZMK
convention).

## Credits

Default keymap layout adapted from ZMK's in-tree
[`lily58.keymap`](https://github.com/zmkfirmware/zmk/blob/main/app/boards/shields/lily58/lily58.keymap)
(MIT licensed, ZMK Contributors). Matrix transform and physical layout
copied from
[`lily58.dtsi`](https://github.com/zmkfirmware/zmk/blob/main/app/boards/shields/lily58/lily58.dtsi)
per the [dongle guide](https://zmk.dev/docs/hardware-integration/dongle).
