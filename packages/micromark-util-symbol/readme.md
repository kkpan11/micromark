# micromark-util-symbol

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][bundle-size-badge]][bundle-size]
[![Sponsors][sponsors-badge]][opencollective]
[![Backers][backers-badge]][opencollective]
[![Chat][chat-badge]][chat]

[micromark][] utility with symbols.

## Contents

* [What is this?](#what-is-this)
* [When should I use this?](#when-should-i-use-this)
* [Install](#install)
* [Use](#use)
* [API](#api)
* [Types](#types)
* [Compatibility](#compatibility)
* [Security](#security)
* [Contribute](#contribute)
* [License](#license)

## What is this?

This package exposes constants used throughout the micromark ecosystem.

## When should I use this?

This package is useful when you are making your own micromark extensions.
It’s useful to reference these constants by name instead of value while
developing.
[`micromark-build`][micromark-build] compiles them away for production code.

## Install

This package is [ESM only][esm].
In Node.js (version 16+), install with [npm][]:

```sh
npm install micromark-util-symbol
```

In Deno with [`esm.sh`][esmsh]:

```js
import * as symbol from 'https://esm.sh/micromark-util-symbol@1'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import * as symbol from 'https://esm.sh/micromark-util-symbol@1?bundle'
</script>
```

## Use

```js
import {codes, constants, types, values} from 'micromark-util-symbol'

console.log(codes.atSign) // 64
console.log(constants.characterReferenceNamedSizeMax) // 31
console.log(types.definitionDestinationRaw) // 'definitionDestinationRaw'
console.log(values.atSign) // '@'
```

## API

This package exports the identifiers `codes`, `constants`, `types`, and
`values`.
There is no default export.

Each identifier is an object mapping strings to values.
See the code for the exposed data.

## Types

This package is fully typed with [TypeScript][].
It exports no additional types.

## Compatibility

Projects maintained by the unified collective are compatible with maintained
versions of Node.js.

When we cut a new major release, we drop support for unmaintained versions of
Node.
This means we try to keep the current release line,
`micromark-util-symbol@2`, compatible with Node.js 16.
This package works with `micromark@3`.

## Security

This package is safe.
See [`security.md`][securitymd] in [`micromark/.github`][health] for how to
submit a security report.

## Contribute

See [`contributing.md`][contributing] in [`micromark/.github`][health] for ways
to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organisation, or community you agree to
abide by its terms.

## License

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[author]: https://wooorm.com

[backers-badge]: https://opencollective.com/unified/backers/badge.svg

[build]: https://github.com/micromark/micromark/actions

[build-badge]: https://github.com/micromark/micromark/workflows/main/badge.svg

[bundle-size]: https://bundlejs.com/?q=micromark-util-symbol

[bundle-size-badge]: https://img.shields.io/badge/dynamic/json?label=minzipped%20size&query=$.size.compressedSize&url=https://deno.bundlejs.com/?q=micromark-util-symbol

[chat]: https://github.com/micromark/micromark/discussions

[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg

[coc]: https://github.com/micromark/.github/blob/main/code-of-conduct.md

[contributing]: https://github.com/micromark/.github/blob/main/contributing.md

[coverage]: https://codecov.io/github/micromark/micromark

[coverage-badge]: https://img.shields.io/codecov/c/github/micromark/micromark.svg

[downloads]: https://www.npmjs.com/package/micromark-util-symbol

[downloads-badge]: https://img.shields.io/npm/dm/micromark-util-symbol.svg

[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c

[esmsh]: https://esm.sh

[health]: https://github.com/micromark/.github

[license]: https://github.com/micromark/micromark/blob/main/license

[micromark]: https://github.com/micromark/micromark

[micromark-build]: https://github.com/micromark/micromark/tree/main/packages/micromark-build

[npm]: https://docs.npmjs.com/cli/install

[opencollective]: https://opencollective.com/unified

[securitymd]: https://github.com/micromark/.github/blob/main/security.md

[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg

[support]: https://github.com/micromark/.github/blob/main/support.md

[typescript]: https://www.typescriptlang.org
