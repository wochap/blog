+++
title = "Make Nvim Blazingly Fast"
date = "2025-03-23"
description = "I'll share how I debugged and discovered the reasons behind my nvim configuration being laggy."
summary = "I'll share how I debugged and discovered the reasons behind my nvim configuration being laggy."
tags = [
  "nvim"
]
showToc = true
showComments = true
+++

## How to Track Down the Lag

First, try to reproduce the freeze or lag. Once you can consistently trigger the sluggishness, start disabling your custom plugins, autocmds, and options until you pinpoint the culprit.

But what if nothing seems to work? What if the lag only appears after hours of coding? Then my friend, [profile.nvim](https://github.com/stevearc/profile.nvim) is your new best buddy. When you notice lag consistently, fire up profile.nvim to capture everything nvim executes. Just keep the recording short—long enough to catch what happens between your keystrokes.

Next, pass the generated `profile.json` to a GUI like [Perfetto UI](https://ui.perfetto.dev/). There, you can see which functions run when you press `j` (or any key) and how long they take, ultimately revealing the sneaky offender.

![Perfetto UI](https://res.cloudinary.com/wochap/image/upload/v1742770964/wochap/assets/2025-03-23-17-56-12.webp)

> **EDIT (March 27, 2025):** [snacks.nvim](https://github.com/folke/snacks.nvim) includes a [profiler tool](https://github.com/folke/snacks.nvim/blob/main/docs/profiler.md), which I prefer over [profile.nvim](https://github.com/stevearc/profile.nvim).
> ![snacks profiler](https://res.cloudinary.com/wochap/image/upload/v1743109719/wochap/assets/Screenshot_2025-03-27_04-07-30.webp)

## Tweaking options

### Disabling built-in syntax highlighting

I set `syntax` to `manual` so that I can still use nvim’s built-in syntax for filetypes that lack treesitter support. This configuration lets you manually enable syntax for those filetypes with an autocmd:

```lua
vim.cmd.syntax("manual")
```

Whenever a filetype doesn’t support treesitter, you can enable its `syntax` using the following autocmd:

```lua
vim.api.nvim_create_autocmd("FileType", {
  pattern = { "gitsendemail", "conf", "editorconfig", "qf", "checkhealth", "less" },
  callback = function(event)
    vim.bo[event.buf].syntax = vim.bo[event.buf].filetype
  end,
})
```

### Setting `synmaxcol`

The `synmaxcol` option limits how much of a long horizontal line is highlighted by nvim’s syntax engine. This setting is especially useful when opening files with extremely long lines, as it prevents nvim from freezing while attempting to highlight the entire line:

```lua
vim.opt.synmaxcol = 500
```

### Managing Signs

Try to avoid enabling signs where possible, as they can add performance overhead—especially when used with custom status column plugins like [statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim). For instance, I disable diagnostic signs since I often have numerous diagnostics (hints and warnings) on screen. Similarly, with plugins like `gitsigns.nvim`, showing signs for new files can also impact performance.

### Disabling Inlay Hints in Insert Mode

Inlay hints execute on every keystroke. If your language server protocol (LSP) is performing poorly, these frequent updates can block nvim. To mitigate this issue, disable inlay hints when entering insert mode and re-enable them upon leaving:

```lua
vim.api.nvim_create_autocmd("InsertEnter", {
  pattern = "*",
  callback = function(event)
    vim.schedule(function()
      vim.lsp.inlay_hint.enable(false, { bufnr = event.buf })
    end)
  end,
})
vim.api.nvim_create_autocmd("InsertLeave", {
  group = utils.augroup("enable_inlay_hints"),
  pattern = "*",
  callback = function(event)
    vim.schedule(function()
      vim.lsp.inlay_hint.enable(true, { bufnr = event.buf })
    end)
  end,
})
```

### LSP

Since I spend most of my time juggling React and Vue, here’s a quick tip: if you're using the `eslint` LSP, configure it to run only on save—because ain't nobody got time for constant linting. Also, consider ditching `ts_ls` in favor of `vtsls`; your editor (and your sanity) will thank you.

```lua
-- eslint LSP config
{
  settings = {
    run = "onSave",
  }
}
```

## Tweaking Plugins

### nvim-treesitter

Disable [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) highlighting for files that are either too large or detected as minified. This prevents performance issues when processing such files. Also, disable `additional_vim_regex_highlighting`:

```lua
highlight = {
  enable = true,
  additional_vim_regex_highlighting = false,
  disable = function(_, bufnr)
    -- Return true if the buffer is a minified file or if its size is too large.
  end,
},
```

> You can view my implementation of the `treesitter.highlight.disable` function [here](https://github.com/wochap/nvim/blob/de57423876ae8aa591285b1f671c77e51151711c/lua/custom/plugins/treesitter.lua#L54-L54) and [here](https://github.com/wochap/nvim/blob/de57423876ae8aa591285b1f671c77e51151711c/lua/custom/utils/init.lua#L12).

### nvim-tree.lua

Be sure to check out the ["Performance Tips"](https://github.com/nvim-tree/nvim-tree.lua/wiki/Troubleshooting#performance-tips) on its wiki page. If you have `filesystem_watchers` enabled, make sure to ignore directories with a large number of files, such as `node_modules`. Additionally, disable the `modified` option since it can cause lag in large codebases:

```lua
{
  filesystem_watchers = {
    enable = true,
    -- I personally ignore the `.git` and `.direnv` directories since I use https://direnv.net
    ignore_dirs = { "node_modules", ".direnv", ".git" },
  },
  modified = {
    enabled = false
  }
}
```

### snacks.nvim

I highly recommend replacing certain plugins with their Snacks counterparts, as Snacks is blazingly fast, for example:

- [statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim) → [Snacks Statuscolumn](https://github.com/folke/snacks.nvim/blob/main/docs/statuscolumn.md)
- [indent-blankline.nvim](https://github.com/lukas-reineke/indent-blankline.nvim) → [Snacks Indent](https://github.com/folke/snacks.nvim/blob/main/docs/indent.md)
- [indentmini.nvim](https://github.com/nvimdev/indentmini.nvim) → [Snacks Indent](https://github.com/folke/snacks.nvim/blob/main/docs/indent.md)
- [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) → [Snacks Picker](https://github.com/folke/snacks.nvim/blob/main/docs/picker.md)
