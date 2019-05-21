# redact-pii

[![NPM Package](https://badge.fury.io/js/redact-pii.svg)](https://www.npmjs.com/package/redact-pii)
[![Dependencies](https://david-dm.org/solvvy/redact-pii.svg)](https://david-dm.org/solvvy/redact-pii)

> **NOTE**: Users of redact-pii@2.x.x please check the [Changelog](CHANGELOG.md) before upgrading .

Remove personally identifiable information from text. 

### Prerequesites

This library is primarily written for node.js but it should work in the browser as well.
It is written in TypeScript and compiles to ES2017. The library makes use of `async` functions and hence needs node.js 8.0.0 or higher (or a modern browser). If this is a problem for you please open an issue and we may consider adapting the compiler settings to support older node.js versions.

### Simple example (synchronous API)

```
npm install redact-pii
```

```js
const { SyncRedactor } = require('redact-pii');
const redactor = new SyncRedactor();
const redactedText = redactor.redact('Hi David Johnson, Please give me a call at 555-555-5555');
// Hi NAME, Please give me a call at PHONE_NUMBER
console.log(redactedText);
```

### Simple example (asynchronous / promise-based API)

```js
const { AsyncRedactor } = require('redact-pii');
const redactor = new AsyncRedactor();
redactor.redactAsync('Hi David Johnson, Please give me a call at 555-555-5555').then(redactedText => {
  // Hi NAME, Please give me a call at PHONE_NUMBER
  console.log(redactedText);
});
```

## Supported Features

- sync and async API variants
- ability to customize what to use as replacement value for detected patterns
- built in regex based redaction rules for:
  - credentials
  - creditCardNumber
  - emailAddress
  - ipAddress
  - name
  - password
  - phoneNumber
  - streetAddress
  - username
  - usSocialSecurityNumber
  - zipcode
  - url
  - digits
  - > **NOTE**: the built-in redaction rules are mostly applicable for identifying (US-)english PII.

## Advanced usage and features

### Customize replacement values

```js
const { SyncRedactor } = require('redact-pii');

// use a single replacement value for all built-in patterns found.
const redactor = new SyncRedactor({ globalReplaceWith: 'TOP_SECRET' });
redactor.redact('Dear David Johnson, I live at 42 Wallaby Way');
// Dear TOP_SECRET, I live at TOP_SECRET

// use a custom replacement value for a specific built-in pattern
const redactor = new SyncRedactor({
  builtInRedactors: {
    names: {
      replaceWith: 'ANONYMOUS_PERSON'
    }
  }
});

redactor.redact('Dear David Johnson');
// Dear ANONYMOUS_PERSON
```

### Add custom patterns or redaction functions

Note that the order of redaction rules matters, therefore you have to decide whether you want your custom redaction rules to run `before` or `after` the built-in ones. Generally it's better to put very specialized patterns or functions `before` the built-in ones and more broad / general ones `after`.

```js
const { SyncRedactor } = require('redact-pii');

// add a custom regexp pattern
const redactor = new SyncRedactor({
  customRedactors: {
    before: [
      {
        regexpPattern: /\b(cat|dog|cow)s?\b/gi,
        replaceWith: 'ANIMAL'
      }
    ]
  }
});

redactor.redact('I love cats, dogs, and cows');
// I love ANIMAL, ANIMAL, and ANIMAL

// add a synchronous custom redaction function
const redactor = new SyncRedactor({
  customRedactors: {
    before: [
      {
        redact(textToRedact) {
          return textToRedact.includes('TopSecret')
            ? 'THIS_FILE_IS_SO_TOP_SECRET_WE_HAD_TO_REDACT_EVERYTHING'
            : textToRedact;
        }
      }
    ]
  }
});

redactor.redact('This document is classified as TopSecret.')
// THIS_FILE_IS_SO_TOP_SECRET_WE_HAD_TO_REDACT_EVERYTHING


import { AsyncRedactor } from './src/index';

// add an asynchronous custom redaction function
const redactor = new AsyncRedactor({
  customRedactors: {
    before: [
      {
        redactAsync(textToRedact) {
          return myCustomRESTApiServer.redactCustomWords(textToRedact);
        }
      }
    ]
  }
});
```

### Disable specific built-in redaction rules

```js
const redactor = new SyncRedactor({
  builtInRedactors: {
    names: {
      enabled: false
    },
    emailAddress: {
      enabled: false
    }
  }
});
```

### Contributing

#### Run tests

You can run the tests via `npm run test`.
