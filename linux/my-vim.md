# Setting Up Neovim

Make a backup of your current Neovim configuration files:

```bash
# Linux / MacOS (Unix)
mv ~/.config/nvim{,.bak}
mv ~/.local/share/nvim{,.bak}
mv ~/.local/state/nvim{,.bak}
mv ~/.cache/nvim{,.bak}
```

## Using LazyVim

Clone the LazyVim starter repository and open Neovim:

```bash
git clone https://github.com/LazyVim/starter ~/.config/nvim && nvim
```

Remove the `.git` folder so you can add it to your own repository later:

```bash
rm -rf ~/.config/nvim/.git
```

Full documentation is available [here](https://www.lazyvim.org/).

## Using NvChad

Clone the NvChad starter repository and open Neovim:

```bash
git clone https://github.com/NvChad/starter ~/.config/nvim && nvim
```

The default mappings are defined
[here](https://github.com/NvChad/NvChad/blob/v2.5/lua/nvchad/mappings.lua).

Check
[configs.md](https://github.com/neovim/nvim-lspconfig/blob/master/doc/configs.md)
to ensure your language's LSP server is present there and edit the
`configs/lspconfig.lua` file to add your language's LSP.

Full documentation is available
[here](https://nvchad.com/docs/quickstart/install).
