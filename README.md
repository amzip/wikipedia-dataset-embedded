# 🗂️ Wikipedia Dataset Embedded  
> An Embedded segment of the *Wikipedia 2023* dataset (Hugging Face), enriched with high‑quality semantic embeddings generated using **nomic-embed-text-v2-moe**.

![Dataset](https://img.shields.io/badge/dataset-Wikipedia%202023-blue.svg)
![Embeddings](https://img.shields.io/badge/embeddings-nomic--v2--moe-purple.svg)
![Size](https://img.shields.io/badge/elements-480k-green.svg)
![License](https://img.shields.io/badge/license-CC%20BY--SA%203.0-yellow.svg)
![Vectors](https://img.shields.io/badge/vector%20dim-768-orange.svg)

---

## 🧠 Overview

This repository contains an **embedded subset** of the *Wikipedia 2023* dataset published on Hugging Face.  
It is designed for:

- 🔍 **Semantic search**  
- 🧩 **RAG pipelines**  
- 🧪 **Embedding model benchmarking**  
- 📊 **Large‑scale semantic analysis**

## 🌐 Language

The dataset is entirely in **English**.  
All extracted text, chunked segments, and generated embeddings originate from the English portion of the Wikipedia 2023 dataset.

---

## 📦 Dataset Composition

### 📁 Source  
The original dataset is the *Wikipedia 2023* parquet dataset available on Hugging Face. (https://huggingface.co/datasets/wikimedia/wikipedia)

### 🔢 Sampling Strategy  
From each original `.parquet` file, the **first 12,000 entries** were selected.

Total embedded items: 12,000 entries × 40 parquet files = 480,000 embedded elements, ≈ 1,000,000 embedding vectors.

### ✂️ Text Chunking  
Each Wikipedia article was split into overlapping token chunks:

- **Chunk size:** 500 tokens  
- **Overlap:** 50 tokens  

This improves retrieval granularity and embedding consistency.

### 🧬 Embeddings  
Embeddings were generated using: nomic-embed-text-v2-moe


Model characteristics:

- 📏 **Vector dimension:** 768  
- ⚙️ Mixture‑of‑Experts routing  
- 🚀 Optimized for retrieval and semantic similarity  

### 🆕 Added Column: `embeddings`

Each parquet file includes a new column: 
**embeddings** : string

This column contains a serialized array of embedding vectors, one per text chunk.  
The structure is:
[
[embedding_vector_chunk_1],
[embedding_vector_chunk_2],
...
]

- Stored as a **string** for parquet compatibility  
- Each inner vector has **768 dimensions**  
- The number of chunks varies depending on the article length  

Example (conceptual):
embeddings = "[[0.12, -0.44, ...], [0.03, 0.88, ...], ...]"

## 🔧 Deserializing the `embeddings` Column

Each parquet row contains an `embeddings` column stored as a **string** representing an array of embedding vectors:


The goal is to deserialize this field into:

- **Python:** `list[list[float]]`  
- **.NET Core:** `List<float[]>`  

Below are working examples for both environments.

---

### 🐍 Python Example (PyArrow / Pandas)

```python
import json
import pandas as pd

# Load parquet file
df = pd.read_parquet("wikipedia_embedded.parquet")

# Read the string column
raw = df.loc[0, "embeddings"]  # string: "[[0.12, -0.44, ...], [...]]"

# Deserialize into Python list[list[float]]
embeddings = json.loads(raw)

print(type(embeddings))          # list
print(type(embeddings[0]))       # list
print(len(embeddings[0]))        # 768
```
### ⚙️ .NET Core Example (System.Text.Json)

```C#
using System.Text.Json;

// raw string from parquet row
string raw = row.embeddings;  
// raw = "[[0.12, -0.44, ...], [0.03, 0.88, ...], ...]"

// Deserialize into List<float[]>
var embeddings = JsonSerializer.Deserialize<List<float[]>>(raw);

Console.WriteLine(embeddings.Count);      // number of chunks
Console.WriteLine(embeddings[0].Length);  // 768
```
