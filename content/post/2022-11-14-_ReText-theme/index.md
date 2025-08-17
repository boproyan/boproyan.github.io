+++
title = 'ReText'
date = 2022-11-14 00:00:00 +0100
categories = markdown
+++
![](retext_logo.png)  
*[ReText](https://github.com/retext-project) est un éditeur simple mais puissant pour les langages de balisage Markdown et reStructuredText.*

## ReText/Archlinux

### Installation

Retext est disponible dans les dépôts

    yay -S retext python-pyqt6-webengine

**python-pyqt6-webengine** : moteur de prévisualisation plus puissant avec prise en charge de JavaScript

<u>Alternative</u> : [Installation via python pip](https://github.com/retext-project/retext/wiki/Installing-ReText)

### Configuration

La configuration est dans le dossier utilisateur `$HOME/.config/ReText\ project/` et contient 3 fichiers

1. **ReText.conf** : Fichier de configuration de base
2. **retext.qss** : Pour la configuration graphique de l'éditeur
3. **markdown-taupe.css** : Thème de la prévisualisation

**ReText.conf**

```
[General]
appStyleSheet=/home/yann/.config/ReText project/retext.qss
recentFileList=
styleSheet=/home/yann/.config/ReText project/markdown-taupe.css
useWebEngine=true
useWebKit=true
```

**retext.qss**

```css
QTextEdit {
  color: black;
  background-color: white;
}
```

**markdown-taupe.css**

```css
body {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 15px;
  line-height: 1.3;
  color: #f6e6cc;
  width: 700px;
  margin: auto;
  background: #27221a;
  position: relative;
  padding: 0 30px;
}
body>:first-child
{
  margin-top:0!important;
}
img {
  max-width: 100%;
}
table {
  width: 100%;
  border-collapse: collapse;
}
th {
  background-color: rgba(0, 0, 0, 0.3);
}
table, th, td {
  padding: 5px;
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 0.4em;
  -moz-border-radius: 0.4em;
  -webkit-border-radius: 0.4em;
}
tr:nth-child(even) {
  background-color: rgba(0, 0, 0, 0.3);
}
p, ul, ol, dl, table, pre {
  margin-bottom: 1em;
}
ul {
  margin-left: 20px;
}
a {
  text-decoration: none;
  cursor: pointer;
  color: #ba832c;
  font-weight: bold;
}
a:focus {
  outline: 1px dotted;
}
a:visited {}
a:hover, a:focus {
  color: #d3a459;
  text-decoration: none;
}
a *, button * {
  cursor: pointer;
}
hr {
  display: none;
}
small {
  font-size: 90%;
}
input, select, button, textarea, option {
  font-family: Arial, "Lucida Grande", "Lucida Sans Unicode", Arial, Verdana, sans-serif;
  font-size: 100%;
}
button, label, select, option, input[type=submit] {
  cursor: pointer;
}
sup {
  font-size: 80%;
  line-height: 1;
  vertical-align: super;
}
h1, h2, h3, h4, h5, h6 {
  line-height: 1.1;
  font-family: Baskerville, "Goudy Old Style", "Palatino", "Book Antiqua", serif;
}
h1 {
  font-size: 24pt;
  margin: 1em 0 0.1em;
}
h2 {
  font-size: 22pt;
}
h3 {
  font-size: 20pt;
}
h4 {
  font-size: 18pt;
}
h5 {
  font-size: 16pt;
}
h6 {
  font-size: 14pt;
}
h1 a, h1 a:hover {
  color: #d7af72;
  font-weight: normal;
  text-decoration: none;
}
::selection {
  background: #745626;
}
::-moz-selection {
  background: #745626;
}
pre {
  background: #1B1812;
  color: #fff;
  padding: 8px 10px;
  overflow-x: hidden;
}
pre code {
  font-size: 10pt;
}
```

## Configurations (en)

ReText stores all of its configuration in a text file: `~/.config/ReText project/ReText.conf`.  
See the [possible configuration options](https://github.com/retext-project/retext/blob/master/configuration.md#color-scheme-setting)

You can change

* the theme of the window, using [Qt stylesheets](http://doc.qt.io/archives/qt-4.8/stylesheet.html).   
  Add `appStyleSheet=/home/YOUR_USERNAME/.config/ReText project/YOUR_FILE.qss` to the configuration file

  <ins>Example (retext.qss)</ins>:

  ```
  QTextEdit {
    color: black;
    background-color: white;
  }
  ```

* the color scheme of the text editor, using a `[ColorScheme]` section in the configuration file

  <ins>Example (ReText.conf)</ins>:

  ```
  [ColorScheme]
  htmlTags=green
  htmlSymbols=#ff8800
  htmlComments=#abc
  ```

* the color scheme of the preview, using CSS.  
  Add `styleSheet=/home/YOUR_USERNAME/.config/ReText project/YOUR_FILE.css`  
  and `useWebKit=true` to the configuration file

  <ins>Example (preview.css)</ins>:

  ```
  body {
    color: black;
    background-color: white;
  }
  ```

Of course, don't forget to put the files in that directory, then restart ReText to see the changes.

### Configuration Options (en) 

ReText stores all of its configuration in a text file. A path to that
file is printed to stdout during ReText startup.

Configuration options that you can set to improve your experience:

option name                    | type      | description
-----------                    | ----      | -----------
`appStyleSheet`                | file path | file containing a Qt stylesheet file
`autoSave`                     | boolean   | whether to automatically save documents (default: false)
`defaultCodec`                 | string    | name of encoding to use by default (default: use system encoding)
`defaultMarkup`                | string    | name of markup to use for unknown files
`detectEncoding`               | boolean   | whether to automatically detect files encoding; needs chardet package (default: true)
`documentStatsEnabled`         | boolean   | whether to show document stats (word count, character count) (default: false)
`editorFont`                   | string    | font to use for editor: name (default: `monospace`)
`editorFontSize`               | integer   | font to use for editor: font size
`font`                         | string    | font to use for previews: name
`fontSize`                     | integer   | font to use for previews: font size
`handleWebLinks`               | boolean   | whether to use ReText preview area to open external links (default: false)
`hideToolBar`                  | boolean   | whether to hide the toolbars from the UI (default: false)
`highlightCurrentLine`         | boolean   | whether to highlight current line in editor (default: false)
`iconTheme`                    | string    | name of the system icon theme to use (see below)
`lineNumbersEnabled`           | boolean   | whether to show column with line numbers in editor (default: false)
`livePreviewByDefault`         | boolean   | whether new tabs and windows should open in live preview mode (default: false)
`markdownDefaultFileExtension` | string    | default file extension for Markdown files (default: `.mkd`)
`openLastFilesOnStartup`       | boolean   | whether to automatically open last documents on startup (default: false)
`paperSize`                    | string    | name of default page size to use for print and export (e.g. A4, Letter)
`pygmentsStyle`                | string    | name of Pygments syntax highlighting style to use (default: `default`)
`relativeLineNumbers`          | boolean   | whether to show line numbers as relative from the current line (default: false)
`restDefaultFileExtension`     | string    | default file extension for reStructuredText files (default: `.rst`)
`rightMargin`                  | integer   | enable drawing of vertical line on defined position (or 0 to disable)
`saveWindowGeometry`           | boolean   | whether to restore window geometry from previous session (default: false)
`spellCheck`                   | boolean   | whether to enable spell checking
`spellCheckLocale`             | string    | short name of spell check locale to use (examples: `en_US`, `ru`, `pt_BR`)
`styleSheet`                   | file path | CSS file to use in preview area
`syncScroll`                   | boolean   | whether to enable synchronized scrolling for Markdown (default: true)
`tabBarAutoHide`               | boolean   | whether to hide the tabs bar when only one tab is open (default: false)
`tabInsertsSpaces`             | boolean   | whether Tab key should insert spaces instead of tabs (default: true)
`tabWidth`                     | integer   | the width of tab character (default: 4)
`uiLanguage`                   | string    | short name of locale to use for interface (examples: `en_US, `ru, `pt_BR`)
`useFakeVim`                   | boolean   | whether to use the FakeVim editor, if available (default: false)
`useWebEngine`                 | boolean   | whether to use the WebEngine (Chromium) as HTML previewer (default: false)
`useWebKit`                    | boolean   | whether to use the WebKit instead of QTextEdit as HTML previewer (default: false)

If the type is 'file path', then the value should be an absolute path
to a file.

These options can be set internally by ReText and should never be set
manually: `recentFileList`, `lastFileList`, `lastTabIndex` and `windowGeometry`.

Icon themes
===========

If ReText starts and does not show icons, go to Preferences dialog
and fill the "icon theme" field with the icon theme being used.

By default Qt (the toolkit used by ReText) can correctly detect icon
theme only on KDE and on a fixed list of Gtk+-based environments (when
the gtk platformtheme is used).

If you don't know name of your icon theme, look at the names of
subdirectories in `/usr/share/icons/` directory.

Color scheme setting
====================

It is possible to configure ReText highlighter to use custom colors set,
by providing these colors in a separate section in the configuration file.

The example of such section is:

    [ColorScheme]
    htmlTags=green
    htmlSymbols=#ff8800
    htmlComments=#abc

Color names for the text editor:

color name             | main setting           | description
----------             | ------------           | -----------
`marginLine`           | `rightMargin`          | the vertical right margin line
`currentLineHighlight` | `highlightCurrentLine` | highlighting of the text line being edited
`infoArea`             |                        | the info box in the bottom-right corner
`lineNumberArea`       | `lineNumbersEnabled`   | the line numbers area background
`lineNumberAreaText`   | `lineNumbersEnabled`   | the line numbers area foreground

Color names for the highlighter:

color name        | description
----------        | -----------
`htmlTags`        | HTML tags, e.g. `<foo>`
`htmlStrings`     | string properties inside HTML tags, e.g. `"baz"` inside `<foo bar="baz">`
`htmlSymbols`     | HTML symbols, e.g. `&bar;`
`htmlComments`    | HTML comments, e.g. `<!-- comment -->`
`markdownLinks`   | Markdown links and images text, e.g. `foo` inside `[foo](http://example.com)`
`blockquotes`     | blockquotes, e.g. `> quote` in Markdown
`codeSpans`       | code spans, e.g. `` `code` `` in Markdown
`restDirectives`  | reStructuredText directives, e.g. `.. math::`
`restRoles`       | reStructuredText roles, e.g. `:math:`
`whitespaceOnEnd` | whitespace at line endings

## Prebuild Color Schemes (en)

| Theme                              | CSS | QSS | Conf | Preview | Source
|---                                 |---  |---  |---   |---      |---
| [Default](Default)                 | ⨯   | ⨯   | ⨯    | ![](Default/preview.png?s=150)     | [retext-project](https://github.com/retext-project/retext/blob/master/ReText/highlighter.py)
| [Github](Github)                   | ✓   | ⨯   | ✓    | ![](Github/preview.png?s=150)      | [EndangeredMassa](https://gist.github.com/EndangeredMassa/8849279/)
| [QDarkStyle](QDarkStyle)           | ✓   | ✓   | ✓    | ![](QDarkStyle/preview.png?s=150)  | [snailhome](http://snailhome.github.io/note/do_someting_after_install_retext.html)
| [Monokai](Monokai)                 | ⨯   | ✓   | ✓    | ![](Monokai/preview.png?s=150)     | [geekthis](https://geekthis.net/post/retext-change-color-scheme/)
| [Monokai Minimal](Monokai-Minimal) | ⨯   | ✓   | ✓    | ![](Monokai-Minimal/preview.png?s=150)     | [geekthis](https://geekthis.net/post/retext-change-color-scheme/)
| [Solarized](Solarized)             | ⨯   | ✓   | ✓    | ![](Solarized/preview.png?s=150)     | [geekthis](https://geekthis.net/post/retext-change-color-scheme/)
| [Solarized Light](Solarized-Light) | ⨯   | ✓   | ✓    | ![](Solarized-Light/preview.png?s=150)     | [geekthis](https://geekthis.net/post/retext-change-color-scheme/)
| [Breeze Dark](Breeze-Dark)         | ✓   | ✓   | ✓    | ![](Breeze-Dark/preview.png?s=150) | [cognifloyd](https://github.com/retext-project/retext/issues/255#issuecomment-281764875)
| [Markdown Taupe](Markdown-Taupe)   | ✓   | ⨯   | ⨯    | ![](Markdown-Taupe/preview.png?s=150) | [jasonm23](https://github.com/jasonm23/markdown-css-themes/blob/gh-pages/screen.css)
| [Houdini](Houdini)                 | ✓   | ✓   | ✓    | ![](Houdini/preview.png?s=150) | [shotgunsoftware](https://github.com/shotgunsoftware/tk-houdini/blob/master/style.qss)
| [Markdown](Markdown)               | ✓   | ⨯   | ⨯    | ![](Markdown/preview.png?s=150) | [elclanrs](https://gist.github.com/elclanrs/f49c0a8cf3838c1ffeb3)

