# dimolto

Theme inspired by [risoto](https://github.com/joeroe/risotto)

## Configure

To use the theme, add `theme = 'dimolto'` to your site's `config.toml`, or `theme: dimolto` to your `config.yaml`.

See `exampleSite/config.toml` for the theme-specific parameters you need to add to your site's `config.toml` or `config.yaml` to configure the theme.

### Colour schemes

dimolto uses the [base16 framework](https://github.com/chriskempson/base16) to define colour schemes that can be used with the `theme.palette` parameter.
A selection of 16 palettes (10 dark, 6 light) are bundled with the theme: `catppuccin-macchiato`, `base16-dark`, `base16-light` and `dracula`.
The default is `base16-dark`.

The easiest way to use other base16 styles is to place .css file from https://github.com/monicfenga/base16-styles/tree/master/css-variables and place it in your `static/css/palettes` directory.

Or to define a wholly custom theme, you will need to define the following CSS variables for the following base16 colours (see [base16-dark.css](blob/main/static/css/palettes/base16-dark.css) for an example):

| Base | Default colour                             | Used for            | Examples                             |
| ---- | ------------------------------------------ | ------------------- | ------------------------------------ |
| 00   | <span class="base00">Dark</span>           | Background          | Page background                      |
| 01   | <span class="base01">│</span>              | Alt. background     | Content background                   |
| 02   | <span class="base02">│</span>              | In-text backgrounds | `<pre>`, `<code>`, `<kbd>`, `<samp>` |
| 03   | <span class="base03">│</span>              | Muted text          | `:before` & `:marker` symbols        |
| 04   | <span class="base04">│</span>              | Alt. foreground     | Aside text                           |
| 05   | <span class="base05">│</span>              | Foreground          | Content text                         |
| 06   | <span class="base06">│</span>              |                     |                                      |
| 07   | <span class="base07">Light</span>          |                     |                                      |
| 08   | <span class="base08">Red</span>            |                     |                                      |
| 09   | <span class="base09">Orange</span>         |                     |                                      |
| 0A   | <span class="base0A">Yellow</span>         | Highlights          | Selected text, `<mark>`              |
| 0B   | <span class="base0B">Green</span>          | Primary accent      | Logo                                 |
| 0C   | <span class="base0C">Cyan</span>           | Active links        | `a:active`, `a:hover`                |
| 0D   | <span class="base0D">Blue</span>           | Links               | `a:link`, `a:visited`                |
| 0E   | <span class="base0E">Magenta</span>        |                     |                                      |
| 0F   | <span class="base0F">Brown</span>          |                     |                                      |

For light mode palettes, the sequence of 00–07 should be reversed (light to dark, not dark to light).
Note that not all colours are currently used in the theme.

You you use these colours directly in content using the convenience classes `.baseXX` and `.bg-baseXX`.
For example:

```html
<span class="base0A">Yellow text</span>
<mark class="base0D">Text highlighted in green</mark>
```

## Favicon

dimolto will automatically use favicons placed in the `static/` directory.
The following files will be detected and included in your site's `<head>` section:

* `favicon.ico`
* `favicon-16x16.png`
* `favicon-32x32.png`
* `apple-touch-icon.png`
* `site.webmanifest`

You can generate these from an image or emoji using [favicon.io](https://favicon.io/) or a similar service.
They must be placed directly under your site's `static/` directory, i.e. not in in a subdirectory or `themes/dimolto/static/`.

## Acknowledgements

The 'cooked rice' emoji used as a favicon for the example site was created by the [Twemoji project](https://twemoji.twitter.com/) and is licensed under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
