# [<img src="https://khan.github.io/KaTeX/katex-logo.svg" width="130" alt="KaTeX">](https://khan.github.io/KaTeX/)
[![Build Status](https://travis-ci.org/Khan/KaTeX.svg?branch=master)](https://travis-ci.org/Khan/KaTeX)
[![codecov](https://codecov.io/gh/Khan/KaTeX/branch/master/graph/badge.svg)](https://codecov.io/gh/Khan/KaTeX)
[![Join the chat at https://gitter.im/Khan/KaTeX](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/Khan/KaTeX?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
![](https://img.badgesize.io/Khan/KaTeX/v0.10.0-alpha/dist/katex.min.js?compression=gzip)

KaTeX is a fast, easy-to-use JavaScript library for TeX math rendering on the web.

 * **Fast:** KaTeX renders its math synchronously and doesn't need to reflow the page. See how it compares to a competitor in [this speed test](http://www.intmath.com/cg5/katex-mathjax-comparison.php).
 * **Print quality:** KaTeX’s layout is based on Donald Knuth’s TeX, the gold standard for math typesetting.
 * **Self contained:** KaTeX has no dependencies and can easily be bundled with your website resources.
 * **Server side rendering:** KaTeX produces the same output regardless of browser or environment, so you can pre-render expressions using Node.js and send them as plain HTML.

KaTeX supports all major browsers, including Chrome, Safari, Firefox, Opera, Edge, and IE 9 - IE 11. More information can be found on the [list of supported commands](https://khan.github.io/KaTeX/function-support.html) and on the [wiki](https://github.com/khan/katex/wiki).

## Usage

You can [download KaTeX](https://github.com/khan/katex/releases) and host it on your server or include the `katex.min.js` and `katex.min.css` files on your page directly from a CDN:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.0-alpha/dist/katex.min.css" integrity="sha384-BTL0nVi8DnMrNdMQZG1Ww6yasK9ZGnUxL1ZWukXQ7fygA1py52yPp9W4wrR00VML" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/katex@0.10.0-alpha/dist/katex.min.js" integrity="sha384-y6SGsNt7yZECc4Pf86XmQhC4hG2wxL6Upkt9N1efhFxfh6wlxBH0mJiTE8XYclC1" crossorigin="anonymous"></script>
```

#### In-browser rendering

Call `katex.render` with a TeX expression and a DOM element to render into:

```js
katex.render("c = \\pm\\sqrt{a^2 + b^2}", element);
```

To avoid escaping the backslash (double backslash), you can use
[`String.raw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/raw)
(but beware that `${`, `\u` and `\x` may still need escaping):
```js
katex.render(String.raw`c = \pm\sqrt{a^2 + b^2}`, element);
```

If KaTeX can't parse the expression, it throws a `katex.ParseError` error.

#### Server side rendering or rendering to a string

To generate HTML on the server or to generate an HTML string of the rendered math, you can use `katex.renderToString`:

```js
var html = katex.renderToString("c = \\pm\\sqrt{a^2 + b^2}");
// '<span class="katex">...</span>'
```

Make sure to include the CSS and font files, but there is no need to include the JavaScript. Like `render`, `renderToString` throws if it can't parse the expression.

#### Security

Any HTML generated by KaTeX *should* be safe from `<script>` or other code
injection attacks.
(See `maxSize` below for preventing large width/height visual affronts,
and see `maxExpand` below for preventing infinite macro loop attacks.)
Of course, it is always a good idea to sanitize the HTML, though you will need
a rather generous whitelist (including some of SVG and MathML) to support 
all of KaTeX.

#### Handling errors

If KaTeX encounters an error (invalid or unsupported LaTeX) and `throwOnError`
hasn't been set to `false`, then it will throw an exception of type
`katex.ParseError`.  The message in this error includes some of the LaTeX
source code, so needs to be escaped if you want to render it to HTML.
In particular, you should convert `&`, `<`, `>` characters to
`&amp;`, `&lt;`, `&gt;` (e.g., using `_.escape`)
before including either LaTeX source code or exception messages in your
HTML/DOM.  (Failure to escape in this way makes a `<script>` injection
attack possible if your LaTeX source is untrusted.)
Alternatively, you can set `throwOnError` to `false` to use built-in behavior
of rendering the LaTeX source code with hover text stating the error.

#### Rendering options

You can provide an object of options as the last argument to `katex.render` and `katex.renderToString`. Available options are:

- `displayMode`: `boolean`. If `true` the math will be rendered in display mode, which will put the math in display style (so `\int` and `\sum` are large, for example), and will center the math on the page on its own line. If `false` the math will be rendered in inline mode. (default: `false`)
- `throwOnError`: `boolean`. If `true` (the default), KaTeX will throw a `ParseError` when it encounters an unsupported command or invalid LaTeX. If `false`, KaTeX will render unsupported commands as text, and render invalid LaTeX as its source code with hover text giving the error, in the color given by `errorColor`.
- `errorColor`: `string`. A color string given in the format `"#XXX"` or `"#XXXXXX"`. This option determines the color that unsupported commands and invalid LaTeX are rendered in when `throwOnError` is set to `false`. (default: `#cc0000`)
- `macros`: `object`. A collection of custom macros. Each macro is a property with a name like `\name` (written `"\\name"` in JavaScript) which maps to a string that describes the expansion of the macro. Single-character keys can also be included in which case the character will be redefined as the given macro (similar to TeX active characters).
- `colorIsTextColor`: `boolean`. If `true`, `\color` will work like LaTeX's `\textcolor`, and take two arguments (e.g., `\color{blue}{hello}`), which restores the old behavior of KaTeX (pre-0.8.0). If `false` (the default), `\color` will work like LaTeX's `\color`, and take one argument (e.g., `\color{blue}hello`).  In both cases, `\textcolor` works as in LaTeX (e.g., `\textcolor{blue}{hello}`).
- `maxSize`: `number`. All user-specified sizes, e.g. in `\rule{500em}{500em}`, will be capped to `maxSize` ems. If set to `Infinity` (the default), users can make elements and spaces arbitrarily large.
- `maxExpand`: `number`. Limit the number of macro expansions to the specified number, to prevent e.g. infinite macro loops. If set to `Infinity` (the default), the macro expander will try to fully expand as in LaTeX.
- `strict`: `boolean` or `string` or `function` (default: `"warn"`). If `false` or `"ignore`", allow features that make writing LaTeX convenient but are not actually supported by (Xe)LaTeX (similar to MathJax). If `true` or `"error"` (LaTeX faithfulness mode), throw an error for any such transgressions. If `"warn"` (the default), warn about such behavior via `console.warn`. Provide a custom function `handler(errorCode, errorMsg, token)` to customize behavior depending on the type of transgression (summarized by the string code `errorCode` and detailed in `errorMsg`); this function can also return `"ignore"`, `"error"`, or `"warn"` to use a built-in behavior.  A list of such features and their `errorCode`s:
  - `"unknownSymbol"`: Use of unknown Unicode symbol, which will likely also
    lead to warnings about missing character metrics, and layouts may be
    incorrect (especially in terms of vertical heights).
  - `"unicodeTextInMathMode"`: Use of Unicode text characters in math mode.
  - `"mathVsTextUnits"`: Mismatch of math vs. text commands and units/mode.
  A second category of `errorCode`s never throw errors, but their strictness
  affects the behavior of KaTeX:
  - `"newLineInDisplayMode"`: Use of `\\` or `\newline` in display mode
    (outside an array/tabular environment).  In strict mode, no line break
    results, as in LaTeX.

For example:

```js
katex.render("c = \\pm\\sqrt{a^2 + b^2}\\in\\RR", element, {
  displayMode: true,
  macros: {
    "\\RR": "\\mathbb{R}"
  }
});
```

#### Automatic rendering of math on a page

Math on the page can be automatically rendered using the auto-render extension. See [the Auto-render README](contrib/auto-render/README.md) for more information.

#### Font size and lengths

By default, KaTeX math is rendered in a 1.21× larger font than the surrounding
context, which makes super- and subscripts easier to read. You can control
this using CSS, for example:

```css
.katex { font-size: 1.1em; }
```

KaTeX supports all TeX units, including absolute units like `cm` and `in`.
Absolute units are currently scaled relative to the default TeX font size of
10pt, so that `\kern1cm` produces the same results as `\kern2.845275em`.
As a result, relative and absolute units are both uniformly scaled relative
to LaTeX with a 10pt font; for example, the rectangle `\rule{1cm}{1em}` has
the same aspect ratio in KaTeX as in LaTeX.  However, because most browsers
default to a larger font size, this typically means that a 1cm kern in KaTeX
will appear larger than 1cm in browser units.

### Common Issues
- Many Markdown preprocessors, such as the one that Jekyll and GitHub Pages use,
  have a "smart quotes" feature.  This changes `'` to `’` which is an issue for
  math containing primes, e.g. `f'`.  This can be worked around by defining a
  single character macro which changes them back, e.g. `{"’", "'"}`.
- KaTeX follows LaTeX's rendering of `aligned` and `matrix` environments unlike
  MathJax.  When displaying fractions one above another in these vertical
  layouts there may not be enough space between rows for people who are used to
  MathJax's rendering.  The distance between rows can be adjusted by using
  `\\[0.1em]` instead of the standard line separator distance.
- KaTeX does not support the `align` environment because LaTeX doesn't support
  `align` in math mode.  The `aligned` environment offers the same functionality
  but in math mode, so use that instead or define a macro that maps `align` to
  `aligned`.
- MathJax defines `\color` to be like `\textcolor` by default; set KaTeX's
  `colorIsTextColor` option to `true` for this behavior.  KaTeX's default
  behavior matches MathJax with its `color.js` extension enabled.

## Libraries

### Angular2+
- [ng-katex](https://github.com/garciparedes/ng-katex) Angular module to write beautiful math expressions with TeX syntax boosted by KaTeX library

### React
- [react-latex](https://github.com/zzish/react-latex) React component to render latex strings, based on KaTeX
- [react-katex](https://github.com/talyssonoc/react-katex) React components that use KaTeX to typeset math expressions

### Ruby

- [katex-ruby](https://github.com/glebm/katex-ruby) Provides server-side rendering and integration with popular Ruby web frameworks (Rails, Hanami, and anything that uses Sprockets).


## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

KaTeX is licensed under the [MIT License](http://opensource.org/licenses/MIT).
