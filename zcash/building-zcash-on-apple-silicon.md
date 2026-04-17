# Building zcash on Apple Silicon (aarch64-apple-darwin)

The zcash build system does not natively support Apple Silicon (arm64/aarch64) Macs. Two issues prevent a successful build on these machines. This document describes the prerequisites, both problems, and their fixes.

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

## Build instructions after applying fixes

After making both changes, clean any previous build artifacts and rebuild:

```bash
make distclean
rm -rf depends/aarch64-apple-darwin*/
rm -rf target/
./zcutil/build.sh -j$(nproc)
```

The full depends rebuild is necessary because the old depends prefix contains the x86_64 Rust toolchain and a `config.site` with the wrong `RUST_TARGET`.

### Verifying the build

```bash
file src/zcashd
# Expected: Mach-O 64-bit executable arm64

src/zcashd --version
```

### Warnings you can safely ignore

The build may emit these warnings, which do not affect functionality:

- `ld: warning: ignoring duplicate libraries: '-lc++'`
- `ld: warning: object file (...librustzcash.a...) was built for newer 'macOS' version (XX.X) than being linked (11.0)`

The macOS version mismatch warning occurs because the Rust toolchain targets a newer minimum macOS version than the C++ build flags specify. The resulting binary will still work correctly on your system.
