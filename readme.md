<h1 align="center">
  <img src="https://raw.githubusercontent.com/micromark/micromark/2e476c9/logo.svg?sanitize=true" alt="micromark" />
</h1>

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][bundle-size-badge]][bundle-size]
[![Sponsors][sponsors-badge]][opencollective]
[![Backers][backers-badge]][opencollective]
[![Chat][chat-badge]][chat]

The smallest CommonMark compliant markdown parser.
With positional info and concrete tokens.

## Feature highlights

<!-- Note: this section has to be in sync with the `micromark` readme. -->

* [x] **[compliant][commonmark]** (100% to CommonMark)
* [x] **[extensions][]** (100% [GFM][], 100% [MDX.js][mdxjs], [directives][],
  [frontmatter][], [math][])
* [x] **[safe][security]** (by default)
* [x] **[robust][test]** (±2k tests, 100% coverage, fuzz testing)
* [x] **[small][size-debug]** (smallest CM parser at ±14kb)

## Contents

* [When should I use this?](#when-should-i-use-this)
* [What is this?](#what-is-this)
* [Install](#install)
* [Use](#use)
* [API](#api)
* [Extensions](#extensions)
  * [List of extensions](#list-of-extensions)
  * [`SyntaxExtension`](#syntaxextension)
  * [`HtmlExtension`](#htmlextension)
  * [Extending markdown](#extending-markdown)
  * [Creating a micromark extension](#creating-a-micromark-extension)
* [Architecture](#architecture)
  * [Overview](#overview)
  * [Preprocess](#preprocess)
  * [Parse](#parse)
  * [Postprocess](#postprocess)
  * [Compile](#compile)
* [Examples](#examples)
  * [GitHub flavored markdown (GFM)](#github-flavored-markdown-gfm)
  * [Math](#math)
  * [Syntax tree](#syntax-tree)
* [Markdown](#markdown)
  * [CommonMark](#commonmark)
  * [Grammar](#grammar)
* [Project](#project)
  * [Comparison](#comparison)
  * [Test](#test)
  * [Size & debug](#size--debug)
  * [Version](#version)
  * [Security](#security)
  * [Contribute](#contribute)
  * [Sponsor](#sponsor)
  * [Origin story](#origin-story)
  * [License](#license)

## When should I use this?

<!-- Note: this section has to be in sync with the `micromark` readme. -->

* If you *just* want to turn markdown into HTML (with maybe a few extensions)
* If you want to do *really complex things* with markdown

See [§ Comparison][comparison] for more info

## What is this?

<!-- Note: this section has to be in sync with the `micromark` readme. -->

`micromark` is an open source markdown parser written in JavaScript.
It’s implemented as a state machine that emits concrete tokens, so that every
byte is accounted for, with positional info.
It then compiles those tokens directly to HTML, but other tools can take the
data and for example build an AST which is easier to work with
([`mdast-util-to-markdown`][mdast-util-to-markdown]).

While most markdown parsers work towards compliancy with CommonMark (or GFM),
this project goes further by following how the reference parsers (`cmark`,
`cmark-gfm`) work, which is confirmed with thousands of extra tests.

Other than CommonMark and GFM, micromark also supports common extensions to
markdown such as MDX, math, and frontmatter.

These npm packages have a sibling project in Rust:
[`markdown-rs`][markdown-rs].

* to learn markdown, see this [cheatsheet and tutorial][cheat]
* for more about us, see [`unifiedjs.com`][site]
* for questions, see [Discussions][chat]
* to help, see [contribute][] and [sponsor][] below

## Install

<!-- Note: this section has to be in sync with the `micromark` readme. -->

This package is [ESM only][esm].
In Node.js (version 16+), install with [npm][]:

```sh
npm install micromark
```

In Deno with [`esm.sh`][esmsh]:

```js
import {micromark} from 'https://esm.sh/micromark@3'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import {micromark} from 'https://esm.sh/micromark@3?bundle'
</script>
```

## Use

<!-- Note: this section has to be in sync with the `micromark` readme. -->

Typical use (buffering):

```js
import {micromark} from 'micromark'

console.log(micromark('## Hello, *world*!'))
```

Yields:

```html
<h2>Hello, <em>world</em>!</h2>
```

You can pass extensions (in this case [`micromark-extension-gfm`][gfm]):

```js
import {micromark} from 'micromark'
import {gfmHtml, gfm} from 'micromark-extension-gfm'

const value = '* [x] contact@example.com ~~strikethrough~~'

const result = micromark(value, {
  extensions: [gfm()],
  htmlExtensions: [gfmHtml()]
})

console.log(result)
```

Yields:

```html
<ul>
<li><input checked="" disabled="" type="checkbox"> <a href="mailto:contact@example.com">contact@example.com</a> <del>strikethrough</del></li>
</ul>
```

Streaming interface:

```js
import {createReadStream} from 'node:fs'
import {stream} from 'micromark/stream'

createReadStream('example.md')
  .on('error', handleError)
  .pipe(stream())
  .pipe(process.stdout)

function handleError(error) {
  // Handle your error here!
  throw error
}
```

## API

See [§ API][api] in the `micromark` readme.

## Extensions

micromark supports extensions.
There are two types of extensions for micromark:
[`SyntaxExtension`][syntax-extension],
which change how markdown is parsed, and [`HtmlExtension`][html-extension],
which change how it compiles.
They can be passed in [`options.extensions`][api-option-extensions] or
[`options.htmlExtensions`][api-option-htmlextensions], respectively.

As a user of extensions, refer to each extension’s readme for more on how to use
them.
As a (potential) author of extensions, refer to
[§ Extending markdown][extending-markdown] and
[§ Creating a micromark extension][create-extension].

### List of extensions

* [`micromark/micromark-extension-directive`][directives]
  — support directives (generic extensions)
* [`micromark/micromark-extension-frontmatter`][frontmatter]
  — support frontmatter (YAML, TOML, etc)
* [`micromark/micromark-extension-gfm`][gfm]
  — support GFM (GitHub Flavored Markdown)
* [`micromark/micromark-extension-gfm-autolink-literal`](https://github.com/micromark/micromark-extension-gfm-autolink-literal)
  — support GFM autolink literals
* [`micromark/micromark-extension-gfm-footnote`](https://github.com/micromark/micromark-extension-gfm-footnote)
  — support GFM footnotes
* [`micromark/micromark-extension-gfm-strikethrough`](https://github.com/micromark/micromark-extension-gfm-strikethrough)
  — support GFM strikethrough
* [`micromark/micromark-extension-gfm-table`](https://github.com/micromark/micromark-extension-gfm-table)
  — support GFM tables
* [`micromark/micromark-extension-gfm-tagfilter`](https://github.com/micromark/micromark-extension-gfm-tagfilter)
  — support GFM tagfilter
* [`micromark/micromark-extension-gfm-task-list-item`](https://github.com/micromark/micromark-extension-gfm-task-list-item)
  — support GFM tasklists
* [`micromark/micromark-extension-math`][math]
  — support math
* [`micromark/micromark-extension-mdx`](https://github.com/micromark/micromark-extension-mdx)
  — support MDX
* [`micromark/micromark-extension-mdxjs`][mdxjs]
  — support MDX.js
* [`micromark/micromark-extension-mdx-expression`][mdx-expression]
  — support MDX (or MDX.js) expressions
* [`micromark/micromark-extension-mdx-jsx`](https://github.com/micromark/micromark-extension-mdx-jsx)
  — support MDX (or MDX.js) JSX
* [`micromark/micromark-extension-mdx-md`](https://github.com/micromark/micromark-extension-mdx-md)
  — support misc MDX changes
* [`micromark/micromark-extension-mdxjs-esm`](https://github.com/micromark/micromark-extension-mdxjs-esm)
  — support MDX.js import/exports

#### Community extensions

* [`wataru-chocola/micromark-extension-definition-list`](https://github.com/wataru-chocola/micromark-extension-definition-list)
  — support definition lists

### `SyntaxExtension`

A syntax extension is an object whose fields are typically the names of hooks,
referring to where constructs “hook” into.
The fields at such objects are character codes, mapping to constructs as values.

The built in [constructs][] are an example.
See it and [existing extensions][extensions] for inspiration.

### `HtmlExtension`

An HTML extension is an object whose fields are typically `enter` or `exit`
(reflecting whether a token is entered or exited).
The values at such objects are names of tokens mapping to handlers.

See [existing extensions][extensions] for inspiration.

### Extending markdown

micromark lets you change markdown syntax, yes, but there are alternatives.
The alternatives are often better.

Over the years, many micromark and remark users have asked about their unique
goals for markdown.
Some exemplary goals are:

1. I want to add `rel="nofollow"` to external links
2. I want to add links from headings to themselves
3. I want line breaks in paragraphs to become hard breaks
4. I want to support embedded music sheets
5. I want authors to add arbitrary attributes
6. I want authors to mark certain blocks with meaning, such as tip, warning,
   etc
7. I want to combine markdown with JS(X)
8. I want to support our legacy flavor of markdown-like syntax

These can be solved in different ways and which solution is best is both
subjective and dependent on unique needs.
Often, there is already a solution in the form of an existing remark or rehype
plugin.
Respectively, their solutions are:

1. [`remark-external-links`](https://github.com/remarkjs/remark-external-links)
2. [`rehype-autolink-headings`](https://github.com/rehypejs/rehype-autolink-headings)
3. [`remark-breaks`](https://github.com/remarkjs/remark-breaks)
4. custom plugin similar to
   [`rehype-katex`](https://github.com/remarkjs/remark-math/tree/main/packages/rehype-katex)
   but integrating [`abcjs`](https://www.abcjs.net)
5. either [`remark-directive`][remark-directive]
   and a custom plugin or with
   [`rehype-attr`](https://github.com/jaywcjlove/rehype-attr)
6. [`remark-directive`][remark-directive]
   combined with a custom plugin
7. combining the existing micromark MDX extensions however you please, such as
   done by [`mdx-js/mdx`][mdx] or
   [`xdm`](https://github.com/wooorm/xdm)
8. Writing a micromark extension

Looking at these from a higher level, they can be categorized:

* **Changing the output by transforming syntax trees**
  (1 and 2)

  This category is nice as the format remains plain markdown that authors are
  already familiar with and which will work with existing tools and platforms.

  Implementations will deal with the syntax tree
  ([`mdast`][mdast]) and the ecosystems
  **[remark][]** and **[rehype][]**.
  There are many existing
  [utilities for working with that tree][utilities].
  Many [remark plugins][remark-plugins] and
  [rehype plugins][rehype-plugins] also exist.
* **Using and abusing markdown to add new meaning**
  (3, 4, potentially 5)

  This category is similar to *Changing the output by transforming syntax
  trees*, but adds a new meaning to certain things which already have
  semantics in markdown.

  Some examples in pseudocode:

  ````markdown
  *   **A list item with the first paragraph bold**

      And then more content, is turned into `<dl>` / `<dt>` / `<dd>` elements

  Or, the title attributes on links or images is [overloaded](/url 'rel:nofollow')
  with a new meaning.

  ```csv
  fenced,code,can,include,data
  which,is,turned,into,a,graph
  ```

  ```js data can="be" passed=true
  // after the code language name
  ```

  HTML, especially comments, could be used as **markers**<!--id="markers"-->
  ````
* **Arbitrary extension mechanism**
  (potentially 5; 6)

  This category is nice when content should contain embedded “components”.
  Often this means it’s required for authors to have some programming
  experience.
  There are three good ways to solve arbitrary extensions.

  **HTML**: Markdown already has an arbitrary extension syntax.
  It works in most places and authors are already familiar with the syntax,
  but it’s reasonably hard to implement securely.
  Certain platforms will remove HTML completely, others sanitize it to varying
  degrees.
  HTML also supports custom elements.
  These could be used and enhanced by client side JavaScript or enhanced when
  transforming the syntax tree.

  **Generic directives**: although
  [a proposal][directive-proposal]
  and not supported on most platforms, directives do work with many tools
  already.
  They’re not the easiest to author compared to, say, a heading, but sometimes
  that’s okay.
  They do have potential: they nicely solve the need for an infinite number of
  potential extensions to markdown in a single markdown-esque way.

  **MDX** also adds support for components by swapping HTML out for JS(X).
  JSX is an extension to JavaScript, so MDX is something along the lines of
  literate programming.
  This does require knowledge of React (or Vue) and JavaScript, excluding some
  authors.
* **Extending markdown syntax**
  (7 and 8)

  Extend the syntax of markdown means:

  * Authors won’t be familiar with the syntax
  * Content won’t work in other places (such as on GitHub)
  * Defeating the purpose of markdown: being simple to author and looking
    like what it means

  …and it’s hard to do as it requires some in-depth knowledge of JavaScript
  and parsing.
  But it’s possible and in certain cases very powerful.

### Creating a micromark extension

This section shows how to create an extension for micromark that parses
“variables” (a way to render some data) and one to turn a default construct off.

> Stuck?
> See [`support.md`][support].

#### Prerequisites

* You should possess an intermediate to high understanding of JavaScript:
  it’s going to get a bit complex
* Read the readme of [unified][] (until you hit the API section) to better
  understand where micromark fits
* Read the [§ Architecture][architecture] section to understand how micromark
  works
* Read the [§ Extending markdown][extending-markdown] section to understand
  whether it’s a good idea to extend the syntax of markdown

#### Extension basics

micromark supports two types of extensions.
Syntax extensions change how markdown is parsed.
HTML extensions change how it compiles.

HTML extensions are not always needed, as micromark is often used through
[`mdast-util-from-markdown`][mdast-util-from-markdown] to parse to a markdown
syntax tree.
So instead of an HTML extension a `from-markdown` utility is needed.
Then, a [`mdast-util-to-markdown`][mdast-util-to-markdown] utility, which is
responsible for serializing syntax trees to markdown, is also needed.

When developing something for internal use only, you can pick and choose which
parts you need.
When open sourcing your extensions, it should probably contain four parts:
syntax extension, HTML extension, `from-markdown` utility, and a `to-markdown`
utility.

On to our first case!

#### Case: variables

Let’s first outline what we want to make: render some data, similar to how
[Liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) and the
like work, in our markdown.
It could look like this:

```markdown
Hello, {planet}!
```

Turned into:

```html
<p>Hello, Venus!</p>
```

An opening curly brace, followed by one or more characters, and then a closing
brace.
We’ll then look up `planet` in some object and replace the variable with its
corresponding value, to get something like `Venus` out.

It looks simple enough, but with markdown there are often a couple more things
to think about.
For this case, I can see the following:

* Is there a “block” version too?
* Are spaces allowed?
  Line endings?
  Should initial and final white space be ignored?
* Balanced nested braces?
  Superfluous ones such as `{{planet}}` or meaningful ones such as
  `{a {pla} net}`?
* Character escapes (`{pla\}net}`) and character references
  (`{pla&#x7d;net}`)?

To keep things as simple as possible, let’s not support a block syntax, see
spaces as special, support line endings, or support nested braces.
But to learn interesting things, we *will* support character escapes and
-references.

Note that this particular case is already solved quite nicely by
[`micromark-extension-mdx-expression`][mdx-expression].
It’s a bit more powerful and does more things, but it can be used to solve this
case and otherwise serve as inspiration.

##### Setup

Create a new folder, enter it, and set up a new package:

```sh
mkdir example
cd example
npm init -y
```

In this example we’ll use ESM, so add `type: 'module'` to `package.json`:

```diff
@@ -2,6 +2,7 @@
   "name": "example",
   "version": "1.0.0",
   "description": "",
+  "type": "module",
   "main": "index.js",
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
```

Add a markdown file, `example.md`, with the following text:

```markdown
Hello, {planet}!

{pla\}net} and {pla&#x7d;net}.
```

To check if our extension works, add an `example.js` module, with the following
code:

```js
import fs from 'node:fs/promises'
import {micromark} from 'micromark'
import {variables} from './index.js'

const buf = await fs.readFile('example.md')
const out = micromark(buf, {extensions: [variables]})
console.log(out)
```

While working on the extension, run `node example` to see whether things work.
Feel free to add more examples of the variables syntax in `example.md` if
needed.

Our extension doesn’t work yet, for one because `micromark` is not installed:

```sh
npm install micromark --save-dev
```

…and we need to write our extension.
Let’s do that in `index.js`:

```js
export const variables = {}
```

Although our extension doesn’t do anything, running `node example` now somewhat
works!

##### Syntax extension

Much in micromark is based on character codes (see [§ Preprocess][preprocess]).
For this extension, the relevant codes are:

* `-5`
  — M-0005 CARRIAGE RETURN (CR)
* `-4`
  — M-0004 LINE FEED (LF)
* `-3`
  — M-0003 CARRIAGE RETURN LINE FEED (CRLF)
* `null`
  — EOF (end of the stream)
* `92`
  — U+005C BACKSLASH (`\`)
* `123`
  — U+007B LEFT CURLY BRACE (`{`)
* `125`
  — U+007D RIGHT CURLY BRACE (`}`)

Also relevant are the content types (see [§ Content types][content-types]).
This extension is a *text* construct, as it’s parsed alongsides links and such.
The content inside it (between the braces) is *string*, to support character
escapes and -references.

Let’s write our extension.
Add the following code to `index.js`:

```js
const variableConstruct = {name: 'variable', tokenize: variableTokenize}

export const variables = {text: {123: variableConstruct}}

function variableTokenize(effects, ok, nok) {
  return start

  function start(code) {
    console.log('start:', effects, code);
    return nok(code)
  }
}
```

The above code exports an extension with the identifier `variables`.
The extension defines a *text* construct for the character code `123`.
The construct has a `name`, so that it can be turned off (optional, see next
case), and it has a `tokenize` function that sets up a state machine, which
receives `effects` and the `ok` and `nok` states.
`ok` can be used when successful, `nok` when not, and so constructs are a bit
similar to how promises can *resolve* or *reject*.
`tokenize` returns the initial state, `start`, which itself receives the current
character code, prints some debugging information, and then returns a call
to `nok`.

Ensure that things work by running `node example` and see what it prints.

Now we need to define our states and figure out how variables work.
Some people prefer sketching a diagram of the flow.
I often prefer writing it down in pseudo-code prose.
I’ve also found that test driven development works well, where I write unit
tests for how it should work, then write the state machine, and finally use a
code coverage tool to ensure I’ve thought of everything.

In prose, what we have to code looks like this:

* **start**:
  Receive `123` as `code`, enter a token for the whole (let’s call it
  `variable`), enter a token for the marker (`variableMarker`), consume
  `code`, exit the marker token, enter a token for the contents
  (`variableString`), switch to *begin*
* **begin**:
  If `code` is `125`, reconsume in *nok*.
  Else, reconsume in *inside*
* **inside**:
  If `code` is `-5`, `-4`, `-3`, or `null`, reconsume in `nok`.
  Else, if `code` is `125`, exit the string token, enter a `variableMarker`,
  consume `code`, exit the marker token, exit the variable token, and switch
  to *ok*.
  Else, consume, and remain in *inside*.

That should be it!
Replace `variableTokenize` with the following to include the needed states:

```js
function variableTokenize(effects, ok, nok) {
  return start

  function start(code) {
    effects.enter('variable')
    effects.enter('variableMarker')
    effects.consume(code)
    effects.exit('variableMarker')
    effects.enter('variableString')
    return begin
  }

  function begin(code) {
    return code === 125 ? nok(code) : inside(code)
  }

  function inside(code) {
    if (code === -5 || code === -4 || code === -3 || code === null) {
      return nok(code)
    }

    if (code === 125) {
      effects.exit('variableString')
      effects.enter('variableMarker')
      effects.consume(code)
      effects.exit('variableMarker')
      effects.exit('variable')
      return ok
    }

    effects.consume(code)
    return inside
  }
}
```

Run `node example` again and see what it prints!
The HTML compiler ignores things it doesn’t know, so variables are now removed.

We have our first syntax extension, and it sort of works, but we don’t handle
character escapes and -references yet.
We need to do two things to make that work:
a) skip over `\\` and `\}` in our algorithm,
b) tell micromark to parse them.

Change the code in `index.js` to support escapes like so:

```diff
@@ -23,6 +23,11 @@ function variableTokenize(effects, ok, nok) {
       return nok(code)
     }

+    if (code === 92) {
+      effects.consume(code)
+      return insideEscape
+    }
+
     if (code === 125) {
       effects.exit('variableString')
       effects.enter('variableMarker')
@@ -35,4 +40,13 @@ function variableTokenize(effects, ok, nok) {
     effects.consume(code)
     return inside
   }
+
+  function insideEscape(code) {
+    if (code === 92 || code === 125) {
+      effects.consume(code)
+      return inside
+    }
+
+    return inside(code)
+  }
 }
```

Finally add support for character references and character escapes between
braces by adding a special token that defines a content type:

```diff
@@ -11,6 +11,7 @@ function variableTokenize(effects, ok, nok) {
     effects.consume(code)
     effects.exit('variableMarker')
     effects.enter('variableString')
+    effects.enter('chunkString', {contentType: 'string'})
     return begin
   }

@@ -29,6 +30,7 @@ function variableTokenize(effects, ok, nok) {
     }

     if (code === 125) {
+      effects.exit('chunkString')
       effects.exit('variableString')
       effects.enter('variableMarker')
       effects.consume(code)
```

Tokens with a `contentType` will be replaced by *postprocess* (see
[§ Postprocess][postprocess]) by the tokens belonging to that content type.

##### HTML extension

Up next is an HTML extension to replace variables with data.
Change `example.js` to use one like so:

```diff
@@ -1,11 +1,12 @@
 import fs from 'node:fs/promises'
 import {micromark} from 'micromark'
-import {variables} from './index.js'
+import {variablesHtml, variables} from './index.js'

 const buf = await fs.readFile('example.md')
-const out = micromark(buf, {extensions: [variables]})
+const html = variablesHtml({planet: '1', 'pla}net': '2'})
+const out = micromark(buf, {extensions: [variables], htmlExtensions: [html]})
 console.log(out)
```

And add the HTML extension, `variablesHtml`, to `index.js` like so:

```diff
@@ -52,3 +52,19 @@ function variableTokenize(effects, ok, nok) {
     return inside(code)
   }
 }
+
+export function variablesHtml(data = {}) {
+  return {
+    enter: {variableString: enterVariableString},
+    exit: {variableString: exitVariableString},
+  }
+
+  function enterVariableString() {
+    this.buffer()
+  }
+
+  function exitVariableString() {
+    var id = this.resume()
+    if (id in data) {
+      this.raw(this.encode(data[id]))
+    }
+  }
+}
```

`variablesHtml` is a function that receives an object mapping “variables” to
strings and returns an HTML extension.
The extension hooks two functions to `variableString`, one when it starts,
the other when it ends.
We don’t need to do anything to handle the other tokens as they’re already
ignored by default.
`enterVariableString` calls `buffer`, which is a function that “stashes” what
would otherwise be emitted.
`exitVariableString` calls `resume`, which is the inverse of `buffer` and
returns the stashed value.
If the variable is defined, we ensure it’s made safe (with `this.encode`) and
finally output that (with `this.raw`).

##### Further exercises

It works!
We’re done!
Of course, it can be better, such as with the following potential features:

* Add support for empty variables
* Add support for spaces between markers and string
* Add support for line endings in variables
* Add support for nested braces
* Add support for blocks
* Add warnings on undefined variables
* Use `micromark-build`, and use `devlop`, `debug`, and
  `micromark-util-symbol` (see [§ Size & debug][size-debug])
* Add [`mdast-util-from-markdown`][mdast-util-from-markdown] and
  [`mdast-util-to-markdown`][mdast-util-to-markdown] utilities to parse and
  serialize the AST

#### Case: turn off constructs

Sometimes it’s needed to turn a default construct off.
That’s possible through a syntax extension.
Note that not everything can be turned off (such as paragraphs) and even if it’s
possible to turn something off, it could break micromark (such as character
escapes).

To disable constructs, refer to them by name in an array at the `disable.null`
field of an extension:

```js
import {micromark} from 'micromark'

const extension = {disable: {null: ['codeIndented']}}

console.log(micromark('\ta', {extensions: [extension]}))
```

Yields:

```html
<p>a</p>
```

## Architecture

micromark is maintained as a monorepo.
Many of its internals, which are used in `micromark` (core) but also useful for
developers of extensions or integrations, are available as separate modules.
Each module maintained here is available in [`packages/`][packages].

### Overview

The naming scheme in [`packages/`][packages] is as follows:

* `micromark-build`
  — Small CLI to build dev code into production code
* `micromark-core-commonmark`
  — CommonMark constructs used in micromark
* `micromark-factory-*`
  — Reusable subroutines used to parse parts of constructs
* `micromark-util-*`
  — Reusable helpers often needed when parsing markdown
* `micromark`
  — Core module

micromark has two interfaces: buffering (maintained in
[`micromark/dev/index.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/index.js))
and streaming (maintained in
[`micromark/dev/stream.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/stream.js)).
The first takes all input at once whereas the last uses a Node.js stream to take
input separately.
They thinly wrap how data flows through micromark:

```text
                                            micromark
+-----------------------------------------------------------------------------------------------+
|            +------------+         +-------+         +-------------+         +---------+       |
| -markdown->+ preprocess +-chunks->+ parse +-events->+ postprocess +-events->+ compile +-html- |
|            +------------+         +-------+         +-------------+         +---------+       |
+-----------------------------------------------------------------------------------------------+
```

### Preprocess

The **preprocessor**
([`micromark/dev/lib/preprocess.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/lib/preprocess.js))
takes markdown and turns it into chunks.

A **chunk** is either a character code or a slice of a buffer in the form of a
string.
Chunks are used because strings are more efficient storage than character codes,
but limited in what they can represent.
For example, the input `ab\ncd` is represented as `['ab', -4, 'cd']` in chunks.

A character **code** is often the same as what `String#charCodeAt()` yields but
micromark adds meaning to certain other values.

In micromark, the actual character U+0009 CHARACTER TABULATION (HT) is replaced
by one M-0002 HORIZONTAL TAB (HT) and between 0 and 3 M-0001 VIRTUAL SPACE (VS)
characters, depending on the column at which the tab occurred.
For example, the input `\ta` is represented as `[-2, -1, -1, -1, 97]` and `a\tb`
as `[97, -2, -1, -1, 98]` in character codes.

The characters U+000A LINE FEED (LF) and U+000D CARRIAGE RETURN (CR) are
replaced by virtual characters depending on whether they occur together: M-0003
CARRIAGE RETURN LINE FEED (CRLF), M-0004 LINE FEED (LF), and M-0005 CARRIAGE
RETURN (CR).
For example, the input `a\r\nb\nc\rd` is represented as
`[97, -5, 98, -4, 99, -3, 100]` in character codes.

The `0` (U+0000 NUL) character code is replaced by U+FFFD REPLACEMENT CHARACTER
(`�`).

The `null` code represents the end of the input stream (called *eof* for end of
file).

### Parse

The **parser**
([`micromark/dev/lib/parse.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/lib/parse.js))
takes chunks and turns them into events.

An **event** is the start or end of a token amongst other events.
Tokens can “contain” other tokens, even though they are stored in a flat list,
by entering before and exiting after them.

A **token** is a span of one or more codes.
Tokens are most of what micromark produces: the built in HTML compiler or other
tools can turn them into different things.
Tokens are essentially names attached to a slice, such as `lineEndingBlank` for
certain line endings, or `codeFenced` for a whole fenced code.

Sometimes, more info is attached to tokens, such as `_open` and `_close` by
`attention` (strong, emphasis) to signal whether the sequence can open or close
an attention run.
These fields have to do with how the parser works, which is complex and not
always pretty.

Certain fields (`previous`, `next`, and `contentType`) are used in many cases:
linked tokens for subcontent.
Linked tokens are used because outer constructs are parsed first.
Take for example:

```markdown
- *a
  b*.
```

1. The list marker and the space after it is parsed first
2. The rest of the line is a `chunkFlow` token
3. The two spaces on the second line are a `linePrefix` of the list
4. The rest of the line is another `chunkFlow` token

The two `chunkFlow` tokens are linked together and the chunks they span are
passed through the flow tokenizer.
There the chunks are seen as `chunkContent` and passed through the content
tokenizer.
There the chunks are seen as a paragraph and seen as `chunkText` and passed
through the text tokenizer.
Finally, the attention (emphasis) and data (“raw” characters) is parsed there,
and we’re done!

#### Content types

The parser starts out with a document tokenizer.
*Document* is the top-most content type, which includes containers such as block
quotes and lists.
Containers in markdown come from the margin and include more constructs
on the lines that define them.

*Flow* represents the sections (block constructs such as ATX and setext
headings, HTML, indented and fenced code, thematic breaks), which like
*document* are also parsed per line.
An example is HTML, which has a certain starting condition (such as `<script>`
on its own line), then continues for a while, until an end condition is found
(such as `</style>`).
If that line with an end condition is never found, that flow goes until the end.

*Content* is zero or more definitions, and then zero or one paragraph.
It’s a weird one, and needed to make certain edge cases around definitions spec
compliant.
Definitions are unlike other things in markdown, in that they behave like *text*
in that they can contain arbitrary line endings, but *have* to end at a line
ending.
If they end in something else, the whole definition instead is seen as a
paragraph.

The content in markdown first needs to be parsed up to this level to figure out
which things are defined, for the whole document, before continuing on with
*text*, as whether a link or image reference forms or not depends on whether
it’s defined.
This unfortunately prevents a true streaming markdown parser.

*Text* contains phrasing content (rich inline text: autolinks, character escapes
and -references, code, hard breaks, HTML, images, links, emphasis, strong).

*String* is a limited *text*-like content type which only allows character
references and character escapes.
It exists in things such as identifiers (media references, definitions),
titles, or URLs and such.

#### Constructs

Constructs are the things that make up markdown.
Some examples are lists, thematic breaks, or character references.

Note that, as a general rule of thumb, markdown is *really weird*.
It’s essentially made up of edge cases rather than logical rules.
When browsing the built in constructs, or venturing to build your own, you’ll
find confusing new things and run into complex custom hooks.

One more reasonable construct is the thematic break
([see code](https://github.com/micromark/micromark/blob/main/packages/micromark-core-commonmark/dev/lib/thematic-break.js)).
It’s an object that defines a `name` and a `tokenize` function.
Most of what constructs do is defined in their required `tokenize` function,
which sets up a state machine to handle character codes streaming in.

### Postprocess

The **postprocessor**
([`micromark/dev/lib/postprocess.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/lib/postprocess.js))
is a small step that takes events, ensures all their
nested content is parsed, and returns the modified events.

### Compile

The **compiler**
([`micromark/dev/lib/compile.js`](https://github.com/micromark/micromark/blob/main/packages/micromark/dev/lib/compile.js))
takes events and turns them into HTML.
While micromark was created mostly to advance markdown parsing irrespective of
compiling to HTML, the common case of doing so is built in.
A built in HTML compiler is useful because it allows us to check for compliancy
to CommonMark, the de facto norm of markdown, specified in roughly 650
input/output cases.
The parsing parts can still be used separately to build ASTs, CSTs, or many
other output formats.

The compiler has an interface that accepts lists of events instead of the whole
at once, but because markdown can’t truly stream, events are buffered before
compiling and outputting the final result.

## Examples

### GitHub flavored markdown (GFM)

To support GFM (autolink literals, strikethrough, tables, and tasklists) use
[`micromark-extension-gfm`][gfm].
Say we have a file like this:

```markdown
# GFM

## Autolink literals

www.example.com, https://example.com, and contact@example.com.

## Footnote

A note[^1]

[^1]: Big note.

## Strikethrough

~one~ or ~~two~~ tildes.

## Table

| a | b  |  c |  d  |
| - | :- | -: | :-: |

## Tag filter

<plaintext>

## Tasklist

* [ ] to do
* [x] done
```

Then do something like this:

```js
import fs from 'node:fs/promises'
import {micromark} from 'micromark'
import {gfmHtml, gfm} from 'micromark-extension-gfm'

const doc = await fs.readFile('example.md')

console.log(micromark(doc, {extensions: [gfm()], htmlExtensions: [gfmHtml()]}))
```

<details>
<summary>Show equivalent HTML</summary>

```html
<h1>GFM</h1>
<h2>Autolink literals</h2>
<p><a href="http://www.example.com">www.example.com</a>, <a href="https://example.com">https://example.com</a>, and <a href="mailto:contact@example.com">contact@example.com</a>.</p>
<h2>Footnote</h2>
<p>A note<sup><a href="#user-content-fn-1" id="user-content-fnref-1" data-footnote-ref="" aria-describedby="footnote-label">1</a></sup></p>
<h2>Strikethrough</h2>
<p><del>one</del> or <del>two</del> tildes.</p>
<h2>Table</h2>
<table>
<thead>
<tr>
<th>a</th>
<th align="left">b</th>
<th align="right">c</th>
<th align="center">d</th>
</tr>
</thead>
</table>
<h2>Tag filter</h2>
&lt;plaintext&gt;
<h2>Tasklist</h2>
<ul>
<li><input disabled="" type="checkbox"> to do</li>
<li><input checked="" disabled="" type="checkbox"> done</li>
</ul>
<section data-footnotes="" class="footnotes"><h2 id="footnote-label" class="sr-only">Footnotes</h2>
<ol>
<li id="user-content-fn-1">
<p>Big note. <a href="#user-content-fnref-1" data-footnote-backref="" class="data-footnote-backref" aria-label="Back to content">↩</a></p>
</li>
</ol>
</section>
```

</details>

### Math

To support math use [`micromark-extension-math`][math].
Say we have a file like this:

```markdown
Lift($L$) can be determined by Lift Coefficient ($C_L$) like the following equation.

$$
L = \frac{1}{2} \rho v^2 S C_L
$$
```

Then do something like this:

```js
import fs from 'node:fs/promises'
import {micromark} from 'micromark'
import {mathHtml, math} from 'micromark-extension-math'

const doc = await fs.readFile('example.md')

console.log(micromark(doc, {extensions: [math], htmlExtensions: [mathHtml()]}))
```

<details>
<summary>Show equivalent HTML</summary>

```html
<p>Lift(<span class="math math-inline"><span class="katex">…</span></span>) can be determined by Lift Coefficient (<span class="math math-inline"><span class="katex">…</span></span>) like the following equation.</p>
<div class="math math-display"><span class="katex-display"><span class="katex">…</span></span></div>
```

</details>

### Syntax tree

A higher level project, [`mdast-util-from-markdown`][mdast-util-from-markdown],
can give you an AST.

```js
import {fromMarkdown} from 'mdast-util-from-markdown' // This wraps micromark.

const result = fromMarkdown('## Hello, *world*!')

console.log(result.children[0])
```

Yields:

```js
{
  type: 'heading',
  depth: 2,
  children: [
    {type: 'text', value: 'Hello, ', position: [Object]},
    {type: 'emphasis', children: [Array], position: [Object]},
    {type: 'text', value: '!', position: [Object]}
  ],
  position: {
    start: {line: 1, column: 1, offset: 0},
    end: {line: 1, column: 19, offset: 18}
  }
}
```

Another level up is [**remark**][remark], which provides a nice interface and
hundreds of plugins.

## Markdown

### CommonMark

The first definition of “Markdown” gave several examples of how it worked,
showing input Markdown and output HTML, and came with a reference implementation
(`Markdown.pl`).
When new implementations followed, they mostly followed the first definition,
but deviated from the first implementation, and added extensions, thus making
the format a family of formats.

Some years later, an attempt was made to standardize the differences between
implementations, by specifying how several edge cases should be handled, through
more input and output examples.
This is known as [CommonMark][commonmark-spec], and many implementations now
work towards some degree of CommonMark compliancy.
Still, CommonMark describes what the output in HTML should be given some
input, which leaves many edge cases up for debate, and does not answer what
should happen for other output formats.

micromark passes all tests from CommonMark and has many more tests to match the
CommonMark reference parsers.
Finally, it comes with [CMSM][], which describes how to parse markup, instead
of documenting input and output examples.

### Grammar

The syntax of markdown can be described in Backus–Naur form (BNF) as:

```abnf
markdown ::= .*
```

No, that’s [not a typo](http://trevorjim.com/a-specification-for-markdown/):
markdown has no syntax errors; anything thrown at it renders *something*.

## Project

### Comparison

There are many other markdown parsers out there and maybe they’re better suited
to your use case!
Here is a short comparison of a couple in JavaScript.
Note that this list is made by the folks who make `micromark` and `remark`, so
there is some bias.

**Note**: these are, in fact, not really comparable: micromark (and remark)
focus on completely different things than other markdown parsers do.
Sure, you can generate HTML from markdown with them, but micromark (and remark)
are created for (abstract or concrete) syntax trees—to inspect, transform, and
generate content, so that you can make things like [MDX][], [Prettier][], or
[Astro][].

###### `micromark`

micromark can be used in two different ways.
It can either be used, optionally with existing extensions, to get HTML easily.
Or, it can give tremendous power, such as access to all tokens with positional
info, at the cost of being hard to get into.
It’s super small, pretty fast, and has 100% CommonMark compliance.
It has syntax extensions, such as supporting 100% GFM compliance (with
`micromark-extension-gfm`), but they’re rather complex to write.
It’s the newest parser on the block, which means it’s fresh and well suited for
contemporary markdown needs, but it’s also battle-tested, and already the 3rd
most popular markdown parser in JavaScript.

If you’re looking for fine grained control, use micromark.
If you just want HTML from markdown, use micromark.

###### `remark`

[remark][] is the most popular markdown parser.
It’s built on top of `micromark` and boasts syntax trees.
For an analogy, it’s like if Babel, ESLint, and more, were one project.
It supports the syntax extensions that micromark has (so it’s 100% CM compliant
and can be 100% GFM compliant), but most of the work is done in plugins that
transform or inspect the tree, and there’s *tons* of them.
Transforming the tree is relatively easy: it’s a JSON object that can be
manipulated directly.
remark is stable, widely used, and extremely powerful for handling complex data.

You probably should use [remark][].

###### `marked`

[marked][] is the oldest markdown parser on the block.
It’s been around for ages, is battle tested, small, popular, and has a bunch of
extensions, but doesn’t match CommonMark or GFM, and is unsafe by default.

If you have markdown you trust and want to turn it into HTML without a fuss, and
don’t care about perfect compatibility with CommonMark or GFM, but do appreciate
a small bundle size and stability, use [marked][].

###### `markdown-it`

[markdown-it][] is a good, stable, and essentially CommonMark compliant markdown
parser, with (optional) support for some GFM features as well.
It’s used a lot as a direct dependency in packages, but is rather big.
It shines at syntax extensions, where you want to support not just markdown, but
*your* (company’s) version of markdown.

If you need a couple of custom syntax extensions to your otherwise
CommonMark-compliant markdown, and want to get HTML out, use [markdown-it][].

###### Others

There are lots of other markdown parsers!
Some say they’re small, or fast, or that they’re CommonMark compliant—but
that’s not always true.
This list is not supposed to be exhaustive (but it’s the most relevant ones).
This list of markdown parsers is a snapshot in time of why (not) to use
(alternatives to) `micromark`: they’re all good choices, depending on what your
goals are.

### Test

micromark is tested with the \~650 CommonMark tests and more than 1.2k extra
tests confirmed with CM reference parsers.
These tests reach all branches in the code, which means that this project has
100% code coverage.
Finally, we use fuzz testing to ensure micromark is stable, reliable, and
secure.

To build, format, and test the codebase, use `$ npm test` after clone and
install.
The `$ npm run test-api` and `$ npm run test-coverage` scripts check either the
unit tests, or both them and their coverage, respectively.

The `$ npm run test-fuzz` script does fuzz testing for 30 minutes.

### Size & debug

micromark is really small.
A ton of time went into making sure it minifies well, by the way code is written
but also through custom build scripts to pre-evaluate certain expressions.
Furthermore, care went into making it compress well with gzip and brotli.

Normally, you’ll use the pre-evaluated version of micromark.
While developing, debugging, or testing your code, you *should* switch to use
code instrumented with assertions and debug messages:

```sh
node --conditions development module.js
```

To see debug messages, use a `DEBUG` env variable set to `micromark`:

```sh
DEBUG="*" node --conditions development module.js
```

### Version

micromark adheres to [semver](https://semver.org) since 3.0.0.

### Security

The typical security aspect discussed for markdown is [cross-site scripting
(XSS)][xss] attacks.
Markdown itself is safe if it does not include embedded HTML or dangerous
protocols in links/images (such as `javascript:` or `data:`).
micromark makes any markdown safe by default, even if HTML is embedded or
dangerous protocols are used, as it encodes or drops them.
Turning on the `allowDangerousHtml` or `allowDangerousProtocol` options for
user-provided markdown opens you up to XSS attacks.

Another security aspect is DDoS attacks.
For example, an attacker could throw a 100mb file at micromark, in which case
the JavaScript engine will run out of memory and crash.
It is also possible to crash micromark with smaller payloads, notably when
thousands of links, images, emphasis, or strong are opened but not closed.
It is wise to cap the accepted size of input (500kb can hold a big book) and to
process content in a different thread or worker so that it can be stopped when
needed.

Using extensions might also be unsafe, refer to their documentation for more
information.

For more information on markdown sanitation, see
[`improper-markup-sanitization.md`][improper] by [**@chalker**][chalker].

See [`security.md`][securitymd] in [`micromark/.github`][health] for how to
submit a security report.

### Contribute

See [`contributing.md`][contributing] in [`micromark/.github`][health] for ways
to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organisation, or community you agree to
abide by its terms.

### Sponsor

<!-- Note: this section has to be in sync with the `micromark` readme. -->

Support this effort and give back by sponsoring on [OpenCollective][]!

<table>
<tr valign="middle">
<td width="100%" align="center" colspan="10">
  <br>
  <a href="https://www.salesforce.com">Salesforce</a> 🏅<br><br>
  <a href="https://www.salesforce.com"><img src="https://images.opencollective.com/salesforce/ca8f997/logo/512.png" width="256"></a>
</td>
</tr>
<tr valign="middle">
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://vercel.com">Vercel</a><br><br>
  <a href="https://vercel.com"><img src="https://avatars1.githubusercontent.com/u/14985020?s=256&v=4" width="128"></a>
</td>
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://motif.land">Motif</a><br><br>
  <a href="https://motif.land"><img src="https://avatars1.githubusercontent.com/u/74457950?s=256&v=4" width="128"></a>
</td>
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://www.hashicorp.com">HashiCorp</a><br><br>
  <a href="https://www.hashicorp.com"><img src="https://avatars1.githubusercontent.com/u/761456?s=256&v=4" width="128"></a>
</td>
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://www.gitbook.com">GitBook</a><br><br>
  <a href="https://www.gitbook.com"><img src="https://avatars1.githubusercontent.com/u/7111340?s=256&v=4" width="128"></a>
</td>
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://www.gatsbyjs.org">Gatsby</a><br><br>
  <a href="https://www.gatsbyjs.org"><img src="https://avatars1.githubusercontent.com/u/12551863?s=256&v=4" width="128"></a>
</td>
</tr>
<tr valign="middle">
</tr>
<tr valign="middle">
<td width="20%" align="center" rowspan="2" colspan="2">
  <a href="https://www.netlify.com">Netlify</a><br><br>
  <!--OC has a sharper image-->
  <a href="https://www.netlify.com"><img src="https://images.opencollective.com/netlify/4087de2/logo/256.png" width="128"></a>
</td>
<td width="10%" align="center">
  <a href="https://www.coinbase.com">Coinbase</a><br><br>
  <a href="https://www.coinbase.com"><img src="https://avatars1.githubusercontent.com/u/1885080?s=256&v=4" width="64"></a>
</td>
<td width="10%" align="center">
  <a href="https://themeisle.com">ThemeIsle</a><br><br>
  <a href="https://themeisle.com"><img src="https://avatars1.githubusercontent.com/u/58979018?s=128&v=4" width="64"></a>
</td>
<td width="10%" align="center">
  <a href="https://expo.io">Expo</a><br><br>
  <a href="https://expo.io"><img src="https://avatars1.githubusercontent.com/u/12504344?s=128&v=4" width="64"></a>
</td>
<td width="10%" align="center">
  <a href="https://boostnote.io">Boost Note</a><br><br>
  <a href="https://boostnote.io"><img src="https://images.opencollective.com/boosthub/6318083/logo/128.png" width="64"></a>
</td>
<td width="10%" align="center">
  <a href="https://markdown.space">Markdown Space</a><br><br>
  <a href="https://markdown.space"><img src="https://images.opencollective.com/markdown-space/e1038ed/logo/128.png" width="64"></a>
</td>
<td width="10%" align="center">
  <a href="https://www.holloway.com">Holloway</a><br><br>
  <a href="https://www.holloway.com"><img src="https://avatars1.githubusercontent.com/u/35904294?s=128&v=4" width="64"></a>
</td>
<td width="10%"></td>
<td width="10%"></td>
</tr>
<tr valign="middle">
<td width="100%" align="center" colspan="8">
  <br>
  <a href="https://opencollective.com/unified"><strong>You?</strong></a>
  <br><br>
</td>
</tr>
</table>

### Origin story

Over the summer of 2018, micromark was planned, and the idea shared in August
with a couple of friends and potential sponsors.
The problem I (**[@wooorm][]**) had was that issues were piling up in remark and
other repos, but my day job (teaching) was fun, fulfilling, and deserved time
too.
It was getting hard to combine the two.
The thought was to feed two birds with one scone: fix the issues in remark with
a new markdown parser (codename marydown) while being financially supported by
sponsors building fancy stuff on top, such as Gatsby, Contentful, and Vercel
(ZEIT at the time).
**[@johno][]** was making MDX on top of remark at the time (important historical
note: several other folks were working on JSX + markdown too).
We bundled our strengths: MDX was getting some traction and we thought together
we could perhaps make something sustainable.

In November 2018, we launched with the idea for micromark to solve all existing
bugs, sustaining the existing hundreds of projects, and furthering the exciting
high-level project MDX.
We pushed a single name: unified (which back then was a small but essential
part of the chain).
Gatsby and Vercel were immediate sponsors.
We didn’t know whether it would work, and it worked.
But now you have a new problem: you are getting some financial support (much
more than other open source projects) but it’s not enough money for rent, and
too much money to print stickers with.
You still have your job and issues are still piling up.

At the start of summer 2019, after a couple months of saving up donations, I
quit my job and worked on unified through fall.
That got the number of open issues down significantly and set up a strong
governance and maintenance system for the collective.
But when the time came to work on micromark, the money was gone again, so I
contracted through winter 2019, and in spring 2020 I could do about half open
source, half contracting.
One of the contracting gigs was to write a new MDX parser, for which I also
documented how to do that with a state machine [in prose][mdx-cmsm].
That gave me the insight into how the same could be done for markdown: I drafted
[CMSM][], which was some of the core ideas for micromark, but in prose.

In May 2020, Salesforce reached out: they saw the bugs in remark, how micromark
could help, and the initial work on CMSM.
And they had thousands of Markdown files.
In a for open source uncharacteristic move, they decided to fund my work on
micromark.
A large part of what maintaining open source means, is putting out fires,
triaging issues, and making sure users and sponsors are happy, so it was
amazing to get several months to just focus and make something new.
I remember feeling that this project would probably be the hardest thing I’d
work on: yeah, parsers are pretty difficult, but markdown is on another level.
Markdown is such a giant stack of edge cases on edge cases on even more
weirdness, what a mess.
On August 20, 2020, I released [2.0.0][200], the first working version of
micromark.
And it’s hard to describe how that moment felt.
It was great.

In 2022, Vercel paid me to make a Rust version: [`markdown-rs`][markdown-rs].
Super cool that I got to continue this work and bring it to a new language.

### License

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[200]: https://github.com/micromark/micromark/releases/tag/2.0.0

[@johno]: https://github.com/johno

[@wooorm]: https://github.com/wooorm

[api]: https://github.com/micromark/micromark/blob/main/packages/micromark/readme.md#api

[api-option-extensions]: https://github.com/micromark/micromark/blob/main/packages/micromark/readme.md#extensions

[api-option-htmlextensions]: https://github.com/micromark/micromark/blob/main/packages/micromark/readme.md#htmlextensions

[architecture]: #architecture

[astro]: https://github.com/withastro/astro

[author]: https://wooorm.com

[backers-badge]: https://opencollective.com/unified/backers/badge.svg

[build]: https://github.com/micromark/micromark/actions

[build-badge]: https://github.com/micromark/micromark/workflows/main/badge.svg

[bundle-size]: https://bundlejs.com/?q=micromark

[bundle-size-badge]: https://img.shields.io/badge/dynamic/json?label=minzipped%20size&query=$.size.compressedSize&url=https://deno.bundlejs.com/?q=micromark

[chalker]: https://github.com/ChALkeR

[chat]: https://github.com/micromark/micromark/discussions

[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg

[cheat]: https://commonmark.org/help/

[cmsm]: https://github.com/micromark/common-markup-state-machine

[coc]: https://github.com/micromark/.github/blob/main/code-of-conduct.md

[commonmark]: #commonmark

[commonmark-spec]: https://commonmark.org

[comparison]: #comparison

[constructs]: /packages/micromark/dev/lib/constructs.js

[content-types]: #content-types

[contribute]: #contribute

[contributing]: https://github.com/micromark/.github/blob/main/contributing.md

[coverage]: https://codecov.io/github/micromark/micromark

[coverage-badge]: https://img.shields.io/codecov/c/github/micromark/micromark.svg

[create-extension]: #creating-a-micromark-extension

[directive-proposal]: https://talk.commonmark.org/t/generic-directives-plugins-syntax/444

[directives]: https://github.com/micromark/micromark-extension-directive

[downloads]: https://www.npmjs.com/package/micromark

[downloads-badge]: https://img.shields.io/npm/dm/micromark.svg

[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c

[esmsh]: https://esm.sh

[extending-markdown]: #extending-markdown

[extensions]: #list-of-extensions

[frontmatter]: https://github.com/micromark/micromark-extension-frontmatter

[gfm]: https://github.com/micromark/micromark-extension-gfm

[health]: https://github.com/micromark/.github

[html-extension]: #htmlextension

[improper]: https://github.com/ChALkeR/notes/blob/master/Improper-markup-sanitization.md

[license]: https://github.com/micromark/micromark/blob/main/license

[markdown-it]: https://github.com/markdown-it/markdown-it

[markdown-rs]: https://github.com/wooorm/markdown-rs

[marked]: https://github.com/markedjs/marked

[math]: https://github.com/micromark/micromark-extension-math

[mdast]: https://github.com/syntax-tree/mdast

[mdast-util-from-markdown]: https://github.com/syntax-tree/mdast-util-from-markdown

[mdast-util-to-markdown]: https://github.com/syntax-tree/mdast-util-to-markdown

[mdx]: https://github.com/mdx-js/mdx

[mdx-cmsm]: https://github.com/micromark/mdx-state-machine

[mdx-expression]: https://github.com/micromark/micromark-extension-mdx-expression

[mdxjs]: https://github.com/micromark/micromark-extension-mdxjs

[npm]: https://docs.npmjs.com/cli/install

[opencollective]: https://opencollective.com/unified

[packages]: packages/

[postprocess]: #postprocess

[preprocess]: #preprocess

[prettier]: https://github.com/prettier/prettier

[rehype]: https://github.com/rehypejs/rehype

[rehype-plugins]: https://github.com/rehypejs/rehype/blob/main/doc/plugins.md#list-of-plugins

[remark]: https://github.com/remarkjs/remark

[remark-directive]: https://github.com/remarkjs/remark-directive

[remark-plugins]: https://github.com/remarkjs/remark/blob/main/doc/plugins.md#list-of-plugins

[security]: #security

[securitymd]: https://github.com/micromark/.github/blob/main/security.md

[site]: https://unifiedjs.com

[size-debug]: #size--debug

[sponsor]: #sponsor

[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg

[support]: https://github.com/micromark/.github/blob/main/support.md

[syntax-extension]: #syntaxextension

[test]: #test

[unified]: https://github.com/unifiedjs/unified

[utilities]: https://github.com/syntax-tree/mdast#list-of-utilities

[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
