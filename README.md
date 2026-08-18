# Steam Session Manager

A lightweight, highly modular session manager designed to run Wayland and X11 compositors tailored for Steam's **GamepadUI** (Gamepad Mode/Deck UI). It offers an interactive terminal menu, built-in system controls, and dynamic compositor discovery out of the box.

---

## Features

- **Dynamic Compositor Discovery**: Automatically scans and populates the session selection menu from scripts inside the `compositors/` directory. No hardcoded configuration changes needed to add new environments.
- **Pre-configured Compositors**:
  - **Gamescope**: Steam's official micro-compositor, optimized for gaming with custom frame pacing, scaling, and low latency.
  - **Niri**: A modern scroll-based Wayland compositor executed safely as a systemd user service.
- **Built-in System Control (`steam-control`)**: Provides unified commands to adjust system volume, toggle microphone mute status (via `wpctl`/PipeWire), and control display brightness (via `brightnessctl`).
- **Gaming Optimizations Built-In**:
  - Automatically enables the Nvidia shader disk cache with a generous 12GB size limit (`__GL_SHADER_DISK_CACHE_SIZE`).
  - Pre-enables MangoHUD overlays.
- **Configurable Cyclical Menu**: Supports custom configuration templates to bypass the menu entirely and directly boot into your favorite compositor in a seamless loop.

---

## Directory Structure

```text
/home/juliano/Workspace/Lab/SteamSession/
├── steam-session          # Main controller script and interactive terminal menu
├── steam-control          # Audio, microphone, and brightness hardware controls helper
├── lib/
│   └── common             # Common variables, configuration loaders, and error-reporters
└── compositors/           # Standalone modular compositor wrappers
    ├── gamescope          # Gamescope (GamepadUI) session launcher
    └── niri               # Niri systemd user-service wrapper
```

---

## Installation & Setup

1. Place the repository files in your preferred location (e.g., `~/Workspace/Lab/SteamSession`).
2. Make sure the primary scripts are executable:
   ```bash
   chmod +x steam-session steam-control compositors/*
   ```
3. To customize the default session compositor or behavior, create a configuration file at `~/.config/steam-session/config`:
   ```bash
   # ~/.config/steam-session/config
   DEFAULT_COMPOSITOR="gamescope"  # Key matching the file name in compositors/
   SKIP_MENU="false"               # Set to "true" to bypass the selection menu on boot
   ```

---

## Usage

Start the session manager by running the main script:
```bash
./steam-session
```

### Overriding Menu via Command Line
You can temporarily bypass configured defaults by passing the compositor's file name or menu number directly to the launcher script:
```bash
./steam-session niri        # Force-starts Niri
./steam-session gamescope   # Force-starts Gamescope
./steam-session menu        # Forces the selection menu to show, bypassing SKIP_MENU="true"
```

### Hardware Controls (`steam-control`)
`steam-control` is designed to be mapped to hotkeys or triggered inside steam shortcuts:
```bash
# Volume Controls
./steam-control volume +5       # Increase volume by 5%
./steam-control volume -5       # Decrease volume by 5%
./steam-control volume mute     # Toggle audio mute

# Microphone Controls
./steam-control mic mute        # Toggle mic mute

# Brightness Controls
./steam-control brightness +10  # Increase brightness by 10%
./steam-control brightness -10  # Decrease brightness by 10%
```

---

## Extending: Adding New Compositors

Adding a new desktop environment or compositor is exceptionally straightforward.

1. Create a new executable script file inside `compositors/` named after your environment (e.g., `compositors/hyprland`).
2. Inside that script, define:
   - `COMPOSITOR_LABEL` (optional): The human-readable string displayed in the menu.
   - A function named `start_<filename>` containing your session initialization command.

### Example (`compositors/hyprland`)
```bash
#!/bin/bash
# ============================================================
# compositors/hyprland
# ============================================================

COMPOSITOR_LABEL="Hyprland (Wayland)"

start_hyprland() {
    clear
    # Exec your compositor here
    exec Hyprland
}
```

Once saved and marked as executable (`chmod +x compositors/hyprland`), the menu will dynamically pick it up on the next startup!
