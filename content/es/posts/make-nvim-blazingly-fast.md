+++
title = "Haz que Nvim sea rápido"
date = "2025-03-23"
description = "Compartiré cómo depuré y descubrí las razones por las que mi configuración de nvim se vuelve lenta."
summary = "Compartiré cómo depuré y descubrí las razones por las que mi configuración de nvim se vuelve lenta."
tags = [
  "nvim"
]
showToc = true
showComments = true
+++

## Cómo rastrear la lentitud

Primero, intenta reproducir el congelamiento o la lentitud. Una vez que puedas reproducir la lentitud de forma consistente, comienza a deshabilitar tus plugins, autocmds y opciones hasta identificar al culpable.

¿Pero qué pasa si nada parece funcionar? ¿Qué pasa si la lentitud solo aparece después de horas de codear? Entonces, amigo mío, [profile.nvim](https://github.com/stevearc/profile.nvim) será tu nuevo mejor compañero. Cuando notes la lentitud de forma constante, activa `profile.nvim` para capturar todo lo que nvim ejecuta. Solo mantén la grabación breve, lo suficientemente larga como para capturar lo que sucede entre tus pulsaciones de tecla.

A continuación, pasa el `profile.json` generado a una interfaz gráfica como [Perfetto UI](https://ui.perfetto.dev/). Allí podrás ver qué funciones se ejecutan al presionar `j` (o cualquier otra tecla) y cuánto tardan, revelando finalmente al culpable furtivo.

![Perfetto UI](https://res.cloudinary.com/wochap/image/upload/v1742770964/wochap/assets/2025-03-23-17-56-12.webp)

> **EDIT (Marzo 27, 2025):** [snacks.nvim](https://github.com/folke/snacks.nvim) incluye un [profiler tool](https://github.com/folke/snacks.nvim/blob/main/docs/profiler.md), el cual prefiero sobre [profile.nvim](https://github.com/stevearc/profile.nvim).
> ![snacks profiler](https://res.cloudinary.com/wochap/image/upload/v1743109719/wochap/assets/Screenshot_2025-03-27_04-07-30.webp)

## Ajustando opciones

### Deshabilitar "built-in syntax highlighting"

He configurado `syntax` en `manual` para poder seguir utilizando el "syntax highlighting" de nvim para los tipos de archivo que no cuentan con soporte de treesitter. Esta configuración te permite habilitar manualmente "syntax highlighting" para esos tipos de archivo con un autocmd:

```lua
vim.cmd.syntax("manual")
```

Cada vez que un tipo de archivo no soporte treesitter, puedes habilitar su `syntax` usando el siguiente autocmd:

```lua
vim.api.nvim_create_autocmd("FileType", {
  pattern = { "gitsendemail", "conf", "editorconfig", "qf", "checkhealth", "less" },
  callback = function(event)
    vim.bo[event.buf].syntax = vim.bo[event.buf].filetype
  end,
})
```

### Configurando `synmaxcol`

La opción `synmaxcol` limita cuánto de una línea horizontal larga es resaltada por el "nvim’s syntax engine". Esta configuración es especialmente útil al abrir archivos con líneas extremadamente largas, ya que evita que nvim se congele al intentar resaltar la línea completa:

```lua
vim.opt.synmaxcol = 500
```

### Gestión de "signs"

Intenta evitar agregar "signs" siempre que sea posible, ya que pueden añadir sobrecarga en el rendimiento—especialmente cuando se usan con plugins personalizados de la columna de estado como [statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim). Por ejemplo, yo deshabilito los "diagnostic signs", ya que a menudo tengo numerosas notificaciones ("hints" y "warnings") en pantalla. De manera similar, con plugins como `gitsigns.nvim`, mostrar signos para archivos nuevos también puede afectar el rendimiento.

### Deshabilitar "Inlay Hints" en "Insert Mode"

Las "Inlay Hints" se ejecutan con cada pulsación de tecla. Si tu protocolo de lenguaje (LSP) no es tan bueno, estas actualizaciones frecuentes pueden bloquear nvim. Para mitigar este problema, deshabilita las "Inlay Hints" al entrar en el "Insert Mode" y vuélvelas a habilitar al salir:

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

Dado que paso la mayor parte de mi tiempo trabajando con React y Vue, aquí tienes un consejo rápido: si usas el LSP `eslint`, configúralo para que se ejecute solo al guardar. Además, considera abandonar `ts_ls` a favor de `vtsls`.

```lua
-- Configuración del LSP eslint
{
  settings = {
    run = "onSave",
  }
}
```

Algunos LSPs pueden devolver una gran cantidad de sugerencias, como es el caso de [vtsls](https://github.com/yioneko/vtsls). Esto puede hacer que tu plugin de autocompletado (por ejemplo, [blink.cmp](https://github.com/Saghen/blink.cmp), [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)) se ralentice. Para evitarlo, es recomendable limitar la cantidad de sugerencias que devuelve el LSP. Por ejemplo, para reducir las sugerencias de `vtsls`, puedes ajustar su configuración de la siguiente manera:

```lua
-- vtsls LSP config
{
  settings = {
    vtsls = {
      experimental = {
        completion = {
          enableServerSideFuzzyMatch = true,
          entriesLimit = 20,
        },
      },
    },
  },
}
```

## Ajustando Plugins

### nvim-treesitter

Deshabilita el resaltado de [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) para archivos que sean demasiado grandes o que sean detectados como minificados. Esto previene problemas de rendimiento al procesar dichos archivos. Además, deshabilita `additional_vim_regex_highlighting`:

```lua
highlight = {
  enable = true,
  additional_vim_regex_highlighting = false,
  disable = function(_, bufnr)
    -- Devuelve true si el buffer es un archivo minificado o si su tamaño es demasiado grande.
  end,
},
```

> Puedes ver mi implementación de la función `treesitter.highlight.disable` [aquí](https://github.com/wochap/nvim/blob/de57423876ae8aa591285b1f671c77e51151711c/lua/custom/plugins/treesitter.lua#L54-L54) y [aquí](https://github.com/wochap/nvim/blob/de57423876ae8aa591285b1f671c77e51151711c/lua/custom/utils/init.lua#L12).

### nvim-tree.lua

Asegúrate de revisar los ["Performance Tips"](https://github.com/nvim-tree/nvim-tree.lua/wiki/Troubleshooting#performance-tips) en su página wiki. Si tienes habilitados los `filesystem_watchers`, asegúrate de ignorar directorios con una gran cantidad de archivos, como `node_modules`. Además, deshabilita la opción `modified`, ya que puede causar lentitud en bases de código grandes:

```lua
{
  filesystem_watchers = {
    enable = true,
    -- Personalmente, ignoro los directorios `.git` y `.direnv` ya que utilizo https://direnv.net
    ignore_dirs = { "node_modules", ".direnv", ".git" },
  },
  modified = {
    enabled = false
  }
}
```

### snacks.nvim

Recomiendo encarecidamente reemplazar ciertos plugins con sus equivalentes de Snacks, ya que Snacks es increíblemente rápido, por ejemplo:

- [statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim) → [Snacks Statuscolumn](https://github.com/folke/snacks.nvim/blob/main/docs/statuscolumn.md)
- [indent-blankline.nvim](https://github.com/lukas-reineke/indent-blankline.nvim) → [Snacks Indent](https://github.com/folke/snacks.nvim/blob/main/docs/indent.md)
- [indentmini.nvim](https://github.com/nvimdev/indentmini.nvim) → [Snacks Indent](https://github.com/folke/snacks.nvim/blob/main/docs/indent.md)
- [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) → [Snacks Picker](https://github.com/folke/snacks.nvim/blob/main/docs/picker.md)
