# Kitty Terminal

## Installation

when using macos or linux just run this command

```bash
curl -L https://sw.kovidgoyal.net/kitty/installer.sh | sh /dev/stdin
```

## My Configuration

edit this file `.config/kitty/kitty.conf`


```
shell /usr/bin/fish
allow_remote_control yes
font_family UbuntuMono Nerd Font
term xterm-256color
```

## Download Nerd Fonts

download [UbuntuMono Nerd Font](https://www.nerdfonts.com/font-downloads)

extract it to this folder `~/.fonts` if not exists create it with

```bash
mkdir ~/.fonts
```

then refresh font cache with this command :

```bash
fc-cache -fv
```
