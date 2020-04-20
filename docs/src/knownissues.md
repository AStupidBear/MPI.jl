# Known issues

## UCX

[UCX](https://www.openucx.org/) is a communication framework used by several MPI implementations.

### RPATH handling

Prior to version 1.7.0, UCX did not [correctly handle dynamic library RPATHs](https://github.com/openucx/ucx/issues/4001). This could cause failures when loading subsequent libraries, giving an error message such as:
```
libXXXX.so: cannot open shared object file: No such file or directory
```

This can be fixed by either building MPI against a newer version of UCX, or setting the following environment variables before `MPI.Init()`:
```
ENV["UCX_MEM_MMAP_RELOC"] = "no"
ENV["UCX_MEM_MALLOC_HOOKS"] = "no"
ENV["UCX_MEM_MALLOC_RELOC"] = "no"
ENV["UCX_MEM_EVENTS"] = "no"
```

### Multi-threading and signal handling

When using [Julia multi-threading](https://docs.julialang.org/en/v1/manual/parallel-computing/#man-multithreading-1), the Julia garbage collector internally [uses `SIGSEGV` to synchronize threads](https://docs.julialang.org/en/v1/devdocs/debuggingtips/#Dealing-with-signals-1).

By default, UCX will error if this signal is raised ([#337](https://github.com/JuliaParallel/MPI.jl/issues/337)), resulting in a message such as:
```
Caught signal 11 (Segmentation fault: invalid permissions for mapped object at address 0xXXXXXXXX)
```

This signal interception can be controlled by setting the environment variable `UCX_ERROR_SIGNALS`: if not already defined, MPI.jl will set it as:
```
ENV["UCX_ERROR_SIGNALS"] = "SIGILL,SIGBUS,SIGFPE"
```
but if set externally, it should be modified to exclude `SIGSEGV` from the list.

## Microsoft MPI

### Custom operators on 32-bit Windows

It is not possible to use [custom operators with 32-bit Microsoft MPI](https://github.com/JuliaParallel/MPI.jl/issues/246), as it uses the `stdcall` calling convention, which is not supported by [Julia's C-compatible function pointers](https://docs.julialang.org/en/v1/manual/calling-c-and-fortran-code/index.html#Creating-C-Compatible-Julia-Function-Pointers-1).