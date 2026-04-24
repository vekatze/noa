# noa

`noa` is a testing framework for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get noa https://github.com/vekatze/noa/raw/main/archive/0-4-22.tar.zst
```

## Types

### Main Definitions

```neut
data noa-kit

define make-noa-kit(
  sink: descriptor,
  buffer-capacity: int,
  num-of-tests: int, // specifies the number of tests executed by `check`.
  max-size: int, // specifies the max size of input generated in `check`.
  verbose: bool, // specifies whether to enable verbose output.
) -> noa-kit

define make-default-noa-kit() -> noa-kit

define make-verbose-noa-kit() -> noa-kit

// performs property-based testing
define-meta check<a>(k: '&noa-kit, label: '&string, !g: 'gen(a), !predicate: '(a) -> bool) -> 'unit

// performs plain testing
define test(k: &noa-kit, label: &string, property: () -> bool) -> unit

// Represents a value generator for property-based testing.
data gen(a) {
| Gen(
    generate: (sample-size) -> a, // creates a value of type `a`, no larger than the given size.
    shrink: (a) -> list(a), // creates a list of values that are "smaller" than the input value.
  )
}

// draws a sample from a generator
define draw<a>(s: sample-size, g: gen(a)) -> a
```

## Preset Generators

```neut
constant bool-gen: gen(bool)

constant float-gen: gen(float)

constant int-gen: gen(int)

constant positive-int-gen: gen(int)

constant negative-int-gen: gen(int)

constant rune-gen: gen(rune)

constant ascii-rune-gen: gen(rune)

constant string-gen: gen(string)

constant ascii-string-gen: gen(string)

inline list-gen<a>(!g: gen(a)) -> gen(list(a))

inline pair-gen<a, b>(!g1: gen(a), !g2: gen(b)) -> gen(pair(a, b))

inline either-gen<a, b>(g1: gen(a), g2: gen(b)) -> gen(either(a, b))

inline vector-gen<a>(!g: gen(a)) -> gen(vector(a))

// Chooses a value from `Cons(x, xs)` randomly.
define one-of<a>(xs: &list(a), x: &a) -> gen(a)

// Generates a value of type `a` or generates `none`
define optional<a>(!g: gen(a)) -> gen(?a)
```

## Example

```neut
import {
  core.list {append, reverse},
  this.check {check},
  this.gen.list {list-gen},
  this.gen.pair {pair-gen},
  this.gen.rune {rune-gen},
  this.make-noa-kit {make-default-noa-kit},
  this.test {test},
}

define zen() -> unit {
  pin k = make-default-noa-kit();
  // a property-based test
  check::(
    k,
    "reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)",
    pair-gen(list-gen(rune-gen), list-gen(rune-gen)),
    (p) => {
      let Pair(!xs, !ys) = p;
      let left = append(reverse(ys), reverse(xs));
      let right = reverse(append(xs, ys));
      eq-data::(left, right)
    },
  );
  // a plain test
  test(
    k,
    "the list `List[1, 2, 3]` contains 2",
    () => {
      pin xs = List[1, 2, 3];
      pin f = (y) => {eq-int(*y, 2)};
      let result = core.list.find(xs, f);
      match result {
      | Left(_) =>
        False
      | Right(_) =>
        True
      }
    },
  );
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
✓ Pass: unpack then pack is identity
✓ Pass: (length as string) == (length as listchars)
✓ Pass: reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)
✗ Fail: one-of (should fail)
  → 2
✗ Fail: shrinking strings (should fail)
  → "AAAAAAAAAA"
✗ Fail: there is no list that contains 15 (should fail and report [15])
  → Vector[15]
```

Verbose mode is also available:

```text
? Check: there is no list that contains 15 (should fail and report [15])
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
✗ Fail: there is no list that contains 15 (should fail and report [15])
  → Vector[15]
```
