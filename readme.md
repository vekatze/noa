# noa

`noa` is a testing framework for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get noa https://github.com/vekatze/noa/raw/main/archive/0-3-34.tar.zst
```

## Types

### Main Definitions

```neut
// Represents a set of testing utilities.
data trope {
| Trope(
    // For property-based testing.
    check: <a>(name: &text, g: gen(a), property: (a) -> bool) -> unit,
    // For plain testing.
    test: (label: &text, property: () -> bool) -> unit,
  )
}

// Represents a configuration of testing utilities.
data config {
| Config(
    num-of-tests: int, // specifies the number of tests executed by `check`.
    max-size: int, // specifies the max size of input generated in `check`.
    verbose: bool, // specifies whether to enable verbose output.
  )
}

// Creates a new test utilities.
inline new-suite(!c: config): trope

// A set of testing utilities.
inline noa: trope

// A set of testing utilities.
inline noa-verbose: trope

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
inline bools: gen(bool)

inline floats: gen(float)

inline ints: gen(int)

inline positive-ints: gen(int)

inline negative-ints: gen(int)

inline runes: gen(rune)

inline ascii-runes: gen(rune)

inline texts: gen(text)

inline ascii-texts: gen(text)

inline lists<a>(!g: gen(a)): gen(list(a))

inline pairs<a, b>(!g1: gen(a), !g2: gen(b)): gen(pair(a, b))

inline eithers<a, b>(g1: gen(a), g2: gen(b)): gen(either(a, b))

// Chooses a value from `xs` randomly.
define one-of<a>(g: gen(a), xs: &list(a)): gen(a)

// Generates a value of type `a` or generates `none`
define optional<a>(!g: gen(a)): gen(?a)
```

## Example

In most cases you import `noa` and use it as follows:

```neut
define zen(): unit {
  // A property-based test
  noa::check(
    "reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)",
    pairs(lists(ints), lists(ints)),
    function (p) {
      let Pair(!xs, !ys) = p in
      let left = append(reverse(!ys), reverse(!xs)) in
      let right = reverse(append(!xs, !ys)) in
      let eq = core.list.eq.as-eq(core.int.eq.as-eq) in
      eq::equal(left, right)
    },
  );
  // A plain test
  noa::test(
    "the list `[1, 2, 3]` contains 2",
    function () {
      pin xs = [1, 2, 3] in
      let result =
        core.list.find(xs, function (y) {
          eq-int(*y, 2)
        })
      in
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
✔ Pass: unpack then pack is identity
✔ Pass: (length as text) == (length as list of chars)
✔ Pass: reverse(ys) ++ reverse(xs) == reverse(xs ++ ys)
✘ Fail: one-of (should fail)
  → 2
✘ Fail: shrinking texts (should fail)
  → "AAAAAAAAAA"
✘ Fail: there is no list that contains 15 (should fail and report [15])
  → [15]
// ...
```

Verbose mode is also available:

```text
? Check: there is no list that contains 15 (should fail and report [15])
  ✔ []
  ✔ [0]
  ✔ [-1, 0]
  ✔ [0, 1, 0]
    ...
  ✔ [-8, 7, 10, -5, -1, 0, 0, 2, -7, -1, 7, -4, -11, -11, -1, 0, 0, 5, 8, -1]
  ✔ [10, 0, 2, 3, 0, 7, -13, -5, -1, 3, 3, 2, -2, -3, 0, -7, 0, -11, 0, 1, 4]
  ✘ [-5, -12, 3, 7, 4, -12, 0, 1, -5, 1, 15, 7, -1, 0, 1, -6, 10, 14, -3, 0, 6, -8]
    ↻ Shrink:
      ✔ []
      ✔ [7, -1, 0, 1, -6, 10, 14, -3, 0, 6, -8]
      ✘ [-5, -12, 3, 7, 4, -12, 0, 1, -5, 1, 15]
      ✔ []
      ✘ [-12, 0, 1, -5, 1, 15]
      ✔ []
      ✘ [-5, 1, 15]
      ✔ []
      ✘ [1, 15]
      ✔ []
      ✘ [15]
      ✔ []
      ✔ [0]
      ✔ [8]
      ✔ [12]
      ✔ [14]
    ✔ Stop
✘ Fail: there is no list that contains 15 (should fail and report [15])
  → [15]
```
