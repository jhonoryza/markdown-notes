---
title: 'Konfigurasi Sublime Text untuk develop aplikasi Laravel'
date: '2023-08-05 19:00:00'
---

Berikut merupakan konfigurasi yang saya gunakan ketika develop aplikasi laravel menggunakan `sublime text` text editor.

## PLUGINS
- git
- git blame
- git gutter
- sidebar menu advanced
- syncedsidebar
- lsp
- lsp intelephense
- lsp bash
- lsp dockerfile
- terminus
- fileicons
- shell exec

untuk menginstall plugin buka command pallete dengan `cmd + shift + p` / `alt + shift + p` pilih install plugin

## LSP

untuk plugin LSP memerlukan `nodejs`, jadi pertama-tama kita install nodejs
- cara install [https://github.com/nvm-sh/nvm#installing-and-updating](https://github.com/nvm-sh/nvm#installing-and-updating)
- cara penggunaan [https://github.com/nvm-sh/nvm#usage](https://github.com/nvm-sh/nvm#usage)
- setelah terinstall jalankan perintah ini:

```bash
npm -g i intelephense
```

- Buka Menu `LSP > Servers > LSP-intelephense` dibawah menu `Package Settings`

```json
{
    "enabled": true,
    "command": [
        "intelephense",
        "--stdio",
    ],
    "scopes": ["source.php", "embedding.php"],
    "syntaxes": ["Packages/PHP/PHP.sublime-syntax"],
    "languageId": "php",
    "initializationOptions": {
        "clearCache": false,
        "licenceKey": "",
    },
}
```

- buka command palette pilih `enable LSP: Enable Language Server `

## Settings Preferences

buka `cmd + ,` / `alt + ,`

```json
{
	"ignored_packages":
	[
		"Vintage",
		"zzz A File Icon zzz",
	],
	"save_on_focus_lost": true,
    "shell_exec_executable": "/bin/zsh",
	"shell_exec_output": "panel",
	"shell_exec_output_word_wrap": false
}
```

## Keybindings

```json
[
    {
        "command": "lsp_symbol_definition",
        "args": {
            "side_by_side": false
        },
        "keys": [
            "super+enter"
        ],
        "context": [
            {
                "key": "lsp.session_with_capability",
                "operator": "equal",
                "operand": "definitionProvider"
            },
            {
                "key": "auto_complete_visible",
                "operator": "equal",
                "operand": false
            }
        ]
    },

    {
        "command": "lsp_format_document",
        "keys": [
            "super+shift+'"
        ],
        "context": [
            {
                "key": "lsp.session_with_capability",
                "operator": "equal",
                "operand": "documentFormattingProvider | documentRangeFormattingProvider"
            }
        ]
    },
    { "keys": ["super+b"], "command": "toggle_side_bar" },
    { "keys": ["super+t"], "command": "toggle_terminus_panel" },
    { "keys": ["super+shift+o"], "command": "close_all" },
    {
        "keys": ["super+shift+t"],
        "caption": "Terminus: Open Default Shell in Split Tab",
        "command": "terminus_open",
        "args": {
            "post_window_hooks": [
                ["carry_file_to_pane", {"direction": "down"}]
            ]
        }
    },
    {
      "keys": ["super+shift+;"],
      "command": "shell_exec_run",
      "args": {
        "format": "./vendor/bin/pint"
      }
    }
]
```

## Mouse bindings

```json
[
    {
        "button": "button1", 
        "count": 1, 
        "modifiers": ["super"],
        "press_command": "drag_select",
        "command": "lsp_symbol_definition"
    },
	{
		"button": "button1", "count": 1, "modifiers": ["ctrl"],
		"press_command": "drag_select",
		"press_args": {"additive": true}
	}
]
```
