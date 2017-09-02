# otplib

> Time-based (TOTP) and HMAC-based (HOTP) One-Time Password library

[![npm][npm-badge]][npm-link]
[![Build Status][circle-badge]][circle-link]
[![Coverage Status][coveralls-badge]][coveralls-link]
[![npm downloads][npm-downloads-badge]][npm-link]
[![PRs Welcome][pr-welcome-badge]][pr-welcome-link]

<img width="150" src="https://yeojz.github.io/otplib/otplib.png" />

- [About](#about)
- [Demo and Documentation](#demo-and-documentation)
- [Installation](#installation)
- [Upgrading](#upgrading)
- [Getting Started](#getting-started)
  - [In node](#in-node)
    - [Using specific OTP implementation](#using-specific-otp-implementation)
    - [Using classes](#using-classes)
  - [In browser](#in-browser)
    - [Browser Compatibility](#browser-compatibility)
- [Advanced Usage](#advanced-usage)
- [Notes](#notes)
  - [Setting Custom Options](#setting-custom-options)
    - [Available Options](#available-options)
  - [Google Authenticator](#google-authenticator)
    - [Base32 Keys and RFC3548](#base32-keys-and-rfc3548)
    - [Seed / secret length](#seed-secret-length)
- [Contributing](#contributing)
- [License](#license)

## About

`otplib` is a JavaScript One Time Password (OTP) library.
It provides both functional and class based interfaces for
dealing with OTP generation and verification.

It implements both [HOTP][rfc-4226-wiki] - [RFC 4226][rfc-4226] and [TOTP][rfc-6238-wiki] - [RFC 6238][rfc-6238], and are tested against the test vectors provided in their respective RFC specifications. These datasets can be found in the `packages/tests` folder.

-   [RFC 4226 Dataset](https://github.com/yeojz/otplib/blob/master/packages/tests/rfc4226.js)
-   [RFC 6238 Dataset](https://github.com/yeojz/otplib/blob/master/packages/tests/rfc6238.js)

This library is also compatible with [Google Authenticator](https://github.com/google/google-authenticator), and includes additional methods to allow you to work with Google Authenticator.

## Demo and Documentation

-   [Documentation][project-docs]
-   [Demo][project-web]

## Installation

Install the library via:

```
$ npm install otplib --save
```

## Upgrading

This library follows `semver`. As such, major version bumps usually mean API changes or behavior changes. Please check [upgrade notes](https://github.com/yeojz/otplib/wiki/upgrade-notes) for more information, especially before making any major upgrades.

You might also want to check out the release notes associated with each tagged versions in the [releases](https://github.com/yeojz/otplib/releases) page.

## Getting Started

### In node

```js
import otplib from 'otplib';

const secret = otplib.authenticator.generateSecret();
const token = otplib.authenticator.generate(secret);
const isValid = otplib.authenticator.check(123456, secret);

// or

const isValid = otplib.authenticator.verify({
  secret,
  token: 123456
});

```

#### Using specific OTP implementation

If you want to include a specific OTP specification, you can import it directly:

```js
import hotp from 'otplib/hotp';
import totp from 'otplib/totp';
import authenticator from 'otplib/authenticator';
```

Do note that you'll have to provide a crypto solution (this is to allow custom crypto solutions),
as long as they implement `createHmac` and `randomBytes`. Take a look at the
[browser implementation](https://github.com/yeojz/otplib/blob/master/packages/otplib-browser) for an example

i.e.

```js
import authenticator from 'otplib/authenticator';
import crypto from 'crypto';
authenticator.options = {crypto}

// Or if you're using the other options
// hotp.options = {crypto}
// totp.options = {crypto}
```

#### Using classes

For ease of use, the default exports are all instantiated instances of their respective classes.
You can access the original classes via it's same name property of an instantiated class.

i.e

```js

import hotp from 'otplib/hotp';
const HOTP = hotp.HOTP;

import totp from 'otplib/totp';
const TOTP = totp.TOTP;

import authenticator from 'otplib/authenticator';
const Authenticator = authenticator.Authenticator;
```

### In browser

A browser-targeted version has been compiled.
You'll need to add the following scripts to your code:

```html
<script src="otplib-browser.js"></script>

<script type="text/javascript">
   // window.otplib
</script>
```

You can find it in `node_modules/otplib` after you install.

Alternatively you can

-   Download from [gh-pages][project-lib].
-   Use unpkg.com

```html
<script src="https://unpkg.com/otplib@6.0.0/otplib-browser.js"></script>
```

For a live example, the [project site][project-web] has been built using `otplib-browser.js`. The source code can be found [here](https://github.com/yeojz/otplib/tree/master/site).

#### Browser Compatibility

In order to reduce the size of the browser package, the `crypto` package has been replaced with a alternative implementation. The current implementation depends on [Uint8Array][mdn-uint8array] and the browser's native [crypto][mdn-crypto] methods, which may only be available in recent browser versions.

To find out more about the replacements, you can take a look at `packages/otplib-browser/crypto.js`

__Output sizes:__

-   with node crypto: ~311Kb
-   with alternative crypto: ~96Kb

## Advanced Usage

This library is primarily a node module (cjs), with a umd browser package provided.

| file                                                                                     | description                                              |
| ---------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| [authenticator.js](https://yeojz.github.io/otplib/docs/module-otplib-authenticator.html) | Google Authenticator bindings                            |
| [browser.js](https://yeojz.github.io/otplib/docs/module-otplib-browser.html.html)        | Browser compatible package built with webpack            |
| [core.js](https://yeojz.github.io/otplib/docs/module-otplib-core.html)                   | Provides all functional parts of the library             |
| [hotp.js](https://yeojz.github.io/otplib/docs/module-otplib-hotp.html)                   | Wraps the functional core into a instantiated HOTP class |
| [otplib.js](https://yeojz.github.io/otplib/docs/module-otplib.html)                      | Entry file for this library                              |
| [totp.js](https://yeojz.github.io/otplib/docs/module-otplib-totp.html)                   | Wraps the functional core into a instantiated TOTP class |
| [utils.js](https://yeojz.github.io/otplib/docs/module-otplib-utils.html)                 | Helper utilities                                         |

For more information about the functions and available files, check out the [documentation][project-docs].

## Notes

### Setting Custom Options

All instantiated classes will have their options inherited from their respective options generator. i.e. HOTP from `hotpOptions` and TOTP/Authenticator from `totpOptions`.

All OTP classes have an object setter and getter method to override these default options.

For example,

```js
import otplib from 'otplib';

// setting
otplib.authenticator.options = {
   step: 30
}

// getting
const opts = otplib.authenticator.options;
```

#### Available Options

| Option           | Type     | Defaults                             | Description                                                                                                   |
| ---------------- | -------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| algorithm        | string   | 'sha1'                               | Algorithm used for HMAC                                                                                       |
| createHmacSecret | function | (hotp) hotpSecret, (totp) totpSecret | Transforms the secret and applies any modifications like padding to it.                                       |
| crypto           | object   | node crypto                          | Crypto module to use.                                                                                         |
| digits           | integer  | 6                                    | The length of the token                                                                                       |
| epoch (totp)     | integer  | null                                 | starting time since the UNIX epoch (seconds). *Note* non-javascript epoch. i.e. `new Date().getTime() / 1000` |
| step (totp)      | integer  | 30                                   | Time step (seconds)                                                                                           |

### Google Authenticator

#### Base32 Keys and RFC3548

Google Authenticator requires keys to be base32 encoded.
It also requires the base32 encoder to be [RFC 3548][rfc-3548] compliant.

OTP calculation will still work should you want to use other base32 encoding methods (like Crockford's Base 32) but it will NOT be compatible with Google Authenticator.

```js
import authenticator from 'otplib/authenticator';

const secret = authenticator.generateSecret(); // base 32 encoded hex secret key
const token = authenticator.generate(secret);
```

#### Seed / secret length

In [RFC 6238][rfc-6238], the secret / seed length for different algorithms is predefined:

```
HMAC-SHA1 - 20 bytes
HMAC-SHA256 - 32 bytes
HMAC-SHA512 - 64 bytes
```

As such, the length of the secret is padded and sliced according to the expected length for respective algrorithms.

## Contributing

-   Check out: [CONTRIBUTING.md][pr-welcome-link]

## License

`otplib` is [MIT licensed](./LICENSE)

[npm-badge]: https://img.shields.io/npm/v/otplib.svg?style=flat-square
[npm-link]: https://www.npmjs.com/package/otplib
[npm-downloads-badge]: https://img.shields.io/npm/dt/otplib.svg?style=flat-square

[circle-badge]: https://img.shields.io/circleci/project/github/yeojz/otplib/master.svg?style=flat-square
[circle-link]: https://circleci.com/gh/yeojz/otplib

[coveralls-badge]: https://img.shields.io/coveralls/yeojz/otplib/master.svg?style=flat-square
[coveralls-link]: https://coveralls.io/github/yeojz/otplib

[pr-welcome-badge]: https://img.shields.io/badge/PRs-Welcome-ff69b4.svg?style=flat-square
[pr-welcome-link]: https://github.com/yeojz/otplib/blob/master/CONTRIBUTING.md

[mdn-uint8array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array
[mdn-crypto]: https://developer.mozilla.org/en-US/docs/Web/API/Window/crypto

[project-web]: https://yeojz.github.io/otplib
[project-docs]: https://yeojz.github.io/otplib/docs
[project-lib]: https://github.com/yeojz/otplib/tree/gh-pages/lib

[rfc-4226]: http://tools.ietf.org/html/rfc4226
[rfc-6238]: http://tools.ietf.org/html/rfc6238
[rfc-3548]: http://tools.ietf.org/html/rfc3548

[rfc-4226-wiki]: http://en.wikipedia.org/wiki/HMAC-based_One-time_Password_Algorithm
[rfc-6238-wiki]: http://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm