# Base-R-plagiarism-detector
A mathematically rigorous text similarity and plagiarism detection engine implemented purely in Base R without external dependencies.

## 
Developed by: **Tatsuki Itagaki**  
Copyright (c) 2026 Tatsuki Itagaki  
License: **MIT License**

## Security & Data Privacy (Zero Data Leakage Risk)

Unlike modern LLM-based plagiarism checkers or cloud-based SaaS solutions, this engine offers **absolute data privacy and zero risk of information leakage**. 

- **100% Local Execution**: All computations are performed strictly in your local R environment. Your sensitive documents, proprietary research, or internal source codes are never sent to external servers or third-party APIs.
- **No Internet Required**: Because the algorithms rely exclusively on native Base R mathematical functions, the entire system can operate safely inside air-gapped networks or secure enterprise environments.
- **No Data Retention**: Texts are processed entirely in volatile memory (RAM) and immediately garbage-collected after evaluation. No temporary files or caches are written to the disk.

## Overview

Many common string distance metrics (such as Jaro-Winkler, Cosine based on 1-grams, or standard Jaccard distance) violate basic geometric principles. They often break the **triangle inequality** or the **identity of indiscernibles** (e.g., treating two entirely different scrambled sentences as identical, yielding a distance of 0).

This repository provides two distinct, mathematically sound approaches to text copy verification that preserve metric space axioms or eliminate topological loopholes, utilizing only standard functions in Base R.

## Features

- **Zero External Dependencies**: Built entirely using native R functions (`adist`, `strsplit`, `intToUtf8`, `matrix operations`).
- **Axiomatic Integrity**: Prevents mathematical "hallucinations" (such as anagrams or structural dilution) that occur in naive similarity packages.
- **Dual-Engine Design**: Optimized separately for **Whole-Text Paraphrasing** and **Partial Text Extraction**.

The screening logic optimizes auditing by categorizing text into three layers: automated rejection of direct matches (Layer 1; Whole-Text Paraphrasing), review of standard domain jargon (Layer 2; Partial Text Extraction), and intense human audit for high-level paraphrasing deception (Layer 3; Passed both). By reducing cognitive load, this approach directs human expertise toward identifying sophisticated, non-literal plagiarism.

---

## 1. Word-Level Edit Distance (Whole-Text Plagiarism)

This engine tokenizes text into discrete word units and maps them to unique UTF-8 Unicode characters via the Private Use Area (`U+E000` onwards) before applying graph-theoretic Levenshtein distance (`adist`). 

### Why it is mathematically sound:
It perfectly satisfies all metric space axioms (Identity of Indiscernibles, Symmetry, and Triangle Inequality). By mapping word IDs into discrete Unicode points, it allows vocabulary sizes up to **64,000+ unique words** without crashing on memory boundaries or raw `nul` character constraints. It successfully identifies and punishes word-shuffling counter-measures.

```R
# Source code available in R/verify_copy_word_level.R
verify_copy_word_level  Successfully penalizes scrambled sentences.

# Execute Engine 2 (Validating Domain / Shared Structural Density)
result_2 <- verify_copy_cosine_ngram(text1, text2, n = 3)
print(result_2)
# Output: 0.5816158 -> Catches the heavy 3-gram sequence density overlap.
```

## License

This project is open-source and available under the [MIT License](LICENSE).
