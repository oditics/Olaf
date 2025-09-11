# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Olaf is a lightweight acoustic fingerprinting library written in C. It can run on traditional computers, embedded devices (ESP32, ARM), and in web browsers via WebAssembly. The system extracts audio fingerprints and either stores them in a database or matches them against stored fingerprints.

## Build System

### Primary Build - Traditional Computers
```bash
make           # Compile default version with LMDB key-value store
make install   # Install to /usr/local/bin/
```

### Alternative Build Methods
```bash
make mem       # Memory-only version (no LMDB, for embedded/testing)
make web       # WebAssembly version using Emscripten
make lib       # Shared library version
zig build      # Cross-platform compilation using Zig
```

### Testing
```bash
make test                           # Compile and run unit tests
ruby eval/olaf_functional_tests.rb  # Functional integration tests
```

### Cleanup
```bash
make clean      # Remove build artifacts
make destroy_db # Delete database files
```

## Architecture

### Core Components

**Audio Processing Pipeline:**
- `olaf_reader_stream.c` - Audio input handling (streaming or single-file)
- `olaf_stream_processor.c` - Audio stream processing coordinator
- `olaf_ep_extractor.c` - Event point extraction from audio
- `olaf_fp_extractor.c` - Fingerprint generation from event points

**Storage Systems:**
- `olaf_db.c` + LMDB - Persistent key-value storage (traditional computers)
- `olaf_db_mem.c` - In-memory storage (embedded/web)
- `olaf_fp_db_writer*.c` - Database writing implementations

**Matching:**
- `olaf_fp_matcher.c` - Fingerprint matching against database
- `olaf_runner.c` - High-level application logic

**Configuration:**
- `olaf_config.c` - Compile-time configuration parameters

### Build Variants

1. **Default** - Full LMDB database for traditional computers
2. **Memory (`-Dmem`)** - Header-file based storage for embedded
3. **Web** - WebAssembly with memory storage

### External Dependencies

- **Required:** `ffmpeg` (audio decoding), `ruby` (CLI interface)
- **Build:** `gcc` (C11), `emcc` (WebAssembly), or `zig`
- **Optional:** `doxygen` (documentation), `threach` gem (multi-threading)

## Development Commands

### Primary Interface
The main interface is the Ruby script `olaf.rb` which wraps the C binary:
```bash
olaf store audio_file.mp3    # Store fingerprints
olaf query audio_file.mp3    # Match against database
olaf stats                   # Database statistics
olaf clear                   # Clear database
```

### Configuration
- Compile-time config in `src/olaf_config.c`
- Ruby script config at top of `olaf.rb` (audio processing, paths)
- Default database location: `~/.olaf/db/`

### Multi-platform Considerations
- Memory version uses header files instead of LMDB
- WebAssembly version excludes LMDB components
- ESP32/embedded builds use memory storage only

## Code Style
- C11 standard with GNU extensions for `pffft.c`
- Object-oriented style in C with clear module separation
- Consistent naming: `olaf_module_function` pattern
- Compile with `-W -Wall -std=c11 -pedantic -O2`