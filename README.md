# [**@f1stnpm2/earum-delectus-praesentium**](https://github.com/f1stnpm2/earum-delectus-praesentium)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
![npm (scoped)](https://img.shields.io/npm/v/@f1stnpm2/earum-delectus-praesentium)
![node-current (scoped)](https://img.shields.io/node/v/@f1stnpm2/earum-delectus-praesentium)
[![Build Status](https://img.shields.io/github/actions/workflow/status/f1stnpm2/earum-delectus-praesentium/git-build.yml?branch=master)](https://img.shields.io/github/actions/workflow/status/f1stnpm2/earum-delectus-praesentium/git-build.yml?branch=master)
[![Coverage Status](https://coveralls.io/repos/github/f1stnpm2/earum-delectus-praesentium/badge.svg?branch=master)](https://coveralls.io/github/f1stnpm2/earum-delectus-praesentium?branch=master)

> Create and verify HMAC's for JSON objects.

Easily create and verify [keyed-hash message authentication codes (HMAC's)](https://en.wikipedia.org/wiki/HMAC) for your JSON objects to ensure data integrity and authenticity.

The generated HMAC is independent of the JSON's attribute order and therefore stable for content-identical objects. See [calculateHmac](#calculatehmacobj-key).

Users of an older version prior to v1.1.0 please see the [important note](#note-for-users-of-older-versions-prior-to-v110).

## Usage

### Create and add HMAC to a JSON object

```js
const oh = require('@f1stnpm2/earum-delectus-praesentium');
const key = 'HmacSecret-0815';

let person = {
    name: 'Max',
    age: 32,
    hobbies: ['sports', 'travelling']
};

oh.createHmac(person, key);

// person = {
//   name: 'Max',
//   age: 32,
//   hobbies: ['sports','travelling'],
//   __hmac:'37c2e448b6f4a72c9d8abc9a1ab6cada602c3785148caeeed5498ed065ddc69f'
// }
```

### Verify HMAC for a JSON object

```js
// person = {
//   name: 'Max',
//   age: 32,
//   hobbies: ['sports','travelling'],
//   __hmac:'37c2e448b6f4a72c9d8abc9a1ab6cada602c3785148caeeed5498ed065ddc69f'
// }

const oh = require('@f1stnpm2/earum-delectus-praesentium');
const key = 'HmacSecret-0815';

let verification = oh.verifyHmac(person, key);
// true

person.age = 33;

let verificationAfterChange = oh.verifyHmac(person, key);
// false
```

### Only calculate HMAC for a JSON object

```js
const oh = require('@f1stnpm2/earum-delectus-praesentium');
const key = 'HmacSecret-0815';

let person = {
    name: 'Max',
    age: 32,
    hobbies: ['sports', 'travelling']
};

let hmac = oh.calculateHmac(person, key);
// 37c2e448b6f4a72c9d8abc9a1ab6cada602c3785148caeeed5498ed065ddc69f
```

## API

### createHmac(obj, key, hmacAttribute)

Calculates the HMAC of `obj` and attaches it as value of attribute `obj[hmacAttribute]`.

#### obj

Type: `Object`

The object to calculate and store the HMAC for.

#### key

Type: `String`

The key to calculate the objects HMAC.

#### hmacAttribute

Type: `String`
Default: `__hmac`

The name of the attribute to store the HMAC value in `obj`. Make sure that the name of the attribute is not overlapping with other attributes already in use.

### verifyHmac(obj, key, hmacAttribute)

Verifies the HMAC attached to `obj`. Returns `true` if the validation was successful, otherwise false `false`.

The verification would fail and return `false`, if...
- `obj` is null
- `obj` doesn't provide a HMAC to check against
- `obj` was manipulated: at least one attribute was changed, added or deleted (deep-inspection including all nested objects/arrays)
- the HMAC of `obj` was manipulated
- `key` is deviating from the one the HMAC was created with

The verification would not fail, just because the JSON's attributes order has changed. For more details see [calculateHmac](#calculatehmacobj-key).

#### obj

Type: `Object`

The object of which the HMAC should be verified. The given HMAC to be verified is assumed to exist as an attribute in the object itself: `obj[hmacAttribute]`.

#### key

Type: `String`

The key to calculate the objects HMAC and validate against the given one. Must be identical to the `key` that was used to create the original HMAC for the object for a successful verification.

#### hmacAttribute

Type: `String`
Default: `__hmac`

The name of the attribute for the HMAC value in `obj` to be verified against.

### calculateHmac(obj, key)

Calculates and returns the HMAC of `obj`.

Takes **all** of `obj` attributes into account for calculating the HMAC. So make sure that there isn't already a HMAC attribute created in the object. Otherwise this would also being used as an input for the calculation.

The calculation of the HMAC is independent of the order of your JSON's attributes. This means that the HMAC of content-identical objects with just another order of attributes will always by the same.

```js
let person = {
    name: 'Max',
    age: 32,
    hobbies: ['sports', 'travelling']
};

let hmac = oh.calculateHmac(person, key);

let person2 = {
    age: 32,
    hobbies: ['sports', 'travelling'],
    name: 'Max'
};

let hmac2 = oh.calculateHmac(person2, key);

/// (hmac === hmac2) is true
```

Please not that this order-independency does not apply to array elements. Arrays containing the same values in another order are not content-identical for obvious reasons. So the HMAC's of `{ hobbies: ['sports', 'travelling'] }` and `{ hobbies: ['travelling', 'sports'] }` are different.

#### obj

Type: `Object`

The object to calculate the HMAC for.

#### key

Type: `String`

The key to calculate the objects HMAC.

## Under the hood

To create and verify the HMAC, standard [NodeJS crypto functions](https://nodejs.org/docs/latest-v12.x/api/crypto.html#crypto_class_hmac) are used.

The HMAC is generated by using the following parameters:
- Hash function: SHA-256
- Digest output encoding: Hexadecimal String

To provide a stable (attribute-order independent) representation of the JSON object, a sorted traversal using the library [@tsmx/json-traverse](https://www.npmjs.com/package/@tsmx/json-traverse) is applied.

### Note for users of older versions prior to v1.1.0

Prior to v1.1.0 the algorithm used to generate the JSON's representation for the HMAC generation didn't to 100% guarantee a deterministic behaviour which could in some cases result in a failing verification although it should succeed.

Therefore it is **strongly recommended** to update to version 1.1.0 or higher. If you have any HMAC's persistently stored which where generated with a 1.0.x version you must re-calculate them with v1.1.0 or higher when upgrading.  

## Test

```
npm install
npm test
```