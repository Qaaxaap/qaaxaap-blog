---
title: A Concise Niri Tutorial
date: 2026-01-27 17:07:41
tags:
slug: niri-tutorial
lang: en
alt_url: /zh/2026/01/27/Niri-简明教程/
---
Written in a hurry, will polish it later

---
## Installation Environment

This assumes you've already installed Arch and have at least one AUR helper (like paru, yay).
I use yay here, so if you don't use yay, swap the commands out yourself — I trust that's not hard for an Arch user.
## Preparation Before Installing

Update your system first before doing anything else

```bash
yay -Syu
```

You need to download the following packages

```bash
sudo pacman -S niri xdg-desktop-portal-gtk xdg-desktop-portal-gnome alacritty swaybg swayidle hyprlock xwayland-satellite dolphin sddm brightnessctl wireplumber grim flameshot breeze wshowkeys-git fcitx5 fcitx5-qt fcitx5-chinese-addons blueman noto-fonts libnotify pipewire pipewire-pulse
yay -S noctalia-shell vicinae ttf-jetbrains-mono misans
```

Noctalia Shell is a UI built with Material Design. It can take over system notifications, sound and display brightness control, and our status bar is provided by it — we don't use waybar.

Vicinae is for launching apps, similar to the function of pressing the Super key on its own in Hyprland

## Configuring systemd

If you weren't already using sddm as your login manager, enable it now. If you have no idea what that means, you've probably already enabled it

```bash
sudo systemctl enable sddm.service
```

Then write a service for swayidle

```bash
vim ~/.config/systemd/user/swayidle.service 
```

Paste this inside

```
[Unit]
PartOf=graphical-session.target
After=graphical-session.target
Requisite=graphical-session.target

[Service]
ExecStart=/usr/bin/swayidle -w timeout 601 'niri msg action power-off-monitors' timeout 600 'hyprlock' before-sleep 'hyprlock'
Restart=on-failure
```

It means: after 600 seconds of no input, hyprlock locks the screen; at 601 seconds the monitors get turned off. For more actions see the official swaylock docs.

Next, edit niri's wants, which is a bit like dependencies

```bash
systemctl --user add-wants niri.service swayidle.service
```

This lets swayidle automatically take over locking, sleeping and so on

__Note: don't follow the many tutorials telling you to enable waybar and mako — we don't use them!__

Next, create the niri config file

```bash
vim ~/.config/niri/config.kdl
```

Then write in the following — but don't copy all of it, read the comments and change what needs changing

```bash
// Configuration related to input devices: keyboard, mouse, touchpad, etc.
input {
    keyboard {
        xkb {
            layout "us"
        }

        // Enable numlock at startup; omitting this setting disables it.
        numlock
    }

    touchpad {
        tap
        natural-scroll
        scroll-method "two-finger"
    }

    mouse {
        // Set mouse movement speed, from -1 to 1 (slow to fast)
        accel-speed 1
    }

    // By default niri takes over the power button to sleep; disable it here so the shutdown feature works
    disable-power-key-handling
    // Switch the mod key: use alt normally, Super inside nested windows.
    mod-key "Super"
    mod-key-nested "Alt"
}

// You can run `niri msg outputs` inside a niri instance to find monitor names.
output "HDMI" {
    // Uncomment to disable this monitor.
    off

    // Focus this monitor by default
    focus-at-startup

    // The format is "<width>x<height>" or "<width>x<height>@<refresh rate>".
    // If the refresh rate is omitted, niri will pick the highest refresh rate for the resolution.
    mode "3840x2160@60.000"

    // You can use an integer or fractional scale; for example, a ratio of 150%.
    scale 2

    // transform allows rotating the display counter-clockwise; the valid values are:
    // normal, 90, 180, 270, flipped, flipped-90, flipped-180 and flipped-270.
    transform "normal"

    // The output's position in the coordinate space of all monitors. Monitors without an explicit position are placed to the right of all placed monitors.
    // position x=1280 y=0
}

// If eDP-2 is not connected, this monitor will be focused by default
output "eDP-2" {
    // off
    focus-at-startup
    mode "2560x1600@300.000"
    transform "normal"
    position x=0 y=0
}

// You can use wev to look up the XKB name corresponding to a particular key
binds {
    Alt+Tab { spawn "niri-switch"; }
    // Mod-Shift-/ shows the list of important hotkeys (usually the same as Mod-?).
    Mod+Shift+Slash { show-hotkey-overlay; }
    Mod+D hotkey-overlay-title="Open the File Manager" { spawn "/usr/bin/dolphin"; }
    // Mod+L hotkey-overlay-title="Lock the Screen: swaylock" { spawn "/usr/bin/swaylock" "-f" "-i" "$HOME/.dotfiles/sway/.config/sway/lock.png"; }
    Mod+L hotkey-overlay-title="Lock the Screen: hyprlock" { spawn "/usr/bin/hyprlock"; }
    Mod+Return hotkey-overlay-title="Open a Terminal" { spawn "/usr/bin/kitty"; }
   // Mod+A hotkey-overlay-title="Run an Application" { spawn "/usr/bin/fuzzel"; }
    Mod+A hotkey-overlay-title="Run an Application" { spawn "/usr/bin/vicinae" "toggle"; }
    Mod+X hotkey-overlay-title="Open a browser: zen" { spawn "/usr/bin/google-chrome-stable"; }
    // Mod+D hotkey-overlay-title="Toggle obs and mpv together" { spawn "/usr/bin/touch" "/tmp/obs_mpv_toggle_pause"; }
    Mod+K hotkey-overlay-title="打开screenkey" { spawn "/usr/bin/wshowkeys" "-a" "right" "-a" "bottom" "-F" "ComicShannsMono Nerd Font 30"; }
    Mod+Shift+K hotkey-overlay-title="关闭screenkey" { spawn "/usr/bin/killall" "wshowkeys"; }
    // Mod+Shift+C hotkey-overlay-title="Restart waybar" { spawn-sh "pkill waybar && waybar"; }

    // Volume control. allow-when-locked=true makes the keys work while the screen is locked too. The wpctl here ships with the wireplumber package
    XF86AudioRaiseVolume allow-when-locked=true { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.1+"; }
    XF86AudioLowerVolume allow-when-locked=true { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.1-"; }
    XF86AudioMute        allow-when-locked=true { spawn-sh "wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle"; }
    XF86AudioMicMute     allow-when-locked=true { spawn-sh "wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle"; }

    // Brightness control. brightnessctl is a separate package
    XF86MonBrightnessUp allow-when-locked=true { spawn "brightnessctl" "set" "+10%"; }
    XF86MonBrightnessDown allow-when-locked=true { spawn "brightnessctl" "set" "10%-"; }

    // Toggle overview
    Mod+Tab repeat=false { toggle-overview; }
    // Close window
    Mod+Q repeat=false { close-window; }

    // Window focus switching, position movement
    Mod+Left  { focus-column-left; }
    Mod+Down  { focus-window-down; }
    Mod+Up    { focus-window-up; }
    Mod+Right { focus-column-right; }
    Mod+N     { focus-column-left; }
    Mod+i     { focus-column-right; }
    Mod+Alt+Left { consume-or-expel-window-left; }
    Mod+Alt+Right {consume-or-expel-window-right; }
    // Mod+N     { spawn-sh "niri msg action focus-column-left && niri msg action center-column"; }
    // Mod+i     { spawn-sh "niri msg action focus-column-right && niri msg action center-column"; }
    Mod+Shift+Left  { move-column-left; }
    Mod+Shift+Down  { move-window-down; }
    Mod+Shift+Up    { move-window-up; }
    Mod+Shift+Right { move-column-right; }
    Mod+Shift+N     { move-column-left; }
    Mod+Shift+I     { move-column-right; }

    Mod+Home { focus-column-first; }
    Mod+End  { focus-column-last; }
    Mod+Shift+Home { move-column-to-first; }
    Mod+Shift+End  { move-column-to-last; }

    // workspace focus switching, moving windows between workspaces
    Mod+Page_Down      { focus-workspace-down; }
    Mod+Page_Up        { focus-workspace-up; }
    Mod+Ctrl+Page_Down { move-column-to-workspace-down; }
    Mod+Ctrl+Page_Up   { move-column-to-workspace-up; }
    // Move the whole workspace up/down
    Mod+Shift+Page_Down { move-workspace-down; }
    Mod+Shift+Page_Up   { move-workspace-up; }

    // Focus switching and position movement shared by windows and workspaces in the up/down directions
    Mod+E     { focus-window-or-workspace-down; }
    Mod+U     { focus-window-or-workspace-up; }
    Mod+Shift+E     { move-window-down-or-to-workspace-down; }
    Mod+Shift+U     { move-window-up-or-to-workspace-up; }

    // Monitor focus switching
    Mod+Ctrl+Left  { focus-monitor-left; }
    Mod+Ctrl+Down  { focus-monitor-down; }
    Mod+Ctrl+Up    { focus-monitor-up; }
    Mod+Ctrl+Right { focus-monitor-right; }
    Mod+Ctrl+N     { focus-monitor-left; }
    Mod+Ctrl+E     { focus-monitor-down; }
    Mod+Ctrl+U     { focus-monitor-up; }
    Mod+Ctrl+I     { focus-monitor-right; }

    // Move windows across monitors
    Mod+Shift+Ctrl+Left  { move-column-to-monitor-left; }
    Mod+Shift+Ctrl+Down  { move-column-to-monitor-down; }
    Mod+Shift+Ctrl+Up    { move-column-to-monitor-up; }
    Mod+Shift+Ctrl+Right { move-column-to-monitor-right; }
    Mod+Shift+Ctrl+N     { move-column-to-monitor-left; }
    Mod+Shift+Ctrl+E     { move-column-to-monitor-down; }
    Mod+Shift+Ctrl+U     { move-column-to-monitor-up; }
    Mod+Shift+Ctrl+I     { move-column-to-monitor-right; }

    // Mouse-related keybinds
    Mod+WheelScrollDown      cooldown-ms=150 { focus-workspace-down; }
    Mod+WheelScrollUp        cooldown-ms=150 { focus-workspace-up; }
    Mod+Ctrl+WheelScrollDown cooldown-ms=150 { move-column-to-workspace-down; }
    Mod+Ctrl+WheelScrollUp   cooldown-ms=150 { move-column-to-workspace-up; }

    Mod+WheelScrollRight      { focus-column-right; }
    Mod+WheelScrollLeft       { focus-column-left; }
    Mod+Ctrl+WheelScrollRight { move-column-right; }
    Mod+Ctrl+WheelScrollLeft  { move-column-left; }

    Mod+Shift+WheelScrollDown      { focus-column-right; }
    Mod+Shift+WheelScrollUp        { focus-column-left; }
    Mod+Ctrl+Shift+WheelScrollDown { move-column-right; }
    Mod+Ctrl+Shift+WheelScrollUp   { move-column-left; }

    Mod+TouchpadScrollDown { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.02+"; }
    Mod+TouchpadScrollUp   { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.02-"; }

    Mod+1 { focus-workspace 1; }
    Mod+2 { focus-workspace 2; }
    Mod+3 { focus-workspace 3; }
    Mod+4 { focus-workspace 4; }
    Mod+5 { focus-workspace 5; }
    Mod+6 { focus-workspace 6; }
    Mod+7 { focus-workspace 7; }
    Mod+8 { focus-workspace 8; }
    Mod+9 { focus-workspace 9; }
    Mod+0 { focus-workspace 10; }
    Mod+Shift+1 { move-column-to-workspace 1; }
    Mod+Shift+2 { move-column-to-workspace 2; }
    Mod+Shift+3 { move-column-to-workspace 3; }
    Mod+Shift+4 { move-column-to-workspace 4; }
    Mod+Shift+5 { move-column-to-workspace 5; }
    Mod+Shift+6 { move-column-to-workspace 6; }
    Mod+Shift+7 { move-column-to-workspace 7; }
    Mod+Shift+8 { move-column-to-workspace 8; }
    Mod+Shift+9 { move-column-to-workspace 9; }
    Mod+Shift+0 { move-column-to-workspace 10; }

    // Maximize and fullscreen
    Mod+W { toggle-windowed-fullscreen; }
    Mod+F { expand-column-to-available-width; }
    Mod+Shift+F { fullscreen-window; }

    // Center non-maximized windows
    Mod+C { center-column; }
    Mod+Ctrl+C { center-visible-columns; }

    // Switch between the preset widths and heights in the layout
    Mod+R { switch-preset-column-width; }
    Mod+Shift+R { switch-preset-window-height; }
    // Width units can be pixels or percentages
    Mod+Minus { set-column-width "-10%"; }
    Mod+Equal { set-column-width "+10%"; }

    // Change height
    Mod+Shift+Minus { set-window-height "-10%"; }
    Mod+Shift+Equal { set-window-height "+10%"; }

    // Toggle floating windows; switch focus between tiled and floating windows
    Mod+Shift+Space       { toggle-window-floating; }
    Mod+Space { switch-focus-between-floating-and-tiling; }

    // Screenshot
    Alt+J { spawn-sh "grim -g \"$(slurp)\" - | satty --filename - --output-filename ~/$(date '+%Y%m%d-%H:%M:%S').png"; }
    Alt+Shift+J { spawn "flameshot" "gui"; }
    Print { screenshot show-pointer=false; }
    Ctrl+Print { screenshot-screen write-to-disk=true; }
    Alt+Print { screenshot-window write-to-disk=true; }

    // Virtual machine software may need keyboard control. allow-inhibiting=false ignores the current keybind itself
    Mod+Escape allow-inhibiting=false { toggle-keyboard-shortcuts-inhibit; }

    // Quitting niri shows a confirmation dialog to avoid quitting by accident.
    Mod+Shift+Q { quit; }

    // Turn off the monitor. Move the mouse or press any key to restore it
    Mod+Shift+P { power-off-monitors; }
}

// Settings that affect the position and size of windows.
layout {
    // Set the gaps around windows in logical pixels.
    gaps 10
    background-color "transparent"
    // When several windows exist, non-maximized windows are not auto-centered, which makes split-screen easier
    center-focused-column "never"
    // Auto-center when there is only one window
    always-center-single-column

    // The width switched between presets by mod+r.
    preset-column-widths {
        proportion 0.5
        proportion 0.2444
        proportion 0.7556
        // fixed sets an exact width in logical pixels. (affected by scale)
        // fixed 1920
    }

    preset-window-heights {
        proportion 0.5
        proportion 0.8
        proportion 1.0
    }
    // Turn off the focus ring
    focus-ring {
        // off
    }

    // Turn off the border
    border {
        off
    }
}

// Override the environment variables of processes started by niri
environment {
    QT_QPA_PLATFORMTHEME "qt5ct"
    ALL_PROXY "http://127.0.0.1:7890"
    LANG "zh_CN.UTF-8"
    LC_CTYPE "zh_CN.UTF-8"
    LC_NUMERIC "zh_CN.UTF-8"
    LC_TIME "zh_CN.UTF-8"
    LC_COLLATE "zh_CN.UTF-8"
    LC_MONETARY "zh_CN.UTF-8"
    LC_MESSAGES "zh_CN.UTF-8"
    LC_PAPER "zh_CN.UTF-8"
    LC_NAME "zh_CN.UTF-8"
    LC_ADDRESS "zh_CN.UTF-8"
    LC_TELEPHONE "zh_CN.UTF-8"
    LC_MEASUREMENT "zh_CN.UTF-8"
    LC_IDENTIFICATION "zh_CN.UTF-8"
    LC_ALL null
    // XDG_DATA_DIRS "$HOME/.local/share" "$XDG_DATA_DIRS"
    // GTK_IM_MODULE "fcitx"
    QT_IM_MODULE "fcitx"
    // https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#Sway
    XMODIFIERS "@im=fcitx"
    QT_IM_MODULES "wayland;fcitx"
    GTK_IM_MODULE null
    SDL_IM_MODULE null
    GLFW_IM_MODULE null
}
spawn-at-startup "niri-switch-daemon"

// Software started automatically when niri starts
spawn-at-startup "/usr/bin/fcitx5"
// spawn-at-startup "/usr/bin/v2rayn"
// spawn-at-startup "/usr/bin/waybar"
spawn-at-startup "/usr/bin/vicinae" "server"
spawn-at-startup "~/.cargo/bin/soteria"
spawn-at-startup "~/Desktop/tools/update_repositories.sh"
spawn-at-startup "qs" "-c" "noctalia-shell"
spawn-at-startup "/usr/bin/hyprlock"
// To run shell commands (with variables, pipes, etc.), use spawn-sh-at-at-startup:
spawn-sh-at-startup "swaybg -i /home/Qaaxaap/Pictures/wallpaper.png -m fill"

hotkey-overlay {
    // Skip the "Important Hotkeys" popup.
    skip-at-startup
}

// Set the path where screenshots are saved; null disables saving to disk
screenshot-path "~/Pictures/ScreenShot/%Y-%m-%d %H-%M-%S.png"

// Ignore the software's own decorations (e.g. title bars)
prefer-no-csd

// Specify the cursor theme and size; hide the cursor while typing
cursor {
    // xcursor-theme "Dracula-cursors"
    xcursor-theme "breeze"
    xcursor-size 24
    hide-when-typing
}

// Use `niri msg windows` to view the Title, App ID and other information
window-rule {
    open-on-output "eDP-2"
    // default-window-height { proportion 0.9; }
    // default-floating-position x=100 y=200 relative-to="bottom-left"
    // default-column-width { proportion 0.7556; }
    geometry-corner-radius 20
    clip-to-geometry true
    border {
        // off
        on
        width 4
        active-gradient from="#bd93f9" to="#94b9fa" angle=135
        inactive-color "#505050"
        urgent-color "#9b0000"
        // active-gradient from="#80c8ff" to="#bbddff" angle=45
        // inactive-gradient from="#505050" to="#808080" angle=45 relative-to="workspace-view"
        // urgent-gradient from="#800" to="#a33" angle=45
    }

    focus-ring{
        off
    }
    // opacity 0.75
}
window-rule {
    open-on-output "eDP-2"
    match app-id="scrcpy"
    default-column-width { proportion 0.2444; }
}
window-rule {
    open-on-output "eDP-2"
    match app-id=r#"chrome"#
    default-column-width { proportion 0.8; }
    // border {
    //    on
    //    width 4
    //    active-color "#61AFEF"
    // }
    // open-focused false
}
window-rule {
    match app-id="com.gabm.satty" title="satty"
    border {
        on
        width 2
        active-color "#61AFEF"
    }
}

// `niri msg layers` shows namespaces, so you can configure waybar transparency here
layer-rule {
    match namespace="^quickshell-overview$"
    place-within-backdrop true
    //     opacity 0.75
}

// Put swaybg inside the overview backdrop.
layer-rule {
    match namespace="^wallpaper$"
    place-within-backdrop true
}

debug {
    honor-xdg-activation-with-invalid-serial
}
// Disable the hot corner at the top-left of the mouse
gestures {
    hot-corners {
        // off
    }
}

animations {
    // Uncomment to turn off all animations.
    // You can also put "off" into each individual animation to disable it.
    // off

    // Slow down all animations by this factor. Values below 1 speed them up instead.
    // slowdown 3.0

    // Individual animations.

    workspace-switch {
        spring damping-ratio=1.0 stiffness=1000 epsilon=0.0001
    }

    window-open {
        duration-ms 150
        curve "ease-out-expo"
    }

    window-close {
        duration-ms 150
        curve "ease-out-quad"
    }

    horizontal-view-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    window-movement {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    window-resize {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }

    config-notification-open-close {
        spring damping-ratio=0.6 stiffness=1000 epsilon=0.001
    }

    exit-confirmation-open-close {
        spring damping-ratio=0.6 stiffness=500 epsilon=0.01
    }

    screenshot-ui-open {
        duration-ms 200
        curve "ease-out-quad"
    }

    overview-open-close {
        spring damping-ratio=1.0 stiffness=800 epsilon=0.0001
    }
}

```

There are a lot of keybindings; go look at them yourself. I'll add more when I get around to it.

Here we use Hyprlock for locking the screen

```
~/.config/hypr
├── hyprlock.conf
└── mocha
    └── mocha.conf
```

Build a directory structure like this

Then back up your original `~/.config/hypr/hyprlock.conf`, and fill it in with the config below

```bash
source = $HOME/.config/hypr/mocha/mocha.conf

$accent = $mauve
$accentAlpha = $mauveAlpha
$font = JetBrains Mono

# GENERAL
general {
  hide_cursor = true
}

# BACKGROUND
background {
  monitor =
  path = /path/to/your/lock/screen/wallpaper.png
  blur_passes = 2
  color = $base
}

# LAYOUT
label {
  monitor =
  text = 键盘布局: $LAYOUT
  color = $text
  font_size = 25
  font_family = $font
  position = 30, -30
  halign = left
  valign = top
}

# TIME
label {
  monitor =
  text = $TIME
  color = $text
  font_size = 90
  font_family = $font
  position = -30, 0
  halign = right
  valign = top
}

# DATE
label {
  monitor =
  text = cmd[update:43200000] date +"%Y年 %m月 %d日, %A"
  color = $text
  font_size = 25
  font_family = $font
  position = -30, -150
  halign = right
  valign = top
}

# FINGERPRINT
{
  monitor = "";
  text = "$FPRINTPROMPT";
  color = "$text";
  font_size = 14;
  font_family = $font;
  position = "0, -107";
  halign = "center";
  valign = "center";
}

# USER AVATAR
image {
  monitor =
  path = $HOME/.face
  # size = 100
  size = 200
  border_color = $accent
  position = 0, 90
  # position = 0, 75
  halign = center
  valign = center
}

# INPUT FIELD
input-field {
  monitor =
  size = 300, 60
  outline_thickness = 4
  dots_size = 0.2
  dots_spacing = 0.2
  dots_center = true
  outer_color = $accent
  inner_color = $surface0
  font_color = $text
  fade_on_empty = false
  # placeholder_text = <span foreground="##$textAlpha"><i>󰌾 Logged in as </i><span foreground="##$accentAlpha">$USER</span></span>
  placeholder_text = <span foreground="##$textAlpha">󰌾 <span foreground="##$accentAlpha">$USER</span> 已登录 </span>
  hide_input = false
  check_color = $accent
  fail_color = $red
  fail_text = <i> 已失败 <b>($ATTEMPTS)</b> 次 </i>
  capslock_color = $yellow
  # position = 0, -47
  position = 0, -80
  halign = center
  valign = center
```

Also copy your avatar over to `~/.face`. (Note: that's creating a `.face` file, not putting your avatar image inside a `.face` folder!)
Do the same for `~/.config/hypr/mocha/mocha.conf` and fill in the following config
```bash
$rosewater = rgb(f5e0dc)
$rosewaterAlpha = f5e0dc

$flamingo = rgb(f2cdcd)
$flamingoAlpha = f2cdcd

$pink = rgb(f5c2e7)
$pinkAlpha = f5c2e7

$mauve = rgb(cba6f7)
$mauveAlpha = cba6f7

$red = rgb(f38ba8)
$redAlpha = f38ba8

$maroon = rgb(eba0ac)
$maroonAlpha = eba0ac

$peach = rgb(fab387)
$peachAlpha = fab387

$yellow = rgb(f9e2af)
$yellowAlpha = f9e2af

$green = rgb(a6e3a1)
$greenAlpha = a6e3a1

$teal = rgb(94e2d5)
$tealAlpha = 94e2d5

$sky = rgb(89dceb)
$skyAlpha = 89dceb

$sapphire = rgb(74c7ec)
$sapphireAlpha = 74c7ec

$blue = rgb(89b4fa)
$blueAlpha = 89b4fa

$lavender = rgb(b4befe)
$lavenderAlpha = b4befe

$text = rgb(cdd6f4)
#$text = rgb(6c8cf5)
#$textAlpha = 6c8cf5
$textAlpha = cdd6f4

$subtext1 = rgb(bac2de)
$subtext1Alpha = bac2de

$subtext0 = rgb(a6adc8)
$subtext0Alpha = a6adc8

$overlay2 = rgb(9399b2)
$overlay2Alpha = 9399b2

$overlay1 = rgb(7f849c)
$overlay1Alpha = 7f849c

$overlay0 = rgb(6c7086)
$overlay0Alpha = 6c7086

$surface2 = rgb(585b70)
$surface2Alpha = 585b70

$surface1 = rgb(45475a)
$surface1Alpha = 45475a

$surface0 = rgb(313244)
$surface0Alpha = 313244

$base = rgb(1e1e2e)
$baseAlpha = 1e1e2e

$mantle = rgb(181825)
$mantleAlpha = 181825

$crust = rgb(11111b)
$crustAlpha = 11111b
```
You can change the colors yourself

If you try it you'll notice you have to type your password twice when logging in, which is no good, so we set up autologin — that way only one password.

Edit `/etc/sddm.conf.d/autologin.conf`
```
[Autologin]
User=#your_username
Session=niri
```
Alright, you can reboot into Niri now — go explore it yourself
