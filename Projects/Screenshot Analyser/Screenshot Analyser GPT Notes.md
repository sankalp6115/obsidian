# The Watcher — Project Documentation

## 1. Overall System

The project, **The Watcher**, is intended to collect screenshots/images from multiple devices and send them to a **central processing server**.

```
Mac ─────┐
         │
Phone ───┴──→ Central Server
                  │
                  ├── Image Embedding
                  │
                  └── OCR
                       ↓
                     Cleaning
                       ↓
                    NLP/Text
                       ↓
                    Storage
```

The notes mention using **Syncthing** for syncing screenshots from devices to the central server.

The alternatives considered were:

- Syncthing — marked as working/preferred
- `fswatch` — marked as not suitable
- `rsync` — mentioned as another possibility

The synchronization approach is intended to support either **constant watching or periodic synchronization**.

---

# 2. Image Processing

For every incoming image, two major operations happen in parallel:

```
                 Image
                /     \
              OCR    Image Embedding
               │
             Clean
               │
        Text Embedding
```

The notes explicitly describe this as a **parallelized pipeline**.

### Image embedding

- 512-dimensional embedding
- OpenAI CLIP or another embedding model
- Used for image similarity/search

### OCR

**Tesseract** is being considered/used to extract text from images.

The extracted OCR text is then passed through the text-cleaning/NLP pipeline.

---

# 3. SQLite Storage

The initial design stores image-related information in **SQLite**.

The proposed fields are:

```
img_id
    UUID / string

name
    string

path
    string

embedding
    BLOB

OCR text
    array of strings

cluster_id
    int

cluster_centroid
    BLOB
```

The notes also mention that the system should work well for **lakhs of samples**.

---

# 4. Search System

The project has two search mechanisms.

## 4.1 Full-Text Search

SQLite's built-in **FTS (Full-Text Search)** is intended to act as the initial text search engine.

The notes mention support for:

- Searching by word
- Multi-word search
- AND/OR logic
- Ranking
- Prefix search

Conceptually:

```
Query
  ↓
SQLite FTS
  ↓
Relevant OCR-text results
```

---

## 4.2 Semantic Search

Image embeddings can be used for semantic/image similarity search.

The notes specify **cosine similarity** rather than Euclidean distance.

Cosine similarity:

- compares the direction of embedding vectors
- gives a value in the range `[-1, 1]`
- allows semantically/similarly related images to be identified even when their raw characteristics differ

---

## 4.3 Hybrid Search

The proposed hybrid search combines both approaches:

```
Query
  │
  ├──→ SQLite FTS
  │       ↓
  │   Favorable results
  │
  └──→ FAISS
          ↓
      Similar results
```

The idea is to first use **SQLite FTS** to obtain favorable text-search results and then use **FAISS** to retrieve more similar results.

The notes describe FAISS as being **fast at scale**.

---

# 5. Clustering

The initial/proposed clustering approach is:

```
SQLite
  ↓
Fetch image IDs + embeddings
  ↓
Clustering
  ↓
Generate clusters
  ↓
Store cluster ID + centroid in DB
```

Algorithms mentioned:

- K-Means
- DBSCAN
- HDBSCAN

---

## Incremental clustering for new images

For a new image:

```
New Image
   │
   ├──→ OCR
   │
   └──→ Image Embedding
             │
             ↓
       Find similar images
             │
       ┌─────┴─────┐
       ↓           ↓
 Cosine Similarity  FAISS
       └─────┬─────┘
             ↓
       K nearest images
             ↓
            KNN
             ↓
      Best-fit cluster
```

The notes describe two possible approaches for finding similar images:

1. Direct cosine similarity
2. FAISS

The **K nearest images** can then be used with KNN to vote for the best-fitting cluster.

The initial approach suggested in the notes is:

> **Cosine similarity as a naive starting approach; FAISS for scale.**

### Cluster maintenance

The notes identify a problem:

> Clusters become unstable over time.

The proposed solution is to **recluster periodically**, approximately every **500–1000 images**.

---

# 6. OCR Text Processing / NLP Pipeline

The notes separate OCR processing into multiple stages.

The overall concept is:

```
OCR text
   ↓
Text preprocessing
   ↓
Tokenization
   ↓
Stopword handling
   ↓
Lemmatization / Stemming
   ↓
NER
   ↓
Semantic filtering
   ↓
Noise classification
   ↓
Structure extraction
   ↓
Post-processing
```

The notes specifically mention **CampusX** as the source/reference for an NLP preprocessing pipeline.

---

# 7. OCR Cleaning Pipeline

The currently planned cleaning stages are:

1. Spell correction
2. Special-character removal
3. Word normalization / repetition removal
4. Emoji filtering
5. Valid-token filtering
6. Lowercasing
7. Punctuation removal

The notes show the sequence as roughly:

```
isInstance
   ↓
lowercase
   ↓
emoji_filter
   ↓
is_valid_token
   ↓
normalize
   ↓
punctuation_remove
   ↓
spell_chr_remove
   ↓
correct_spelling
   ↓
filter_unique_words
```

The exact ordering has some handwritten arrows/corrections, so this appears to be an evolving implementation rather than a finalized specification.

The notes explicitly say:

> Beginning cleaning — keeping limited to phase ① & ②.

---

# 8. OCR Cleaning Details

## 8.1 Spell correction

Libraries/models considered:

- SymSpell
- BERT
- NLTK
- `pyspellchecker`
- TextBlob

The notes distinguish:

- **SymSpell** — fast spell correction
- **BERT** — context-aware correction

There is also a warning about blindly correcting technical/domain-specific words.

Examples noted:

```
lib → lab
dev → de
cpu → colo
```

The concern is that important words can be changed and **meaning can be lost**.

The notes therefore mention **hardcoding certain words so they aren't modified or removed**.

---

## 8.2 Special-character removal

The goal is to remove unnecessary symbols/noise from OCR text.

---

## 8.3 Word normalization

The notes mention removing repetitions.

Example:

```
heyyy → hey
```

---

## 8.4 Emoji handling

Two approaches are documented.

### Regex approach

A Unicode-range regex is proposed for removing emoji characters.

### `emoji` package

The alternative is:

```
import emoji

emoji.demojize(text)
```

This converts an emoji into its textual representation.

For example, conceptually:

```
Python is 🐍
```

becomes:

```
Python is :snake:
```

---

# 9. Tokenization

Tokenization converts a text document into smaller units such as:

```
word
character
sentence
```

The notes emphasize that **bad tokenization can confuse the model** and that good tokenization is important.

Tokenization concepts noted:

### Prefix

Character at the beginning.

### Suffix

Character at the end.

### Infix

Character within the word.

### Exception rules

Special rules can prevent certain strings from being split incorrectly when punctuation rules are applied.

Methods/libraries considered:

- `str.split()`
- Regex
- NLTK
- spaCy

The notes mark **spaCy as producing better results**.

---

# 10. Stemming

Stemming reduces inflected words to a common stem.

Example:

```
walk
walked
walking
```

The notes describe the goal as mapping related words to the same stem, even if the resulting stem isn't necessarily a valid English word.

Libraries/algorithms mentioned:

- NLTK
- Porter Stemmer
- Snowball Stemmer

---

# 11. Lemmatization

Lemmatization reduces inflected words to their proper root/base form.

The notes define the root word as the **lemma**.

Example:

```
running → run
```

The notes mention:

- WordNet
- Parts of speech
- spaCy

WordNet is described as a **lexical dictionary** and the notes point out that dictionary lookup makes lemmatization slower than stemming.

---

# 12. Named Entity Recognition

The notes propose an NER stage to ensure important entities aren't unnecessarily removed during preprocessing.

Examples of entities:

```
names
brands
etc.
```

Libraries/models mentioned:

- spaCy NER
- BERT NER

---

# 13. Semantic Filtering

Two approaches are listed:

### A. Embedding-based

Use embeddings to determine whether extracted text is semantically useful.

### B. Keyword extraction

Extract relevant keywords from the OCR text.

---

# 14. Noise Classification

The idea is to train a classifier to determine whether a token/sentence is useful or useless.

```
Input: Token / Sentence
             ↓
         Classifier
          /      \
      Useful    Useless
```

Models mentioned:

- Logistic Regression
- BERT

---

# 15. Structure Extraction

The OCR text can contain different structural components.

The notes specifically mention:

- **Code blocks**
- **Paragraphs**

This stage would identify/extract those structures.

---

# 16. Post-Processing

The final NLP post-processing stage is intended to produce useful outputs for downstream applications.

The notes mention:

- Search engine
- Summarizer
- Knowledge extraction
- Classification

---

# 17. Current Implementation / Progress

A dated entry on **24/6/26** records the following:

```
Image metadata + clean OCR
        ↓
      SQLite
```

The notes state that this was successfully tested for **text-based search**.

The next step documented is:

> **Now using text embeddings for clustering**

---

# 18. Current Clustering Experiment

The latest page documents **HDBSCAN** with cosine distance.

The distance matrix is noted as:

```
metrics.pairwise_distances
```

with cosine distance.

Parameters being considered:

```
min_samples
min_cluster_size
cluster_selection_method
cluster_selection_epsilon
```

---

# 19. Assignment / Experiment

One section describes an assignment:

```
Create PS
   ↓
Text preprocessing
   ↓
Multi-class classification
   ↓
Classes such as:
movie
song
...
```

The exact intended classifier/model isn't specified further in the notes.

---

# Consolidated Architecture

Putting the pages together, the project's intended architecture looks like this:

```
                  ┌───────────────┐
                  │ Mac / Phone   │
                  └───────┬───────┘
                          │
                     Screenshot
                          │
                          ↓
                 ┌─────────────────┐
                 │ Central Server  │
                 └────────┬────────┘
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
             OCR              Image Embedding
              │                       │
              ↓                       │
        Text Cleaning                 │
              │                       │
              ↓                       │
        Text Embedding                │
              │                       │
              └───────────┬───────────┘
                          ↓
                     SQLite Storage
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
         Text Search              Clustering
          SQLite FTS          HDBSCAN / KNN / FAISS
              │                       │
              └───────────┬───────────┘
                          ↓
                    Search / Analysis
```

And the **current state** documented in the notebook appears to be:

**Screenshot synchronization → OCR → cleaning → metadata + cleaned OCR stored in SQLite → text-based search tested → moving toward text-embedding-based clustering.**