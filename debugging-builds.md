# Troubleshooting Common Build Failures

Compiling software from source can be a complex process, and builds can fail for many reasons. This guide covers some of the most common errors you might encounter and provides strategies for debugging them.


## General Debugging Strategies

Before diving into specific errors, here are some general tips that can help you solve almost any build problem:

1.  **Read the Error Message Carefully**: This sounds obvious, but it's the most important step. Scroll up from the end of the build output. The first error message is often the most important one; subsequent errors are frequently a cascade caused by the initial problem.
2.  **Check the Logs**: If the build system creates log files (like `config.log`), they contain a wealth of information. Use `grep` to search these logs for keywords like "error", "fail", or the name of the library that's causing problems.
3.  **Understand Your Environment**: Run `module list` to see what modules you have loaded. Is it the correct compiler? Have you loaded the necessary libraries (e.g., `openmpi`, `hdf5`)? Is it possible you have conflicting modules loaded?
4.  **Start Clean**: If you are re-running a build after making changes, it's often a good idea to start from a clean directory. Remnants from a previous failed build can sometimes cause strange errors. Run `make clean` or remove the build directory and start over.

## Common Errors and How to Fix Them

### Error: `./configure: command not found` or `make: command not found`

-   **What it means**: The shell cannot find the `configure` script or the `make` program in your `$PATH`.
-   **How to fix it**:
    -   Make sure you are in the correct directory (the top-level source directory of the software). Use `ls` to check if the `configure` script is there.
    -   If the script is there but not executable, run `chmod +x configure`.
    -   The `make` command and other essential build tools (like a C compiler) are usually provided by the system or a "build essentials" type of module. On many systems, you may need to load a `gcc` or similar compiler module to bring these tools into your path.

### Error: `configure: error: C compiler cannot create executables`

-   **What it means**: This is a generic error from a `configure` script indicating that the compiler is not working correctly. The real reason is usually found in the `config.log` file.
-   **How to fix it**:
    -   Open `config.log` and search for the last occurrence of "error".
    -   Often, this is caused by the compiler not being able to find a required library or because of an incompatibility between the compiler and the system libraries.
    -   Ensure you have a compiler module loaded. Sometimes, you may need to switch to a different version of the compiler.

### Error: `fatal error: some_header.h: No such file or directory`

-   **What it means**: The compiler cannot find a header file (`.h` file) that the source code needs. Header files define the interfaces to libraries.
-   **How to fix it**:
    -   Identify which software package provides `some_header.h`. You can often find this with a web search or by using your system's package manager commands if applicable.
    -   Load the module for that software package. For example, if `mpi.h` is missing, you need to `module load openmpi` (or another MPI implementation).
    -   If you have the dependency installed in a non-standard location, you may need to tell the compiler where to find it. This is usually done by setting environment variables before running `./configure`:
        ```bash
        export CFLAGS="-I/path/to/dependency/include"
        export LDFLAGS="-L/path/to/dependency/lib"
        ./configure ...
        ```
        The `-I` flag adds a directory to the header search path.

### Error: `ld: cannot find -l<library_name>` (e.g., `ld: cannot find -lssl`)

-   **What it means**: The **linker** (`ld`) cannot find a required library file. The `-l` flag is a shorthand; for example, `-lssl` tells the linker to look for a file named `libssl.so` (shared library) or `libssl.a` (static library).
-   **How to fix it**:
    -   This is the library-equivalent of the missing header file error. You need to find which package provides the library and load its module.
    -   If the library is in a non-standard path, you need to tell the linker where to look. This is done with the `LDFLAGS` environment variable, which uses the `-L` flag to add a directory to the library search path.
        ```bash
        export LDFLAGS="-L/path/to/dependency/lib"
        ./configure ...
        ```

### Error: `undefined reference to 'some_function'`

-   **What it means**: The linker found the library, but the function `some_function` is not defined in any of the libraries it was told to link against.
-   **How to fix it**:
    -   This is one of the trickiest errors. It can mean several things:
        -   You are not linking against the correct library. You might have forgotten a `-l<another_library>` flag. Check the software's documentation for its dependencies.
        -   There is a version mismatch. The header file you are using might be from a newer version of a library than the actual library file you are linking against.
        -   If it's a C++ program, the error might be due to a name mangling issue, often caused by trying to link C code with C++ code incorrectly or by a compiler mismatch.
    -   The first step is always to double-check the software's required dependencies and make sure you have loaded the correct modules for all of them.

## Asking for Help

If you get stuck, don't hesitate to ask for help from Computational Scientists. To help them solve your problem quickly, please provide the following information:

-   The name and version of the software you are trying to build.
-   The commands you used to configure and run the build.
-   The output of `module list`.
-   The full, unedited error message and the last 20-30 lines of output before the error occurred. If possible, attach the full log file.
