# Fast Check
##### Yet another property based testing framework _BUT_ written in TypeScript

[![Build Status](https://travis-ci.org/dubzzz/fast-check.svg?branch=master)](https://travis-ci.org/dubzzz/fast-check)
[![npm version](https://badge.fury.io/js/fast-check.svg)](https://badge.fury.io/js/fast-check)
[![dependencies Status](https://david-dm.org/dubzzz/fast-check/status.svg)](https://david-dm.org/dubzzz/fast-check)
[![devDependencies Status](https://david-dm.org/dubzzz/fast-check/dev-status.svg)](https://david-dm.org/dubzzz/fast-check?type=dev)

[![Coverage Status](https://coveralls.io/repos/github/dubzzz/fast-check/badge.svg)](https://coveralls.io/github/dubzzz/fast-check)
[![Test Coverage](https://api.codeclimate.com/v1/badges/7cb8cb395740446a3108/test_coverage)](https://codeclimate.com/github/dubzzz/fast-check/test_coverage)
[![Maintainability](https://api.codeclimate.com/v1/badges/7cb8cb395740446a3108/maintainability)](https://codeclimate.com/github/dubzzz/fast-check/maintainability)

## Getting started

Install the module with: `npm install fast-check`

## Usage

Using fast-check with [mocha](http://mochajs.org/) is really straightfoward.
It can be used directly in `describe`, `it` blocks with no extra care.

The following snippets written in Javascript shows an example featuring two properties:

```js
const fc = require('fast-check');

// Code under tests
const contains = (text, pattern) => text.indexOf(pattern) >= 0;

// Properties
describe('properties', () => {
    it('should always contain itself', () => {
        fc.assert(fc.property(fc.string(), text => contains(text, text)));
    });
    it('should always contain its substrings', () => {
        fc.assert(fc.property(fc.string(), fc.string(), fc.string(), (a,b,c) => contains(a+b+c, b)));
    });
});
```

In case of failure, the tests would raise a red flag and the output should help you to diagnose what went wrong in your implementation (example with a failing implementation of contain):

```
1) should always contain its substrings
    Property failed after 1 tests (seed: 1515709471288): [,,]
    Got error: Property failed by returning false
```

## Documentation

### Properties

- `fc.property`: define a new property ie. a list of arbitraries and a test function to assess the success

The predicate would be considered falsy if its throws or if `output == null || output == true` evaluate to `false`.
```typescript
function property<T1>(
        arb1: Arbitrary<T1>,
        predicate: (t1:T1) => (boolean|void)): Property<[T1]>;
function property<T1,T2>(
        arb1: Arbitrary<T1>, arb2: Arbitrary<T2>,
        predicate: (t1:T1,t2:T2) => (boolean|void)): Property<[T1,T2]>;
...
```

- `fc.asyncPoperty`: define a new property ie. a list of arbitraries and an asynchronous test function to assess the success

The predicate would be considered falsy if its throws or if `output == null || output == true` evaluate to `false` (after `await`).
```typescript
function asyncProperty<T1>(
        arb1: Arbitrary<T1>,
        predicate: (t1:T1) => Promise<boolean|void>): AsyncProperty<[T1]>;
function asyncProperty<T1,T2>(
        arb1: Arbitrary<T1>, arb2: Arbitrary<T2>,
        predicate: (t1:T1,t2:T2) => Promise<boolean|void>): AsyncProperty<[T1,T2]>;
...
```

## Runners

- `fc.assert`: run the property and throws in case of failure

**This function has to be awaited in case it is called on an asynchronous property.**

This function is ideal to be called in `describe`, `it` blocks.
It does not return anything in case of success.

It can be parametrized using its second argument.

```typescript
export interface Parameters {
    seed?: number;     // optional, initial seed of the generator: Date.now() by default
    num_runs?: number; // optional, number of runs before success: 100 by default 
    logger?: (v: string) => void; // optional, log output: console.log by default
}
```

```typescript
function assert<Ts>(property: IProperty<Ts>, params?: Parameters);
```

- `fc.check`: run the property and return an object containing the test status along with other useful details

**This function has to be awaited in case it is called on an asynchronous property.**

It should never throw whatever the status of the test.

It can be parametrized with the same parameters than `fc.assert`.

The details returned by `fc.check` are the following:

```typescript
interface RunDetails<Ts> {
    failed: boolean,         // false in case of failure, true otherwise
    num_runs: number,        // number of runs (all runs if success, up and including the first failure if failed)
    num_shrinks: number,     // number of shrinks (depth required to get the minimal failing example)
    seed: number,            // seed used for the test
    counterexample: Ts|null, // failure only: shrunk conterexample causig the property to fail
    error: string|null,      // failure only: stack trace and error details
}
```

```typescript
function check<Ts>(property: IProperty<Ts>, params?: Parameters);
```

- `fc.sample`: sample generated values of an `Arbitrary<T>` or `Property<T>`

It builds an array containing all the values that would have been generated for the equivalent test.

It also accept `Parameters` as configuration in order to help you diagnose the shape of the inputs that will be received by your property.

```typescript
type Generator<Ts> = Arbitrary<Ts> | IProperty<Ts>;

function sample<Ts>(generator: Generator<Ts>): Ts[];
function sample<Ts>(generator: Generator<Ts>, params: Parameters): Ts[];
function sample<Ts>(generator: Generator<Ts>, num_generated: number): Ts[];
```

- `fc.statistics`: classify the values produced by an `Arbitrary<T>` or `Property<T>`

It provides useful statistics concerning generated values.
In order to be able to gather those statistics it has to be provided with a classifier function that can classify the generated value in zero, one or more categories (free labels).

It also accept `Parameters` as configuration in order to help you diagnose the shape of the inputs that will be received by your property.

Statistics are dumped into `console.log` but can be redirected to another source by modifying the `logger` key in `Parameters`.

```typescript
type Generator<Ts> = Arbitrary<Ts> | IProperty<Ts>;
type Classifier<Ts> = ((v: Ts) => string) | ((v: Ts) => string[]);

function statistics<Ts>(generator: Generator<Ts>, classify: Classifier<Ts>): void;
function statistics<Ts>(generator: Generator<Ts>, classify: Classifier<Ts>, params: Parameters): void;
function statistics<Ts>(generator: Generator<Ts>, classify: Classifier<Ts>, num_generated: number): void;
```

## Arbitraries

Arbitraries are responsible for the random generation (but deterministic) and shrink of datatypes.
They can be combined together to build more complex datatypes.

### Boolean (:boolean)

- `fc.boolean()` either `true` or `false`

### Numeric (:number)

Integer values:

- `fc.integer()` all possible integers ie. from -2147483648 (included) to 2147483647 (included)
- `fc.integer(max: number)` all possible integers between -2147483648 (included) and max (included)
- `fc.integer(min: number, max: number)` all possible integers between min (included) and max (included)
- `fc.nat()` all possible positive integers ie. from 0 (included) to 2147483647 (included)
- `fc.nat(max: number)` all possible positive integers between 0 (included) and max (included)

Floating point numbers:

- `fc.float()` uniformly distributed `float` value between 0.0 (included) and 1.0 (excluded)
- `fc.double()`uniformly distributed `double` value between 0.0 (included) and 1.0 (excluded)

### String (:string)

Single character only:

- `fc.hexa()` one character in `0123456789abcdef` (lower case)
- `fc.base64()` one character in `A-Z`, `a-z`, `0-9`, `+` or `/`
- `fc.char()` between 0x20 (included) and 0x7e (included) , corresponding to printable characters (see https://www.ascii-code.com/)
- `fc.ascii()` between 0x00 (included) and 0x7f (included)
- `fc.unicode()` between 0x0000 (included) and 0xffff (included)

Multiple characters:

- `fc.hexaString()` or `fc.hexaString(maxLength: number)` string based on characters generated by `fc.hexa()`
- `fc.base64String()` or `fc.base64String(maxLength: number)` string based on characters generated by `fc.base64()`. Provide valid base 64 strings: length always multiple of 4 padded with '=' characters
- `fc.string()` or `fc.string(maxLength: number)` string based on characters generated by `fc.char()`
- `fc.asciiString()` or `fc.asciiString(maxLength: number)` string based on characters generated by `fc.ascii()`
- `fc.unicodeString()` or `fc.unicodeString(maxLength: number)` string based on characters generated by `fc.unicode()`

Strings that mimic real strings, with words and sentences:

- `fc.lorem()`, `fc.lorem(maxWordsCount: number)` or `fc.lorem(maxWordsCount: number, sentencesMode: boolean)` lorem ipsum strings. Generator can be configured by giving it a maximum number of characters by using `maxWordsCount` or switching the mode to sentences by setting `sentencesMode` to `true` in which case `maxWordsCount` is used to cap the number of sentences allowed. **This arbitrary is not shrinkable**

### Combinors of arbitraries (:T)

- `fc.constant<T>(value: T): Arbitrary<T>` constant arbitrary only able to produce `value: T`
- `fc.oneof<T>(...arbs: Arbitrary<T>[]): Arbitrary<T>` randomly chooses an arbitrary at each new generation. Should be provided with at least one arbitrary. All arbitraries are equally probable and shrink is still working for the selected arbitrary
- `fc.option<T>(arb: Arbitrary<T>): Arbitrary<T | null>` or `fc.option<T>(arb: Arbitrary<T>, freq: number): Arbitrary<T | null>` arbitrary able to nullify its generated value. When provided a custom `freq` value it changes the frequency of `null` values so that they occur one time over `freq` tries (eg.: `freq=5` means that 20% of generated values will be `null` and 80% would be produced through `arb`). By default: `freq=5`
- `fc.array<T>(arb: Arbitrary<T>): Arbitrary<T[]>` or `fc.array<T>(arb: Arbitrary<T>, maxLength: number): Arbitrary<T[]>` array of random length containing values generated by `arb`. By setting the parameter `maxLength`, the user can change the maximal size allowed for the generated array. By default: `maxLength=10`
- `fc.tuple<T1,T2,...>(arb1: Arbitrary<T1>, arb2: Arbitrary<T2>, ...): Arbitrary<[T1,T2,...]>` tuple generated by aggregating the values of `arbX` like `generate: () => [arb1.generate(), arb2.generate(), ...]`. This arbitrary perfectly handle shrinks and is able to shink on all the generators
