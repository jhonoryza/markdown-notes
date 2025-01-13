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

### My config

edit this file `~/.config/nvim/lua/plugins/init.lua` add this plugins

```lua
  {
    "rachartier/tiny-inline-diagnostic.nvim",
    event = "VeryLazy",
    config = function()
      require "configs.inline-diagnostics"
    end,
  },
  { "nvchad/volt",     lazy = true },
  {
    "nvchad/minty",
    lazy = true,
    config = function()
      require "configs.minty"
    end,
  },
  { "nvchad/menu",     lazy = true },

  { "nvchad/showkeys", cmd = "ShowkeysToggle", opts = { position = "top-center" } },

  { "nvchad/timerly",  cmd = "TimerlyToggle" },

  {
    "jose-elias-alvarez/null-ls.nvim",
    config = function()
      require("null-ls").setup({
        sources = {
          -- Menggunakan golangci-lint-server untuk linting
          null_ls.builtins.diagnostics.golangci_lint.with({
             command = "golangci-lint-server",  -- Pastikan path ke golangci-lint-server sesuai
          }),
        },
      })
    end,
  },
```

edit this file `~/.config/nvim/lua/configs/lspconfig.lua`

```lua
local servers = { "html", "cssls", "gopls", "volar", "svelte", "emmet_language_server", "css_variables", "denols", "tailwindcss", "rust_analyzer", "intelephense", "ansiblels", "bashls", "cmake", "docker_compose_language_service", "dockerls", "ts_ls", "lua_ls", "clangd" }

lspconfig.ts_ls.setup{
  on_attach = nvlsp.on_attach,
  on_init = nvlsp.on_init,
  capabilities = nvlsp.capabilities,
  filetypes = { "typescript", "typescriptreact", "typescript.tsx" },
}
  lspconfig.volar.setup {
    init_options = {
      typescript = {
        tsdk = '/home/ubuntu/.local/share/nvim/mason/packages/typescript-language-server/node_modules/typescript/lib'
      }
    }
  }
```

add this file `~/.config/nvim/lua/configs/inline-diagnostics.lua`

```lua
-- Default configuration
require("tiny-inline-diagnostic").setup {
  signs = {
    left = "",
    right = "",
    diag = "●",
    arrow = "    ",
    up_arrow = "    ",
    vertical = " │",
    vertical_end = " └",
  },
  hi = {
    error = "DiagnosticError",
    warn = "DiagnosticWarn",
    info = "DiagnosticInfo",
    hint = "DiagnosticHint",
    arrow = "NonText",
    background = "CursorLine", -- Can be a highlight or a hexadecimal color (#RRGGBB)
    mixing_color = "None", -- Can be None or a hexadecimal color (#RRGGBB). Used to blend the background color with the diagnostic background color with another color.
  },
  blend = {
    factor = 0.27,
  },
  options = {
    -- Show the source of the diagnostic.
    show_source = false,

    -- Throttle the update of the diagnostic when moving cursor, in milliseconds.
    -- You can increase it if you have performance issues.
    -- Or set it to 0 to have better visuals.
    throttle = 20,

    -- The minimum length of the message, otherwise it will be on a new line.
    softwrap = 15,

    -- If multiple diagnostics are under the cursor, display all of them.
    multiple_diag_under_cursor = false,

    -- Enable diagnostic message on all lines.
    -- /!\ Still an experimental feature, can be slow on big files.
    multilines = false,

    overflow = {
      -- Manage the overflow of the message.
      --    - wrap: when the message is too long, it is then displayed on multiple lines.
      --    - none: the message will not be truncated.
      --    - oneline: message will be displayed entirely on one line.
      mode = "wrap",
    },

    -- Format the diagnostic message.
    -- Example:
    -- format = function(diagnostic)
    --     return diagnostic.message .. " [" .. diagnostic.source .. "]"
    -- end,
    format = nil,

    --- Enable it if you want to always have message with `after` characters length.
    break_line = {
      enabled = false,
      after = 30,
    },

    virt_texts = {
      priority = 2048,
    },
  },
}
```

open nvim and type `:Mason` run this command

```bash
:MasonInstall gopls intelephense html-lsp json-lsp kotlin-language-server lua-language-server markdownlint-cli2 phpmd phpstan prisma-language-server python-lsp-server sql-formatter sqls css-lsp deno docker-compose-language-service dockerfile-language-server emmet-language-server eslint-lsp gofumpt goimports golangci-lint golangci-lint-langserver golines gomodifytags tailwindcss-language-server ts_ls vue-language-server svelte-language-server clangd rust-analyzer ansible-language-server bash-language-server blade-formatter cmake-language-server terraform-ls typescript-language-server 
```
