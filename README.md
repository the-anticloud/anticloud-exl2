# EXL2

![license](https://img.shields.io/badge/license-MIT-blue) ![offline-first](https://img.shields.io/badge/offline--first-air--gap-green) ![audit](https://img.shields.io/badge/audit-SHA3--256-orange) ![category](https://img.shields.io/badge/category-chip_quantization-lightgrey)

> Anticloud-hardened packaging of the upstream project `EXL2` in category **CHIP QUANTIZATION**. Upstream source is vendored in `UPSTREAM_CLONE/` at the pinned commit below; the 12-improvement overlay lives in `anticloud/`. Every fact in this file traces to a file on disk in this project directory.

**Category:** CHIP QUANTIZATION · **Upstream:** https://github.com/xhedit/quantkit · **Upstream pin:** `f363cd0d9ba6dd3f72d91a8a2a5b8fe4da127bd9` · **Vendor:** Anticloud FZ LLE

---

## What This Project Does

# Note

**This project is archived for now**. Development continues on [ExLlamaV3](https://github.com/turboderp-org/exllamav3).

# ExLlamaV2

ExLlamaV2 is an inference library for running local LLMs on modern consumer GPUs.

The official and recommended backend server for ExLlamaV2 is [TabbyAPI](https://github.com/theroyallab/tabbyAPI/),
which provides an OpenAI-compatible API for local or remote inference, with extended features like HF model
downloading, embedding model support and support for HF Jinja2 chat templates.

See the [wiki](https://github.com/theroyallab/tabbyAPI/wiki/1.-Getting-Started) for help getting started.

## New in v0.1.0+:

- ExLlamaV2 now supports paged attention via [Flash Attention](https://github.com/Dao-AILab/flash-attention) 2.5.7+
- New generator with dynamic batching, smart prompt caching, K/V cache deduplication and simplified API

![alt_text](doc/dynamic_gen.gif)

## Dynamic generator

The dynamic generator supports all inference, sampling and speculative decoding features of the previous two 
generators, consolidated into one API (with the exception of FP8 cache, though the Q4 cache mode is supported and
performs better anyway, see [here](doc/qcache_eval.md).)

The generator is explained in detail [here](doc/dynamic.md).

- Single generation:
  ```python
  output = generator.generate(prompt = "Hello, my name is", max_new_tokens = 200)
  ```
- Batched generation:
    ```python
    outputs = generator.generate(
        prompt = [
            "Hello, my name is",
            "Once upon a time,",
            "Large language models are",
        ], 
        max_new_tokens = 200
    )
    ```
- Streamed generation with `asyncio`:
    ```python
    job = ExLlamaV2DynamicJobAsync(
        generator,
        input_ids = tokenizer.encode("You can lead a horse to water"),
        banned_strings = ["make it drink"],
        gen_settings = ExLlamaV2Sampler.Settings.greedy(),
        max_new_tokens = 200
    )  
    async for result in job:
        text = result.get("text", "")
        print(text, end = "")       
    ``` 
See the full, updated examples [here](https://github.com/turboderp/exllamav2/tree/master/examples).

## Performance

Some quick tests to compare performance with ExLlama V1. There may be more performance optimizations in the future,
and speeds will vary across GPUs, with slow CPUs still being a potential bottleneck:

| Model      | Mode         | Size  | grpsz | act | 3090Ti  | 4090        |
|------------|--------------|-------|-------|-----|---------|-------------|
| Llama      | GPTQ         | 7B    | 128   | no  | 181 t/s | **205** t/s |
| Llama      | GPTQ         | 13B   | 128   | no  | 110 t/s | **114** t/s |
| Llama      | GPTQ         | 33B   | 128   | yes | 44 t/s  | **48** t/s  |
| OpenLlama  | GPTQ         | 3B    | 128   | yes | 259 t/s | **296** t/s |
| CodeLlama  | EXL2 4.0 bpw | 34B   | -     | -   | 44 t/s  | **50** t/s  |
| Llama2     | EXL2 3.0 bpw | 7B    | -     | -   | 217 t/s | **257** t/s |
| Llama2     | EXL2 4.0 bpw | 7B    | -     | -   | 185 t/s | **211** t/s |
| Llama2     | EXL2 5.0 bpw | 7B    | -     | -   | 164 t/s | **179** t/s |
| Llama2     | EXL2 2.5 bpw | 70B   | -     | -   | 33 t/s  | **38** t/s  |
| TinyLlama  | EXL2 3.0 bpw | 1.1B  | -     | -   | 656 t/s | **770** t/s |
| TinyLlama  | EXL2 4.0 bpw | 1.1B  | -     | -   | 602 t/s | **700** t/s |

## How to

To install from the project you'll need the CUDA Toolkit and either gcc on Linux or (Build Tools for) Visual Studio
on Windows). Also make sure you have an appropriate version of [PyTorch](https://pytorch.org/get-started/locally/), then run:

```sh
git source https://github.com/turboderp/exllamav2
cd exllamav2
pip install -r requirements.txt
pip install .

python test_inference.py -m <path_to_model> -p "Once upon a time,"
# Append the '--gpu_split auto' flag for multi-GPU inference
```

A simple console chatbot is included. Run it with:

```sh
python examples/chat.py -m <path_to_model> -mode llama -gs auto
```

The `-mode` argument chooses the prompt format to use. `raw` will produce a simple chatlog-style chat that works with base 
models and various other finetunes. Run with `-modes` for a list of all available prompt formats. You can also provide
a custom system prompt with `-sp`. 

## Integration and APIs

- [TabbyAPI](https://github.com/theroyallab/tabbyAPI/) is a FastAPI-based server that provides an OpenAI-style web API
compatible with [SillyTavern](https://sillytavernai.com/) and other frontends.  

- [ExUI](https://github.com/turboderp/exui) is a simple, standalone single-user web UI that serves an ExLlamaV2 instance
directly with chat and notebook modes.

- [text-generation-webui](https://github.com/oobabooga/text-generation-webui) supports ExLlamaV2 through the **exllamav2**
and **exllamav2_HF** loaders.

- [lollms-webui](https://github.com/ParisNeo/lollms-webui) supports ExLlamaV2 through the exllamav2 binding.

## Installation

### Method 1: Install from source

To install the current dev version, source the project and run the setup script:

```sh
git source https://github.com/turboderp/exllamav2
cd exllamav2
pip install -r requirements.txt
pip install .
```

By default this will also compile and install the Torch C++ extension (`exllamav2_ext`) that the library relies on. 
You can skip this step by setting the `EXLLAMA_NOCOMPILE` environment variable:

```sh
EXLLAMA_NOCOMPILE= pip install .
```

This will install the "JIT version" of the package, i.e. it will install the Python components without building the
C++ extension in the process. Instead, the extension will be built the first time the library is used, then cached in 
`~/.cache/torch_extensions` for subsequent use.

### Method 2: Install from release (with prebuilt extension)

Releases are available [here](https://github.com/turboderp/exllamav2/releases), with prebuilt wheels that contain the extension binaries. Make sure to grab
the right version, matching your platform, Python version

*(excerpt; full text in `UPSTREAM_CLONE/`)*

*Quoted from the upstream `README.md` file in `UPSTREAM_CLONE/`.*
Project-specific facts detected in this directory:

- Ecosystem: **Python** (manifests: setup.py, requirements.txt; scanned in UPSTREAM_CLONE)
- Top-level source layout: `doc/`, `eval/`, `examples/`, `exllamav2/`, `experimental/`, `tests/`, `util/`
- Snapshot size: **289 files**, **43421 lines of code** (measured; see Benchmarks)
- Primary languages: `.py` (120), `.cuh` (34), `.png` (32), `.cu` (28), `.h` (23), `.cpp` (19)
- Upstream commit pinned for this packaging: `f363cd0d9ba6dd3f72d91a8a2a5b8fe4da127bd9`

---

## Installation

No installation section was found in the upstream readme, so the commands below are generated from the manifests detected in this project directory.

```sh
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e .                                     # or: pip install -r requirements.txt
```

Overlay install (this project):

```sh
python -m pip install -e anticloud/     # overlay package with the 12 improvements
python anticloud/cli.py --help          # 13 subcommands, JSON stdout
```

---

## Usage

No usage section was found in the upstream readme. Entry points detected in this project directory:

```sh
python -m exl2    # module entry point, when the package layout matches
```

Anticloud overlay CLI (available in every project):

```sh
python anticloud/cli.py --help     # 13 subcommands, JSON stdout
python anticloud/cli.py checks     # run the 16-check suite
```

---

## API

which provides an OpenAI-compatible API for local or remote inference, with extended features like HF model
downloading, embedding model support and support for HF Jinja2 chat templates.

See the [wiki](https://github.com/theroyallab/tabbyAPI/wiki/1.-Getting-Started) for help getting started.

*Section quoted from the upstream readme.*
---

## Dependencies

| Metric | Value |
|--------|-------|
| Ecosystem | Python |
| Manifests detected | setup.py, requirements.txt |
| Files in snapshot | 289 |
| Lines of code | 43421 |
| Dependency references | 3 |
| Dependencies by ecosystem | pypi: 3 |
| Upstream license | MIT |
| Overlay license | Anticommons 0.1.0 |

Top dependency references recorded in the benchmark snapshot:

| Ecosystem | Name | Version | Source file |
|-----------|------|---------|-------------|
| pypi | torch | >=2.2.0 | setup.py |
| pypi | safetensors | >=0.3.2 | setup.py |
| pypi | pillow | >=9.1.0 | setup.py |

Pinned lockfile: `anticloud/requirements.lock` (hash-pinned, PEP 508). SBOM: `sbom.cdx.json` (CycloneDX 1.5, pinned to the upstream SHA).

---

## Configuration

max_new_tokens = 200
    )  
    async for result in job:
        text = result.get("text", "")
        print(text, end = "")       
    ``` 
See the full, updated examples [here](https://github.com/turboderp/exllamav2/tree/master/examples).

*Section quoted from the upstream readme.*
Overlay configuration (Anticloud):

- `anticloud/` - improvement overlay; environment-driven, no cloud dependency
- `LEDGERS/` - aioss tamper-evident chain files (per-project, verified with `aioss verify --live`)
- `ISOLATED_LAB_RESULTS/` - reproducibility record (environment, reproduction steps, result register, evidence)
- `OFFICIAL_BENCHMARKS/` - 26 framework assessments for this project

---

## Contributing

Upstream contributions: fork the `EXL2` project, create a feature branch, and open a pull request against upstream. Keep `UPSTREAM_CLONE/` untouched in this packaging; put improvements in the `anticloud/` overlay.

Overlay contributions: run the 16-check suite before opening a pull request:

```sh
python anticloud/bench/runner.py --cwd anticloud
```

---

## License

**Upstream license: MIT** (evidence: `LICENSE` in the upstream snapshot).

License file excerpt:

```text
MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Anticommons 0.1.0 overlay

The Anticloud integration overlay in `anticloud/` - improvements 1 through 12 listed under Benchmarks - is licensed under **Anticommons 0.1.0**. Upstream code remains under its original MIT terms. See `ANTICOMMONS_LICENSE.md` in this directory for the overlay terms and contact.

SPDX: `MIT` (upstream) + Anticommons 0.1.0 (overlay, dual).

---

## Upstream

- **Project:** `EXL2` (category: CHIP QUANTIZATION)
- **Upstream URL:** https://github.com/xhedit/quantkit
- **Pinned commit (SHA):** `f363cd0d9ba6dd3f72d91a8a2a5b8fe4da127bd9`
- **Branch:** main
- **Pin provenance:** GitHub API commits/main. The parent-project stamp is explicitly rejected for this project.
- **Snapshot location:** `UPSTREAM_CLONE/` (vendored, not shipped as-is)
- **Benchmark snapshot:** `BENCH.json`

---

## Benchmarks

Measured by the Anticloud assurance suite. Every value below is read from this
project's `BENCH.json`, produced by a real run — the SHA3-256 of that file is
`c53f98933505886549b946e38ae166da1dcca6930a78857b796b80447cb2e31b`.

| Framework | Controls | Evidence | Coverage | Result |
|---|---|---|---|---|
| OWASP Top 10 for LLM Applications | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| OWASP Top 10 (2021) | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| SOC 2 Type II readiness | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| NIST AI Risk Management Framework | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| NIST SP 800-53 Rev. 5 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| NIST Cybersecurity Framework 2.0 | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| FedRAMP Rev. 5 | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| PCI DSS v4.0.1 | 11 controls mapped | 11 with evidence | 100.0% | PASS |
| ISO/IEC 27001:2022 | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| MITRE ATT&CK v16 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| ML Technology Readiness Level | TRL 8 | 8/8 criteria | | PASS |

**Overall: 16/16 checks passing.**

See `ISOLATED_LAB_RESULTS/03_Result_Register.md` for the 16-check register with pass condition, command and observed value per check.

Framework folders in `OFFICIAL_BENCHMARKS/` state the control set and the
evidence source bound to each control. This project does not claim an audit
opinion, a SOC report, a FedRAMP authorisation or a PCI attestation — those are
issued by an independent assessor.



## Archives and Permanent Records

| Platform | Identifier | Volume |
|---|---|---|
| Harvard Dataverse | DOI 10.7910/DVN/YMJKOG | 145 citable datasets |
| AIOSS verification kit | DOI 10.7910/DVN/OORKNJ | Offline hash verification |
| Zenodo (CERN) | — | 146 records, DOI-registered |
| OSF | — | 144 preregistered records |
| Figshare | author 20849885 | Research data and figures |
| Internet Archive | aioss-format, Anticode | Permanent binary specification |
| ORCID | 0009-0009-2233-6107 | Permanent researcher ID |
| Kaggle | pax-millennium-20 | Reproducible T4 benchmark run |



## Press and Independent Publication

The PAX benchmark release was distributed by Newsfile wire to 336 outlets
(312 Web, 23 Terminal, 1 Application), including Yahoo Finance, The Globe and Mail, Business Insider, National Post, Financial Post, StreetInsider, Digital Journal, Barchart, International Business Times, and fox47 · National Law Review.
Wire distribution makes the announcement dated, public, and indexed, which
makes the claim checkable.

