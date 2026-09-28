# Crosses 54 firmware build findings

**Recorded:** 2026-09-28
**Status:** The firmware configuration builds successfully in GitHub Actions. The
firmware has not been tested on the physical keyboard as part of this
investigation.

## Hardware context

The owner's order details describe a wireless Crosses 54 (4x6 plus thumb keys)
with a PMW3610 trackball on each half, nice!nano v2 controllers, and no screen.
This is the owner's supplied order specification; it has not been independently
verified against the physical device.

## Finding

The repository's `config/west.yml` imports the Crosses shield support from
Good-Great-Grand-Wonderful's `gggw-zmk-keebs` project. That upstream shield
already defines the trackball hardware and split-input path: a local sensor on
each half and forwarding of the peripheral half's input to the central half.
The local overlays had added another trackball setup on top of that support.

The five removed local files were:

- `config/crosses_left.overlay`
- `config/crosses_right.overlay`
- `config/crosses_left.conf`
- `config/crosses_right.conf`
- `config/crosses_trackball_shared.dtsi`

The overlays redeclared SPI/pinctrl and listener configuration, and included
references such as `&tb_left_sensor` and `&tb_central_sensor` that were not
defined by the upstream shield. This made the local overlays conflict with the
shield's device tree and gave a concrete explanation for the repeated build
failures.

The previous GitHub Actions logs had expired when investigated, so their exact
compiler or devicetree error messages could not be checked. Therefore the
specific cause of each earlier failure is not confirmed from those logs. The
configuration conflict was identified by comparing the local files with the
upstream shield, and removing those files was followed by successful builds.

## Change made

The conflicting local files were removed, restoring the template's upstream
shield configuration. The existing `build.yaml` targets were kept:

- `crosses_54_left` on `nice_nano_v2`
- `crosses_54_right` on `nice_nano_v2`, with ZMK Studio enabled
- `settings_reset`

The keymap and the other base configuration files were not changed in this
cleanup. The change was merged into `main` in
[pull request #1](https://github.com/mcwii/Crosses54_keyboard/pull/1)
(merge commit `a8356438e983dfff6794520a3ce328b9e3e85e1d`).

## Build verification

The GitHub Actions build passed for both keyboard halves and the settings-reset
target. The merged-PR run is
[available here](https://github.com/mcwii/Crosses54_keyboard/actions/runs/36343581507).
Its build artifacts verify that firmware can be compiled for those targets;
they do not prove that the sensors work on the owner's particular hardware
until the firmware is flashed and tested.

The repository is public and GitHub Actions successfully ran the build, so
repository visibility was not the cause of the prior build failures. The
cleanup also did not change or recover firmware already installed on either
controller.

## References

- Local west manifest: `config/west.yml`
- Local build targets: `build.yaml`
- Upstream shield source:
  [Good-Great-Grand-Wonderful/gggw-zmk-keebs](https://github.com/Good-Great-Grand-Wonderful/gggw-zmk-keebs/tree/main/boards/shields/crosses)
- Merged fix: [PR #1](https://github.com/mcwii/Crosses54_keyboard/pull/1)
