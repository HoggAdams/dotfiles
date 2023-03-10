# Picom

[picom](https://github.com/yshui/picom) is a standalone [compositor](https://wiki.archlinux.org/title/Xorg#Composite) for [Xorg](https://wiki.archlinux.org/title/Xorg), suitable for use with [window managers](https://wiki.archlinux.org/title/Window_managers) that do not provide compositing. picom is a fork of [compton](https://github.com/chjj/compton/), which is a fork of [xcompmgr-dana](https://web.archive.org/web/20150429182855/http://oliwer.net/xcompmgr-dana/), which in turn is a fork of [xcompmgr](https://wiki.archlinux.org/title/Xcompmgr).



## Installation

[Install](https://wiki.archlinux.org/title/Install) the [picom](https://archlinux.org/packages/?name=picom) package or [picom-git](https://aur.archlinux.org/packages/picom-git/)AUR for the development version. 

## Configuration

The default configuration is available in `/etc/xdg/picom.conf`. For modifications, it can be copied to `~/.config/picom/picom.conf` or `~/.config/picom.conf`.

To use another custom configuration file with picom, use the following command:

```
$ picom --config path/to/picom.conf
```

See [picom(1) § CONFIGURATION FILES](https://man.archlinux.org/man/picom.1#CONFIGURATION_FILES) for details.

### Disable shadows for some windows

The `shadow-exclude` option can disable shadows for windows if required. For currently disabled windows, see [[1\]](https://github.com/yshui/picom/blob/b66e5fd422820460f030db853109e4d50bb3549d/picom.sample.conf#L46).

To disable shadows for menus add the following to `wintypes` in `picom.conf`:

```
# menu        = { shadow = false; };
dropdown_menu = { shadow = false; };
popup_menu    = { shadow = false; };
utility       = { shadow = false; };
```

The other `WINDOW_TYPE` values that can be used are defined in the EWMH standard: `unknown`, `desktop`, `dock`, `toolbar`, `menu`, `utility`, `splash`, `dialog`, `normal`, `dropdown_menu`, `popup_menu`, `tooltip`, `notification`, `combo`, and `dnd`.

### Opacity

To set opacity (in effect transparency) for focused and unfocused  windows (for example terminal emulators), add the following to your `picom.conf`:

```
opacity-rule = [
  "90:class_g = 'URxvt' && focused",
  "60:class_g = 'URxvt' && !focused"
];
```

See also [#Tabbed windows (shadows and transparency)](https://wiki.archlinux.org/title/Picom#Tabbed_windows_(shadows_and_transparency)).

## Usage

picom may be manually enabled or disabled at any time during a  session, or autostarted as a background process for sessions. There are  also several optional arguments that may be used to tweak the  compositing effects provided. These include:

- `-b`: Run as a background process for a session (e.g. when [autostarting](https://wiki.archlinux.org/title/Autostarting) for a window manager such as [Openbox](https://wiki.archlinux.org/title/Openbox))
- `-c`: Enable shadow effects
- `-C`: Disable shadow effects on panels and docks (deprecated, use [#Disable shadows for some windows](https://wiki.archlinux.org/title/Picom#Disable_shadows_for_some_windows))
- `-G`: Disable shadow effects for application windows and drag-and-drop objects (deprecated, use [#Disable shadows for some windows](https://wiki.archlinux.org/title/Picom#Disable_shadows_for_some_windows))
- `--config`: Use a specified configuration file

Many more options are available, including setting timings, displays  to be managed, the opacity of menus, window borders, and inactive  application menus. See [picom(1)](https://man.archlinux.org/man/picom.1).

**Note:** If a different [composite manager](https://wiki.archlinux.org/title/Composite_manager) is running, it should be disabled before starting *picom*.

To manually enable default compositing effects during a session, use the following command:

```
$ picom &
```

To autostart picom as a background process for a session, the `-b` argument can be used (may cause a display freeze):

```
$ picom -b
```

Here is an example where additional arguments that require values to be set have been used:

```
$ picom -cfF -o 0.38 -O 200 -I 200 -t 0 -l 0 -r 3 -D2 -m 0.88
```

## Multihead

If a [multihead](https://wiki.archlinux.org/title/Multihead) configuration is used without xinerama - meaning that X server is  started with more than one screen - then picom will start on only one  screen by default. It can be started on all screens by using the `DISPLAY` environment variable. For example, to run on X screen 0 in the background:

```
 DISPLAY=":0" picom -b
```

The above should work on all monitors, but if it does not try an older method that manually specifies each one:

```
seq 0 3 | xargs -l1 -I@ picom -b -d :0.@
```

## Grayscale

It is possible to convert windows to grayscale by use of [shaders](https://learnopengl.com/Getting-started/Shaders).

As per [picom(1)](https://man.archlinux.org/man/picom.1), start by editing the default shader from the [picom's sources](https://github.com/yshui/picom/blob/next/compton-default-fshader-win.glsl).

```
/path/to/shader/file.glsl
uniform float opacity;
uniform bool invert_color;
uniform sampler2D tex;

void main() {
	vec4 c = texture2D(tex, gl_TexCoord[0].xy);
	float g = 0.2126 * c.r + 0.7152 * c.g + 0.0722 * c.b;   // CIELAB luma, based on human tristimulus.
	c = vec4(vec3(g), c.a);
 	if (invert_color)
		c = vec4(vec3(c.a, c.a, c.a) - vec3(c), c.a);
	c *= opacity;
	gl_FragColor = c;
}
```

Then start picom by including the full text of the shader. The `glx` backend will also, probably, be necessary.

```
$ picom --backend glx --glx-fshader-win "$(cat /path/to/shader/file.glsl)"
```

## Troubleshooting

Recent versions of picom had some problem with DRI2 acceleration and exhibited severe flickering when DRI2 is in use ([picom bug](https://github.com/yshui/picom/issues/47), [mesa bug](https://bugs.freedesktop.org/show_bug.cgi?id=108651)).  This has been worked around and reported to be working, but may still  affect some users.  DRI3 is unaffected by this particular issue.

The use of compositing effects may on occasion cause issues such  as visual glitches when not configured correctly for use with other  applications and programs.

### Conky

To disable shadows around [Conky](https://wiki.archlinux.org/title/Conky) windows, have the following in `~/.conkyrc`:

```
own_window_class conky
```

In the case this solution fail with blur effect, you can try this in `~/.conkyrc`:

```
own_window_type= 'desktop'
```

### dwm and dmenu

dwm's statusbar is not detected by any of picom's functions to  automatically exclude window manager elements. Neither dwm statusbar nor dmenu have a static window id. If you want to exclude it from inactive  window transparency (or other), you will have to either patch a window  class into the source code of each, or exclude by less precise  attributes. The following example is with dwm's status on top, which  allows a resolution independent of location exclusion:

```
$ picom <any other arguments> --focus-exclude "x = 0 && y = 0 && override_redirect = true"
```

Otherwise, where using a configuration file:

```
focus-exclude = "x = 0 && y = 0 && override_redirect = true";
```

The override redirect property seems to be false for most windows-  having this in the exclusion rule prevents other windows drawn in the  upper left corner from being excluded (for example, when dwm statusbar  is hidden, x0 y0 will match whatever is in dwm's master stack).

### Firefox

See [#Disable shadows for some windows](https://wiki.archlinux.org/title/Picom#Disable_shadows_for_some_windows).

To disable shadows for Firefox elements add the following to shadow-exclude in `picom.conf`:

```
"class_g = 'firefox' && argb",
```

See [[2\]](https://github.com/chjj/compton/issues/201#issuecomment-45288510) for more information.

### slock

Where inactive window transparency has been enabled (the `-i` argument when running as a command), this may provide troublesome results when also using [slock](https://wiki.archlinux.org/title/Slock). One solution is to amend the transparency to `0.2`. For example, where running picom arguments as a command:

```
$ picom <any other arguments> -i 0.2
```

Otherwise, where using a configuration file:

```
inactive-dim = 0.2;
```

Alternatively, you may try to exclude slock by its window id, or by excluding all windows with no name.

**Note:** Some programs change their id for every new instance, but slock's  appears to be static. Someone more knowledgeable will have to confirm  that slock's id is in fact static- until then, use at your own risk.

Exclude all windows with no name from picom using the following options:

```
$ picom <other arguments> --focus-exclude "! name~=''"
```

Find your slock's window id by running the command:

```
$ xwininfo & slock
```

Quickly click anywhere on the screen (before slock exits), then type  your password to unlock. You should see the window id in the output:

```
xwininfo: Window id: 0x1800001 (has no name)
```

Take the window id and exclude it from picom with:

```
$ picom <any other arguments> --focus-exclude 'id = 0x1800001'
```

Otherwise, where using a configuration file:

```
focus-exclude = "id = 0x1800001";
```

### Flicker

Applies to fully maximized windows (in sessions without any panels) with the default `picom.conf` caused and resolved by the following option:

```
unredir-if-possible = false;
```

See [[3\]](https://github.com/chjj/compton/issues/402) for more information.

### Fullscreen tearing

If you observe screen tearing of video playback only in fullscreen, see [#Flicker](https://wiki.archlinux.org/title/Picom#Flicker).

### Lag when using xft fonts

If you experience heavy lag when using Xft fonts in applications such as [xterm](https://wiki.archlinux.org/title/Xterm) or [urxvt](https://wiki.archlinux.org/title/Urxvt) try:

```
--xrender-sync --xrender-sync-fence
```

or the xrender backend.

See [[4\]](https://github.com/chjj/compton/issues/152) for more information.

### Screen artifacts/screenshot issues when using AMD's Catalyst driver

Try running picom with:

```
--backend xrender
```

or add

```
backend = "xrender";
```

to your `picom.conf` file.

See [[5\]](https://github.com/chjj/compton/issues/208) for more information.

### Tabbed windows (shadows and transparency)

When windows with transparency are tabbed, the underlying tabbed  windows are still visible because of transparency. Each tabbed window  also draws its own shadow resulting in multiple shadows.

Removing the multiple shadows issue can be done by adding the following to the already existing [shadow-exclude list](https://github.com/yshui/picom/blob/248bffede73e520a4929dd7751667d29d4169d59/picom.sample.conf#L175-L181):

```
"_NET_WM_STATE@:32a *= '_NET_WM_STATE_HIDDEN'"
```

Not drawing underlying tabbed windows can be enabled by adding the following to your `picom.conf`:

```
opacity-rule = [
  "95:class_g = 'URxvt' && !_NET_WM_STATE@:32a",
  "0:_NET_WM_STATE@[0]:32a *= '_NET_WM_STATE_HIDDEN'",
  "0:_NET_WM_STATE@[1]:32a *= '_NET_WM_STATE_HIDDEN'",
  "0:_NET_WM_STATE@[2]:32a *= '_NET_WM_STATE_HIDDEN'",
  "0:_NET_WM_STATE@[3]:32a *= '_NET_WM_STATE_HIDDEN'",
  "0:_NET_WM_STATE@[4]:32a *= '_NET_WM_STATE_HIDDEN'"
];
```

Note that `URxvt` is the Xorg class name of your terminal. Change this if you use a different terminal. You can query a window's  class by running the command `xprop WM_CLASS` and clicking the window.

See [[6\]](https://www.reddit.com/r/unixporn/comments/330zxl/webmi3_no_more_overlaying_shadows_and_windows_in/) for more information.

**Warning:** With i3 and kitty as terminal, doing this will currently (as of  2020-08-31) freeze all hidden (tabbed) instances of kitty when you  reload i3: [[7\]](https://github.com/kovidgoyal/kitty/issues/1681) a possible fix is suggested for a similar issue at [Intel graphics#DRI3 issues](https://wiki.archlinux.org/title/Intel_graphics#DRI3_issues)

### Unable to change the background color with xsetroot

Currently, picom is incompatible with `xsetroot`'s `-solid` option, a workaround is to use [hsetroot](https://archlinux.org/packages/?name=hsetroot) to set the background color:

```
$ hsetroot -solid '#000000'
```

See [[8\]](https://github.com/chjj/compton/issues/162) for more information.

### Screentearing with NVIDIA's proprietary drivers

Try this setting in `picom.conf`:

```
vsync = true;
```

### Lag with NVIDIA proprietary drivers and FullCompositionPipeline

See [#Screen artifacts/screenshot issues when using AMD's Catalyst driver](https://wiki.archlinux.org/title/Picom#Screen_artifacts/screenshot_issues_when_using_AMD's_Catalyst_driver).

Another option to reduce lag with the glx backend is to disable "allow flipping" [[9\]](https://github.com/yshui/picom/issues/620#issuecomment-869666038) in nvidia settings (OpenGL section). This can also be done from the command line:

```
$ nvidia-settings -a 'AllowFlipping=0'
```

To load settings after reboot (see [Autostarting](https://wiki.archlinux.org/title/Autostarting)) run

```
$ nvidia-settings --load-config-only
```

### Xorg leaking GPU memory with NVIDIA proprietary drivers

See [#Screen artifacts/screenshot issues when using AMD's Catalyst driver](https://wiki.archlinux.org/title/Picom#Screen_artifacts/screenshot_issues_when_using_AMD's_Catalyst_driver).

### Slock after suspend

When using a systemd service to trigger slock on a suspend or  hibernate action, one may find the screen unlocked for a few seconds  after resume. To prevent, disable window fading:

```
$ picom --no-fading-openclose
```

### Testing with old compton

Many issues are regressions from the old compton, which picom is a fork of. As a last resort you can try [compton-old-git](https://aur.archlinux.org/packages/compton-old-git/)AUR and see if it helps.

### Screen sharing

A shadowed overlay on screen sharing and shadows of [Zoom Meetings](https://wiki.archlinux.org/title/Zoom_Meetings) pop-up windows might be avoided by adding the following to `shadow-exclude`.

```
shadow-exclude = [
  "name = 'cpt_frame_xcb_window'",
  "class_g ?= 'zoom'",
];
```

Blurred screen sharing is disabled by adding [Zoom Meetings](https://wiki.archlinux.org/title/Zoom_Meetings) to `blur-background-exclude` with

```
blur-background-exclude = [
  "class_g ?= 'zoom'",
];
```

For Microsoft Teams, the red border around the shared content is  implemented with a mostly transparent window. Having blur enabled makes  it impossible to work with and should be disabled as follows:

```
shadow-exclude = [
  "name = 'rect-overlay'",
];

blur-background-exclude = [
  "name = 'rect-overlay'",
];
```

### Disable window fade in and fade out effect when switching between workspaces

Adding `--fade-in-step=1 --fade-out-step=1 --fade-delta=0` flag can disable the fade in and fade out effect when switching to a new workspace. [[10\]](https://github.com/yshui/picom/issues/354#issuecomment-835809210)

## See also

- [Howto: Using Compton for tear-free compositing on XFCE or LXDE](https://ubuntuforums.org/showthread.php?t=2144468&p=12644745#post12644745)
