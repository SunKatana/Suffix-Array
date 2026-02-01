# Assignment 1 — Naive Search vs Suffix Array Search (Python + C++) by the Jonas Brothers 

<img width="1536" height="1024" alt="ChatGPT Image Jan 13, 2026, 12_40_28 AM" src="https://github.com/user-attachments/assets/98c0b52f-8786-4f6b-b9ab-1a825fbbea90" />

This repository contains implementations and benchmarks for exact string search on DNA data:

* **Naive search** (baseline): scans the reference for each query
* **Suffix array search** (indexed): builds a suffix array once and answers queries using binary search

The **Python versions** are used for the experiments/benchmarking. The **C++ files** are included as part of the submission.

---

## Contents

### Python

* `common.py` — shared helpers (FASTA loading, query handling)
* `naive_search.py` — naive exact search
* `suffixarray_search.py` — suffix array build + binary search queries

### C++

* `naive_search.cpp`
* `suffixarray_search.cpp`

---

## Requirements

Recommended environment: **Linux** (e.g., FU compute servers)

* Python **3.9+**
* `python3-venv`

Optional (only if your system needs build tools for any dependency):

* `build-essential`
* `python3-dev`

Install (Debian/Ubuntu):

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-venv python3-dev build-essential
```

---
## What we did (implementation summary)

We started from the official template repository **SGSSGene/ImplementingSearch**:

https://github.com/SGSSGene/ImplementingSearch

Following the instructions provided in the template README, we:

1. **Cloned the repository (including submodules)** to get the full project structure and dependencies.
2. Implemented the missing parts marked as `//!TODO ImplementMe` in:
   - `src/naive_search.cpp`
   - `src/suffixarray_search.cpp`
3. Built the C++ project using CMake and the provided build setup.
4. Ran the compiled binaries and tested different input sizes by changing the `--query_ct` argument.
5. Measured and compared the runtime of:
   - the **naive search** approach
   - the **suffix array based** search approach

The goal was to compare the performance difference between a direct search baseline and an indexed search method.

---
## Setup

Create a virtual environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate

python -m pip install -U pip wheel
python -m pip install -U iv2py
```

Sanity check:

```bash
python -c "import iv2py; print('iv2py OK')"
```

---

## Data

You need:

* a reference FASTA file (e.g. `hg38_partial.fasta.gz`)
* a query FASTA file (Illumina reads, e.g. `illumina_reads_100.fasta.gz`)

Example paths on FU:

```bash
DATA=/home/mi/danim02/advalg-assignment1-python/c++/ImplementingSearch/data
R=$DATA/hg38_partial.fasta.gz
```

Available query files (example):

* `illumina_reads_40.fasta.gz`
* `illumina_reads_60.fasta.gz`
* `illumina_reads_80.fasta.gz`
* `illumina_reads_100.fasta.gz`

---

## How to run

### Naive search (baseline)

```bash
python naive_search.py --reference "$R" --query "$DATA/illumina_reads_40.fasta.gz" --query_ct 1000
```

### Suffix array search

```bash
python suffixarray_search.py --reference "$R" --query "$DATA/illumina_reads_40.fasta.gz" --query_ct 1000
```

### Arguments

* `--reference` : path to reference FASTA/FASTA.GZ
* `--query` : path to query FASTA/FASTA.GZ
* `--query_ct` : number of queries to take from the query file (used for benchmarking)

---

## Output

### `naive_search.py`

Prints:

* `total_hits    <int>`
* `search_ms     <int>`

Example:

```
total_hits    12345
search_ms     789
```

### `suffixarray_search.py`

Prints:

* `total_hits    <int>`
* `build_ms      <int>`
* `search_ms     <int>`

Example:

```
total_hits    12345
build_ms      120
search_ms     50
```

---

## Correctness check

For the **same** inputs (`reference`, `query`, `query_ct`), both implementations must report the **same**:

* `total_hits`

---

## Benchmarking

Recommended on Linux with GNU `time` to measure both runtime and memory.

### A) Query length = 100, varying number of queries (10³ / 10⁴ / 10⁵ / 10⁶)

```bash
Q100=$DATA/illumina_reads_100.fasta.gz

for N in 1000 10000 100000 1000000; do
  echo "== naive len=100 N=$N =="
  /usr/bin/time -v python naive_search.py --reference "$R" --query "$Q100" --query_ct "$N"

  echo "== suffix array len=100 N=$N =="
  /usr/bin/time -v python suffixarray_search.py --reference "$R" --query "$Q100" --query_ct "$N"
done
```

Record from `/usr/bin/time -v`:

* `Elapsed (wall clock) time`
* `Maximum resident set size (kbytes)`

Also record from program output:

* naive: `search_ms`
* suffix array: `build_ms`, `search_ms`

---

### B) Fixed number of queries `N`, varying query length (40 / 60 / 80 / 100)

Pick an `N` that finishes reasonably fast for naive search (e.g. `1000` or `10000`):

```bash
N=10000

for L in 40 60 80 100; do
  Q=$DATA/illumina_reads_${L}.fasta.gz

  echo "== naive len=$L N=$N =="
  /usr/bin/time -v python naive_search.py --reference "$R" --query "$Q" --query_ct "$N"

  echo "== suffix array len=$L N=$N =="
  /usr/bin/time -v python suffixarray_search.py --reference "$R" --query "$Q" --query_ct "$N"
done
```

### Assignment 2 python version:


```bash
DATA=/home/mi/tiloa00/Suffix-Array-main/data
R=$DATA/reference/text.dna4.short.fasta
```

Implement an fmindex based search
It requires the same dependencies and environment, in the addition of tracemalloc for memory checking.
Benchmarking Results:
```bash
for N in 1000 10000 100000 1000000; do

  for L in 100; do
    Q=$DATA/illumina_reads_${L}.fasta.gz

    echo "== suffix array len=$L N=$N =="
    /usr/bin/time -v python fmindex_search.py --reference "$R" --query "$Q" --query_ct "$N"
done
```

python suffix Benchmarks on home computer with text.dna4.short.fasta.index(Issues with running on the server):
--query_ct "100"
total_hits	40
build_ms	12544
search_ms	1

--query_ct "1000"
total_hits	448
build_ms	12545
search_ms	14

python fmindex Benchmarks on home computer:
--query_ct "100"
total_hits	40
build_ms	91156
search_ms	3

--query_ct "1000"
total_hits	448
build_ms	91220
search_ms	29

--query_ct "10000"
total_hits	4456
build_ms	94796
search_ms	286

--query_ct "100000"
total_hits	45335
build_ms	91413
search_ms	2641

--query_ct "1000000"
total_hits	453350
build_ms	91679
search_ms	26269

This Python's fmindex is always worse than the suffix array implementation in runtime, since it also requires a suffix array, and due to
```bash
self.Occ = {c: [0] * (self.n + 1) for c in self.alphabet}
```
since that uses Python integers and has a huge memory footprint.
It technically works, by having a stable FMindex buildtime of ~91500 ms, with a linearly scaling search ms time.

python fmindex Benchmarks on home computer while checking for memory usage using tracemalloc:
--query_ct "1000"
total_hits	448
index_mem_kb	10742296

--query_ct "10000"
total_hits	4456
index_mem_kb	10742295

Comparing to Server python execution:

== suffix array len=100 N=1000 ==
total_hits      448
build_ms        99906
search_ms       34
User time (seconds): 101.27
System time (seconds): 4.30
Elapsed (wall clock) time (h:mm:ss or m:ss): 1:45.63

== suffix array len=100 N=1000000 ==
total_hits      453350
build_ms        97869
search_ms       30759
User time (seconds): 130.09
System time (seconds): 4.32
Elapsed (wall clock) time (h:mm:ss or m:ss): 2:14.50

On the server, the runtime was a bit slower, but we can see with the linux time benchmark, that the runtime increase scales well with larger query_ct.

Benchmark the Human reference genome:

```bash
R=$DATA/reference/GCF_000001405.26_GRCh38_genomic.fna
N=10000
for L in 40 60 80 100; do
  Q=$DATA/illumina_reads_${L}.fasta.gz
  echo "== suffix array len=$L N=$N =="
  /usr/bin/time -v python fmindex_search.py --reference "$R" --query "$Q" --query_ct "$N"
done
```
At the time of submitting, it was still running. Attempts at running it locally caused PC crashes, and I only managed server access very late.


### Conclusion
Normally, the FMindex should be vastly superior in both runtime and memory compared to the suffix array. Our implementation is lacking in both. We also attempted implementing it in C++, but had issues with the Seqan3 integration, as described in the github, and didn't finish it in time. Our cpp FMindex is attached in the submission.
Our implementation did however achieve a runtime scaling well with query_ct.

Due to the bad runtimes of python implementations, we redid all implementations in C++:

# Search Algorithms implemented in C++:

This implementation work on the base of the given git directory, installable by using

```bash
git clone --recurse-submodules https://github.com/SGSSGene/ImplementingSearch
```

The submitted cpp files get put into the src directory. To use the code, we have to use the following code from the ImplementingSearch directory:

```bash
$ # We are assuming you are in the terminal/console inside the repository folder
$ mkdir build # creates a folder for our build system
$ cd build
$ cmake ..    # configures our build system
$ make        # builds our software, repeat this command to recompile your software
```

## Naive search
The first Algorithm implemented is the naive search. It works by sliding the query over the references, comparing at each possible position if the two sequences match. that has a O(n*m) runtime, which scales really badly.
To see how badly, here are the Benchmarks, executed from the build directory (Shown values are wall-time runtime and Maximum resident set size in kb:

```bash
N = 1000
for L in 40 60 80 100; do  
  echo "== naive search len=$L N=$N =="
  /usr/bin/time -f "Elapsed: %E\nMax RSS: %M KB" ./bin/naive_search --reference ../data/hg38_partial.fasta.gz --query ../data/illumina_reads_${L}.fasta.gz --query_ct "$N"
done
```

== naive search len=40 N=1000 ==

Total hits: 1300
Elapsed: 5:43.98
Max RSS: 212896 KB

== naive search len=60 N=1000 ==

Total hits: 720
Elapsed: 5:43.94
Max RSS: 217668 KB

== naive search len=80 N=1000 ==

Total hits: 563
Elapsed: 5:44.17
Max RSS: 218048 KB

== naive search len=100 N=1000 ==

Total hits: 448
Elapsed: 5:43.44
Max RSS: 219600 KB

== naive search len=40 N=10000 ==

Total hits: 20069
Elapsed: 57:08.87
Max RSS: 212876 KB

L is the size of the query moved over the reference. Larger L do not increase the runtime, but generally lead to less hits, especially if the smaller queries are sub-sequences of the longer ones. Larger N on the other hand mean larger References, which leads to linearly scaling runtime increases. In this example, increasing the Reference size by a factor of 10 leads to an increase of runtime by factor 10. The required memory remains pretty much the same over the different L and N. (All Benchmarks were performed on the server)

## Suffix array
The second Algorithm is the suffix array. It creates an array of integers giving the starting positions of all suffixes of the reference, sorted in lexicographic order. We used divsufsort for the suffix array creation, and used an naive binary search to find the query in the reference. This algorithm scales better than the naive search, with the array construction taking O(n log(n)), and the search taking O(m log(n)), with m= amount queries and n=length of reference.

```bash
for L in 40 60 80 100; do  
  echo "== suffix_array_search len=$L =="
  /usr/bin/time -f "Elapsed: %E\nMax RSS: %M KB" ./bin/suffixarray_search --reference ../data/hg38_partial.fasta.gz --query ../data/illumina_reads_${L}.fasta.gz 
done
```

== suffix_array_search len=40 ==

total_hits	50
Elapsed: 0:11.73
Max RSS: 603060 KB

== suffix_array_search len=60 ==

total_hits	43
Elapsed: 0:12.15
Max RSS: 606572 KB

== suffix_array_search len=80 ==

total_hits	40
Elapsed: 0:12.80
Max RSS: 607740 KB

== suffix_array_search len=100 ==

total_hits	40
Elapsed: 0:12.21
Max RSS: 609736 KB

We can see a huge increase in runtime, cutting it down from 5.44 minutes to 12 seconds. The cost however is a increased memory usage, going from 200MB to 600MB, since all suffixes of the Reference are saved. Changes in query length does not impact the runtime in a significant way.

## FMindex
The third Algorithm is the FMindex. fmindex_construct is first used, to create an fmindex. The code for the construct is given, but there is some uncertainty. The code itself creates a unidirectional fmindex with 
```bash
seqan3::fm_index index{reference}
```
while stating it saves a Bidirectional 2fmindex with the line

```bash
seqan3::debug_stream << "Saving 2FM-Index ... " << std::flush;
```
The index later gets loaded into the search with the given code

```bash
using Index = decltype(seqan3::fm_index{std::vector<std::vector<seqan3::dna5>>{}}); // Some hack
seqan3::debug_stream << "Loading 2FM-Index ... " << std::flush;
```
which is stating again its a 2FMindex, while loading a unidirectional fmindex. to load a bidirectional fm index, "decltype(seqan3::bi_fm_index{...})" would be used. We implemented the search using seqan3::search, which detects the optimal runtime between handling the index as uni- and bidirectional automatically, meaning the index type doesn't matter for the implementation, but it is still confusing.

The runtime of the FMindex search is O(Q⋅m⋅logσ) with Q=number of queries, m=avg. query length and σ=5 (DNA5 ranks)

FMindex benchmarking:

```bash
for N in 1000 10000 100000 1000000; do

  for L in 100; do  
    echo "== construct fm=$L N=$N =="
    ./bin/fmindex_construct --reference ../data/hg38_partial.fasta.gz --index myIndex.index

    echo "== search fm len=$L N=$N =="
    /usr/bin/time -f "Elapsed: %E\nUser CPU: %U s\nSys CPU: %S s\nMax RSS: %M KB" ./bin/fmindex_search --index myIndex.index --query "../data/illumina_reads_${L}.fasta.gz" --query_ct "$N" --errors 0
  done
done
```

== construct fm=100 N=1000 ==

Saving 2FM-Index ... done

== search fm len=100 N=1000 ==

Loading 2FM-Index ... done
Total hits: 448
Elapsed: 0:00.30
User CPU: 0.22 s
Sys CPU: 0.07 s
Max RSS: 83164 KB

== construct fm=100 N=10000 ==

Saving 2FM-Index ... done

== search fm len=100 N=10000 ==

Loading 2FM-Index ... done
Total hits: 4456
Elapsed: 0:00.29
User CPU: 0.23 s
Sys CPU: 0.05 s
Max RSS: 83636 KB

== construct fm=100 N=100000 ==

Saving 2FM-Index ... done

== search fm len=100 N=100000 ==

Loading 2FM-Index ... done
Total hits: 45335
Elapsed: 0:01.14
User CPU: 1.08 s
Sys CPU: 0.05 s
Max RSS: 83176 KB

== construct fm=100 N=1000000 ==

Saving 2FM-Index ... done

== search fm len=100 N=1000000 ==

Loading 2FM-Index ... done
Total hits: 453350
Elapsed: 0:09.80
User CPU: 9.62 s
Sys CPU: 0.15 s
Max RSS: 282548 KB

In terms of runtime, this is the fastest search yet, with results less than a second on N<10000. Memory also stays around 83MB within 100000 N. There is a noticable increase in memory and runtime for the N=1000000, but it still performs better than all previous algorithms.


Benchmark for reference genome GCF_000001405.26_GRCh38_genomic.fna:

```bash
./bin/fmindex_construct --reference ../data/GCF_000001405.26_GRCh38_genomic.fna --index myIndex.index
Saving 2FM-Index ... done

for L in 40 60 80 100; do
  echo "== loading index len=$L =="
  /usr/bin/time -f "Elapsed: %E\nMax RSS: %M KB" ./bin/fmindex_search --index myIndex.index --query ../data/illumina_reads_${L}.fasta.gz --query_ct 1000 --errors 0
done
```

== loading index len=40 ==

Loading 2FM-Index ... done
Total hits: 24391
Elapsed: 0:01.38
Max RSS: 2266652 KB

== loading index len=60 ==

Loading 2FM-Index ... done
Total hits: 8768
Elapsed: 0:01.32
Max RSS: 2270028 KB

== loading index len=80 ==

Loading 2FM-Index ... done
Total hits: 4561
Elapsed: 0:01.24
Max RSS: 2271120 KB

== loading index len=100 ==

Loading 2FM-Index ... done
Total hits: 929
Elapsed: 0:01.21
Max RSS: 2272616 KB

Due to multiple crashes (probably due to memory restrictions on the server, we managed to do it once we deleted some old files), we chose a query_ct of 1000, and performed the FMindex search on the human genome reference. We got the same runtime on all different query lengths, with a memory of 2.3GB needed. The index of the original 3.2GB of fasta data was 1.2GB big, and for the algortihm the entire index was read into memory, leading to a significantly higher memory usage than other algorithms.

## Pigeon hole search

The Pigeonhole Principle states that if you allow k errors in a pattern, and you divide that pattern into k+1 pieces, at least one of those pieces must match the reference exactly. So the query is broken off into k+1 pieces, and each piece is queried with seqan3::search(part, index, cfg). If a part finds an exact match, the entire sequence is tested, if it fits into the area with errors<=k. If it fits, it counts as a hit.
The given part of the code for the pigeon search again loads a unidirectional index, while stating its a bidirectional fmindex.

Benchmark comparing pigeonhole to fm search:

```bash
for E in 0 1 2 3; do
  echo "== fmindex_construct E=$E =="
  ./bin/fmindex_construct --reference ../data/hg38_partial.fasta.gz --index myIndex.index
  echo "== fmindex_search E=$E =="
  /usr/bin/time -f "Elapsed: %E\nMax RSS: %M KB" ./bin/fmindex_search --index myIndex.index --query ../data/illumina_reads_40.fasta.gz --query_ct 100 --errors $E
  echo "== pigeon_search E=$E =="
  /usr/bin/time -f "Elapsed: %E\nMax RSS: %M KB" ./bin/fmindex_pigeon_search --reference ../data/hg38_partial.fasta.gz --index myIndex.index --query ../data/illumina_reads_40.fasta.gz --query_ct 100 --errors $E
done
```

== fmindex_construct E=0 ==

Saving 2FM-Index ... done

== fmindex_search E=0 ==

Total hits: 50
Elapsed: 0:00.14
Max RSS: 76904 KB

== pigeon_search E=0 ==

Total hits: 50
Elapsed: 0:00.98
Max RSS: 276700 KB

== fmindex_construct E=1 ==

Saving 2FM-Index ... done

== fmindex_search E=1 ==

Total hits: 232
Elapsed: 0:00.16
Max RSS: 76928 KB

== pigeon_search E=1 ==

Total hits: 161
Elapsed: 0:00.84
Max RSS: 276812 KB

== fmindex_construct E=2 ==

Saving 2FM-Index ... done

== fmindex_search E=2 ==

Total hits: 1124
Elapsed: 0:00.39
Max RSS: 76904 KB

== pigeon_search E=2 ==

Loading 2FM-Index ... done

Total hits: 661
Elapsed: 0:01.07
Max RSS: 276956 KB

== fmindex_construct E=3 ==

Saving 2FM-Index ... done

== fmindex_search E=3 ==

Total hits: 5000
Elapsed: 0:04.04
Max RSS: 77072 KB

== pigeon_search E=3 ==

Total hits: 3081
Elapsed: 0:02.08
Max RSS: 284060 KB

On lower error count, the FMindex search outperfomres the pigeon hole FMindex search, but from E>=3, pigeonhole outperformes the FMindex search in terms of runtime. Pigeon generally uses more memory, and finds less hits. The lower hit count is due to the pigeon hole implementation using hamming distance, which doesn't account for insertions and deletions. It also skips hits, that are close to the beginning due to "if (hit_pos < begin){continue;}", which filters hits out, that are starting before the reference starts.

# Conclusion

These are our C++ implementations of the ImplementingSearch git repository. Last week, we had some communication and technical issues, leading to a badly implemented fm search being submitted, with a badly written report. We also had some issues properly interacting with the server (it stated we had to connect to compute03.mi.fu-berlin.de instead of using compute03.imp.fu-berlin.de). We hope that this redo of the report can rectify it.




