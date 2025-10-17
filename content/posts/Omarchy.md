---
title: I switched to Omarchy; you should, too
date: 2025-10-17
draft: false
---
Ever since I put Arch Linux on my laptop, I've always known I would eventually switch my Windows desktop to Linux as well. I just hadn't found a time or reason to do so. Then Omarchy came out and there was no longer an excuse about "it's going to take forever to configure another Arch machine".

##  Installation
The installation itself was trivial, head to the [Omarchy website](https://omarchy.org/), download the ISO, and load it on an external drive with [Rufus](https://rufus.ie/en/), [balenaEtcher](https://etcher.balena.io/), or my personal favorite, [Ventoy](https://www.ventoy.net/en/index.html)
Then in your computer's BIOS/UEFI, set that drive as your first bootable drive. Once in the Omarchy installer, read through what it says and install away. One interesting thing I found is that Omarchy's installer won't let you install on a drive's partition, only letting you select an entire drive, meaning the entire drive will be formatted and wiped. Make backups!

After that it tells you to restart and remove the bootable media, and you're in!

## Basic Configuration
First things first, I don't like SUPER being the Windows key, maybe it's just that I'm used to ALT, but I much prefer it.
Omarchy Menu -> Setup -> Input
```
# ~/.config/hypr/input.conf

kb_options = compose:caps, altwin:swap_alt_win
```
Then, I don't like the default `SUPER + W` to close a window, I prefer `SUPER + W`. I can change that in the Omarchy Menu -> Setup -> Keybindings:
```
# ~/.config/hypr/bindings.conf

unbind = SUPER, W
bindd = SUPER, Q, Close active window, killactive
```
Oh, and having to type `nvim` bothers me, so I added this alias to `~/.bashrc`:
```
# ~/.bashrc

alias vim='nvim'
```

## VPN
I use NordVPN so I want to get this going ASAP. Turns out it's super simple. Install the package `nordvpn-bin` and then `nordvpn-gui` (technically optional but I want it).
Adding myself to the `nordvpn` group with
```
sudo usermod -aG nordvpn $USER
```
And restarting
After a restart, I enable the nordvpn service:
```
sudo systemctl enable --now nordvpnd
```
And now that that's working, it's time to log in. Unfortunately, the web login page doesn't properly redirect back to the app, so we need to log in via the command line. Logging into NordVPN on the website, I can go to the NordVPN service, then scroll down to the access token, generate a new one, copy it, and then in the command line:
```
nordvpn login --token <token>
```
And just like that, VPN is working!

## Games
Luckily there has been a lot of work towards this in the past years so the process was painless. Omarchy makes it trivial to download Steam through their menu, and Proton is better than ever, meaning I don't expect to run into any issues with any of the games I play

## Multiple Monitors
I was delighted to see that without any configuration of my own, Omarchy automatically picked up both of my monitors and used them both properly. I did, however, need to modify the setup a bit.
The relevant configuration file is `~/.config/hypr/monitors.conf`
The config was set automatically for any monitors, but I wanted more control about exactly what was happening and where.
An easy way to detect your monitors is to run `hyprctl monitors` in the terminal. The important thing you need to know are the names, resolutions, and refresh rates. This was the output for me:
```
Monitor HDMI-A-1 (ID 0):
	1920x1080@60.00000 at 0x0
	description: Acer Technologies EK271 H 73260BEFA3W01
	make: Acer Technologies
	model: EK271 H
	physical size (mm): 600x330
	serial: 73260BEFA3W01
	active workspace: 11 (11)
	special workspace: 0 ()
	reserved: 0 26 0 0
	scale: 1.00
	transform: 0
	focused: no
	dpmsStatus: 1
	vrr: false
	solitary: 0
	solitaryBlockedBy: windowed mode,missing candidate
	activelyTearing: false
	tearingBlockedBy: next frame is not torn,user settings,missing candidate
	directScanoutTo: 0
	directScanoutBlockedBy: user settings,missing candidate
	disabled: false
	currentFormat: XRGB8888
	mirrorOf: none
	availableModes: 1920x1080@60.00Hz 1920x1080@100.00Hz 1920x1080@74.97Hz 1920x1080@59.94Hz 1920x1080@50.00Hz 1680x1050@59.95Hz 1280x1024@75.03Hz 1280x1024@60.02Hz 1440x900@59.89Hz 1280x960@60.00Hz 1280x800@59.81Hz 1152x864@75.00Hz 1280x720@60.00Hz 1280x720@59.94Hz 1280x720@50.00Hz 1024x768@75.03Hz 1024x768@70.07Hz 1024x768@60.00Hz 800x600@75.00Hz 800x600@72.19Hz 800x600@60.32Hz 800x600@56.25Hz 720x576@50.00Hz 720x480@59.94Hz 640x480@75.00Hz 640x480@72.81Hz 640x480@59.94Hz 640x480@59.93Hz

Monitor DP-2 (ID 1):
	2560x1440@144.00000 at 1920x0
	description: LG Electronics LG QHD 0x0000AC28
	make: LG Electronics
	model: LG QHD
	physical size (mm): 700x390
	serial: 0x0000AC28
	active workspace: 1 (1)
	special workspace: 0 ()
	reserved: 0 26 0 0
	scale: 1.00
	transform: 0
	focused: yes
	dpmsStatus: 1
	vrr: false
	solitary: 0
	solitaryBlockedBy: windowed mode,missing candidate
	activelyTearing: false
	tearingBlockedBy: next frame is not torn,user settings,missing candidate
	directScanoutTo: 0
	directScanoutBlockedBy: user settings,missing candidate
	disabled: false
	currentFormat: XRGB8888
	mirrorOf: none
	availableModes: 2560x1440@144.00Hz 2560x1440@120.00Hz 2560x1440@74.97Hz 2560x1440@59.95Hz 1920x1080@74.91Hz 1920x1080@60.00Hz 1680x1050@59.95Hz 1600x900@60.00Hz 1280x1024@75.03Hz 1280x1024@60.02Hz 1280x800@59.81Hz 1152x864@59.96Hz 1280x720@60.00Hz 1024x768@75.03Hz 1024x768@60.00Hz 800x600@75.00Hz 800x600@60.32Hz 640x480@75.00Hz 640x480@59.94Hz
```
And the relevant bits of information:
```
HDMI-A-1
	Resolution: 1920x1080
	Refresh Rate: 60Hz (in availableModes you can see it can go up to 100Hz but as my secondary monitor I left it as is)

DP-2
	Resolution: 2560x1440
	Refresh Rate: 144Hz
```
Once gathered I can modify the line in my monitor config file:
```
# ~/.config/hypr/monitors.conf

monitor=,preferred,auto,auto
```
to
```
# ~/.config/hypr/monitors.conf

monitor=HDMI-A-1,1920x1080@60,0x0,1
monitor=DP-2,2560x1440@144,1920x0,1
```
Note the second monitor has `1920x0`, this is the "absolute" position in space. This is to ensure that Hyprland knows that HDMI-A-1 is to the left of DP-2, exactly 1920 pixels to the right and 0 pixels down.

## Networking
There's one ("one") thing I dislike about Omarchy. It uses `iwd` as its networking software instead of `NetworkManager`. I was unable to connect to `eduroam` with the system in place, so I switched to NetworkManager using [this comment](https://github.com/basecamp/omarchy/issues/1414#issuecomment-3262272669) from an Issue in the Omarchy Github. The only thing I had to change is to not run the `systemctl mask --now wpa_supplicant.service` command, as for some reason that broke all networking on my laptop. Once that was done, I just use `nmtui` for connecting to new networks and imported my old `eduroam.nmconnection` file and it worked perfectly. I wish it came with NetworkManager by default, but at least it's simple enough to change.
I've noticed the waybar icon still shows network details, which seems to imply that it knows of the connection and its speed, but I haven't tried using the app to connect to any networks, YMMV.

## Workspaces
In Omarchy the default workspace configuration is perfectly fine, but I wanted more rigid control of where my workspaces were, since none were bound to any particular monitors.
I created 20 workspaces, 10 bound to each monitor, and since there's no "workspace" configuration file, I made my own and added this line to the base hyprland configuration file:
```
# ~/.config/hypr/hyprland.conf

source = ~/.config/hypr/workspaces.conf
```
```
# ~/.config/hypr/workspaces.conf

# Workspaces 1–10 → DP-2 (right)
workspace=1,monitor:DP-2
workspace=2,monitor:DP-2
workspace=3,monitor:DP-2
workspace=4,monitor:DP-2
workspace=5,monitor:DP-2
workspace=6,monitor:DP-2
workspace=7,monitor:DP-2
workspace=8,monitor:DP-2
workspace=9,monitor:DP-2
workspace=10,monitor:DP-2

# Workspaces 11–20 → HDMI-A-1 (left)
workspace=11,monitor:HDMI-A-1
workspace=12,monitor:HDMI-A-1
workspace=13,monitor:HDMI-A-1
workspace=14,monitor:HDMI-A-1
workspace=15,monitor:HDMI-A-1
workspace=16,monitor:HDMI-A-1
workspace=17,monitor:HDMI-A-1
workspace=18,monitor:HDMI-A-1
workspace=19,monitor:HDMI-A-1
workspace=20,monitor:HDMI-A-1
```
And then I rebound the keybindings to allow me to use them as I'm most comfortable with:
```
# ~/.config/hypr/bindings.conf

# Workspaces
# Unbind
unbind = SUPER, code:10
unbind = SUPER, code:11
unbind = SUPER, code:12
unbind = SUPER, code:13
unbind = SUPER, code:14
unbind = SUPER, code:15
unbind = SUPER, code:16
unbind = SUPER, code:17
unbind = SUPER, code:18
unbind = SUPER, code:19
unbind = SUPER SHIFT, code:10
unbind = SUPER SHIFT, code:11
unbind = SUPER SHIFT, code:12
unbind = SUPER SHIFT, code:13
unbind = SUPER SHIFT, code:14
unbind = SUPER SHIFT, code:15
unbind = SUPER SHIFT, code:16
unbind = SUPER SHIFT, code:17
unbind = SUPER SHIFT, code:18
unbind = SUPER SHIFT, code:19

# Bind
# Main Monitor
bindd = SUPER, code:10, Switch to workspace 1, workspace, 1
bindd = SUPER, code:11, Switch to workspace 2, workspace, 2
bindd = SUPER, code:12, Switch to workspace 3, workspace, 3
bindd = SUPER, code:13, Switch to workspace 4, workspace, 4
bindd = SUPER, code:14, Switch to workspace 5, workspace, 5
bindd = SUPER, code:15, Switch to workspace 6, workspace, 6
bindd = SUPER, code:16, Switch to workspace 7, workspace, 7
bindd = SUPER, code:17, Switch to workspace 8, workspace, 8
bindd = SUPER, code:18, Switch to workspace 9, workspace, 9
bindd = SUPER, code:19, Switch to workspace 10, workspace, 10
bindd = SUPER SHIFT, code:10, Move window to workspace 1, movetoworkspace, 1
bindd = SUPER SHIFT, code:11, Move window to workspace 2, movetoworkspace, 2
bindd = SUPER SHIFT, code:12, Move window to workspace 3, movetoworkspace, 3
bindd = SUPER SHIFT, code:13, Move window to workspace 4, movetoworkspace, 4
bindd = SUPER SHIFT, code:14, Move window to workspace 5, movetoworkspace, 5
bindd = SUPER SHIFT, code:15, Move window to workspace 6, movetoworkspace, 6
bindd = SUPER SHIFT, code:16, Move window to workspace 7, movetoworkspace, 7
bindd = SUPER SHIFT, code:17, Move window to workspace 8, movetoworkspace, 8
bindd = SUPER SHIFT, code:18, Move window to workspace 9, movetoworkspace, 9
bindd = SUPER SHIFT, code:19, Move window to workspace 10, movetoworkspace, 10
# Secondary Monitor
bindd = SUPER CTRL, code:10, Switch to workspace 11, workspace, 11
bindd = SUPER CTRL, code:11, Switch to workspace 12, workspace, 12
bindd = SUPER CTRL, code:12, Switch to workspace 13, workspace, 13
bindd = SUPER CTRL, code:13, Switch to workspace 14, workspace, 14
bindd = SUPER CTRL, code:14, Switch to workspace 15, workspace, 15
bindd = SUPER CTRL, code:15, Switch to workspace 16, workspace, 16
bindd = SUPER CTRL, code:16, Switch to workspace 17, workspace, 17
bindd = SUPER CTRL, code:17, Switch to workspace 18, workspace, 18
bindd = SUPER CTRL, code:18, Switch to workspace 19, workspace, 19
bindd = SUPER CTRL, code:19, Switch to workspace 20, workspace, 20
bindd = SUPER SHIFT CTRL, code:10, Move window to workspace 11, movetoworkspace, 11
bindd = SUPER SHIFT CTRL, code:11, Move window to workspace 12, movetoworkspace, 12
bindd = SUPER SHIFT CTRL, code:12, Move window to workspace 13, movetoworkspace, 13
bindd = SUPER SHIFT CTRL, code:13, Move window to workspace 14, movetoworkspace, 14
bindd = SUPER SHIFT CTRL, code:14, Move window to workspace 15, movetoworkspace, 15
bindd = SUPER SHIFT CTRL, code:15, Move window to workspace 16, movetoworkspace, 16
bindd = SUPER SHIFT CTRL, code:16, Move window to workspace 17, movetoworkspace, 17
bindd = SUPER SHIFT CTRL, code:17, Move window to workspace 18, movetoworkspace, 18
bindd = SUPER SHIFT CTRL, code:18, Move window to workspace 19, movetoworkspace, 19
bindd = SUPER SHIFT CTRL, code:19, Move window to workspace 20, movetoworkspace, 20
```
This to me is much more comfortable, I know exactly how to go to any workspace at any time from anywhere, no relative arrows or vim motions. Absolute precision.
And finally I had to modify how the waybar showed the workspaces. As far as I know, you can't have one instance of waybar running 2 separate bars for 2 separate monitors, and I don't want to have all 20 workspaces on my waybar. Ideally I'd be able to see each monitor's workspaces in each monitor. But since I don't care enough to actually handle 2 separate waybars, I just have the waybar on my secondary monitor not show any workspace labels.
```
# ~/.config/waybar/config.jsonc

"hyprland/workspaces": {
    "monitor": "DP-2",
    "on-click": "activate",
    "format": "{icon}",
    "format-icons": {
      "default": "",
      "1": "1",
      "2": "2",
      "3": "3",
      "4": "4",
      "5": "5",
      "6": "6",
      "7": "7",
      "8": "8",
      "9": "9",
      "10": "10",
      "active": "󱓻"
    }
  },
```
Importantly, I set it only to show these labels on my primary monitor (DP-2), and I removed `persistent-workspaces`, since it's ridiculous to see empty workspaces.

## Window Rules
Here's where I set rules for windows, I always want certain apps on certain workspaces, so I can set up some window rules to configure that, along with some other tweaks.
In a new file (like with the workspaces):
```
# ~/.config/hypr/windowrules.conf

windowrule=workspace 2, class:zen
windowrule=workspace 5, class:obsidian
windowrule=workspace 6, class:steam
windowrule=workspace 11, class:vesktop

windowrule=tile, class:steam
```
Here I define where I want these apps to open, and since Steam decided to be a bit annoying and not tile by default, I configure it to.
If you want to do this for your own apps, you can open it and in any terminal run `hyprctl clients`, find your window, and copy the class name.

## Application Launcher
In Omarchy, pressing `SUPER + Space` brings up the Application Launcher, where you can fuzzy find apps/web apps and other things and run them. A lot of the apps shown there are apps I'm never going to use (ChatGPT, Grok, X, Zoom, etc) and so I don't want them to even show up.
Luckily this menu uses `walker` which pulls the relevant information from `.desktop` files in `~/.local/share/applications` and `/usr/share/applications`. It's as simple as either deleting the relevant ones or renaming them to something like `X.desktop.disabled` so `walker` won't pick them up anymore.
On a side note, I now know how and where to change/set an icon or make a new app or custom program I can run, and might do so in the future.

## Movement Keybindings
Omarchy by default uses `SUPER + <arrow key>` to navigate between windows, but I'm much more comfortable with vim keybinds, and moving to the arrow keys is something I like to avoid if I can. So I'm going to change that by adding this to my bindings.conf *Omarchy Menu -> Setup -> Keybindings*
```
# ~/.config/hypr/bindings.conf

# Rebind some keybinds to allow for vim motions for window navigation
unbind = SUPER, K
unbind = SUPER, J
bindd = SUPER CTRL, K, Show key bindings, exec, omarchy-menu-keybindings
bindd = SUPER CTRL, J, Toggle split, togglesplit

# Vim Motions for window navigation
unbind = SUPER, left
unbind = SUPER, right
unbind = SUPER, up
unbind = SUPER, down
bindd = SUPER, H, Move focus left, movefocus, l
bindd = SUPER, L, Move focus right, movefocus, r
bindd = SUPER, K, Move focus up, movefocus, u
bindd = SUPER, J, Move focus down, movefocus, d

unbind = SUPER SHIFT, left
unbind = SUPER SHIFT, right
unbind = SUPER SHIFT, up
unbind = SUPER SHIFT, down
bindd = SUPER SHIFT, H, Swap window to the left, swapwindow, l
bindd = SUPER SHIFT, L, Swap window to the right, swapwindow, r
bindd = SUPER SHIFT, K, Swap window up, swapwindow, u
bindd = SUPER SHIFT, J, Swap window down, swapwindow, d
```
## Themes
I don't mind the default themes on Omarchy. My favorite is Catpuccin, but there was one on the [Omarchy Extra Theme Browser](https://learn.omacom.io/2/the-omarchy-manual/90/extra-themes) that I really liked. [Void](https://github.com/vyrx-dev/omarchy-void-theme). Luckily Omarchy makes it super simple to install and use themes, so under the Omarchy Menu you can navigate to *Install -> Style -> Theme* and paste in the GitHub link, and it just works.
## NeoVim
Another extremely simple task, but one I had to do regardless. Omarchy comes packaged with a pretty good LazyVim configuration for NeoVim; but I'm used to mine, and don't like some of the "extra fancy" things it adds.
I renamed `~/.config/nvim` to `~/.config/nvim-omarchy` and then cloned my own nvim config from GitHub.
[If you want it](https://github.com/Nixo371/nvim)
## SSH
I have a few things that depend on SSH keys. Most importantly, GitHub and my home media server. Both of these are simple to set up again, but GitHub is the easiest.
Make a new SSH key with `ssh-keygen` (I make one for each thing), save it in the default location and *set a passkey*. I can't stress this enough, I know it's your computer, but get in the habit of giving your SSH keys a passkey, I've seen interns use company laptops with their personal ssh keys, and when they leave, don't remove the ssh keys. You can use `ssh-agent`, but ***use a passkey***. Anyways, once that's done I log into GitHub, remove the old ones, add the new one, and that's done.
The media server one is slightly more tricky.
I ssh into the server with my laptop, whose ssh key is registered. From inside I modify the ssh config file:
```
# /etc/ssh/sshd_config

PasswordAuthentication yes
```
Now I'm able to ssh into my media server with my password, `ssh-copy-id`my new ssh key over, delete the old key by finding the old hostname in `~/.ssh/authorized_keys` and deleting the line in the file.
Once that's done, I re-disable password authentication as I enabled it above, and I'm done!

## Misc
Along with all the above, I downloaded Thunderbird as a mail client, Dolphin as my file explorer (I've heard great things), and Vesktop instead of Discord.
From the time I've had Omarchy everything has just felt *effortless*, and it might be the first time where I can genuinely recommend an OS like Omarchy to anyone interested in starting with Linux. Ubuntu and Linux Mint will always be the OG "beginner" distributions, but Omarchy definitely throws its hat in the ring.