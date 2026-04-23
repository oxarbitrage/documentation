# Building zcash on Apple Silicon (aarch64-apple-darwin)

The zcash build system does not natively support Apple Silicon (arm64/aarch64) Macs. Several issues prevent a successful build and runtime operation. This document describes the prerequisites, all four known problems, and their fixes.

## Prerequisites

### Xcode Command Line Tools

```bash
xcode-select --install
```

### Homebrew packages

```bash
brew install autoconf automake coreutils libtool pkgconf
```

These provide the autotools toolchain (`autoconf`, `automake`, `libtool`), GNU coreutils (for `nproc`, `readlink -f`, etc.), and `pkg-config` needed by the depends build system and `./configure`.

---

## Problem 1: Rust toolchain built for the wrong architecture

### Symptom

The build completes compilation but fails at link time with hundreds of undefined symbols:

```
ld: symbol(s) not found for architecture arm64
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
```

The missing symbols come from the Rust side of the codebase and include names like:

- `_tracing_callsite`, `_tracing_log`, `_tracing_span_create`, ...
- `_wallet$cxxbridge1$*` (Sapling note decryption, batch scanner, etc.)
- `_stream$cxxbridge1$*`
- `_unified_full_viewing_key_*`
- `_zcash_address_*`, `_zcash_transaction_*`
- `_zip339_*`
- `_transparent_key_ovks`

### Cause

The depends build system in `depends/packages/native_rust.mk` hardcodes `x86_64-apple-darwin` as the Rust target for all Darwin hosts. The `rust_target` macro reads:

```makefile
$(if ...,$(if $(findstring darwin,$(3)),x86_64-apple-darwin,...))
```

This means even on an aarch64 Mac, the Rust toolchain downloaded is the x86_64 version, and `RUST_TARGET` in `config.site` is set to `x86_64-apple-darwin`. The resulting `librustzcash.a` contains x86_64 object files that the arm64 linker cannot use.

### Fix

Three changes to `depends/packages/native_rust.mk`:

**1. Add the aarch64 Darwin Rust toolchain download (after the existing darwin entries):**

```makefile
$(package)_file_name_aarch64_darwin=rust-$($(package)_version)-aarch64-apple-darwin.tar.gz
$(package)_sha256_hash_aarch64_darwin=60a41dea4ae0f4006325745a6400e6fdc3e08ad3f924fac06f04c238cf23f4ec
```

The depends `funcs.mk` resolves `file_name_$(host_arch)_$(host_os)` before `file_name_$(host_os)`, so on aarch64 Macs this takes precedence over the x86_64 entry.

**2. Add the aarch64-apple-darwin rust-std SHA-256 hash (in the "Mapping from Rust targets to SHA-256 hashes" section):**

```makefile
$(package)_rust_std_sha256_hash_aarch64-apple-darwin=44809c3b92c7500c64517151f1e3389b32913a35414553395104bc4a0ee35f69
```

**3. Fix the `rust_target` macro to detect aarch64 Darwin:**

Change:

```makefile
define rust_target
$(if $($(1)_rust_target_$(2)),$($(1)_rust_target_$(2)),$(if $(findstring darwin,$(3)),x86_64-apple-darwin,$(if $(findstring freebsd,$(3)),x86_64-unknown-freebsd,$(2))))
endef
```

To:

```makefile
define rust_target
$(if $($(1)_rust_target_$(2)),$($(1)_rust_target_$(2)),$(if $(findstring darwin,$(3)),$(if $(findstring aarch64,$(2)),aarch64-apple-darwin,x86_64-apple-darwin),$(if $(findstring freebsd,$(3)),x86_64-unknown-freebsd,$(2))))
endef
```

The new logic checks if the canonical host contains `aarch64` before falling back to `x86_64-apple-darwin`.

> Note: The SHA-256 hashes above are for Rust 1.81.0. If the project updates its Rust version, regenerate the hashes using `./contrib/devtools/update-rust-hashes.sh` or by downloading the new archives and running `shasum -a 256` on them.

---

## Problem 2: `getauxval` linker error from crc32c

### Symptom

After fixing the Rust architecture issue, the build fails with:

```
Undefined symbols for architecture arm64:
  "_getauxval", referenced from:
      crc32c::Extend(unsigned int, unsigned char const*, unsigned long) in libcrc32c.a(libcrc32c_a-crc32c.o)
ld: symbol(s) not found for architecture arm64
```

### Cause

`getauxval` is a Linux-only function (it reads values from the ELF auxiliary vector). It does not exist on macOS.

The crc32c library uses `getauxval` at runtime to detect ARM CRC32C instruction support. It has two modes: "strong" (via `#include <sys/auxv.h>`) and "weak" (declaring `getauxval` with `__attribute__((weak))`).

During `./configure`, the strong check correctly fails on macOS because `<sys/auxv.h>` does not exist. However, the weak check in `configure.ac` uses `AC_COMPILE_IFELSE`, which only tests compilation, not linking. Since clang on macOS accepts the `__attribute__((weak))` declaration as valid syntax, the compile test passes and sets `HAVE_WEAK_GETAUXVAL=1`. At link time, the Mach-O linker cannot resolve `getauxval` because the symbol does not exist anywhere on the system.

On Linux ELF, a weak undefined symbol resolves to zero at link time. On macOS Mach-O, this does not work the same way.

### Fix

In `configure.ac`, change the weak `getauxval` check from `AC_COMPILE_IFELSE` to `AC_LINK_IFELSE`:

```diff
 AC_MSG_CHECKING(for weak getauxval support in the compiler)
-AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
+AC_LINK_IFELSE([AC_LANG_PROGRAM([[
     unsigned long getauxval(unsigned long type) __attribute__((weak));
     #define AT_HWCAP 16
   ]], [[
     getauxval(AT_HWCAP);
   ]])],
  [ AC_MSG_RESULT(yes); HAVE_WEAK_GETAUXVAL=1 ],
  [ AC_MSG_RESULT(no); HAVE_WEAK_GETAUXVAL=0 ]
 )
```

This causes the configure test to actually link the test program. On macOS the link fails, correctly setting `HAVE_WEAK_GETAUXVAL=0`. On Linux the test continues to pass since ELF handles weak symbols correctly.

With `HAVE_WEAK_GETAUXVAL=0` and `HAVE_STRONG_GETAUXVAL=0`, the crc32c code skips the `getauxval` runtime check entirely. On Apple Silicon the CRC32 instructions are always available as part of the base architecture, so no runtime detection is needed.

---

## Problem 3: BerkeleyDB mutex failure at runtime

### Symptom

zcashd builds successfully but crashes on startup when opening the wallet database:

```
CDBEnv::Open: Error -30972 opening database environment: BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery
```

The BDB error log (`regtest/db.log`) shows:

```
BDB2015 Unable to acquire/release a mutex; check configuration
BDB0061 PANIC: BDB0069 DB_LOCK_NOTGRANTED: Lock not granted
```

The process may also abort with:

```
Assertion failed: (!posix::pthread_mutex_destroy(&m)), function ~mutex
```

### Cause

BerkeleyDB 6.2.23 has a mutex implementation selection bug on macOS ARM64. During its configure step, BDB's `dist/aclocal/mutex.m4` determines which mutex backend to use:

1. **POSIX shared pthreads are skipped on all Darwin** (lines 184-193 of `mutex.m4`). A comment explains that macOS 10.7 broke `pthread_*_setpshared(PTHREAD_PROCESS_SHARED)` — it returns success instead of EINVAL, but the shared mutexes don't actually work. BDB avoids testing for them entirely on Darwin.

2. **With shared pthreads skipped, BDB falls through to test-and-set mutexes.** On macOS it finds `Darwin/_spin_lock_try` (line 443), which uses the deprecated `OSSpinLock` internals. On Apple Silicon, these spin locks have known priority inversion bugs and may fail to acquire/release properly.

3. **The result**: BDB selects a broken mutex backend, and any attempt to open a database environment fails immediately.

### Fix

In `depends/packages/bdb.mk`, add `--enable-posixmutexes` to the Darwin config options:

```diff
 $(package)_config_opts_aarch64=--disable-atomicsupport
+$(package)_config_opts_darwin+=--enable-posixmutexes
```

This forces BDB's configure to set `db_cv_mutex=posix_only`. Since shared pthreads are skipped on Darwin, the configure script falls through to test **private** (thread-only) POSIX pthreads (`POSIX/pthreads/private`), which sets both `HAVE_MUTEX_PTHREADS` and `HAVE_MUTEX_THREAD_ONLY`.

Thread-private POSIX mutexes work correctly on macOS — only the process-shared variant is broken. Since zcashd uses BDB within a single process, thread-private mutexes are sufficient.

---

## Problem 4: Compilation errors with newer Xcode / macOS SDK

### Symptom

On systems with a recent Xcode (Clang 20+) and a newer macOS SDK (e.g. `MacOSX26.4.sdk`), the build fails with errors like:

```
./support/allocators/secure.h:25:28: error: 'pointer' is deprecated [-Werror,-Wdeprecated-declarations]
   25 |     typedef typename base::pointer pointer;
```

```
./util/time.h:32:28: error: 'instance' may be duplicated when built into a shared library: it is mutable,
    with external linkage and hidden visibility [-Werror,-Wunique-object-duplication]
   32 |         static SystemClock instance;
```

These appear across many translation units, producing dozens of errors.

### Cause

The zcash build uses `-Weverything -Werror` and selectively disables specific warnings. Two warnings that didn't exist (or didn't fire) with the older SDK/Clang shipped in the depends system are now triggered by newer system SDK headers:

1. **`-Wdeprecated-declarations`**: In C++17, `std::allocator::pointer`, `const_pointer`, `reference`, and `const_reference` typedefs were deprecated. The newer libc++ in the macOS SDK marks them with `[[deprecated]]`. The zcash custom allocators (`secure_allocator` in `support/allocators/secure.h` and `zero_after_free_allocator` in `support/allocators/zeroafterfree.h`) inherit these typedefs from `std::allocator`, triggering the warning.

2. **`-Wunique-object-duplication`**: A new warning introduced in Clang 20 that flags mutable static variables with external linkage and hidden visibility (`-fvisibility=hidden`), because they could be duplicated across shared library boundaries. This fires on the `CALLSITE` variables in the tracing/logging macros (`rust/include/tracing.h`, `rust/include/rust/metrics.h`) and singleton clock instances in `util/time.h`. This is a false positive for zcashd since it is a statically linked executable, not a shared library.

### Fix

In `configure.ac`, add both warnings to the `DISABLED_WARNING_CXXFLAGS` block (around line 358):

```diff
 DISABLED_WARNING_CXXFLAGS="\
   -Wno-c++20-compat -Wno-cast-align -Wno-cast-qual -Wno-comma -Wno-conditional-uninitialized \
   -Wno-covered-switch-default -Wno-ctad-maybe-unsupported \
   -Wno-delete-non-abstract-non-virtual-dtor -Wno-deprecated-copy -Wno-deprecated-copy-dtor \
+  -Wno-deprecated-declarations \
   -Wno-deprecated-dynamic-exception-spec -Wno-disabled-macro-expansion -Wno-documentation \
   ...
   -Wno-unsafe-buffer-usage \
+  -Wno-unique-object-duplication \
   -Wno-unused-but-set-variable -Wno-unused-exception-parameter -Wno-unused-function \
```

These are safe to suppress. The deprecated allocator typedefs are a known C++17 cleanup that does not affect functionality. The object duplication warning is irrelevant because zcashd is a static binary — the flagged variables cannot actually be duplicated.

> Note: This problem only affects systems with a newer Xcode/SDK than the one used by the depends build system's bundled Clang. If you are building on an older macOS version, you may not encounter this issue.

---

## Build instructions after applying all fixes

After making all changes (Problems 1-3 are required; Problem 4 only if you have a newer Xcode/SDK), clean any previous build artifacts and rebuild:

```bash
make distclean
rm -rf depends/aarch64-apple-darwin*/
rm -rf target/
./zcutil/build.sh -j$(nproc)
```

The full depends rebuild is necessary because the old depends prefix contains the x86_64 Rust toolchain, a `config.site` with the wrong `RUST_TARGET`, and BDB built with the wrong mutex backend.

### Verifying the build

```bash
file src/zcashd
# Expected: Mach-O 64-bit executable arm64

src/zcashd --version
```

### Verifying the wallet works

Create a regtest data directory and config, then confirm zcashd starts without BDB errors:

```bash
mkdir -p /tmp/zcash-regtest
echo "i-am-aware-zcashd-will-be-replaced-by-zebrad-and-zallet-in-2025=1" > /tmp/zcash-regtest/zcash.conf
src/zcashd -regtest -datadir=/tmp/zcash-regtest -printtoconsole=1
```

You should see `CDBEnv::Open: LogDir=...` followed by successful initialization (no `DB_RUNRECOVERY` error). In another terminal, verify the node responds:

```bash
src/zcash-cli -regtest -datadir=/tmp/zcash-regtest getblockchaininfo
```

### Warnings you can safely ignore

The build may emit these warnings, which do not affect functionality:

- `ld: warning: ignoring duplicate libraries: '-lc++'`
- `ld: warning: object file (...librustzcash.a...) was built for newer 'macOS' version (XX.X) than being linked (11.0)`

The macOS version mismatch warning occurs because the Rust toolchain targets a newer minimum macOS version than the C++ build flags specify. The resulting binary will still work correctly on your system.
