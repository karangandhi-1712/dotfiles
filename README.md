# dotfiles — karangandhi-1712

Bare-repo dotfiles for an **Ubuntu 24.04 / GNOME 46** machine dressed up with a
macintosh-flavored desktop: WhiteSur theme, SF Pro fonts, `bigsur` sound theme.

## Manage these dotfiles

The `cfg` alias is defined in `.bashrc` by the tracked file itself:

```
cfg status
cfg add <path>          # add a file/dir explicitly
cfg commit -m "…"
cfg push
```

### Fresh machine bootstrap

```
git clone https://github.com/karangandhi-1712/dotfiles.git ~/.dotfiles
echo 'alias cfg="git --git-dir=\"$HOME/.dotfiles\" --work-tree=\"$HOME\""' >> ~/.bashrc
source ~/.bashrc
cfg config status.showUntrackedFiles no
cfg checkout -f main        # materializes every tracked file into $HOME
```

## What's inside

| Area | Path | Notes |
|---|---|---|
| Shell | `.bashrc`, `.profile`, `.gitconfig` | ROS 2 Jazzy + `colcon_cd`, nvm, discrete-GPU sim aliases |
| GTK 3/4 | `.config/gtk-3.0`, `.config/gtk-4.0` | `settings.ini` + WhiteSur **libadwaita** `gtk-*.css` (assets + windows-assets included) |
| GNOME settings | `.config/gnome/dconf.dconf` | Text export of the whole dconf tree: fonts (`SF Pro Display 11`), `WhiteSur-Dark`, `prefer-dark`, sound theme `bigsur`, shell/extension prefs |
| Theme | `.themes/WhiteSur-Dark/` | Only the active variant; see below for the full installer |
| Fonts | `.fonts`, `.local/share/fonts` | SF Pro Display + Text (`.otf`, Apple license included), AvantGarde, GE Inspira, DejaVu ExtraLight |
| Wallpapers | `.local/share/backgrounds/` | 4 wallpapers (26 MB) |
| Sounds | `.local/share/sounds/bigsur/` | The GNOME sound theme currently in use (`bigsur`) |
| Flatpak | `.local/share/flatpak/overrides/` | User overrides: `global` + per-app (`app.zen_browser.zen`) |
| ROS 2 | `.config/ros.org`, `.rviz2/` | Machine config + saved RViz2 UI state (SDFormat logs excluded) |
| Dev tools | `kitty`, `htop`, `fontconfig`, `.config/Code/User/settings.json` | VS Code user settings; snippets are empty |
| Session | `autostart/`, `plank/`, `tiling-assistant/layouts.json`, `systemd/user/openclaw-gateway.service`, `background` | Dock items, tiling layouts, one user unit |

## Restore order on a fresh Ubuntu

1. **Base GNOME bits** — `sudo apt install dmz-cursor-theme sassc gtk2-engines-murrine optipng inkscape`
2. **WhiteSur theme** (re-generates `~/.themes/*`, restores the gtk-4.0 libadwaita css):
   ```
   git clone --depth=1 https://github.com/vinceliuice/WhiteSur-gtk-theme.git
   cd WhiteSur-gtk-theme && ./install.sh -c dark && ./libadwaita-tweaks.sh -c dark
   ```
   The tracked `~/.config/gtk-4.0/` already carries the css overrides, so this
   step is optional — but it also fixes the `gtk.gresource` symlink into
   `/usr/share/themes`.
3. **dconf** — `dconf load / < ~/.config/gnome/dconf.dconf` (fonts, theme,
   color scheme, sounds, extensions) then log out/in once.
4. **Fonts** — `fc-cache -fv`
5. **Flatpak** — `flatpak install -y flathub app.zen_browser.zen`; user
   overrides arrive with the checkout. For the **system-level** override:
   ```
   sudo tee /var/lib/flatpak/overrides/global <<'EOF'
   [Context]
   filesystems=xdg-config/gtk-3.0;xdg-config/gtk-4.0;
   EOF
   ```
6. **ROS 2** — install [Jazzy](https://docs.ros.org/en/jazzy/) + `colcon`
   (`apt install ros-jazzy-desktop ros-dev-tools`). The `.bashrc` linearly
   sources `/opt/ros/jazzy/setup.bash`, `colcon_cd`, and a **project workspace**
   (`~/VectorInternship/vectorbot_in_warehouse/sim/install/setup.bash`) — delete
   that last line on machines without the workspace.
7. **VS Code / other** — tracked settings land via the checkout; install the
   editor as usual.
8. **User systemd unit** (OpenClaw gateway) — comes with checkout; then
   `systemctl --user daemon-reload` if you keep it.

## Deliberately NOT tracked

- `~/mac-fonts/` (751 MB full macOS font pack) and `~/macos-sounds/`
  (139 MB) — only the ~15 MB of installed fonts and the active `bigsur`
  sound theme are committed.
- `~/.var/` — Flatpak **app data** (Zen browser profile: history, secrets).
- Secrets/state: `.ssh`, `.gnupg`, `.config/gh`, remmina/transmission,
  bash history, agent dirs (`.claude`, `.gemini`, …), `.dotfiles` itself.
- VS Code bulk: `globalStorage/`, `workspaceStorage/`, `History/`, `sync/`.
- `~/.ros/log`, `~/.sdformat`, caches (`.cache*`, `.nvm`, `.cargo`, …).

A belt-and-suspenders exclude list lives in `~/.dotfiles/info/exclude` so
nothing above can slip in even if `status.showUntrackedFiles` is ever reset.
