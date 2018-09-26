# nano-i18n

Tiny experimental i18n library for JavaScript using [tagged template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

I'm starting small and evolving as the need arises.

## Installation

### Try it out

You can [try out the API on RunKit](https://npm.runkit.com/nano-i18n) without installing the library.

### As a Node.js module

Install the library using `npm` or `yarn`:

```bash
npm install nano-i18n
```

```bash
yarn add nano-i18n
```

### Using unpkg

You can load the library through a `<script>` tag using `unpkg.com`:

```html
<script src='https://unpkg.com/nano-i18n'></script>
```

## Usage

```js
import { load, k, v, t } from 'nano-i18n';

// 1. Load some translations

// Simple replacement
load(k`Hello, ${'World'}!`, v`Salut, ${0}!`);

// Swap the order of values in the translation
load(k`My name is ${'mine'}, yours is ${'yours'}`, v`Your name is ${1}, mine is ${0}`);

// 2. Translate some messages
console.log(t`Hello, ${'Dan'}!`);
// => "Salut, Dan!"

console.log(t`My name is ${'Dan'}, yours is ${'Alex'}`);
// => "Your name is Alex, mine is Dan"
```

## API reference

### Managing translations

§ **load**(key: _string_, value: _string_ | _function_)

Set up a translation:

```js
import { load } from 'nano-i18n';

load(k`Hello, ${'World'}!`, v`Salut, ${0}!`);
```

You can also add a batch of translations:

```js
import { load } from 'nano-i18n';

let translations = {};
translations[k`Hello, ${'World'}!`] = v`Salut, ${0}!`;
translations[k`My name is ${'name'}`] = v`Numele meu este ${0}`;

load(translations);
```

Or using the [computed property names](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names) syntax:

```js
import { load } from 'nano-i18n';

load({
	[k`Hello, ${'World'}!`]: v`Salut, ${0}!`,
	[k`My name is ${'name'}`]: v`Numele meu este ${0}`
});
```

Or skipping the `k` literal tag altogether and using a string key directly:

```js
import { load } from 'nano-i18n';

load('Hello, {}!', v`Salut, ${0}!`);
```

You can also skip the `v` literal tag and send a simple string translation:

```js
import { load } from 'nano-i18n';

load('Hello, World!', 'Salut, lume!');
```

§ **clear**()

Clears all the translations:

```js
import { clear } from 'nano-i18n';

clear();
```

§ **config**(options: _object_)

Configures the library. Available options:

_log_ (Number, default `0`)

The log level for the library. Possible values:

-   `0` disables logging
-   `1` logs a warning in the console on a missing translation
-   `2` throws an error when encountering a missing translation

_placeholder_ (String, default `{}`)

The string to use as placeholder for interpolated values when generating the key for a translation.

### Literal tags

§ **k**

Obtains the _key_ for a certain literal:

```js
import { k } from 'nano-i18n';

k`Hello, ${'World'}`;
// => "Hello, {}"
```

Keys are generated by replacing all values with the `{}` sequence.

**Note:** That means that currently, the strings you wish to translate may not contain the `{}` sequence.

You may change this by using the `config` method:

```js
import { config } from 'nano-i18n';

config({
	placeholder: '@@'
});
```

§ **v**

Generates a translation function for a literal:

```js
import { v } from 'nano-i18n';

v`Salut, ${0}!`;
// => function(strings, values) {}
```

When providing a translation, the interpolated values need to be numbers (i.e. `${0}`, `${1}`, et cetera). They don't necessarily need to be in sequential order, so you can swap them around if the translation needs it.

§ **t**

Get a translated string:

```js
import { t } from 'nano-i18n';

t`Hello, ${'Dan'}!`;
// => "Salut, Dan!"
```

If there's no translation available, it returns the normal interpolated string instead, and logs a warning to the console.

You can also use `t` as a normal function for translating dynamic strings:

```js
import { t } from 'nano-i18n';

t('Hello World');
```

This is useful for passing the `t` function to templating languages such as Mustache.

### Usage with Webpack

This library can be used with Webpack to load translations from external files with the `nano-i18n/loader` loader. `nano-i18n` comes with the loader, so you don't have to install any other packages.

**webpack.config.js**

```js
module.exports = {
	module: {
		rules: [
			{
				test: /\.json$/,
				include: /locales/,
				loader: 'nano-i18n/loader'
			}
		]
	}
};
```

By default, external files are expected to be JSONs containing an **array of `key`/`val` pairs**, i.e.:

**locales/ro-RO.json**

```js
[
	{ key: "Hello, ${'World'}", val: 'Salut, ${0}' },
	{ key: 'Some String', val: 'Un Șir Oarecare' }
	// etc.
];
```

With this setup in place, we can load the JSON as ready-to-use translations:

```js
import { load } from 'nano-i18n';
load(require('path/to/locales/ro-RO.json'));
```

`nano-i18n/loader` also accepts a `parse` option with you can use to keep the external translation files in any format, as long as you format it to `key`/`val` pairs.

Let's say you want to keep the translations in a CSV file so that it's easy to edit them in Microsoft Excel or macOS Numbers:

**locales/ro-RO.csv**

```csv
"key","val"
"Hello, ${'World'}","Salut, ${0}"
"Some String","Un Șir Oarecare"
```

In our Webpack configuration, we'll parse the CSV file using the [`d3-dsv`](https://github.com/d3/d3-dsv) package:

**webpack.config.js**

```js
let { csvParse } = require('d3-dsv');

module.exports = {
	module: {
		rules: [
			{
				test: /\.csv$/,
				include: /locales/,
				loader: 'nano-i18n/loader',
				options: {
					parse: text => csvParse(text)
				}
			}
		]
	}
};
```

> 💡 The `parse` option accepts a function which receives as its only argument the original text to parse, and which must return an array of `key`/`val` pairs.

## Further reading

-   [i18n with tagged template literals in ECMAScript 2015](https://jaysoo.ca/2014/03/20/i18n-with-es2015-template-literals/) by Jack Hsu
-   [Easy i18n in 10 lines of JavaScript (PoC)](https://codeburst.io/easy-i18n-in-10-lines-of-javascript-poc-eb9e5444d71e) by Andrea Giammarchi
-   [i18n Tagged Template Literals](http://i18n-tag.kolmer.net/) by Steffen Kolmer
