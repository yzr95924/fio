# Agent Guidelines for fio Development

This document provides guidelines for AI agents working on the fio (Flexible I/O Tester) codebase. It covers build commands, test execution, code style, and development practices.

## Overview

fio is a C project with a modular architecture. The codebase includes:
- Core I/O engine and job management (`fio.c`, `init.c`, `io_u.c`)
- I/O engines in `engines/` directory
- Library components in `lib/`, `crc/`, `oslib/`
- Unit tests in `t/` and `unittests/`
- Functional tests in `t/jobs/` executed via Python test runner

## Build Commands

### Initial Configuration
```bash
./configure            # Generate config-host.mak and detect system capabilities
```

### Building
```bash
make                  # Build fio and unit test binaries
make -j$(nproc)       # Parallel build
```

### Installation
```bash
make install          # Install fio and scripts (requires appropriate permissions)
```

### Clean Build
```bash
make clean            # Remove object files
make distclean        # Remove all generated files including config
```

## Test Commands

### Smoke Test
```bash
make test             # Quick verification of basic functionality
```

### Full Test Suite
```bash
python3 t/run-fio-tests.py              # Run all functional tests
python3 t/run-fio-tests.py --debug      # With debug output
python3 t/run-fio-tests.py --run-only 1 2 3  # Run specific test IDs
python3 t/run-fio-tests.py --skip 6 1007 1008  # Skip specific tests
```

### Unit Tests
Individual unit test executables are built in `t/` and `unittests/`:
```bash
./t/stest             # Smalloc test
./t/lfsr-test         # LFSR test
./t/axmap             # Bitmap test
./unittests/unittest  # CUnit-based tests
```

### Specialized Tests
```bash
make fulltest         # Zoned block device tests (requires null_blk kernel module)
```

### CI Integration
The CI system uses scripts in `ci/`:
- `ci/actions-install.sh` - Install dependencies
- `ci/actions-build.sh` - Build with -Werror
- `ci/actions-smoke-test.sh` - Run smoke tests
- `ci/actions-full-test.sh` - Run full test suite

## Code Style Guidelines

### Language and Compiler
- C99 with GNU extensions (specified via `-std=gnu99`)
- Compiler warnings treated as errors (`-Werror` in CI)
- Use `__attribute__` macros defined in `compiler/compiler.h`

### Naming Conventions
- **Variables**: `snake_case`
- **Functions**: `snake_case`
- **Macros**: `UPPER_SNAKE_CASE`
- **Types**: `snake_case` with `_t` suffix for typedefs
- **Struct members**: `snake_case`

### Formatting
- **Indentation**: 1 tab = 8 spaces (Linux kernel style)
- **Braces**: Opening brace on its own line for functions, same line for statements
- **Line length**: Aim for 80 columns, but not strictly enforced
- **Spacing**: Space after keywords, no space before parentheses

#### Example
```c
static void example_function(struct thread_data *td)
{
	if (td->error) {
		log_info("fio: %s\n", td->verror);
		return;
	}

	do_something(td->io_u);
}
```

### Header Files
- Use include guards with `#ifndef FIO_FILENAME_H`
- System headers first, then project headers
- Group related includes logically
- Use forward declarations when possible

#### Example Include Order
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include "fio.h"
#include "parse.h"
#include "smalloc.h"
```

### Error Handling
- Use `errno` for system call errors
- Log errors with `log_info()`, `log_err()`, `log_dbg()` from `log.h`
- Return negative error codes or NULL for failure
- Use `__must_check` attribute for critical functions

### Memory Management
- Use `smalloc()` and `sfree()` from `smalloc.h` for shared memory
- Use standard `malloc()`/`free()` for thread-local allocations
- Always check return values of allocation functions
- Use `fio_unused` attribute for unused parameters

### Threading and Concurrency
- Use pthreads primitives directly
- Use `fio_sem`, `rwlock`, `pshared` abstractions for synchronization
- Document thread safety assumptions

### I/O Engine Development
- Follow patterns in existing engines in `engines/` directory
- Implement required functions defined in `ioengines.h`
- Use `dprint()` for debug logging within engines
- Handle `FIO_Q_BUSY` and `FIO_Q_QUEUED` states properly

## Linting and Static Analysis

### Compiler Warnings
The project is built with `-Wall -Wwrite-strings -Wdeclaration-after-statement`. Ensure code compiles without warnings.

### Static Analysis Tools
While not part of standard build, consider:
- `sparse` for semantic checking
- `cppcheck` for C/C++ analysis
- `clang-tidy` for modern C checks

### Code Review Checks
- Verify thread safety in concurrent code
- Check for memory leaks in error paths
- Validate I/O engine resource cleanup
- Ensure proper error propagation

## Commit Guidelines

### Commit Messages
Follow standard Linux kernel style:
```
subsystem: brief description

Detailed explanation of changes, rationale, and any side effects.
Wrap lines at 72 characters.

Signed-off-by: Name <email>
```

### Examples
```
init: fix memory leak in thread_data initialization

When job setup fails, clean up partially allocated thread_data
structures to prevent memory leaks.

Signed-off-by: John Doe <john@example.com>
```

```
ioengine/null: add support for trim operations

Implement trim callback for null ioengine to improve test coverage
for discard operations.

Signed-off-by: Jane Smith <jane@example.com>
```

### Testing Before Commit
- Run `make test` to ensure basic functionality
- For I/O engine changes, run relevant functional tests
- Consider running `./t/run-fio-tests.py --run-only <test_ids>` for specific areas

## Cursor and Copilot Rules

No Cursor rules (`.cursorrules` or `.cursor/rules`) or Copilot instructions (`.github/copilot-instructions.md`) are present in this repository. If added later, agents should follow them.

## Additional Resources

- `HOWTO.rst` - User documentation and examples
- `README.rst` - Project overview and build instructions
- `examples/` - Example job files for various workloads
- `doc/` - Additional documentation
- `tools/` - Helper scripts for parsing and plotting results

## Notes for AI Agents

- Always verify changes compile with `./configure && make`
- Run relevant tests before proposing changes
- Follow existing patterns in similar code
- Use project-specific abstractions (`smalloc`, `log_*`, `dprint`)
- Document non-obvious design decisions in code comments
- Consider cross-platform implications (Linux, macOS, Windows, BSD)
