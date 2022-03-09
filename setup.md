# Try 1: Modifying regolith files (status: Help Needed)
## Install sway
```bash
sudo apt install sway brightnessctl network-manager-gnome pulseaudio-utils playerctl xdg-desktop-portal swayidle swaylock redshift policykit-1-gnome kanshi gnome-keyring gnome-session-bin gnome-settings-daemon-common
```
## Add DM entry
This is needed for regolith-sway to be accessible from the Display Manager (login manager).
- Copy regolith.desktop from /usr/share/xsessions/regolith.desktop to /usr/share/wayland-sessions/regolith-sway.desktop
- open /usr/share/wayland-sessions/regolith-sway.desktop 
- Change the Name field to `Regolith-Sway` to differential in DM while logging in.
- Change `TryExec` from `i3` to `sway`
- Change `Exec` from `regolith` to `regolith-sway-session` (We will create an executable by this name later on).
- Change `DesktopNames` to `Regolith-Sway;GNOME-Flashback;GNOME`

## Add Gnome session file for regolith
- cd into /usr/share/gnome-session/sessions/
- copy regolith.session file as regolith-sway.session
- Open the regolith-sway.session file.
- change `Name` to `Regolith-Sway` (Needed to Launch the required componeents with Regolith-Sway as a session).
- change `RequiredComponents` as follows to add regolith-sway at the end:
```
RequiredComponents=org.gnome.SettingsDaemon.A11ySettings;org.gnome.SettingsDaemon.Color;org.gnome.SettingsDaemon.Datetime;org.gnome.SettingsDaemon.Housekeeping;org.gnome.SettingsDaemon.Keyboard;org.gnome.SettingsDaemon.MediaKeys;org.gnome.SettingsDaemon.Power;org.gnome.SettingsDaemon.PrintNotifications;org.gnome.SettingsDaemon.Rfkill;org.gnome.SettingsDaemon.ScreensaverProxy;org.gnome.SettingsDaemon.Sharing;org.gnome.SettingsDaemon.Smartcard;org.gnome.SettingsDaemon.Sound;org.gnome.SettingsDaemon.Wacom;org.gnome.SettingsDaemon.XSettings;gnome-flashback;regolith-sway;
```

## Create regolith-sway-session script which is run by the display manager
- cd into /usr/bin/
- Create a copy of `regolith-session` as `regolith-sway-session` in the same directory and open it in an editor.
- Set `XDG_CURRENT_DESKTOP` to `Regolith-Sway` (try 1) or `sway` (try 2).
- Change all instances of `--session=regolith` to `--session=regolith-sway` as shown below.
```bash
if [ $COMPACT_VERSION -gt 3320 ]; then
    exec gnome-session --builtin --session=regolith-sway --disable-acceleration-check "$@"
else
    exec gnome-session --session=regolith-sway --disable-acceleration-check "$@"
fi
```

# Create regolith-session-init file 
This file is responsible for setting Xresources(will try to keep those if possible for themeing) and registering and closing the sway session with gnome and actually launching sway. 

- `touch /usr/bin/regolith-sway-session-init`
- open the above file and put the following in it (this is everything regolith-session-init provides minus xresources conig). **Note**: You can also add your environment here.
```bash
# Register with gnome-session so that it does not kill the whole session thinking it is dead.
# See https://zork.net/~st/jottings/gnome-i3.html for details
if [ -n "$DESKTOP_AUTOSTART_ID" ]; then
    dbus-send --print-reply --session --dest=org.gnome.SessionManager "/org/gnome/SessionManager" org.gnome.SessionManager.RegisterClient "string:Regolith-Sway" "string:$DESKTOP_AUTOSTART_ID"
fi

# Launch swaywm with the Regolith configuration
echo "Regolith is launching i3-gaps with $REGOLITH_I3_CONFIG_FILE"
# i3 -c $REGOLITH_I3_CONFIG_FILE
sway

# Close session when sway exits.
if [ -n "$DESKTOP_AUTOSTART_ID" ]; then
    dbus-send --print-reply --session --dest=org.gnome.SessionManager "/org/gnome/SessionManager" org.gnome.SessionManager.Logout "uint32:1"
fi
```

## Create desktop file for regolith-sway
- cd into /usr/share/application/ directory.
- Create a copy of `regolith.desktop` file as `regolith-sway.dektop` and open it in a editor.
- Change `Name` to `regolith-sway`.
- Change `Exec` to `regolith-sway-session-init` (Launches regolith-sway-session-init executable, which registers the current session with gnome).
- Change `X-GNOME-WMName` to `Regolith-Sway` (To load the required componenets defined in the /usr/share/gnome-session/sessions/regolith-sway.session file).

## Try logging into sway
PS: Didn't work

References/Credits: 
- [sway-gnome](https://github.com/alexpearce/sway-gnome) by alexpearce.
- Regolith

