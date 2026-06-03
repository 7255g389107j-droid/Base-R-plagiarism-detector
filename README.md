# Base-R-plagiarism-detector
A mathematically rigorous text similarity and plagiarism detection engine implemented purely in Base R without external dependencies.

## Security & Data Privacy (Zero Data Leakage Risk)

Unlike modern LLM-based plagiarism checkers or cloud-based SaaS solutions, this engine offers **absolute data privacy and zero risk of information leakage**. 

- **100% Local Execution**: All computations are performed strictly in your local R environment. Your sensitive documents, proprietary research, or internal source codes are never sent to external servers or third-party APIs.
- **No Internet Required**: Because the algorithms rely exclusively on native Base R mathematical functions, the entire system can operate safely inside air-gapped networks or secure enterprise environments.
- **No Data Retention**: Texts are processed entirely in volatile memory (RAM) and immediately garbage-collected after evaluation. No temporary files or caches are written to the disk.
  
Developed by: **Tatsuki Itagaki**  
Copyright (c) 2026 Tatsuki Itagaki  
License: **MIT License**

## Overview

Many common string distance metrics (such as Jaro-Winkler, Cosine based on 1-grams, or standard Jaccard distance) violate basic geometric principles. They often break the **triangle inequality** or the **identity of indiscernibles** (e.g., treating two entirely different scrambled sentences as identical, yielding a distance of 0).

This repository provides two distinct, mathematically sound approaches to text copy verification that preserve metric space axioms or eliminate topological loopholes, utilizing only standard functions in Base R.

## Features

- **Zero External Dependencies**: Built entirely using native R functions (`adist`, `strsplit`, `matrix operations`).
- **Axiomatic Integrity**: Prevents mathematical "hallucinations" (such as anagrams or structural dilution) that occur in naive similarity packages.
- **Dual-Engine Design**: Optimized separately for **Whole-Text Paraphrasing** and **Partial Text Extraction**.

---

## 1. Word-Level Edit Distance (Whole-Text Plagiarism)

This engine tokenizes text into discrete word units and maps them to a strict, single-byte character space before applying graph-theoretic Levenshtein distance (`adist`). 

### Why it is mathematically sound:
It perfectly satisfies all metric space axioms (Identity of Indiscernibles, Symmetry, and Triangle Inequality). By treating words rather than letters as the atomic units, it remains computationally efficient while successfully punishing word-shuffling counter-measures.

```R
# Source code available in R/verify_copy_word_level.R
verify_copy_word_level <- function(text_a, text_b) {
  words_a <- unlist(strsplit(gsub("\\s+", " ", text_a), " "))
  words_b <- unlist(strsplit(gsub("\\s+", " ", text_b), " "))
  
  unique_words <- unique(c(words_a, words_b))
  
  map_to_char <- function(words, dict) {
    match_idx <- match(words, dict)
    rawToChar(as.raw(match_idx + 32))
  }
  
  encoded_a <- map_to_char(words_a, unique_words)
  encoded_b <- map_to_char(words_b, unique_words)
  
  edit_dist <- as.numeric(adist(encoded_a, encoded_b))
  max_len <- max(length(words_a), length(words_b))
  similarity <- 1 - (edit_dist / max_len)
  
  return(list(word_distance = edit_dist, similarity = similarity))
}
```

---

## 2. Character N-Gram Cosine Similarity (Partial Plagiarism)

This engine breaks sentences down into overlapping sequences of $N$ characters (default $N=3$) and applies linear algebraic cosine operations to evaluate structural density.

### Why it is mathematically sound:
Standard 1-gram bag-of-words models suffer from severe anagram vulnerabilities. By enforcing a strict local sequence constraint ($N \ge 3$), this implementation eliminates the mathematical loophole where different sentences generate identical vector representations. It isolates exact stolen segments from vastly larger documents without score dilution.

```R
# Source code available in R/verify_copy_cosine_ngram.R
verify_copy_cosine_ngram <- function(text_a, text_b, n = 3) {
  get_ngrams <- function(text, n) {
    chars <- strsplit(text, "")[]
    if(length(chars) < n) return(character(0))
    sapply(1:(length(chars) - n + 1), function(i) paste(chars[i:(i+n-1)], collapse=""))
  }
  
  grams_a <- get_ngrams(text_a, n)
  grams_b <- get_ngrams(text_b, n)
  
  all_grams <- unique(c(grams_a, grams_b))
  
  vec_a <- table(factor(grams_a, levels = all_grams))
  vec_b <- table(factor(grams_b, levels = all_grams))
  
  dot_product <- sum(vec_a * vec_b)
  norm_a <- sqrt(sum(vec_a^2))
  norm_b <- sqrt(sum(vec_b^2))
  
  if (norm_a == 0 || norm_b == 0) return(0)
  
  return(dot_product / (norm_a * norm_b))
}
```

---

## Usage & Validation

```R
# Case A: Testing Sentence-Level Copying with Word Shuffling
text1 <- "The quick brown fox jumps over the lazy dog"
text2 <- "lazy dog over the jumps fox brown quick The" 

verify_copy_word_level(text1, text2)\$similarity
# [1] ~0.22 -> Successfully identifies the scramble and drops the score.

# Case B: Testing Partial Plagiarism (Extracting one phrase out of a long paper)
long_doc <- "This is a highly rigorous academic paper containing unique insights. The quick brown fox jumps over the lazy dog."
stolen_snippet <- "The quick brown fox jumps over the lazy dog. Adding some random text here."

verify_copy_cosine_ngram(long_doc, stolen_snippet, n = 3)
# [1] ~0.55 -> Accurately flags density overlaps despite the mismatched document lengths.
```

## License

This project is open-source and available under the [MIT License](LICENSE).
