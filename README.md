# Jellyfin Desktop

> [!WARNING]
> This client is still under active development and may have bugs or missing features.

A [Jellyfin](https://jellyfin.org) desktop client built on [CEF](https://bitbucket.org/chromiumembedded/cef) and [mpv](https://mpv.io/). A complete rewrite of the previous [Qt-based client](https://github.com/jellyfin-archive/jellyfin-desktop-qt/).

## Downloads
### Linux
**Note:** Wayland only for now; see https://github.com/jellyfin/jellyfin-desktop/issues/140

- [AppImage](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-linux-appimage/main/linux-appimage-x86_64.zip)
- Arch Linux (AUR): [jellyfin-desktop-git](https://aur.archlinux.org/packages/jellyfin-desktop-git)
- [Flatpak (non-Flathub bundle)](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-linux-flatpak/main/linux-flatpak.zip)

### macOS
- [Apple Silicon](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-macos/main/macos-arm64.zip)
- [Intel](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-macos/main/macos-x86_64.zip)

After installing, remove quarantine: 
```
sudo xattr -cr /Applications/Jellyfin\ Desktop.app
```

### Windows
- [x64](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-windows/main/windows-x64.zip)
- [arm64](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-windows/main/windows-arm64.zip)


## Building

See [dev/](dev/README.md) for build instructions.

## Fork changes

- **Audio/subtitle track desync fix**: When playing the next episode, Jellyfin selects audio/subtitle tracks from server preferences, but the desktop client was ignoring them and playing defaults. Tracks are now applied after `FILE_LOADED` fires (post-`onPlaying`) instead of being passed atomically in `loadfile`, so late `setAudioStreamIndex`/`setSubtitleStreamIndex` calls from the playback manager are not overridden by the load command.
- **Silent audio mute fix**: `DefaultAudioStreamIndex` unset by the server caused `aid=0` (`TRACK_DISABLE`) in `loadfile`, silently muting playback. Default is now `TRACK_AUTO` (`aid=-1`).
- **Aspect ratio menu fix**: The "Aspect Ratio" option under the gear icon was non-functional (empty `getSupportedAspectRatios()` → no dialog). Now returns Normal/Fill/Cover options backed by mpv's `keepaspect`/`panscan` properties.
- **Wayland "Window is not responding" false positive fix** (Hyprland and other compositors): The input thread reads Wayland events on behalf of all queues — including `xdg_wm_base` ping events destined for mpv's default queue — but mpv's VO thread may be sleeping between frames and unaware pending events exist. The input thread now pokes mpv's VO wakeup pipe after each `wl_display_read_events()` call, ensuring mpv dispatches its queue (and sends the pong) promptly rather than waiting until the next frame or timeout.
