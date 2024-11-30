---
title: 'my zsh terminal configuration'
date: '2023-08-30 00:07:00'
---

this is my zsh terminal configuration.

## required

clone this repo:

```bash
git clone https://github.com/jhonoryza/lazyvim-setup-scripts.git
./install_zsh.sh
./install_lazyvim.sh
./install_lazygit.sh
```

Open ~/.zshrc

```bash
nvim ~/.zshrc
#Find the line which says plugins=(git).
#Replace that line with plugins=(git zsh-autosuggestions zsh-syntax-highlighting fast-syntax-highlighting zsh-autocomplete)
```

how to change shell to zsh `chsh -s /usr/bin/zsh` by default after install oh-my-zsh you will be asked about default shell
download this font in your terminal: https://www.nerdfonts.com/font-downloads choose jetbrains mono or ubuntu mono
to install in macos just copy .ttf files `cp -vf *.ttf ~/Library/Fonts`


## lazyvim

install nodejs

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash && \
echo "export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion" >> .zshenv && \
source .zshenv && \
nvm install --lts
```

untuk install LSP tinggal menggunakan comand `:Mason` dan `:MasonInstall <package>`
untuk intelephense license key buat file `YOUR_HOME_DIR/intelephense/licence.txt` lalu restart nvim

tambahkan `~/.config/nvim/lua/plugins/codeium.lua`, setelah install `:Codeium Auth` ikut petunjuk yg ada
untuk accept suggestion ctrl+g

```lua
return {
  {
    "Exafunction/codeium.vim",
    event = "BufEnter",
  },
}
```

keymap yg sering dipakai di lazyvim

```
- space + space -> find file
- space + ` -> switch between last buffer
- shift + h or l` -> switch between buffer
- space + , -> select buffer
- space + b + d -> delete current buffer
- space + e -> toggle sidebar
- shift + > or < -> indent code sebelumnya mesti visual block dlu
- gcc -> comment bisa dengan visual block
```

secara default auto format ketika file di save


## optional jess terminal

pertama jika mau pake jess terminal uninstall dulu lazyvim

```bash
apt-get install tmux lua lua-check
git clone git@github.com:jhonoryza/dotfiles.git
cd dotfiles
./install

vim .
:PackerInstall
git clone https://github.com/jessarcher/zsh-artisan.git ~/.oh-my-zsh/custom/plugins/artisan

npm install -g @tailwindcss/language-server
npm install -g intelephense
export PATH="/home/ubuntu/.npm-global/bin"

wget https://github.com/LuaLS/lua-language-server/releases/download/3.6.24/lua-language-server-3.6.24-linux-x64.tar.gz
# move to /opt/lua-language-server
```
