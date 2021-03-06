# GaloisFields.jl - finite fields for Julia

| **Build Status**                                                | **Test coverage**                                       |
|:---------------------------------------------------------------:|:-------------------------------------------------------:|
| [![][travis-img]][travis-url] [![][appveyor-img]][appveyor-url] | [![Coverage Status][codecov-img]][codecov-url]      |

## Introduction

This module defines types representing [finite fields][galois-fields-wiki]. It
supports both fields of prime order and of prime power order.

[galois-fields-wiki]: https://en.wikipedia.org/wiki/Finite_field

## Synopsis

The easiest way to create Galois fields is with the `@GaloisField` and `@GaloisField!`
macros. Typically, you use the former for a field of prime order and the latter
for a field of prime power order. In the prime power case, you pass a display
name / variable name for the primitive element.

```julia
using GaloisFields

const F = @GaloisField 29     # ℤ/29ℤ
const G = @GaloisField! 27 β   # degree-3 extension of ℤ/3ℤ; multiplicatively generated by β

F(2)^29 == F(2)
β^27 == β
```

The exclamation mark `!` is intended to convey that the macro has a side-effect:
for example, in the code above, it assigns a variable called `β`.

The macros also accept special symbols for specifying the field. This is more
difficult to type ([docs][unicode-input]) but more elegant to read:

```julia
const F = @GaloisField ℤ/29ℤ
const G = @GaloisField 𝔽₂₇ β
```

If you want to pass your own generator for the representation of a field
of order ``q = p^n``, you can:

```julia
const F = @GaloisField! 𝔽₃ β^2 + β + 2
β^2 + β + 2 == 0
```

Lastly, there's also function interfaces in cases where macros are not
appropriate:

```julia
const F = GaloisField(29)               # ℤ/29ℤ
const G, β = GaloisField(81, :β)        # degree-4 extension of ℤ/3ℤ
const G, β = GaloisField(3, 4, :β)      # same; avoid having to factorize 81
const F, β = GaloisField(3, :β => [2, 0, 0, 2, 1]) # same; pass our own custom minimum polynomial
```

## Fast multiplications
In some cases, we make use of [Zech's logarithms][zech] for faster multiplications.
By default, this happens if the order of the field is less than ``2^16``, if the
characteristic is not 2, and if the primitive element is also a multiplicative
generator. However, you can override this by calling either of

```julia
GaloisFields.enable_zech_multiplication(F)
GaloisFields.disable_zech_multiplication(F)
```

_before_ doing any multiplication operation. If you call this function on a
field whose primitive element is _not_ a multiplicative generator, this will
throw a warning.

[zech]: https://en.wikipedia.org/wiki/Zech's_logarithm

## Conversions
If you specify your own minimum polynomial, we make no assumptions about
conversions between fields. For example, when defining
```julia
const F = @GaloisField! 𝔽₂ β^2 + β + 1
const G = @GaloisField! 𝔽₂ γ^2 + γ + 1
```
an operation like
```julia
G(β)
```
will throw an error. The mathematical reason is that  the fields ``F`` and ``G``
are isomorphic, but there is two different isomorphisms. ("They are not _canonically_
isomorphic.") To choose an identification, you can use the `identify` function
(which is not exported by default, so we use its full path):
```julia
GaloisFields.identify(β => γ^2)
GaloisFields.identify(γ => β^2)
```
This allows for conversions such as
```julia
G(β)
convert(F, γ + 1)
```
The inner workings of this distinction are based on the symbol names. So
if you define ``F`` and ``G`` with the _same_ symbol and minimum polynomial:
```julia
const F = @GaloisField! 𝔽₂ β^2 + β + 1
const G = @GaloisField! 𝔽₂ β^2 + β + 1
```
then they are just considered equal and conversions work without extra work.

## Conversions for the default minimum polynomials
If you do not specify a minimum polynomial, for example by using
```julia
const F = @GaloisField! 𝔽₈₁ β
const G = @GaloisField! 𝔽₉ γ
```
then we use [Conway polynomials][conway]. They have special compatibility
relations between them, allowing conversions:
```julia
β^10 == γ
```
This works provided `F` and `G` have the same characteristic `p`. If the order
of either is a power of the other, we convert into the bigger field. If not, we
convert both into the field of order `p^N`, where `N` is the least common
multiple of the extension degrees of `F` and `G` over ℤ/pℤ.

## Acknowledgements

This package uses [Frank Lübeck's][lubeck] [database of Conway polynomials][db].
For security, we make a [copy available over https][https-db] for this package.
It is downloaded as part of the install process.

[conway]: https://en.wikipedia.org/wiki/Conway_polynomial_(finite_fields)


[travis-img]: https://travis-ci.org/tkluck/GaloisFields.jl.svg?branch=master
[travis-url]: https://travis-ci.org/tkluck/GaloisFields.jl

[appveyor-img]: https://ci.appveyor.com/api/projects/status/74ti0cx7honk82yj?svg=true
[appveyor-url]: https://ci.appveyor.com/project/tkluck/galoisfields-jl

[codecov-img]: https://codecov.io/gh/tkluck/GaloisFields.jl/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/tkluck/GaloisFields.jl

[lubeck]: http://www.math.rwth-aachen.de/~Frank.Luebeck/index.html
[db]: http://www.math.rwth-aachen.de/~Frank.Luebeck/data/ConwayPol/index.html?LANG=en
[https-db]: https://gist.githubusercontent.com/tkluck/e1cd1746c69aa17e4a37114d22649627/raw/7fbe9763fae27f14924262ad03606f1c3af4400e/CPImport.txt
[unicode-input]: https://docs.julialang.org/en/v1.1/manual/unicode-input/#Unicode-Input-1
