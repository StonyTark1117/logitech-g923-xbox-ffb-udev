# logitech-g923-xbox-ffb-udev

Udev rule that enables force feedback for the **Logitech G923 Racing Wheel
for Xbox One and PC** (`046d:c26e`) when running games through Proton on
Linux.

## Why this is needed

Proton's `winebus.sys` opens the wheel's `/dev/hidraw` nodes to read the HID
descriptor and advertise DirectInput FFB effect bits to Windows games. By
default those nodes are `0600 root:root`, so `winebus` silently falls back to
the `evdev` path. On Xbox-class devices the `evdev` path only exposes
`FF_RUMBLE`, so games like *Forza Horizon* hide the force-feedback UI entirely
and only buzzy rumble is emitted.

This package ships a single udev rule that gives the `input` group read/write
access to the wheel's hidraw nodes. After installation, `winebus` can read the
descriptor, the DirectInput FFB path becomes available, and in-game FFB
settings appear normally.

This is **only required for the Xbox/PC variant** (`c26e`). The PlayStation/PC
variant (`c266`) uses the Classic FFB protocol via `hid-lg4ff` and is already
covered by `oversteer`'s sysfs rules.

## Relationship to other packages

- **Requires [`logitech-g923-xbox-udev`](https://aur.archlinux.org/packages/logitech-g923-xbox-udev)** — that package handles the `usb_modeswitch` from the wheel's boot PID (`c26d`) to its PC PID (`c26e`). Without it, this rule has nothing to match.
- **Complementary to [`oversteer`](https://github.com/berarma/oversteer)** — oversteer's `99-logitech-wheel-perms.rules` chmods the *sysfs* attributes (`range`, `leds/*/brightness`) for `c26e` but does not touch `/dev/hidraw*`. This package fills that gap.

## Installation

From the AUR:

```sh
paru -S logitech-g923-xbox-ffb-udev
# or
yay -S logitech-g923-xbox-ffb-udev
```

From a local clone:

```sh
git clone https://github.com/StonyTark1117/logitech-g923-xbox-ffb-udev.git
cd logitech-g923-xbox-ffb-udev
makepkg -si
```

The post-install hook reloads udev and retriggers hidraw automatically. Unplug
and replug the wheel afterwards to ensure existing file descriptors pick up
the new permissions.

## Verifying it worked

After installation and a replug:

```sh
ls -l /dev/input/by-id/usb-Logitech_G923_Racing_Wheel_for_Xbox_One_and_PC_*-hidraw \
    | xargs -I{} readlink -f {} | xargs ls -l
```

You should see `crw-rw---- root input` on both hidraw nodes (one per HID
interface). Your user must be in the `input` group:

```sh
groups | grep -q input || sudo gpasswd -a "$USER" input
```

Then launch your game. In Forza Horizon and similar DX12 titles, the force
feedback section of the wheel settings should now be visible.

## License

MIT. See [LICENSE](LICENSE).
