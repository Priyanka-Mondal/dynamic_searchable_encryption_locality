# Dynamic Searchable Encryption with Locality

A C++ implementation of **locality-aware dynamic searchable symmetric encryption (SSE)** — specifically the *de-amortized SDd* scheme. It allows a client to insert, delete, and search over an encrypted index on an untrusted server, while leaking minimal information about access patterns.

## How it works

- **Insertions / Deletions**: keyword-file pairs are added or removed from an encrypted index using oblivious operations
- **Search**: the client queries for all files associated with a keyword; the server learns only the encrypted result, not the keyword or access pattern
- **Locality-aware design**: data is organized to minimize non-contiguous memory accesses, improving I/O efficiency
- **De-amortization**: expensive rebuild operations are spread across queries so each operation runs in bounded time

Core cryptographic primitives: AES-256-CBC (OpenSSL) + oblivious RAM (ORAM) + oblivious map (OMAP).

---

## Dependencies

- Linux (tested on GNU/Linux)
- `g++` with C++14 support
- `make`
- OpenSSL development headers

Install on Ubuntu/Debian:
```bash
sudo apt-get install build-essential libssl-dev
```

---

## Build

```bash
git clone https://github.com/Priyanka-Mondal/dynamic_searchable_encryption_locality.git
cd dynamic_searchable_encryption_locality

# Debug build (default)
make build

# Release build
make CONF=Release build
```

Binaries are placed in:
- Debug: `dist/Debug/GNU-Linux/sse-locality-sda`
- Release: `dist/Release/GNU-Linux/sse-locality-sda`

---

## Run

```bash
# Run with default config
./dist/Debug/GNU-Linux/sse-locality-sda

# Run with a custom config file
./dist/Debug/GNU-Linux/sse-locality-sda smallconfig.txt
```

### Config file format (`config.txt`)

```
<overwrite: true/false>      # true = generate fresh test data; false = reuse existing search.txt
<inMemory: true/false>       # true = keep index in RAM
<number of test cases>
<N: database size>
<result size 1>
<result size 2>
<result size 3>
<deletion count>
<stash size>
<dummy rate>
<instance count>
<block size>
<page size>
<use OMAP: 0/1>
```

Example (`config.txt`):
```
false
true
1
5000
25
4
1000
0
355
0
20
10
10
1
```

The first run should use `overwrite = true` to generate the test data. Subsequent runs can set it to `false` to reuse the same keyword set (for reproducible benchmarks).

---

## Benchmark

To run benchmarks over 1M-record datasets across varying result sizes and deletion rates (10 iterations each):

```bash
bash test.sh
```

Results are appended to files under `results/`. Make sure the `tests/` and `results/` directories exist and contain the appropriate dataset files before running.

---

## Project structure

| File | Description |
|------|-------------|
| `main.cpp` | Entry point — reads config, runs insert/delete/search |
| `DeAmortizedSDdNoOMAP.{h,cpp}` | Core de-amortized searchable encryption client |
| `ORAM.{hpp,cpp}` | Oblivious RAM tree implementation |
| `OMAP.{h,cpp}` | Oblivious map |
| `AES.{hpp,cpp}` | AES-256-CBC encryption wrapper |
| `OneChoiceClient/Server/Storage` | One-choice bin protocol |
| `TwoChoicePPwithStash*` | Two-choice bin protocol with stash |
| `AVLTree.{h,cpp}` | AVL tree for indexing |
| `bucketSort.{h,c}` | Oblivious bucket sort |
| `Utilities.{h,cpp}` | Config parsing, timers, test case generation |
| `config.txt` | Default runtime configuration |
| `test.sh` | Benchmark script |