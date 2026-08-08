# Titan 2 fork notes

This branch (`titan2-keyboard`) adds dedicated support for the **Unihertz Titan 2**
physical keyboard on top of [ClassicOldSong/moonlight-android](https://github.com/ClassicOldSong/moonlight-android)
(for use with the [Apollo](https://github.com/ClassicOldSong/Apollo) host).

## What the hardware sends (measured, firmware EEA V01.00.14, Android 16)

Source: Pastiera project's Titan 2 device archive
(`docs/device-archives/unihertz-titan2` in https://github.com/palsoftware/pastiera).

| Key | Android keycode | Scancode |
| --- | --- | --- |
| Letters Q–M | standard `KEYCODE_*` | standard evdev (16–50) |
| Shift | `KEYCODE_SHIFT_LEFT` (59) | 42 |
| Alt | `KEYCODE_ALT_LEFT` (58) | 100 |
| Fn (when set to Ctrl in system settings) | `KEYCODE_CTRL_LEFT` (113) | 29 |
| SYM | `KEYCODE_SYM` (63) | 253 |
| Enter / Backspace / Space | standard | 28 / 14 / 57 |
| Keyboard touch gestures | phantom `KEYCODE 322 / 404` from a 2nd input device | 322 / 404 |
| Keyboard input device name | `titan2` (used for auto-detection) | — |

## What this branch changes

1. **Alt-layer fix (the big one).** On the Titan 2, `Alt+Q` must type `0`, but the
   event still carries `KEYCODE_Q` + Alt meta, so stock Moonlight sends **Alt+Q as a
   host shortcut** instead of the printed symbol. When Titan 2 mode is active and Alt
   is held with a letter key whose Unicode char differs from the base letter, the
   printed symbol is sent as UTF-8 text instead. Key-up events are consumed
   symmetrically. Digits row, `(` `)` `-` `_` `/` `:` `@` `*` `#` `+` `"` `'` `!` `.`
   `,` `?` etc. all type correctly now.
2. **SYM key → Left Ctrl.** `KEYCODE_SYM` is unmapped upstream and did nothing. It now
   acts as Left Ctrl (modifier-tracked through the normal special-key path), so
   `SYM+C/X/V/Z` work as host shortcuts even if Fn is left as Home/Assistant.
3. **Touch-gesture filter.** Phantom keycodes 322/404 injected by the keyboard's
   capacitive touch layer are swallowed.
4. **Auto-detection + toggle.** Enabled automatically when an input device named
   `titan2` is present; can be turned off in
   *Settings → Input Settings → Unihertz Titan 2 keyboard support*.

Files touched: `Game.java` (hook `handleTitan2Key()` in `handleKeyDown`/`handleKeyUp`),
`PreferenceConfiguration.java`, `preferences.xml`, `strings.xml`.

## Recommended Titan 2 setup

- System settings: set the **Fn key to Ctrl** — then you have two Ctrl keys (Fn + SYM).
- Enable Moonlight's **keyboard accessibility service** (already in this fork) so
  Android doesn't eat system-level combos.
- Do **not** enable *Ignore synthetic events* if you use an IME like Pastiera —
  IME-injected events have `deviceId <= 0` and would be dropped.

## Known gaps / ideas for v2

- No Esc, Tab, arrows, forward-Delete, F-keys yet. Candidates: `SYM+Space` → Tab,
  `SYM+Backspace` → Del, `SYM+I/J/K/L` → arrows (conflicts with Ctrl+arrows — needs a
  decision), long-press SYM → Esc.
- SYM opened the stock IME's symbol panel outside of streaming; while streaming the
  panel is suppressed by design (SYM is Ctrl). Symbols live on the Alt layer anyway.
- Behaviour with a third-party IME (Pastiera) installed but disabled is untested.

## Building

No Android SDK is available in the environment where this patch was written, so build
on your machine:

```bash
git clone https://github.com/ClassicOldSong/moonlight-android.git
cd moonlight-android
git remote add titan2 <your-fork-url>   # or apply the patch below
git checkout titan2-keyboard
./gradlew assembleDebug   # APK in app/build/outputs/apk/debug (nonRoot flavor)
```

or open in Android Studio and run the `nonRootDebug` variant. The upstream project
also ships `appveyor.yml` if you prefer CI builds.
