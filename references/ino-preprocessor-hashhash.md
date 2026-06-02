# `##line` / `stray '##' in program` — Arduino Preprocessor Bug

## Error signature

```
error: stray '##' in program
    1 | ##line 11 ".../005_setup.ino"
          ^~
error: 'line' does not name a type
```

The generated `.ino.cpp` starts with `##line` instead of `#line`, and the last
`#line` directive is missing its `#` (showing just `line 1`).

## Root cause

**Arduino-cli 1.5.0** with **esp32:esp32 core 3.3.8** has a preprocessor bug:
when the main `.ino` file does NOT contain explicit prototypes for `setup()`
and `loop()`, the preprocessor auto-generates them — but corrupts the first
`#line` directive into `##line`.

The `.cpp` shows this structure:
```
##line 11 ".../005_setup.ino"     ← broken: ## instead of #
void setup();
#line 9 ".../006_main_loop.ino"
void loop();
#line 0 ".../005_setup.ino"       ← unusual (#line 0)
line 1 ".../0503.ino"            ← broken: missing leading #
<merged content>
```

## Reproduction

Minimal case — two files in a sketch directory:

**project.ino:**
```cpp
#include <Arduino.h>
```

**005_setup.ino:**
```cpp
void setup() { Serial.begin(115200); }
```

Compiling produces `##line 1 ".../005_setup.ino"`.

## Fix — 3 options (pick one)

### Option A: Add prototypes to main `.ino` (quick workaround)

At the end of the prototypes section in the main `.ino` file:
```cpp
void setup();
void loop();
```

### Option B: Prefer `.h`/`.cpp` structure (recommended for non-trivial projects)

Keep `setup()` and `loop()` in the single `.ino` file; move everything else
to proper `.h`/`.cpp` modules. The preprocessor only scans `.ino` files — if
setup/loop are already defined in the main `.ino`, no prototypes are needed.

```
sketch/
├── sketch.ino       # setup() + loop() only
├── module.h
├── module.cpp
└── ...
```

Note: prototypes `void setup(); void loop();` at the bottom of the `.ino` are
still needed as a safety measure (see Option A).

### Option C: Merge all code into a single `.ino` file

When the project is small enough, a single `.ino` with everything inline
avoids the multi-file preprocessor entirely.

## How this was discovered

1. The `.cpp.merged` file (preprocessor output after concatenation) was correct
2. The `.cpp` file (with auto-generated prototypes prepended) was corrupted
3. Binary search eliminated all other `.ino` files until only the main `.ino`
   + one function file remained
4. A minimal test with `#include <Arduino.h>` + `void setup(){}` in a separate
   file reproduced it consistently
5. Multiple `#line` directives were shifted: first one doubled `##`, last one
   lost its `#`
6. Confirmed NOT caused by: CLAUDE.md, project.json, BOM encoding, CRLF, or
   oversized comment blocks
