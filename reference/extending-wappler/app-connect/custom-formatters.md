# Custom Formatters

### Introduction

Writing your own formatters can be easily done if you have common knowledge about JavaScript. We will explain how you register your own formatters and give some samples.

Formatters are registered for a specific data type, the types available in App Connect are `string`, `number`, `boolean`, `array`, `object`, and `null`. There are also some special types `undefined`, `global` and `any`. We will explain them all in some examples.

There are 2 ways to register formatters, the first is registering a single formatter using `dmx.Formatter(datatype, name, function)` and secondly for registering multiple formatters for a specific datatype using `dmx.Formatters(datatype, object)`.

The first argument of the formatter function is the input data, the other arguments are the ones passed in the expression. It is important that the function has no side effects, when the same input is given it should always return the same output again.

Formatters should only be used for formatting data and are always sync functions. When you need async function or functions with side effects you should create a flow action instead.

#### Registering multiple string formatters

```javascript
dmx.Formatters('string', {
  escapeHTML: function(str) {
    return str
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/'/g, '&#39;')
      .replace(/"/g, '&quot;');
  },

  unescapeHTML: function(str) {
    return str
      .replace(/&amp;/g, '&')
      .replace(/&lt;/g, '<')
      .replace(/&gt;/g, '>')
      .replace(/&#39;/g, "'")
      .replace(/&quot;/g, '"');
  },

  stripHTML(str) {
    return str.replace(/<[^>]*/g, '');
  },
});

```

#### Registering a single formatter function with argument

```javascript
// using the expression {{"ha".repeat(4)}} will return "hahahaha"
dmx.Formatter('string', 'repeat', function(str, n) {
  return new Array(n + 1).join(str);
});
```

#### More Examples

```javascript
// Localization using Intl

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat
dmx.Formatter('number', 'formatLocaleNumber', function(num, locales, options) {
  // input: {{(123456.789).formatLocalNumber('de-DE', {style: 'currency', currency: 'EUR'})}}
  // output: "123.456,79 €"
  return new Intl.NumberFormat(locales, options).format(num);
});

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat
dmx.Formatter('array', 'formatLocaleList', function(arr, locales, options) {
  // input: {{['Motorcycle', 'Bus', 'Car'].formatLocaleList('de', {style: 'long', type: 'conjunction'})}}
  // output: "Motorcycle, Bus, and Car"
  return new Intl.ListFormat(locales, options).format(arr);
});

// Datetime are passed as UTC strings and not as Date objects.
// Most browsers understand the ISO standard and you can simply use
// new Date(dateString) to get the date. Alternative you can use
// the dmx.parseDate() to parse the string, it also converts numbers
// as unix timestamps and the string `now` will return the current date.

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat
// We use any type to make the formatter available on all data types
dmx.Formatter('any', 'formatLocaleDate', function(obj, locales, options) {
  // input: {{'2022-12-20'.formatLocaleDate('en-US)}}
  // output: "12/20/2020"
  return new Intl.DateTimeFormat(locales, options).format(dmx.parseDate(obj));
});

// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat/format
dmx.Formatter('number', 'formatLocaleRelativeTime', function(num, unit, locales, options) {
  // input: {{(-1).formatLocaleRelativeTime('day', 'en', {style: 'narrow'})}}
  // output: "1 day ago"
  // input: {{(2).formatLocaleRelativeTime('day', 'es', {numeric: 'auto'})}}
  // output: "pasado mañana"
  return new Intl.RelativeTimeFormat(locales, options).format(num, unit);
});

// Global formatters are not applied on data but are global functions.
// The following formatter will log the data in the console.
// The expression {{debug('calc:', 20*4)}} will return 80 and show "calc: 80" in the console.
dmx.formatter('global', 'debug', function(name, obj) {
  console.log(name, obj);
  return obj;
});
```

For complex functions that use heavy calculations you can use Memoization which is an optimization technique that makes the application more efficient and hence faster. It does this by storing computation results in cache, and retrieving that same information from the cache the next time it’s needed instead of computing it again.

```javascript
// Simple example with lookup table.
// We wrap it in an anonymous function to prevent the variables from leaking
// to the global scope where they can potentially cause some conflicts.
(function() {
  // We use a Map for our cache
  const cache = new Map();

  dmx.Formatter('number', 'heavyCalculation', function(n) {
    if (!cache.has(n)) {
      const result = doHeavyCalculation(n);
      cache.set(n, result);
      return result;
    }

    return cache.get(n);
  });
})()

// Example using the Fibonacci sequence and a recursive function.
// https://www.scaler.com/topics/javascript-memoization/
dmx.Formatter('number', 'fib', function fib(n, memo) {
  memo = memo || {};
  if (memo[n]) return memo[n];
  if (n <= 1) return n;
  return memo[n] = fib(n-1, memo) + fib(n-2, memo);
});
```

Some data types that are recognized but currently not officially supported are `date`, `regexp`, `blob`, `file`, `filelist`, `arraybuffer`, `imagebitmap`, `imagedata`, `map` and `set`.
