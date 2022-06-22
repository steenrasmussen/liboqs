The following options can be passed to CMake before the build file generation process to customize the way liboqs is built. The syntax for doing so is: `cmake .. [ARGS] [-D<OPTION_NAME>=<OPTION_VALUE>]...`, where `<OPTON_NAME>` is:

- [BUILD_SHARED_LIBS](#BUILD_SHARED_LIBS)
- [CMAKE_BUILD_TYPE](#CMAKE_BUILD_TYPE)
- [CMAKE_INSTALL_PREFIX](#CMAKE_INSTALL_PREFIX)
- [OQS_BUILD_ONLY_LIB](#OQS_BUILD_ONLY_LIB)
- [OQS_ENABLE_KEM_\<ALG\>/OQS_ENABLE_SIG_\<ALG\>](#OQS_ENABLE_KEM_\<ALG\>/OQS_ENABLE_SIG_\<ALG\>)
- [OQS_MINIMAL_BUILD](#OQS_MINIMAL_BUILD)
- [OQS_DIST_BUILD](#OQS_DIST_BUILD)
- [OQS_USE_OPENSSL](#OQS_USE_OPENSSL)
- [OQS_OPT_TARGET](#OQS_OPT_TARGET)
- [OQS_SPEED_USE_ARM_PMU](#OQS_SPEED_USE_ARM_PMU)
- [USE_SANITIZER](#USE_SANITIZER)
- [OQS_ENABLE_TEST_CONSTANT_TIME](#OQS_ENABLE_TEST_CONSTANT_TIME)

## BUILD_SHARED_LIBS

Can be set to `ON` or `OFF`. When `ON`, liboqs is built as a shared library. It is `OFF` by default, which means liboqs is built as a static library by default.

## CMAKE_BUILD_TYPE

Can be set to the following values:

- `Debug`: This turns off all compiler optimizations and produces debugging information. When the compiler is Clang, the [USE_SANITIZER](#USE_SANITIZER) option can also be specified to enable a Clang sanitizer. **This value only has effect when the compiler is GCC or Clang**

- `Release`: This compiles code at the `O3` optimization level, and sets other compiler flags that reduce the size of the binary.

## CMAKE_INSTALL_PREFIX

See the [CMake documentation](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html).

## OQS_ENABLE_KEM_\<ALG\>/OQS_ENABLE_SIG_\<ALG\>

This can be set to `ON` or `OFF`, and is `ON` by default. When `OFF`, `<ALG>` and its code are excluded from the build process. When `ON`, made available are additional options whereby individual variants of `<ALG>` can be excluded from the build process. 

For example: if `OQS_ENABLE_KEM_BIKE` is set to `ON`, the options `OQS_ENABLE_KEM_bike1_l1_cpa`, `OQS_ENABLE_KEM_bike1_l1_fo`, `OQS_ENABLE_KEM_bike1_l3_cpa`, `OQS_ENABLE_KEM_bike1_l3_fo` are made available (and are set to be `ON` by default). 

For a full list of such options and their default values, consult [.CMake/alg_support.cmake](https://github.com/open-quantum-safe/liboqs/blob/master/.CMake/alg_support.cmake).

## OQS_BUILD_ONLY_LIB

Can be `ON` or `OFF`. When `ON`, only liboqs is built, and all the targets: `run_tests`, `gen_docs`, and `prettyprint` are excluded from the build system.

## OQS_MINIMAL_BUILD

If set, this defines a semicolon deliminated list of algorithms to be contained in a minimal build of `liboqs`: Only algorithms explicitly set here are included in a build: For example running `cmake -DOQS_MINIMAL_BUILD="OQS_ENABLE_KEM_kyber_768;OQS_ENABLE_SIG_dilithium_3" ..` will build a minimum-size `liboqs` library only containing support for Kyber768 and Dilithium3.

The full list of identifiers that can set are listed [here for KEM algorithms](https://github.com/open-quantum-safe/liboqs/blob/main/src/kem/kem.h#L34) and [here for Signature algorithms](https://github.com/open-quantum-safe/liboqs/blob/f3caccff9e6225e7c50ca27f5ee6e58b7bc74188/src/sig/sig.h#L34). Default setting is empty, thus including all [supported algorithms](https://github.com/open-quantum-safe/liboqs#supported-algorithms) in the build.

## OQS_DIST_BUILD

Can be `ON` or `OFF`. When `ON`, build liboqs for distribution. When `OFF`, build liboqs for use on a single machine.

The library is always built for a particular architecture, either x86-64, ARM32v7, or ARM64v8, depending on the setting of CMAKE_SYSTEM_PROCESSOR. But liboqs contains code that is optimized for micro-architectures as well, e.g. x86-64 with the AVX2 extension.

When built for distribution, the library will run on any CPU of the target architecture. Function calls will be dispatched to micro-architecture optimized routines at run-time using CPU feature detection.

When built for use on a single machine, the library will only include the best available code for the target micro-architecture (see [OQS_OPT_TARGET](#OQS_OPT_TARGET)).

## OQS_USE_OPENSSL

This can be set to `ON` or `OFF`. When `ON`, the additional options `OQS_USE_AES_OPENSSL`, `OQS_USE_SHA2_OPENSSL`, and `OQS_USE_SHA3_OPENSSL` are made available to control whether liboqs uses OpenSSL's AES, SHA-2, and SHA-3 implementations. By default, `OQS_USE_AES_OPENSSL` and `OQS_USE_SHA2_OPENSSL` are `ON` while `OQS_USE_SHA3_OPENSSL` is `OFF`.

When `OQS_USE_OPENSSL` is `ON`, CMake also scans the filesystem to find the minimum version of OpenSSL required by liboqs (which happens to be 1.1.1). The `OPENSSL_ROOT_DIR` option can be set to aid CMake in its search.

## OQS_OPT_TARGET

An optimization target. Only has an effect if the compiler is GCC or Clang and `OQS_DIST_BUILD=OFF`. Can take any valid input to the `-march` (on x86_64) or `-mcpu` (on ARM32v7 or ARM64v8) option for `CMAKE_C_COMPILER`. Can also be set to one of the following special values.
  - `auto`: Use `-march=native` or `-mcpu=native` (if the compiler supports it).
  - `generic`: Use `-march=x86-64` on x86-64, or `-mcpu=cortex-a5` on ARM32v7, or `-mcpu=cortex-a53` on ARM64v8.

The default value is `auto`.

## OQS_SPEED_USE_ARM_PMU

Can be `ON` or `OFF`. When `ON`, the benchmarking script will try to use the ARMv8 Performance Monitoring Unit (PMU). This will make cycle counts on ARMv8 platforms significantly more accurate.

In order to use this option, user mode access to the PMU must be enabled via a kernel module. If user mode access is not enabled via kernel module, benchmarking will throw an `Illegal Instruction` error. A kernel module that has been found to work on several platforms can be found [here for linux](https://github.com/mupq/pqax#enable-access-to-performance-counters). Follow the instructions there (i.e., clone the repository, `cd enable_ccr` and `make install`) to load the kernel module, after which benchmarking should work. Superuser permissions are required. Linux header files must also be installed on your platform, which may not be present by default.

Note that this option is not known to work on Apple M1 chips.

## USE_SANITIZER

This has effect when the compiler is Clang and when [CMAKE_BUILD_TYPE](#CMAKE_BUILD_TYPE) is `Debug`. Then, it can be set to:

- `Address`: This enables Clang's `AddressSanitizer`
- `Memory`: This enables Clang's `MemorySanitizer`
- `MemoryWithOrigins`: This enables Clang's `MemorySanitizer` with the added functionality of being able to track the origins of uninitialized values
- `Undefined`: This enables Clang's `UndefinedBehaviorSanitizer`. The `BLACKLIST_FILE` option can be additionally set to a path to a file listing the entities Clang should ignore.
- `Thread`: This enables Clang's `ThreadSanitizer`
- `Leak`: This enables Clang's `LeakSanitizer`

## OQS_ENABLE_TEST_CONSTANT_TIME

This is used in conjunction with `tests/test_constant_time.py` to use Valgrind to look for instances of secret-dependent control flow.  liboqs must also be compiled with [CMAKE_BUILD_TYPE](#CMAKE_BUILD_TYPE) set to `Debug`.  See the documentation in [`tests/test_constant_time.py`](https://github.com/open-quantum-safe/liboqs/blob/main/tests/test_constant_time.py) for more information on usage.