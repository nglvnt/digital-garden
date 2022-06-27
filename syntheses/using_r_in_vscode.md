# Using R in VSCode

## Introduction

## Setting up R in VSCode

0) install [VSCode](https://code.visualstudio.com/) and [R](https://cloud.r-project.org/) (version 3.4.0 is needed at least)
1) install the [languageserver](https://cran.r-project.org/package=languageserver) package in R
2) install the [httpgd](https://cran.r-project.org/web/packages/httpgd/index.html) package in R
3) install the [vscDebugger](https://github.com/ManuelHentschel/vscDebugger) package from GitHub in R
4) install the [R extension for VSCode](https://marketplace.visualstudio.com/items?itemName=REditorSupport.r)
5) install the [R Debugger](https://marketplace.visualstudio.com/items?itemName=RDebugger.r-debugger) extension for VSCode
6) install the [radian](https://github.com/randy3k/radian) console

## Using radian as the console

In order to use the radian console, in the settings change the "r.rterm.linux" to the directory where radian is (find it with "which radian").

One could also alias it as `r`, hence `R` would still open the traditional console.

```bash
alias r="radian"
```

## Recommended VSCode settings

* "r.alwaysUseActiveTerminal": true
* "r.bracketedPaste": true
* "r.sessionWatcher": true

## Handling graphics

### Built-in

### The httpgd package

freetype2 library is required for the systemfonts R package

```bash
apt install libfontconfig1-dev
```

Add the following to `.Rprofile`

```r
if (interactive() && Sys.getenv("TERM_PROGRAM") == "vscode") {
  if ("httpgd" %in% .packages(all.available = TRUE)) {
    options(vsc.plot = FALSE)
    options(device = function(...) {
      httpgd::hgd(silent = TRUE)
      .vsc.browser(httpgd::hgd_url(), viewer = "Beside")
    })
  }
}
```

It is also suggested to add the following keybinding if the plot window is closed accidentally (running the command that produces the plot is not enough to re-open it).

```json
{
  "key": "ctrl+alt+p",
  "command": "r.runCommand",
  "when": "editorTextFocus && editorLangId == 'r'",
  "args": ".vsc.browser(httpgd::hgd_url(), viewer = \"Beside\")"
}
```

## Editor layout and appearance

## Start coding

* new file, select R language mode
* running R code: sending code to the R terminal. Create new R terminal, Ctrl+Enter to run selected code, Ctrl+Shift+S to source the whole file.

More info: <https://github.com/REditorSupport/vscode-R/wiki/Interacting-with-R-terminals>

## Features

R language service: <https://github.com/REditorSupport/vscode-R/wiki/R-Language-Service>

From VSCode doc on R:

* Syntax highlighting
* Code completion
* Linting
* Formatting
* Rename symbols
* Find references
* Interacting with R terminals
* Viewing data, plots, workspace variables, help pages
* Managing packages
* Working with RMarkdown documents

From Kun Ren:

* Completion
* Scope completion
* Document highlight
* Diagnostics
* Formatting
* Function signature
* Hover
* Definition
* Document link
* Document color
* Document/workspace symbols
* Code sections
* Folding ranges
* Find references
* Rename symbol
* Selection range
* Call hierarchy

## Code linting and styling with lintr and styler

styler is slow for large scripts, hence "editor.formatOnSave": false

## How debugging works

<https://github.com/ManuelHentschel/vscDebugger/>

<https://github.com/ManuelHentschel/VSCode-R-Debugger>

## Multi-session

## Initializing renv in VSCode

## Things needed

* **VSCode**
* **R**
* **languageserver R package**: [CRAN](https://cran.r-project.org/web/packages/languageserver/index.html) [GitHub](https://github.com/REditorSupport/languageserver/)
* **httpgd R package**: [website](https://nx10.github.io/httpgd/) [CRAN](https://cran.r-project.org/package=httpgd) [GitHub](https://github.com/nx10/httpgd)
* **vscDebugger package**: [website](https://manuelhentschel.github.io/vscDebugger/) [GitHub](https://github.com/ManuelHentschel/vscDebugger/)
* **R extension for VSCode**: [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=REditorSupport.r) [GitHub](https://github.com/REditorSupport/vscode-R) [Wiki](https://github.com/REditorSupport/vscode-R/wiki)
* **R Debugger extension for VSCode** [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=RDebugger.r-debugger) [GitHub](https://github.com/ManuelHentschel/VSCode-R-Debugger)
* **radian console**: [GitHub](https://github.com/randy3k/radian)

## Resources

* <https://code.visualstudio.com/docs/languages/r>
* <https://github.com/REditorSupport/vscode-R/wiki>
* <https://renkun.me/2019/12/11/writing-r-in-vscode-a-fresh-start/>
* <https://renkun.me/2019/12/26/writing-r-in-vscode-interacting-with-an-r-session/>
* <https://renkun.me/2020/06/16/using-httpgd-in-vscode-a-web-based-svg-graphics-device/>
* <https://renkun.me/2022/03/06/why-i-switched-from-rstudio-to-vs-code/>
* <https://renkun.me/2022/03/06/my-recommendations-of-vs-code-extensions-for-r/>
* <https://github.com/renkun-ken/using-rstats-in-vscode>
  * <https://github.com/renkun-ken/using-rstats-in-vscode/blob/main/Using%20R%20in%20VSCode%20-%20Ren%20Kun%20-%202021.pdf>
  * <https://www.youtube.com/watch?v=9xXBDU2z_8Y>
* <https://luongbui.com/r-and-radian-on-macos-and-vscode/>
* about radian: <https://fromthebottomoftheheap.net/2019/06/18/radian-console-for-r/>
* <https://towardsdatascience.com/vscode-vs-rstudio-worth-the-switch-7a4415fc3275>
* <https://www.infoworld.com/article/3625488/how-to-run-r-in-visual-studio-code.html>
