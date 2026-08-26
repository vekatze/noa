# noa

`noa` is a testing framework for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get noa https://github.com/vekatze/noa/raw/main/archive/0.5.1.tar.zst
```

## Types

### Main Definitions

```neut
data noa-kit

define make-noa-kit(
  sink: descriptor,
  buffer-capacity: int,
  test-count: int, // specifies the number of tests executed for each property.
  max-size: int, // specifies the max size of input generated in `check`.
  verbose: bool, // specifies whether to enable verbose output.
) ->> noa-kit

define make-default-noa-kit() ->> noa-kit

define make-verbose-noa-kit() ->> noa-kit

// Represents a test case.
data spec

// Performs all the given tests.
define check(k: &noa-kit, cases: list(spec)) -> unit

// Represents a value generator for property-based testing.
data gen(a) {
| Gen(run: <r>(&gen-kit) ->> control(r, a))
}

// Creates a property-based test case.
inline-meta property<a>(label: '&string, !g: 'gen(a), !prop: '(&a) -> bool) -> 'spec

// Creates a plain test case.
inline-meta example(label: '&string, !prop: 'bool) -> 'spec

// Creates a property-based test using a derived generator.
inline-meta quickprop<a>(label: '&string, !prop: '(&a) -> bool) -> 'spec
```

## Generators

```neut
inline gen-int(lo: int, hi: int, pivot: int) -> gen(int)

inline gen-float(lo: float, hi: float, pivot: float) -> gen(float)

inline-meta gen-array<a>(!g: 'gen(a)) -> 'gen(array(a))

define gen-list<a>(g: gen(a)) -> gen(list(a))

define gen-vector<a>(g: gen(a)) -> gen(vector(a))

// Chooses a value from `values` randomly.
define one-of<a>(values: list(a)) -> gen(a)
```

### Derivation

```neut
// Derives a generator for `a`.
inline-meta derive<a>() -> 'gen(a)

// Categories available through `derive`'s `rune-category` argument.
data rune-category {
| Printable-Ascii
| Full-Ascii
| Unicode-Scalar
| Alphanumeric
}
```

## Example

```neut
import {
  core::eq.generic {eq-data},
  core::list {append, reverse},
  this::gen.generic {Printable-Ascii, derive},
  this::suite {make-default-noa-kit},
  this::suite.spec {check, example, property},
}

define zen() -> unit {
  pin k = make-default-noa-kit();
  check(k, List::[
    // a property-based test
    property::(
      "reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)",
      derive::()[rune-category := Printable-Ascii],
      (p: &pair(list(rune), list(rune))) => {
        let Pair(!xs, !ys) = p;
        let left = append(reverse(ys), reverse(xs));
        let right = reverse(append(xs, ys));
        eq-data::(left, right)
      },
    ),
    // a plain test
    example::("the list `List::[1, 2, 3]` contains 2", eq-data::(List::[1, 2, 3], List::[1, 2, 3])),
  ])
}
```

## Example Outputs

You can quickly try `noa` on your machine as follows:

```sh
git clone https://github.com/vekatze/noa
cd noa
neut build test --execute
```

This should result in something like the following:

```text
✓ Pass: int stays within [-1000, 1000]
✓ Pass: derive rgb record
✓ Pass: one-shot List::[1, 2, 3] is non-empty
✗ Fail: fail: no list contains 15 (shrink to [15])
  → Cons(15, Nil)
✗ Fail: fail: one-of {2, 4, 6} is odd (shrink to 2)
  → 2
```

Verbose mode is also available:

```text
? Check: fail: no list contains 15 (shrink to [15])
  ✓ Vector[]
  ✓ Vector[0]
  ✓ Vector[1, 0]
  ...
  ✗ Vector[15]
    ↻ Shrink:
      ✓ Vector[]
      ✓ Vector[0]
      ✓ Vector[8]
      ✓ Vector[12]
      ✓ Vector[14]
    ✓ Stop
✗ Fail: fail: no list contains 15 (shrink to [15])
  → Cons(15, Nil)
```
