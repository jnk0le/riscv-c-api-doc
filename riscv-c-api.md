# RISC-V C API Specification

This file contains the RISC-V additions to the standard C APIs.  It
merges together the compiler command-line API, the preprocessor API, and
the C language extensions (function attributes and intrinsic functions).

## Copyright and license information

 &copy; 2018 Palmer Dabbelt <palmer@sifive.com>

 &copy; 2018 SiFive, Inc

It is licensed under the Creative Commons Attribution 4.0 International
License (CC-BY 4.0). The full license text is available at
https://creativecommons.org/licenses/by/4.0/.

## Compiler Command-Line Arguments

* `-mabi=ABI`
* `-march=ISA`
* `-mtune=MICRO_ARCHITECTURE`
* `-mplt` `-mno-plt`
* `-mcmodel=CODE_MODEL`
* `-mstrict-align` `-mno-strict-align`
* `-mfdiv` `-mno-fdiv`
* `-mdiv` `-mno-div`
* `-mpreferred-stack-boundary=N`
* `-msmall-data-limit=N`
* `-mexplicit-relocs` `-mno-explicit-relocs`
* `-mrelax` `-mno-relax`
* `-msave-restore` `-mno-save-restore`
* `-mbranch-cost=N`

## Preprocessor Definitions

| Name                | Value | When defined                  |
| ------------------- | ----- | ----------------------------- |
| __riscv             | 1     | Always defined.               |
| __riscv_xlen        | <ul><li>32 for rv32</li><li>64 for rv64</li><li>128 for rv128</ul> | Always defined.             |
| __riscv_flen        | <ul><li>32 if the F extension is available **or**</li><li>64 if `D` extension available **or**</li><li>128 if `Q` extension available</li></ul> | `F` extension is available. |
| __riscv_32e         | 1     | `E` extension is available.   |
| __riscv_vector      | 1     | Implies that any of the vector extensions (`v` or `zve*`) is available |
| __riscv_v_min_vlen    | <N> (see [__riscv_v_min_vlen](#__riscv_v_min_vlen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen     | <N> (see [__riscv_v_elen](#__riscv_v_elen)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_v_elen_fp  | <N> (see [__riscv_v_elen_fp](#__riscv_v_elen_fp)) | The `V` extension or one of the `Zve*` extensions is available. |
| __riscv_misaligned_fast | 1  | Misaligned accesses are fast. |
| __riscv_misaligned_slow | 1  | Misaligned accesses are supported, but may be substantially slower than aligned accesses. |
| __riscv_misaligned_avoid | 1  | Misaligned accesses are not supported and could trap. (see [ __riscv_misaligned_{fast,slow,avoid}](#__riscv_misaligned_{fast,slow,avoid}) |

### __riscv_v_min_vlen

The `__riscv_v_min_vlen` macro expands to the minimal VLEN, in bits, mandated
by the available vector extension, if any.

The value of `__riscv_v_min_vlen` is defined by the following rules:
- 128, if the `V` extension is present;
- 32, if one of the `Zve32{x,f}` extensions is present;
- 64, if one of the `Zve64{x,f,d}` extensions is present;
- `N`, if one of the `Zvl<N>b` extensions, `N` in `{32,64,128,256,512,1024}`,
is present.

If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_min_vlen` is undefined.

Examples:

- `__riscv_v_min_vlen` is 128 for `rv64gcv`
- `__riscv_v_min_vlen` is 512 for `rv32gcv_zvl512b`
- `__riscv_v_min_vlen` is 256 for `rv32gcv_zvl32b_zvl256b`
- `__riscv_v_min_vlen` is 128 for `rv64gcv_zvl32b`

### __riscv_v_elen

The `__riscv_v_elen` macro expands to the supported element length, in bits,
of any non-floating-point vector operand of any vector instruction in the
available vector extension, if any. (Stricter upper bounds may apply to
particular operands of particular instructions.)


The value of `__riscv_v_elen` is defined by the following rules:
- 64, if the `V` extension or one of the `Zve64{x,f,d}` extensions is present; and
- 32, if one of the `Zve32{x,f}` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen` is undefined.

### __riscv_v_elen_fp

The `__riscv_v_elen_fp` macro expands to the supported element length, in bits,
of any floating-point vector operand of any vector instruction in the available
vector extension, if any. (Stricter upper bounds may apply to particular
operands of particular instructions.)

The value of `__riscv_v_elen_fp` is defined by the following rules:
- 64, if one of the `V` or `Zve64d` extensions is present;
- 32, if one of the `Zve{32,64}f` extensions is present; and
- 0, if one of the `Zve{32,64}x` extensions is present.
If multiple rules apply, the maximum value is taken.
If none of the rules apply, `__riscv_v_elen_fp` is undefined.

### __riscv_misaligned_{fast,slow,avoid}

These can be used in common library code to compile time segregate code which relies
on misaligned access being fast or not.
A typical complier could (but not necessarily) map fast variant to -mno-strict-align
and avoid to -mstrict-align, if specified.
Perhaps obvious, but these are mutually exclusive, so only one is defined at a time
for a compilation unit.


### Architecture Extension Test Macro

Architecture extension test macro is a new set of test macro to checking the
availability and version for certain extension, however not all compilers are
supported, so you should check `__riscv_arch_test` to make sure this compiler
is supporting those preprocessor definitions.

The value of architecture extension test macro are defined as its version,
which is compute by following formula:

```
<MAJOR_VERSION> * 1,000,000 + <MINOR_VERSION> * 1,000 + <REVISION_VERSION>
```

For example:
- F-extension v2.2 will define `__riscv_f` as `2002000`.

| Name                    | Value        | When defined                  |
| ----------------------- | ------------ | ----------------------------- |
| __riscv_arch_test       | 1            | Defined if compiler support new architecture extension test macro. |
| __riscv_i               | Arch Version | `I` extension is available.   |
| __riscv_e               | Arch Version | `E` extension is available.   |
| __riscv_m               | Arch Version | `M` extension is available.   |
| __riscv_a               | Arch Version | `A` extension is available.   |
| __riscv_f               | Arch Version | `F` extension is available.   |
| __riscv_d               | Arch Version | `D` extension is available.   |
| __riscv_c               | Arch Version | `C` extension is available.   |
| __riscv_p               | Arch Version | `P` extension is available.   |
| __riscv_v               | Arch Version | `V` extension is available.   |
| __riscv_zawrs           | Arch Version | `Zawrs` extension is available. |
| __riscv_zba             | Arch Version | `Zba` extension is available. |
| __riscv_zbb             | Arch Version | `Zbb` extension is available. |
| __riscv_zbc             | Arch Version | `Zbc` extension is available. |
| __riscv_zbs             | Arch Version | `Zbs` extension is available. |
| __riscv_zfh             | Arch Version | `Zfh` extension is available. |

### ABI Related Preprocessor Definitions

| Name                     | Value | When defined                  |
| ------------------------ | ----- | ----------------------------- |
| __riscv_abi_rve          | 1     | Defined if using `ilp32e` ABI |
| __riscv_float_abi_soft   | 1     | Defined if using `ilp32`, `ilp32e` or `lp64` ABI. |
| __riscv_float_abi_single | 1     | Defined if using `ilp32f` or `lp64f` ABI. |
| __riscv_float_abi_double | 1     | Defined if using `ilp32d` or `lp64d` ABI. |
| __riscv_float_abi_quad   | 1     | Defined if using `ilp32q` or `lp64q` ABI. |

### Code Model Related Preprocessor Definitions

| Name                  | Value    | When defined                          |
| --------------------- | -------- | ------------------------------------- |
| __riscv_cmodel_medlow | 1        | Defined if using `medlow` code model. |
| __riscv_cmodel_medany | 1        | Defined if using `medany` code model. |

### Deprecated Preprocessor Definitions

| Name                  | Value    | When defined                          | Alternative |
| --------------------- | -------- | ------------------------------------- | ----------- |
| __riscv_cmodel_pic    | 1        | GCC defines this when compiling with `-fPIC`, `-fpic`, `-fPIE` or `-fpie`. | `__PIC__` or `__PIE__` |
| __riscv_mul           | 1        | `M` extension is available.   | `__riscv_m` |
| __riscv_div           | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_muldiv        | 1        | `M` extension is available and `-mno-div` is not given.*[1]    | `__riscv_m` |
| __riscv_atomic        | 1        | `A` extension is available.   | `__riscv_a` |
| __riscv_fdiv          | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_fsqrt         | 1        | `F` extension is available and `-mno-fdiv` is not given.*[1]   | `__riscv_f` or `__riscv_d` |
| __riscv_compressed    | 1        | `C` extension is available.   | `__riscv_c` |

*[1] Not all compilers provide `-mno-div` and `-mno-fdiv` option.

## Function Attributes

### `__attribute__((naked))`

The compiler won't generate the prologue/epilogue for those functions with
`naked` attributes. This attribute is usually used when you want to write a
function with an inline assembly body.

This attribute is incompatible with the `interrupt` and `prestacked` attribute.

NOTE: Be aware that compilers might have further restrictions on naked
functions. Please consult your compiler's manual for more information.

### `__attribute__((interrupt))`, `__attribute__((interrupt("user")))`, `__attribute__((interrupt("supervisor")))` `__attribute__((interrupt("machine")))`

The interrupt attribute specifies that a function is an interrupt handler.
The compiler will save/restore all used registers in the prologue/epilogue
regardless of the ABI, all used registers including floating point
register/vector register if `F` extension/vector extension is enabled.

The interrupt attribute can have an optional parameter to specify the mode.
The possible values are `user`, `supervisor`, or `machine`.
The default value `machine` is used, if the mode is not specified.

The function can specify only one mode; the compiler should raise an error if a
function declares more than one mode or an undefined mode.

This attribute is incompatible with the `naked` attribute.

### `__attribute__((prestacked("<reglist>"))`

The prestacked attribute instructs the compiler which registers can be
used without saving/restoring them. It enables efficient use of 
custom interrupt controllers that stack some of the architectural registers.
Without the need for compiler builds with a custom attributes.

If used together with `interrupt` attribute, `prestacked` annotation overrides
its register preservation functionality.

`<reglist>` is a string literal, listing all registers available for use 
in a given function, with a following syntax rules:

- no whitespaces
- raw register names rather than ABI mnemonics
- register range cover all registers between and including specified ("x4-x6"
is equivalent to "x4,x5,x6")
- registers/ranges are separated by comma
- annotated callee saved registers have to be properly handled as a temporary ones
- CSRs taking part in calling conventions are also subject to this mechanism
- registers must be sorted (integer, floating point, vector, custom, then by 
lowest numbered)
- CSRs must be put after the architectural regfiles, those don’t have to be sorted
- shall not imply `interrupt` attribute

> **_NOTE:_** Strict syntax rules allow better portability across compilers and ABIs.

To support auxiliary purposes, annotated functions should be callable by regular code.

> **_NOTE:_** If `x1` (aka `ra`) is included in the list then a special return
mechanism must be used (e.g. `mret` from `interrupt` attribute)

This attribute is incompatible with the `naked` attribute.

#### usage examples

psABI with F extension, caller saved:\
`__attribute__((prestacked("x5-x7,x10-x17,x28-x31,f0-f7,f10-f17,f28-f31,fcsr")))`

standard risc-v irq, ilp32e, caller saved and `ra`:\
`__attribute__((interrupt, prestacked("x1,x5-x7,x10-x15")))`

standard risc-v irq with simplified range (e.g. shadow register file):\
`__attribute__((interrupt, prestacked("x8-x15")))`

custom irq controller, F + P extensions (`ra`,`a0`,`a1` pushed on stack, shadow registers 
where bit 2 of register operand is set):\
`__attribute__((prestacked("x4-x7,x10,x11,x12-x15,x20-x23,x28-x31,fcsr,vxsat")))`

optimization for `noreturn` functions (psABI with F extension):\
`__attribute__((noreturn, prestacked("x1,x5-x31,f0-f31,fcsr")))`

> **_NOTE:_** Compilers are intentionally preserving full prologues, of `noreturn` functions, to 
allow backtracing and throwing exceptions. Which leads to stack and codespace
bloating. Prestacked annotation can be abused to get rid of the prologues stacking
without the risk of underflowing the stack as would happen with `naked` attribute.

pure assembly function (FP compute kernel) using only subset of caller saved
registers (`a0` argument not modified during execution):\
`__attribute__((prestacked("x5,x11-x15,f8-f11,v0,v1,v8-v31,fcsr,vl,vtype,vstart")))`

> **_NOTE:_** This use case is necessary for efficient IPRA compilations. Beneficial even without IPRA.

## Intrinsic Functions

Intrinsic functions (or intrinsics or built-ins) are expanded into instruction sequences by compilers.
They typically provide access to functionality that is otherwise not synthesizable by compilers.
Some intrinsics expand to different code sequences depending on the available instructions from the enabled ISA extensions.

Compilers typically come with their own architecture-independent intrinsics (e.g. synchronization primitives, byte-swap, etc.).
The RISC-V compiler backend can define additional target-specific intrinsics.
Providing functionality via architecture-independent intrinsics is the preferred method, as it improves code portability.

Some intrinsics are only available if a particular header file is included.
RISC-V header files that enable intrinsics require the prefix `riscv_` (e.g. `riscv_vector.h` or `riscv_crypto.h`).

RISC-V specific intrinsics use the common prefix `__riscv_` to avoid namespace collisions.

The intrinsic name describes the functional behaviour of the function.
In case the functionality can be expressed with a single instruction, the instruction's name (any '.' replaced by '_') is the preferred choice.
Note, that intrinsics that are restricted to RISC-V vendor extensions need to include the vendor prefix (as documented in the RISC-V toolchain conventions).

If intrinsics are available for multiple data types, then function overloading is preferred over multiple type-specific functions.
If an intrinsic function is has parameters or return values that reference registers with XLEN bits, then the data type `long` should be used.
In case a function is only available for one data type and this type cannot be derived from the function's name, then the type should be appended to the function name, delimited by a '_' character.
Typical type postfixes are "32" (32-bit), "i32" (signed 32-bit), "i8m4" (vector register group consisting of 4 signed 8-bit vector registers).

RISC-V intrinsics follow the following naming rule:

```
INTRINSIC ::= PREFIX NAME [ '_' TYPE ]
PREFIX ::= "__riscv_"
NAME ::= Name of the intrinsic function.
TYPE ::= Optional type postfix.
```

RISC-V intrinsics examples:

```
type __riscv_orc_b (type rs); // orc.b rd, rs

long __riscv_clmul (long a, long b); // clmul rd, rs1, rs2

#include <riscv_vector.h> // make RISC-V vector intrinsics available
vint8m1_t __riscv_vadd_vv_i8m1(vint8m1_t vs2, vint8m1_t vs1, size_t vl); // vadd.vv vd, vs2, vs1
```

### NTLH Intrisics 

The RISC-V zihintntl extension provides the RISC-V specific intrinsic functions for generating non-temporal memory accesses. These intrinsic functions provide the domain parameter to specify the behavior of memory accesses.

In order to access the RISC-V NTLH intrinsics, it is necessary to
include the header file `riscv_ntlh.h`.

The functions are only available if the compiler enables the zihintntl extension.

```
type __riscv_ntl_load (type *ptr, int domain);
void __riscv_ntl_store (type *ptr, type val, int domain);
```

There are overloaded functions of `__riscv_ntl_load` and `__riscv_ntl_store`. When these intrinsic functions omit the `domain` argument, the `domain` is implied as `__RISCV_NTLH_ALL`.

```
type __riscv_ntl_load (type *ptr);
void __riscv_ntl_store (type *ptr, type val);
```

The types currently supported are:

- Integer types.
- Floating-point types.
- Fixed-length vector types.

The `domain` parameter could pass the following values. Each one is mapped to the specific zihintntl instruction.

```
enum {
  __RISCV_NTLH_INNERMOST_PRIVATE = 2,
  __RISCV_NTLH_ALL_PRIVATE,
  __RISCV_NTLH_INNERMOST_SHARED,
  __RISCV_NTLH_ALL
};
```

| Domain Value                     | Instruction | 
| ---------------------------------| ------------| 
| `__RISCV_NTLH_INNERMOST_PRIVATE` | `ntl.p1`    |
| `__RISCV_NTLH_ALL_PRIVATE`       | `ntl.pall`  |
| `__RISCV_NTLH_INNERMOST_SHARED`  | `ntl.s1`    |
| `__RISCV_NTLH_ALL`               | `ntl.all`   |

## Constraints on Operands of Inline Assembly Statements

This section lists operand constraints that can be used with inline assembly
statements, including both RISC-V specific and common operand constraints.


| Constraint         |                                    | Note        |
| ------------------ | ---------------------------------- | ----------- |
| m                  | An address that is held in a general-purpose register with offset.      |      |
| A                  | An address that is held in a general-purpose register.      |      |
| r                  | General purpose register                      |      |
| f                  | Floating-point register                       |      |
| i                  | Immediate integer operand                     |      |
| I                  | 12-bit signed immediate integer operand      |      |
| K                  | 5-bit unsigned immediate integer operand     |      |
| J                  | Zero integer immediate operand                |      |
| vr                 | Vector register                    |            |
| vd                 | Vector register, excluding v0      |            |
| vm                 | Vector register, only v0           |            |

NOTE: Immediate value must be a compile-time constant.

### The Difference Between `m` and `A` Constraints

The difference between `m` and `A` is whether the operand can have an offset;
some instructions in RISC-V do not allow an offset for the address operand,
such as atomic or vector load/store instructions.

The following example demonstrates the difference; it is trying
to load value from `foo[10]` and using `m` and `A` to pass that address.

```c
int *foo;
void bar() {
 int x;
 __asm__ volatile ("lw %0, %1" : "=r"(x) : "m" (foo[10]));
 __asm__ volatile ("lw %0, %1" : "=r"(x) : "A" (foo[10]));
}
```

Then we compile with GCC with `-O` option:

```shell
$ riscv64-unknown-elf-gcc x.c -o - -O -S
...
bar:
       lui    a5,%hi(foo)
       ld     a5,%lo(foo)(a5)
 #APP
# 4 "x.c" 1
       lw a4, 40(a5)
# 0 "" 2
 #NO_APP
       addi   a5,a5,40
 #APP
# 5 "x.c" 1
       lw a5, 0(a5)
# 0 "" 2
 #NO_APP
       ret

```

The compiler uses an immediate offset of 40 for the `m` constraint, but for the
`A` constraint uses an extra addi instruction instead.

### Operand Modifiers

This section lists operand modifiers that can be used with inline assembly
statements, including both RISC-V specific and common operand modifiers.

| Modifiers    | Description                                                                       | Note        |
| ------------ | --------------------------------------------------------------------------------- | ----------- |
| z            | Print `zero` (`x0`) register for immediate 0, typically used with constraints `J` |             |
| i            | Print `i` if corresponding operand is immediate.                                  |             |
