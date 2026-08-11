# fnaf-python-bot

This program can complete Fnaf 1 and unlock 3 stars without user input. Even if a night is failed, it will attempt it again. Please note that this wasn't made with the intention of being a cheat to unlock everything. (You can easily do that by editing the save file.)

> **Fork note:** this fork focuses on making Bonnie (and the other door checks) reliable across different resolutions / rendering modes. See the changelog below.

This program was made and tested on Windows 11 using Python version 3.14.3.

## Required modules

- `pyautogui`
- `pillow` (for pyautogui)
- `psutil` (for detecting when the app opens; not needed to beat the game)
- `pygetwindow` (fallback window-title detection when the exe process name isn't found)

## How to run

First, have Python with pip installed: https://www.python.org/downloads/

Next, install the dependencies:

```bash
pip install pyautogui pillow psutil pygetwindow
```

Finally, run the program:

```bash
python beatfnaf1.py
```

Now just open `FiveNightsatFreddys.exe` and after a while it should start automatically moving the mouse and clicking to control the game. The program may break if you attempt to interfere with it while it's running.

To quit the program: Press `esc` to close the game, then press `ctrl + c` in the console to terminate the script.
NOTE: The program doesn't detect when the game closes. So be aware that when you close the game, the mouse may still move around and click on things.

The program will quit automatically if 3 stars are detected on the menu.

## Detection & tuning

Detection logic lives in `beatfnaf1.py`. The key behavior:

- **Bonnie** is detected by counting **bright-blue body pixels** in the left door area (`countBluePixels()` on the `bonnieBlueRegion`). This was calibrated from real gameplay recordings and is far more reliable than single-pixel color/shadow anchors, which produced false positives on some setups.
- **Door latch**: once an animatronic is detected at a door, that door stays closed for `DOOR_LATCH_TIME` (default 20s), refreshed on every re-detection. A single missed frame can no longer reopen the door while the animatronic is still there.
- **Chica** uses a color-region check (`chicaDoorRegion`) with multi-pixel tolerance.
- **Foxy** is checked via the west hall / hall corner cameras.
- The game-process check matches the exe name case-insensitively and falls back to a `pygetwindow` window-title match for builds whose process name differs (e.g. running in a VM).

Tunable constants at the top of `beatfnaf1.py`:

| Constant | Default | Purpose |
|----------|---------|---------|
| `LIGHT_WINDOW` | `0.20` | Seconds the light is held on per check (power usage) |
| `LIGHT_SAMPLES` | `5` | Max fresh screenshots per light window |
| `DOOR_LATCH_TIME` | `20.0` | Seconds a closed door stays closed after last detection |
| `BONNIE_BLUE_MIN` | `250` | Bright-blue pixel threshold before Bonnie counts as present |
| `REGION_RADIUS` / `REGION_MIN_HITS` | `3` / `2` | Region-scan size / minimum matching pixels |

If Bonnie is still missed on your setup, lower `BONNIE_BLUE_MIN` a bit; if the door false-closes too often, raise it.

## Changelog (fork)

- **bright-blue detection + door latch**: replaced unreliable color/shadow anchors with bright-blue body-pixel counting for Bonnie, added a 20s door latch, shortened the light window to save power.
- **robust multi-res sampling**: region-scan around anchors instead of one-pixel exact matches; case-insensitive game check with window-title fallback.
- **tolerance fix**: raised door tolerance, loosened shadow check tolerances, re-grab fresh screenshot during the light window.
