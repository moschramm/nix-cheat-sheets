# Nix Language Basics
Summary of https://nix.dev/tutorials/nix-language
## Overview
- Nix is a **purely functional, lazily evaluated, dynamically typed language** designed to describe *derivations* (build instructions).
- Used in **Nixpkgs** (software distribution) and **NixOS** (Linux distro configuration).
- Key concepts: names/values, functions, attribute sets, impurities (file inputs), derivations.

---
## Basic Syntax and Values
### Primitive Types & Collections
```nix
{
  string = "hello";
  integer = 1;
  float = 3.141;
  bool = true;
  null = null;
  list = [ 1 "two" false ];
  attributeSet = { a = "hello"; b = 2; };
}
```

### Attribute Sets `{ ... }`
- Collection of unique name-value pairs.
- Access attributes with dot: `attrset.a`
- Nested attributes: `attrset.a.b.c`

### Recursive Attribute Sets `rec { ... }`
Allows references within the set:
```nix
rec {
  one = 1;
  two = one + 1;
  three = two + 1;
}
```

### `let ... in ...`
Local bindings with scope limited to the expression after `in`:
```nix
let
  a = 1;
  b = a + 1;
in
a + b  # evaluates to 3
```

---
## Special Constructs

### `with ...; ...`
Shortcut to access attributes without repeating the set name:
```nix
with a; [ x y z ]  # equivalent to [ a.x a.y a.z ]
```

### `inherit`
Shorthand to assign variables from outer scope:
```nix
let
  x = 1;
  y = 2;
in
{ inherit x y; }  # { x = 1; y = 2; }
```
Can inherit from attribute sets:
```nix
inherit (a) x y;  # equivalent to x = a.x; y = a.y;
```

---
## Strings

### Interpolation `${ ... }`
Insert expression values into strings:
```nix
let name = "Nix"; in "hello ${name}"  # "hello Nix"
```
Only strings or string-coercible values allowed.

### Indented Strings `'' ... ''`
Multi-line strings with common indentation trimmed:
```nix
''
  line1
   line2
''
# "line1\n line2\n"
```

---
## Paths
- Absolute paths start with `/`.
- Relative paths contain `/` but don't start with `/`, relative to current file.
- `./.` denotes current directory.
- `<nixpkgs>` is a lookup path to Nixpkgs directory (impure, non-reproducible, **bad**).

---
## Functions
- Always take **one argument**.
- Syntax: `arg: body`
- Multiple arguments via nesting (currying):
```nix
x: y: x + y
```
- Attribute set arguments (destructuring):
```nix
{ a, b }: a + b
```
- Default values:
```nix
{ a, b ? 0 }: a + b
```
- Allow extra attributes:
```nix
{ a, b, ... }: a + b
```
- Named attribute set argument:
```nix
args@{ a, b, ... }: a + b + args.c
```

### Calling Functions
```nix
let f = x: x + 1; in f 1  # 2
```
Use parentheses if needed:
```nix
(x: x + 1) 1  # 2
```

---
## Libraries

### `builtins`
Primitive functions built into Nix interpreter:
```nix
builtins.toString  # primitive function
```
`import` is a special built-in to load and evaluate Nix files:
```nix
import ./file.nix
```

### `pkgs.lib`
Standard library from Nixpkgs, accessed via `pkgs.lib`:
```nix
let pkgs = import <nixpkgs> {}; in pkgs.lib.strings.toUpper "hello"
# "HELLO"
```

---
## Impurities (Build Inputs)
- File system paths and fetchers are impure operations.
- Using a path copies the file/directory to the Nix store:
```nix
"${./data}"  # results in Nix store path of 'data': /nix/store/<hash>-data
```
- Fetchers to get remote files:
```nix
builtins.fetchurl "https://example.com/file.tar.gz"
builtins.fetchTarball "https://example.com/archive.tar.gz"
```

---
## Derivations (Build Descriptions)
- Core concept: describe how to build software.
- Created with `derivation` (primitive) or `stdenv.mkDerivation` (wrapper).
- Evaluates to an attribute set representing a build.
- Can be referenced as a (nix store) path via string interpolation:
```nix
let pkgs = import <nixpkgs> {}; in "${pkgs.nix}"
```

---
## Example: Simple Function and Attribute Set
```nix
let
  greet = name: "Hello, ${name}!";
  person = { firstName = "Nix"; lastName = "User"; };
in
greet person.firstName  # "Hello, Nix!"
```

---
## Example: Package with `mkDerivation`

```nix
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {
  pname = "hello";
  version = "2.12";
  src = fetchurl {
    url = "mirror://gnu/${pname}/${pname}-${version}.tar.gz";
    sha256 = "1ayhp9v4m4rdhjmnl2bq3cibrbqqkgjbl3s7yk2nhlh8vj3ay16g";
  };
  meta = with lib; {
    license = licenses.gpl3Plus;
  };
}
```

---
## Useful Commands
- Interactive evaluation: `nix repl`
- Evaluate file: `nix-instantiate --eval file.nix`
- Garbage collect unused builds: `nix-collect-garbage`

---
## Summary
- Nix language is simple but powerful: **attribute sets**, **functions**, **lazy evaluation**.
- Use `let` for local bindings, `with` for attribute shortcuts, `inherit` for convenience.
- Functions take one argument; multiple arguments via currying or attribute sets.
- Most relevant impurity is reading files as *build inputs*.
- Derivations describe build steps and produce paths in the Nix store.

---
## References
- [Nix manual: Nix language](https://nix.dev/manual/nix/stable/language/index.html)
- [Nix manual: String interpolation](https://nix.dev/manual/nix/stable/language/index.html)
- [Nix manual: Built-in Functions](https://nix.dev/manual/nix/stable/language/index.html)
- [Nix manual: `nix repl`](https://nix.dev/manual/nix/stable/command-ref/new-cli/nix3-repl.html)
- [Nixpkgs manual: Functions reference](https://nixos.org/manual/nixpkgs/stable/#sec-functions-library)
- [Nixpkgs manual: Fetchers](https://nixos.org/manual/nixpkgs/stable/#chap-pkgs-fetchers)
