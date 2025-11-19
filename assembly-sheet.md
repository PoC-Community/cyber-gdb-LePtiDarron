# 🧠 **x86 Assembly Cheat Sheet — Detailed Version**

# **1. Registers**

## 🔹 Main Registers

* **`%esp` — Stack Pointer**

  * Always points to the **top of the stack** (the last pushed element).
  * Decreases (`sub`) when performing a `push`.
  * Used to allocate/deallocate local variables.

* **`%ebp` — Base Pointer**

  * Marks the **stack frame** of a function.
  * Does not change during the function → makes accessing local variables and arguments easier.
  * Optional in optimized code (e.g., `-fomit-frame-pointer`).

* **`%ecx` — Counter / General-purpose register**

  * Naturally used by `rep movs`, `rep stos`, `loop`, etc.
  * Very often used for loops.

## 🔹 Other General-purpose Registers

* **`%eax`**

  * Used for **fast calculations**.
  * Holds the **return value** of a function.
  * Also used for division (`div`, `idiv`) and multiplication (`mul`, `imul`).

* **`%ebx`**

  * General-purpose, but **callee-saved** → must be restored if modified.

* **`%edx`**

  * Used with `%eax` for 64-bit arithmetic operations:

    * Multiplication: result in `EDX:EAX`
    * Division: dividend in `EDX:EAX`

* **`%esi` / `%edi`**

  * Used as **memory pointers** when working with arrays or strings.
  * Frequently used with copy/scan instructions (`movs`, `stos`, `scas`, `lods`).

## 🔹 Instruction Pointer

* **`%eip`**

  * Address of the **next instruction**.
  * Modified only via `call`, `ret`, `jmp`, or conditional jumps.

## 🔹 Flags (FLAGS)

* **`ZF` — Zero Flag**: 1 if result == 0.
* **`SF` — Sign Flag**: most significant bit (indicates negative result).
* **`CF` — Carry Flag**: carry for **unsigned arithmetic**.
* **`OF` — Overflow Flag**: overflow for **signed arithmetic**.

➡️ These flags determine the behavior of **conditional jumps**.


# **2. The Stack 📦**

## 🔹 General Behavior

* **LIFO** (Last In, First Out) organization.
* Each `push` **decreases** `esp`.
* Each `pop` **increases** `esp`.

## 🔹 Common Uses

✔ Local variables
✔ Register saving
✔ Function arguments (depending on calling convention)
✔ Saving the instruction pointer (`eip`) during `call`

## 🔹 Stack Frame

```
[ +── high memory ──+ ]
       arguments
     return address
       old EBP
     local variables
[ +── low memory ──+ ]
                 ↑
                ESP
```

➡️ This diagram corresponds to the moment just **after the function prologue**.

## 🔹 Alignment

* Some ABIs (notably Linux + GCC) require the stack to be **16-byte aligned** before a function call.
* Misalignment → crashes during SSE function calls (`movaps`).


# **3. Memory Addressing**

## 🔹 Common Addressing Modes

* `8(%ebp)`
  Address = `EBP + 8` → first argument passed on the stack.

* `(%eax, %ecx, 4)`
  Address = `EAX + ECX * 4`
  Ideal for accessing an `int` array.

* `0x10(%esp)`
  Address = `ESP + 16`

## 🔹 `lea` (Load Effective Address)

* Does **not** read memory → only computes an address.
* Does **not** modify flags.
* Common uses:

  ✔ Fast calculations (often replaces `add`)
  ✔ Complex arithmetic: `lea (%eax,%eax,4), %edx` → `EDX = EAX * 5`


# **4. Basic Instructions**

## 🔹 `mov`

* Copies source → destination.
* Does not affect flags.

## 🔹 `push` / `pop`

* Stack manipulation.
* Commonly used to save callee-saved registers.

## 🔹 `add` / `sub`

* Modify flags.
* Frequently used to adjust `esp`.

## 🔹 `cmp`

* Performs `destination – source`, **without storing the result**, but updates flags.
* Useful before a conditional jump.

## 🔹 Jumps

* `jmp`: unconditional jump.
* Conditional jumps:

  * `je` / `jne`
  * `jg`, `jge`, `jl`, `jle` (signed arithmetic)
  * `ja`, `jb`, `jbe`, `jae` (unsigned arithmetic)

## 🔹 `call` / `ret`

* `call`: pushes return address + jumps to function.
* `ret`: pops the address and returns.

## 🔹 Logical Instructions

* `and` **— Bitwise AND**

  * Performs a **bitwise AND** between source and destination.
  * Common uses:
    ✔ Mask specific bits (`and $0x0F, %al` keeps only the lower 4 bits)
    ✔ Partially reset a register or test bits
    ✔ Align addresses (`and $0xFFFFFFF0, %esp`)

* `or` **— Bitwise OR**

  * Performs a **bitwise OR**.
  * Common uses:
    ✔ Set specific bits (`or $0x04, %al` sets the 3rd bit)
    ✔ Combine masks or flags

* `xor` **— Bitwise XOR**

  * Performs a **bitwise exclusive OR**.
  * Common uses:
    ✔ Flip specific bits
    ✔ Quickly zero a register (`xor %eax, %eax`)
    ✔ Parity or simple toggling operations

* `not` **— Bitwise NOT**

  * Performs a **bitwise inversion** (`0 → 1` and `1 → 0`).
  * Useful for generating masks or inverting values.
