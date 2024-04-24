# noa

`noa` is a property-based testing framework for the [Neut](https://vekatze.github.io/neut/) programming language.

## Installation

```sh
neut get noa https://github.com/vekatze/noa/raw/main/archive/0-1-0.tar.zst
```

## Usage

See `source/test.nt` (for now).

## Example Outputs

The tests in the file should result in something like the following:

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
```

Verbose mode is also available:

```text
? Check: there is no list that contains 15 (should fail and report [15])
  ✔ []
  ✔ [0]
  ✔ [-1, 0]
  ✔ [0, 1, 0]
  ✗ ...
  ✔ [-8, 7, 10, -5, -1, 0, 0, 2, -7, -1, 7, -4, -11, -11, -1, 0, 0, 5, 8, -1]
  ✔ [10, 0, 2, 3, 0, 7, -13, -5, -1, 3, 3, 2, -2, -3, 0, -7, 0, -11, 0, 1, 4]
  ✗ [-5, -12, 3, 7, 4, -12, 0, 1, -5, 1, 15, 7, -1, 0, 1, -6, 10, 14, -3, 0, 6, -8]
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

You can try it on your machine:

```sh
git clone https://github.com/vekatze/noa
cd noa
neut build --execute
```
