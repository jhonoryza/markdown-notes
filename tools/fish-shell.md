# Fish Shell

## Installation

when using linux with ubuntu run this command:

```bash
sudo apt install fish
```

## My Configuration

edit this file `.config/fish/config.fish`

```
if status is-interactive
    # Commands to run in interactive sessions can go here
end

function vi
    nvim $argv
end

function q
    exit
end

function l
    ls -la $argv
end

set -U fish_user_paths $HOME/.local/bin $fish_user_paths
set -U fish_user_paths /usr/local/go/bin $fish_user_paths
set -U fish_user_paths /usr/local/nvim/bin $fish_user_paths

# fnm
set FNM_PATH /home/ubuntu/.local/share/fnm
if test -d $FNM_PATH
    set -x PATH $FNM_PATH $PATH
    fnm env | source
end
```
