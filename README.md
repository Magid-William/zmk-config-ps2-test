# zmk-config-ps2-test

Driver: [zmk-ps2-trackpoint-driver](https://github.com/Magid-William/zmk-ps2-trackpoint-driver)

Direct PS/2 trackpoint bench config for the nice!nano — no ATtiny/Pro Mini
coprocessor. The trackpoint streams standard 3-byte PS/2 packets on power-up
and rejects every host command, so the driver runs in
`ZMK_INPUT_MOUSE_PS2_NO_HOST_COMMANDS` mode (never sends a command).

## Wiring

| TrackPoint | nice!nano |
|---|---|
| DAT | P0.08 |
| CLK | P0.06 |
| RST | float (it rejects everything anyway) |

Both lines use the internal pull-ups; the TP is powered from the P0.13 EXT
rail.

## Features

- **Direction** — `zip_xy_transform INPUT_TRANSFORM_XY_SWAP` on the listener.
- **Touch layer-toggle** — touching the nub activates `tp_layer` (layer 2)
  for a 500 ms idle timeout: MB1/MB2/MB3 on the right thumb cluster (row 3).
- **Scroll (hold J)** — `&lt 3 J` layer-tap; tilt becomes vertical scroll
  (Y-inverted, `zip_scroll_scaler 1 2`). MB buttons stay available.
- **Volume (hold K)** — `&lt 4 K` layer-tap; tilt becomes volume (te9no
  keybind, `tick 32`, Y-inverted) while `volume_move_zero`
  (`zip_xy_scaler 0 1`) keeps the cursor put.
- **PowerCurve** — on-device (RawAccel "Power"), tuning in the `tpoint0`
  node.
- **Smooth scrolling** — `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y`
  (HID Resolution Multiplier).

## Layers

| # | Name | Trigger |
|---|---:|---|
| 0 | QWERTY | base (J/K are layer-taps) |
| 1 | LOWER | `&mo 1` (function keys) |
| 2 | tp_layer | touch the nub (500 ms) |
| 3 | scroll_layer | hold J |
| 4 | volume_layer | hold K |

## Config — minimal change set for all features

Everything below is already in this repo; this is the minimal combination
that turns all features on.

`ps2test_right.conf`:

```ini
CONFIG_PS2=y
CONFIG_PS2_UART=n
CONFIG_PS2_GPIO=y
CONFIG_ZMK_INPUT_MOUSE_PS2_NO_HOST_COMMANDS=y
CONFIG_PS2_GPIO_NO_RESEND=y
CONFIG_PS2_GPIO_INTERNAL_PULLUP=y
CONFIG_PS2_GPIO_TIMING_SCL_CYCLE_MAX=8000
CONFIG_PS2_GPIO_WORK_QUEUE_CB_STACK_SIZE=4096
CONFIG_ZMK_INPUT_MOUSE_PS2_POWER_CURVE=y
CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y
```

On the `zmk,input-mouse-ps2` node (`tpoint0`):

```dts
&tpoint0 {
    curve-sens = <128>;
    curve-rate = <18>;
    curve-exponent = <256>;
    curve-start = <77>;
};
```