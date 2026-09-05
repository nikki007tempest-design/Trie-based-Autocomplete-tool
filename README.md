# Trie-Based Autocomplete Tool

A prefix-search autocomplete engine implemented from scratch as a C++ **Trie**,
exposed over a small REST API, with a lightweight web UI on top.

## Why this project

Autocomplete is a classic showcase for Trie data structures: insertion,
deletion, and prefix search all run in **O(L)** time, where `L` is the length
of the word/prefix — independent of dictionary size. This project pairs that
core DSA implementation with a thin web layer so it's demoable end-to-end,
not just a console program.

## Features

- **Trie core** (`src/Trie.h`, `src/Trie.cpp`)
  - `insert(word, weight)` — add a word, with an optional popularity weight
  - `remove(word)` — delete a word, pruning now-unused nodes
  - `contains(word)` — exact match lookup
  - `autocomplete(prefix, limit)` — top-`k` suggestions for a prefix, ranked
    by frequency (descending), then alphabetically
  - `loadFromFile(path)` — bulk-load a dictionary (`word,weight` per line)
- **REST API** (`src/main.cpp`, via [cpp-httplib](https://github.com/yhirose/cpp-httplib))
  - `GET /api/suggest?prefix=he&limit=8` — ranked suggestions
  - `POST /api/words` (`word=...&weight=...`) — insert a word at runtime
  - `DELETE /api/words/:word` — remove a word
  - `GET /api/stats` — number of words currently indexed
- **Frontend** (`frontend/index.html`) — a single-page debounced search box
  that calls the API and renders ranked suggestions with click-to-select.

## Project structure

```
trie-autocomplete/
├── CMakeLists.txt
├── include/
│   └── httplib.h        # vendored single-header HTTP library
├── src/
│   ├── Trie.h
│   ├── Trie.cpp
│   └── main.cpp
├── data/
│   └── words.txt         # sample dictionary (word,weight)
└── frontend/
    └── index.html         # search UI, calls the REST API
```

## Building and running

### Option 1: CMake

```bash
mkdir build && cd build
cmake ..
cmake --build .
./trie_server
```

### Option 2: Direct g++ (no CMake required)

```bash
g++ -std=c++17 -O2 -Iinclude -pthread src/main.cpp src/Trie.cpp -o trie_server
cp -r frontend data .   # if building outside the project root
./trie_server
```

Then open **http://localhost:8080** in a browser, or hit the API directly:

```bash
curl "http://localhost:8080/api/suggest?prefix=dev&limit=5"
# ["developer","development","develop","dev"]
```

## Design notes

- **Ranking**: matches are sorted by frequency (a rough proxy for
  "popularity"), then lexicographically, so common words surface first.
- **Concurrency**: the trie is guarded by a single mutex since HTTP handlers
  can run on multiple threads; this project prioritizes correctness over
  fine-grained locking.
- **No external JSON/HTTP framework dependency** beyond the vendored
  single-header `httplib.h`, keeping the build simple for a portfolio project.

## Possible extensions

- Fuzzy/typo-tolerant matching (edit-distance search over the trie)
- Persisting runtime inserts back to `data/words.txt`
- Weighting suggestions by recency in addition to frequency
- Swapping the in-memory trie for a compressed trie (DAWG) for larger dictionaries

## License

MIT — feel free to use or adapt this for your own learning/portfolio.
