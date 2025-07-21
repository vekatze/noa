# noa

`noa` is a testing framework for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get noa https://github.com/vekatze/noa/raw/main/archive/0-4-9.tar.zst
```

## Types

### Main Definitions

```neut
data noa-kit

define make-default-noa-kit(): noa-kit

define make-verbose-noa-kit(): noa-kit

define make-noa-kit(
  sink: descriptor,
  buffer-capacity: int,
  num-of-tests: int, // specifies the number of tests executed by `check`.
  max-size: int, // specifies the max size of input generated in `check`.
  verbose: bool // specifies whether to enable verbose output.
): noa-kit

// performs property-based testing
define check<a>(k: &noa-kit, label: &text, !g: gen(a), !predicate: (a) -> bool): unit

// performs plain testing
define test(k: &noa-kit, label: &text, property: () -> bool): unit

// Represents a value generator for property-based testing.
data gen(a) {
| Gen(
    generate: (size-upper-bound) -> a, // creates a value of type `a`, no larger than the given size.
    shrink: (a) -> list(a), // creates a list of values that are "smaller" than the input value.
    viewer: show(a), // provides a way to convert values of type `a` into texts.
  )
}
```

### Preset Generators

```neut
inline bool-gen: gen(bool)

inline float-gen: gen(float)

inline int-gen: gen(int)

inline positive-int-gen: gen(int)

inline negative-int-gen: gen(int)

inline rune-gen: gen(rune)

inline ascii-rune-gen: gen(rune)

inline text-gen: gen(text)

inline ascii-text-gen: gen(text)

inline list-gen<a>(!g: gen(a)): gen(list(a))

inline pair-gen<a, b>(!g1: gen(a), !g2: gen(b)): gen(pair(a, b))

inline either-gen<a, b>(g1: gen(a), g2: gen(b)): gen(either(a, b))

inline vector-gen<a>(!g: gen(a)): gen(vector(a))

// Chooses a value from `Cons(x, xs)` randomly.
define one-of<a>(g: gen(a), xs: &list(a), fallback: &a): gen(a)

// Generates a value of type `a` or generates `none`
define optional<a>(!g: gen(a)): gen(?a)
```

## Example

```neut
define zen(): unit {
  pin k = make-default-noa-kit();
  // a property-based test
  check(
    k,
    "reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)",
    pair-gen(list-gen(rune-gen), list-gen(rune-gen)),
    function (p) {
      let Pair(!xs, !ys) = p;
      let left = append(reverse(!ys), reverse(!xs));
      let right = reverse(append(!xs, !ys));
      let Eq of {equal} = core.list.eq.as-eq(core.rune.eq.as-eq);
      equal(left, right)
    },
  );
  // a plain test
  test(
    k,
    "the list `[1, 2, 3]` contains 2",
    function () {
      pin xs = [1, 2, 3];
      let result =
        core.list.find(xs, function (y) {
          eq-int(*y, 2)
        });
      match result {
      | Left(_) =>
        False
      | Right(_) =>
        True
      }
    },
  )
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
✓ Pass: (length as text) == (length as list of chars)
✓ Pass: reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)
✗ Fail: one-of (should fail)
  → 2
✗ Fail: shrinking texts (should fail)
  → "AAAAAAAAAA"
✗ Fail: there is no list that contains 15 (should fail and report [15])
  → [15]
// ...
```

Verbose mode is also available:

```text
? Check: there is no list that contains 15 (should fail and report [15])
  ✓ []
  ✓ [0]
  ✓ [-1, 0]
  ✓ [0, 1, 0]
    ...
  ✓ [-8, 7, 10, -5, -1, 0, 0, 2, -7, -1, 7, -4, -11, -11, -1, 0, 0, 5, 8, -1]
  ✓ [10, 0, 2, 3, 0, 7, -13, -5, -1, 3, 3, 2, -2, -3, 0, -7, 0, -11, 0, 1, 4]
  ✗ [-5, -12, 3, 7, 4, -12, 0, 1, -5, 1, 15, 7, -1, 0, 1, -6, 10, 14, -3, 0, 6, -8]
    ↻ Shrink:
      ✓ []
      ✓ [7, -1, 0, 1, -6, 10, 14, -3, 0, 6, -8]
      ✗ [-5, -12, 3, 7, 4, -12, 0, 1, -5, 1, 15]
      ✓ []
      ✗ [-12, 0, 1, -5, 1, 15]
      ✓ []
      ✗ [-5, 1, 15]
      ✓ []
      ✗ [1, 15]
      ✓ []
      ✗ [15]
      ✓ []
      ✓ [0]
      ✓ [8]
      ✓ [12]
      ✓ [14]
    ✓ Stop
✗ Fail: there is no list that contains 15 (should fail and report [15])
  → [15]
```
