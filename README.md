Recitation 11: Solutions
=========

### Basic Linker knowledge

No error are produced on compiling `list.c`. We compile as `gcc -g -Wall -Wextra -std=c99 -c list.c -o list.o`.

Warnings are produced on compiling `main.c`. We compile as `gcc -g -Wall -Wextra -std=c99 -c main.c -o list.o`.

The first few say that the functions `insert`, `delete`, and `find` are implicitly declared. We should explicitly put function declarations (not implementations) for them along with the `extern` keyword to say that they are implemented in a different file.
The other warnings say that we do not use the command line arguments `int argc` and `char **argv` so we can safely remove them, and instead change the definition for `main` to be `int main()` instead.

Now we can link the two object files `main.o` and `list.o` together to the binary `linkedlist`. Link as `gcc list.o main.o -o linkedlist`.

In the original `list.c` before we fix the output, the linker symbols are:
* `head` weak linker symbol
* `n_inserts` weak linker symbol
* `n_deletes` weak linker symbol
* `insert` strong linker symbol
* `prev` not a linker symbol
* `curr` not a linker symbol
* `malloc` undefined linker symbol
* `delete` strong linker symbol
* `free` undefined linker symbol
* `find` strong linker symbol

Or nm list.o Output:
```
000000000000005a T delete
00000000000000dc T find
                 U free
0000000000000008 C head
0000000000000000 T insert
                 U malloc
0000000000000004 C n_deletes
0000000000000004 C n_inserts
```

And some explanation of why the symbols are of types "T" (program text/code/strong symbols), "U" (undefined), or "C" (uninitialized/ weak symbols).

In the original `main.c` before we fix the output, the linker symbols are:
* `delete` undefined linker symbol
* `find` undefined linker symbol
* `insert` undefined linker symbol
* `main` strong linker symbol
* `n_deletes` strong linker symbol
* `n_inserts` strong linker symbol
* `printf` undefined linker symbol

Or nm main.o Output:
```
                 U __assert_fail
                 U delete
                 U find
                 U insert
0000000000000000 T main
0000000000000004 B n_deletes
0000000000000000 B n_inserts
000000000000009c r __PRETTY_FUNCTION__.2216 //Ignore
                 U printf
```

Followed by explanation.

In the binary `linkedlist` that we compiled with the original code, the linker symbols are:
* `delete` strong linker symbol
* `find` strong linker symbol
* `free` undefined linker symbol located in GLIBC
* `head` weak linker symbol
* `insert` strong linker symbol
* `main` strong linker symbol
* `malloc` undefined linker symbol located in GLIBC
* `n_deletes` picks the strong linker symbol from `main.c`
* `n_inserts` picks the strong linker symbol from `main.c`
* `printf` undefined linker symbol located in GLIBC

Or
```
U __assert_fail@@GLIBC_2.2.5
0000000000601058 B __bss_start
0000000000601058 b completed.6973
0000000000601048 D __data_start
0000000000601048 W data_start
00000000004007a4 T delete
0000000000400550 t deregister_tm_clones
00000000004005c0 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000601050 D __dso_handle
0000000000600e28 d _DYNAMIC
0000000000601058 D _edata
0000000000601070 B _end
0000000000400826 T find
00000000004008e4 T _fini
00000000004005e0 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400b40 r __FRAME_END__
                 U free@@GLIBC_2.2.5
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000601068 B head
0000000000400490 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
000000000040074a T insert
00000000004008f0 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
                 w _Jv_RegisterClasses
00000000004008e0 T __libc_csu_fini
0000000000400870 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
000000000040060d T main
                 U malloc@@GLIBC_2.2.5
0000000000601060 B n_deletes
000000000060105c B n_inserts
0000000000400994 r __PRETTY_FUNCTION__.2216
                 U printf@@GLIBC_2.2.5
0000000000400580 t register_tm_clones
0000000000400520 T _start
0000000000601058 D __TMC_END__
```

Followed by explanation.

### Linker and Shared library

The shared library we want to look at is located at a machine dependent path, but probably at `/lib/x86_64-linux-gnu/libc.so.6`. If we examine the output from `nm -D`, we see `malloc`, `free`, and `printf` defined in this shared library.

### Programming errors in the linking process

The output is incorrect because there are two definitions for `n_inserts` and `n_deletes`. One is a weak linker symbol, and the other is a strong linker symbol. If we look at the definition for `n_inserts` in `main.c`, we see that is it of type `int` and initialized to `0`. In `list.c`, `n_inserts` is defined as `float` and not initialized. In `main.c`, any operation with `n_inserts` treats the value there as an `int`. In `list.c`, any operation with `n_inserts` treats the value there as a `float`. That means in `list.c` whenever we say `n_inserts++`, a floating point instruction is executed. One solution is to simply change the definition of `n_inserts` and `n_deletes` in `list.c` to type `int`. The better solution is to only have one place where `n_inserts` and `n_deletes` are defined, and everywhere else use the `extern` keyword. Don't forget to recompile and relink, and then we have the output we expect.
