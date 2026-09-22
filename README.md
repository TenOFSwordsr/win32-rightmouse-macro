# Fun Projects (Ruby) - Windows Right-Mouse Toggle Macro

A single small Ruby script that holds the right mouse button down on demand: F7 presses and holds
the right button, F8 releases it, F9 exits. It polls every virtual key through `GetAsyncKeyState` in
a tight loop and synthesises button events with `mouse_event`, which makes it a Win32 input API
experiment rather than anything reusable. Saved with a `.TXT` extension, so nothing runs it by
accident.

**Suggested repo name:** `win32-rightmouse-macro`
**Stack:** Ruby with the `win32api` and `win32con` gems (`user32` bindings, `Keymap` constants); Windows only
**Status:** archived
**Last modified:** 2019-12-04

## What it does

- Binds `GetAsyncKeyState` and `mouse_event` from `user32.dll` through `Win32::API`.
- `Input` module keeps a 255-entry `@keystate` array; `update` walks every virtual key and, for each
  one currently down (`& 0x8000`), increments its counter, resetting to 0 when released. That gives
  three queries: `trigger?` (counter == 1, i.e. just pressed), `pressed?` (>= 1) and `repeat?` (how
  long it has been held, in ticks).
- `right_down` / `right_up` / `right_click` wrap `MOUSEEVENTF_RIGHTDOWN` and `MOUSEEVENTF_RIGHTUP`.
- Main loop: sleep, `Input.update`, press on `vk_F7`, release on `vk_F8`, `exit` on `vk_F9`; an
  `ensure` block releases the button on the way out so the game or app left behind does not see a
  permanently held button.

## Layout

```
ac.RB.TXT   the whole macro (~85 lines)
```

## Running it

Rename to `ac.rb`, install the gems, and run in a console:

```
gem install win32api win32con
ruby ac.rb
```

(Both are from the old `rubinutils` family that provides `Win32::API` plus the `win32con/keymap`
constants; there is no Gemfile here to pin them against.)

## Notes

- `TickSleepTime = (1 / 120)` is integer division in Ruby, so it evaluates to `0` and
  `sleep(0)` does nothing: the loop actually spins as fast as the interpreter allows rather than the
  intended 120 Hz, hammering `GetAsyncKeyState` 255 times per pass. Use `(1.0 / 120)` to fix it.
- The `FPS = 120` constant is declared but never used; the divisor is hardcoded a second time.
- Requires admin-free but hook-free conditions: key state is read globally, so it works on the active
  window only, and the emitted click lands wherever the cursor is.
