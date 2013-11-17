# JSVerify

> Property based checking.

[![Build Status](https://secure.travis-ci.org/phadej/jsverify.png?branch=master)](http://travis-ci.org/phadej/jsverify)
[![NPM version](https://badge.fury.io/js/jsverify.png)](http://badge.fury.io/js/jsverify)
[![Dependency Status](https://gemnasium.com/phadej/jsverify.png)](https://gemnasium.com/phadej/jsverify)
[![Code Climate](https://codeclimate.com/github/phadej/jsverify.png)](https://codeclimate.com/github/phadej/jsverify)

## Getting Started

Install the module with: `npm install jsverify`

## Synopsis

```js
var jsc = require("jsverify");

// forall (f : bool -> bool) (b : bool), f (f (f b)) = f(b).
var bool_fn_applied_thrice =
  jsc.forall(jsc.fun(jsc.bool()), jsc.bool(), function (f, b) {
    return f(f(f(b))) === f(b);
  });

jsc.assert(bool_fn_applied_thrice);
// OK, passed 100 tests
```

## Documentation

### Use with [jasmine](http://pivotal.github.io/jasmine/) 1.3.x

Check [jasmineHelpers.js](speclib/jasmineHelpers.js) file.

## API

> _Testing shows the presence, not the absence of bugs._
>
> Edsger W. Dijkstra

To show that propositions hold, we need to construct proofs.
There are two extremes: proof by example (unit tests) and formal (machine-checked) proof.
Property-based testing is something in between.
We formulate propositions, invariants or other properties we believe to hold, but
only test it to hold for numerous (random generated) values.

Types and function signatures are written in [Coq](http://coq.inria.fr/)/[Haskell](http://www.haskell.org/haskellwiki/Haskell) influented style:
C# -style `List<T> filter(List<T> v, Func<T, bool> predicate)` is represented by
`filter (v : array T) (predicate : T -> bool) : array T` in our style.

`jsverify` can operate with both synchronous and asynchronous-promise properties.
Generally every property can be wrapped inside [functor](http://learnyouahaskell.com/functors-applicative-functors-and-monoids),
for now in either identity or promise functor, for synchronous and promise properties respectively.

Some type definitions to keep developers sane:

- Functor f => property (size : nat) : f result
- result := true | { counterexample: any }
- Functor f => property_rec := f (result | property)
- generator a := { arbitrary : a, shrink : a -> [a] }

### Properties

#### forall (gens : generator a ...) (prop : a -> property_rec) : property

Property constructor

#### check (prop : property) (opts : checkoptions) : promise result + result

Run random checks for given `prop`. If `prop` is promise based, result is also wrapped in promise.

Options:
- `opts.tests` - test count to run, default 100
- `opts.size`  - maximum size of generated values, default 5
- `opts.quiet` - do not `console.log`

#### assert (prop : property) (opts : checkoptions) : void

Same as `check`, but throw exception if property doesn't hold.

### Primitive generators

#### integer (maxsize : nat) : generator integer

Integers, ℤ

#### nat (maxsize : nat) : generator nat

Natural numbers, ℕ (0, 1, 2...)

#### number (maxsize : number) : generator number

JavaScript numbers, "doubles", ℝ. `NaN` and `Infinity` are not included.

#### bool () : generator bool

Booleans, `true` or `false`.

#### oneof (args : array any) : generator any

Random element of `args` array.

#### string () : generator string

Strings

#### value : generator value

JavaScript value: boolean, number, string, array of values or object with `value` values.

#### array (gen : generator a) : generator (array a)

#### pair (a : generator A) (b : generator B) : generator (A * B)

If not specified `a` and `b` are equal to `value()`.

#### fun (gen : generator a) : generator (b -> a)

Unary functions.

### Generator combinators

#### suchthat (gen : generator a) (p : a -> bool) : generator {a | p a == true}

Generator of values that satisfy `p` predicate. It's adviced that `p`'s accept rate is high.

#### nonshrink (gen : generator a) : generator a

Non shrinkable version of generator `gen`.

### jsc._ - miscellaneous utilities

#### assert (exp : bool) (message : string) : void

Throw an error with `message` if `exp` is falsy.
Resembles [node.js assert](http://nodejs.org/api/assert.html).

#### isEqual (a b : value) : bool

Equality test for `value` objects. See `value` generator.

#### random (min max : int) : int

Returns random int from `[min, max]` range inclusively.

```js
getRandomInt(2, 3) // either 2 or 3
```

#### random.number (min max : number) : number

Returns random number from `[min, max)` range.

#### FMap (eq : a -> a -> bool) : FMap a

Finite map, with any object a key.

Short summary of member functions:

- FMap.insert (key : a) (value : any) : void
- FMap.get (key : a) : any
- FMap.contains (key : a) : obool


## Contributing

In lieu of a formal styleguide, take care to maintain the existing coding style.

- Add unit tests for any new or changed functionality.
- Lint and test your code using [Grunt](http://gruntjs.com/).
- Use `istanbul cover grunt simplemocha` to run tests with coverage with [istanbul](http://gotwarlost.github.io/istanbul/).
- Create a pull request

### Before release

Don't add `README.md` or `jsverify.standalone.js` into pull requests.
They will be regenerated before each release.

- run `npm run-script prepare-release`
   - run `grunt literate` to regenerate `README.md`
   - run `npm run-script browserify` to regenerate `jsverify.standalone.js`

## Release History

- 0.2.0 Use browserify
- 0.1.4 Mocha test suite
    - major cleanup
- 0.1.3 gen.show and exception catching
- 0.1.2 Added jsc.assert
- 0.1.1 Use grunt-literate
- 0.1.0 Usable library
- 0.0.2 Documented preview
- 0.0.1 Initial preview

## Related work

### JavaScript

- [JSCheck](http://www.jscheck.org/)
- [claire](https://npmjs.org/package/claire)
- [gent](https://npmjs.org/package/gent)
- [fatcheck](https://npmjs.org/package/fatcheck)
- [quickcheck](https://npmjs.org/package/quickcheck)
- [qc.js](https://bitbucket.org/darrint/qc.js/)

### Others

- [Wikipedia - QuickCheck](http://en.wikipedia.org/wiki/QuickCheck)
- [Haskell - QuickCheck](http://hackage.haskell.org/package/QuickCheck) [Introduction](http://www.haskell.org/haskellwiki/Introduction_to_QuickCheck1)
- [Erlang - QuviQ](http://www.quviq.com/index.html)
- [Erlang - triq](https://github.com/krestenkrab/triq)
- [Scala - ScalaCheck](https://github.com/rickynils/scalacheck)
Copyright Oleg Grenrus 2013

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.

    * Redistributions in binary form must reproduce the above
      copyright notice, this list of conditions and the following
      disclaimer in the documentation and/or other materials provided
      with the distribution.

    * Neither the name of Oleg Grenrus nor the names of other
      contributors may be used to endorse or promote products derived
      from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
