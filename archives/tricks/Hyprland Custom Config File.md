---
tags:
  - type/permanent
  - status/evergreen
  - topic/learning
  - attr/technique
---

## Definition

The file is in that path `~/.config/hypr/conf/custom.conf`

``` conf
# Add your additional Hyprland configurations here



$mainMod = SUPER
$HYPRSCRIPTS = ~/.config/hypr/scripts
$SCRIPTS = ~/.config/ml4w/scripts

exec-once = fcitx5
exec-once = clash-verge
exec-once = $HYPRSCRIPTS/notifications.sh
exec-once = /usr/lib/pam_kwallet_init

# workspace

unbind = $mainMod CTRL, right
unbind = $mainMod CTRL, left
unbind = $mainMod TAB
unbind = $mainMod CTRL, down

bind = $mainMod CTRL, right, workspace, r+1
bind = $mainMod CTRL, left, workspace, r-1
bind = $mainMod CTRL, down, workspace, 2
bind = $mainMod CTRL, up, workspace, 1
bind = $mainMod CTRL ALT, right, movetoworkspace, r+1
bind = $mainMod CTRL ALT, left, movetoworkspace, r-1

unbind = $mainMod, F
unbind = $mainMod CTRL, F
bind = $mainMod CTRL, F, togglefloating
bind = $mainMod, F, fullscreen, 0

unbind = $mainMod, S
unbind = $mainMod CTRL ALT, S
bind = $mainMod, S, togglespecialworkspace
bind = $mainMod CTRL ALT, S, movetoworkspace, special




# Screen Shot

unbind = $mainMod, P
unbind = $mainMod ALT, P
unbind = $mainMod SHIFT, P

bind = $mainMod, p, exec, $HYPRSCRIPTS/screenshot.sh 
bind = $mainMod ALT, P, exec, $HYPRSCRIPTS/screenshot.sh --instant # Take an instant full-screen screenshot
bind = $mainMod SHIFT, P, exec, $HYPRSCRIPTS/screenshot.sh --instant-area # Take an instant area screenshot


# hot fix
xwayland {
    force_zero_scaling = true
}
dwindle {
    pseudotile = true
    preserve_split = true
}
# Default software

unbind = $mainMod, B

bind = $mainMod, B, exec, google-chrome-stable
bind = $mainMod, C, exec, code

unbind = $mainMod, R
bind = $mainMod, R, exec, rofi -show run

unbind = $mainMod, O
bind = $mainMod, O, exec, obsidian
```



---
## **Related**