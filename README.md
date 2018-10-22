# percollate

Percollate is a command-line tool to turn web pages into beautifully formatted PDFs. See [How it works](#how-it-works).

<img alt="Example Output" src="https://raw.githubusercontent.com/danburzo/percollate/master/img/dimensions-of-colour.png">

_Example spread from the generated PDF of [a chapter in Dimensions of Colour](http://www.huevaluechroma.com/072.php); rendered here in black & white for a smaller image file size._

## Table of Contents

-   [Installation](#installation)
-   [Usage](#usage)
    -   [Available commands](#available-commands)
    -   [Available options](#available-options)
-   [Examples](#examples)
    -   [Basic PDF Generation](#basic-pdf-generation)
    -   [The `--css` option](#the---css-option)
    -   [Using a custom HTML template](#using-a-custom-html-template)
    -   [Using a custom CSS stylesheet](#using-a-custom-css-stylesheet)
    -   [Customizing the page header / footer](#customizing-the-page-header--footer)
-   [How it works](#how-it-works)
-   [Limitations](#limitations)
-   [Troubleshooting](#troubleshooting)
-   [Contributing](#contributing)
-   [See also](#see-also)

## Installation

> 💡 `percollate` needs Node.js version 8.6.0 or later, as it uses new(ish) JavaScript syntax. If you get _SyntaxError: Unexpected token_ errors, check your Node version with `node --version`.

You can install `percollate` globally:

```bash
# using npm
npm install -g percollate

# using yarn
yarn global add percollate
```

To keep the package up-to-date, you can run:

```bash
# using npm, upgrading is the same command as installing
npm install -g percollate

# yarn has a separate command
yarn global upgrade --latest percollate
```

## Usage

> 💡 Run `percollate --help` for a list of available commands. For a particular command, `percollate <command> --help` lists all available options.

### Available commands

| Command           | What it does                                                             |
| ----------------- | ------------------------------------------------------------------------ |
| `percollate pdf`  | Bundles one or more web pages into a PDF                                 |
| `percollate epub` | _Not implemented [yet](https://github.com/danburzo/percollate/issues/8)_ |
| `percollate html` | _Not implemented [yet](https://github.com/danburzo/percollate/issues/7)_ |

### Available options

The `pdf`, `epub`, and `html` commands have these options:

| Option         | What it does                                                                                                   |
| -------------- | -------------------------------------------------------------------------------------------------------------- |
| `-o, --output` | The path of the resulting bundle; when ommited, we derive the output file name from the title of the web page. |
| `--individual` | Export each web page as an individual file.                                                                    |
| `--template`   | Path to a custom HTML template                                                                                 |
| `--style`      | Path to a custom CSS                                                                                           |
| `--css`        | Additional CSS styles you can pass from the command-line to override the default/custom stylesheet styles      |

## Examples

### Basic PDF generation

To transform a single web page to PDF:

```bash
percollate pdf --output some.pdf https://example.com
```

To bundle _several_ web pages into a single PDF, specify them as separate arguments to the command:

```bash
percollate pdf --output some.pdf https://example.com/page1 https://example.com/page2
```

You can use common Unix commands and keep the list of URLs in a newline-delimited text file:

```bash
cat urls.txt | xargs percollate pdf --output some.pdf
```

To transform several web pages into individual PDF files at once, use the `--individual` flag:

```bash
percollate pdf --individual https://example.com/page1 https://example.com/page2
```

### The `--css` option

The `--css` option lets you pass a small snippet of CSS to percollate. Here are some common use-cases:

#### Custom page size / margins

The default page size is A5 (portrait). You can use the `--css` option to override it using [any supported CSS `size`](https://www.w3.org/TR/css3-page/#page-size):

```bash
percollate pdf --css "@page { size: A3 landscape }" http://example.com
```

Similarly, you can define:

-   custom margins, e.g. `@page { margin: 0 }`
-   the base font size: `html { font-size: 10pt }`

#### Remove the appended HREFs from hyperlinks

The idea with percollate is to make PDFs that can be printed without losing where the hyperlinks point to. However, for some link-heavy pages, the appended HREFs can become bothersome. You can remove them using:

```bash
percollate pdf --css "a:after { display: none }" http://example.com
```

### Using a custom HTML template

> ⚠️ TODO add example here

### Using a custom CSS stylesheet

> ⚠️ TODO add example here

### Customizing the page header / footer

> ⚠️ TODO add example here

## How it works

1. Fetch the page(s) using [`got`](https://github.com/sindresorhus/got)
2. [Enhance](./src/enhancements.js) the DOM using [`jsdom`](https://github.com/jsdom/jsdom)
3. Pass the DOM through [`mozilla/readability`](https://github.com/mozilla/readability) to strip unnecessary elements
4. Apply the [HTML template](./templates/default.html) and the [print stylesheet](./templates/default.css) to the resulting HTML
5. Use [`puppeteer`](https://github.com/GoogleChrome/puppeteer) to generate a PDF from the page

## Limitations

Percollate inherits the limitations of two of its main components, Readability and Puppeteer (headless Chrome).

The imperative approach Readability takes will not be perfect in each case, especially on HTML pages with atypical markup; you may occasionally notice that it either leaves in superfluous content, or that it strips out parts of the content. You can confirm the problem against [Firefox's Reader View](https://blog.mozilla.org/firefox/reader-view/). In this case, consider [filing an issue on `mozilla/readability`](https://github.com/mozilla/readability/issues).

Using a browser to generate the PDF is a double-edged sword. On the one hand, you get excellent support for web platform features. On the other hand, [print CSS](https://www.smashingmagazine.com/2018/05/print-stylesheets-in-2018/) as defined by W3C specifications is only partially implemented, and it seems unlikely that support will be improved any time soon. However, even with modest print support, I think Chrome is the best (free) tool for the job.

## Troubleshooting

On some Linux machines you'll need to [install a few more Chrome dependencies](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-doesnt-launch) before `percollate` works correctly. (_Thanks to @ptica for [sorting it out](https://github.com/danburzo/percollate/issues/19#issuecomment-428496041)_)

The `percollate pdf` command supports the `--no-sandbox` Puppeteer flag, but make sure you're [aware of the implications](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-fails-due-to-sandbox-issues) before disabling the sandbox.

## Contributing

Contributions of all kinds are welcome! See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

## See also

Here are some other projects to check out if you're interested in building books using the browser:

-   [weasyprint](https://github.com/Kozea/WeasyPrint) ([website](https://weasyprint.org/))
-   [bindery.js](https://github.com/evnbr/bindery) ([website](https://evanbrooks.info/bindery/))
-   [HummusJS](https://github.com/galkahana/HummusJS)
-   [Editoria](https://gitlab.coko.foundation/editoria/editoria) ([website](https://editoria.pub/))
-   [pagedjs](https://gitlab.pagedmedia.org/tools/pagedjs) ([article](https://www.pagedmedia.org/pagedjs-sneak-peeks/))
-   [Mercury](https://mercury.postlight.com/)
