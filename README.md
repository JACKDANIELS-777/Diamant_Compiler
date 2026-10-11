# Diamant Compiler

A compiler for my Diamant language, written in Python (about **2,300 lines**). It turns Diamant source into
**32-bit x86 NASM assembly** for **bare metal**: the generated code doesn't need an operating system, and `print`
writes straight into VGA text memory at `0xB8000`.

```
s = "Hello"; print s
```

compiles to an `output.asm` with a `.bss` slot for `s`, a `.data` pointer to video memory, and a generated `print`
routine that copies the string into VGA memory one character at a time.

## How it works

1. **Lexer and parser** (`main.py`), built with [SLY](https://github.com/dabeaz/sly). The grammar produces a tree
   of tuples such as `('var_assign', 'x', ('num', 5))`. Operator precedence is declared in the parser.
2. **Code generator** (`BasicExecute.walkTree`, about 1,300 lines). It walks the tree and writes assembly into
   separate buckets for `.bss`, `.data`, `.text` and each function, then joins them into `output.asm`: `_start`,
   the functions, and `main` with a proper `ebp` stack frame.
3. **Smallest type that fits** (`Builtins.py`). For each value it picks the smallest NASM data size that can hold
   it (`db`, `dw`, `dd`, `dq`).
4. **Runtime routines** (`Builtins.py`), such as `puts_pm` for printing in protected mode. They're added to the
   output only when a program uses them.

## The language

Statements are separated by `;`.

| Feature | Syntax |
|---|---|
| Variables | `x = 5` |
| Typed variables | `x -> byte = 5` (types: `byte`, `word`, `int`, `long`, `str`) |
| Arithmetic and comparison | `+ - * /`, `== >= <= > <`, `and`, `or`, `not` |
| Strings and output | `s = "Hello"; print s` |
| Input | `input name 10` |
| Branches | `IF x > 3 THEN … ELSE …` |
| Loops | `FOR i = 0 TO 5 THEN …`, `WHILE x < 10 -> …` |
| Functions | `FUN greet() -> …`, `FUN add (a,b) {…}`, calls `greet()` / `add(2,3)` |
| Structs | `struct Point {x:int, y:int}` and `p = struct Point (1,2)` |
| Arrays and tuples | `[1,2,3]`, `(1,2)` |
| Pointers | `&x` (address), `*p` (dereference) |
| Inline assembly | `asm {"mov eax, 1"}` |
| Comments | `# …` |

## Status: early prototype

I compiled one small program per feature and assembled each `output.asm` with `nasm -f elf32`:

| Works (compiles and assembles) | Not yet |
|---|---|
| constant arithmetic: `y = 3 + 2 * 4` | reading one variable into another: `y = x`, `y = x + 1` |
| strings and `print`, including several prints | statements inside `IF`, `FOR` and function bodies |
| `WHILE` loops | function arguments |
| typed variables: `x -> byte = 5` | pointers: `p = &x` |
| struct definitions and instances | |
| inline assembly blocks | |

The parser already accepts the whole language above; the code generator covers the left column so far.

## Running it

```sh
pip install sly
python main.py program.dia      # writes output.asm
nasm -f elf32 output.asm -o output.o
```

The output is meant to run without an OS: it writes to VGA memory directly, so it needs to be started in 32-bit
protected mode, for example by a bootloader or GRUB.

---

Part of my Diamant projects. Each repository is its own separate project.
