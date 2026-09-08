Important:
- Why `all-MiniLM-L6-v2`?
- How exactly do different embedding models change the representation?
- What does the embedding geometry mean?
- Why does your particular multimodal combination work?
- How should clustering quality actually be evaluated?
- How should an ingestion pipeline be architected for concurrency?
- What are the precise scaling characteristics of your retrieval system?

Absolutely. For a project like **The Watcher**, an interviewer will usually not ask 20 unrelated questions. They will start broad and then **keep drilling into whatever you claim you implemented**.

So the useful way to prepare is as **question chains**.

# 1. Project Overview → Architecture

### Q1. What is The Watcher?

**Possible follow-up:**

* What problem were you trying to solve?
* Why did you build it?
* What happens when a screenshot is captured?
* What are the major components?
* Why did you choose a central server?
* How do your devices communicate with the server?

### Q2. Explain the complete flow of the system.

**Cross-questions:**

* Where does OCR happen?
* Where does image embedding happen?
* Are OCR and image embedding sequential or parallel?
* What happens after OCR?
* Where do you store the results?
* Why SQLite?
* What exactly do you store for each image?

### Q3. Why did you use Syncthing?

**Follow-ups:**

* Why not `rsync`?
* Why not `fswatch`?
* Is Syncthing continuously watching or periodically syncing?
* What happens if the server is temporarily unavailable?
* What happens if the same image is synchronized twice?

---

# 2. OCR → Cleaning Pipeline

Your resume/project description gives an interviewer a lot of room to drill here.

### Q4. Why do you need OCR in this project?

**Follow-ups:**

* What OCR engine did you use?
* Why Tesseract?
* How accurate is it?
* What kind of errors does OCR typically introduce?
* What happens if the screenshot contains code instead of normal English?
* What happens if the screenshot contains no text?

### Q5. Walk me through your OCR cleaning pipeline.

You should be prepared to explain every stage:

```text
OCR
 ↓
Lowercase
 ↓
Emoji filtering
 ↓
Token validation
 ↓
Normalization
 ↓
Punctuation removal
 ↓
Special-character removal
 ↓
Spell correction
 ↓
Unique-word filtering
```

Then the interviewer can attack individual decisions.

### Q6. Why lowercase the text?

**Cross-question:**

> Are there situations where lowercasing is harmful?

---

### Q7. Why remove punctuation?

**Cross-question:**

> Would you remove punctuation from source code?

This is particularly important because your project deals with **screenshots**, and your notes explicitly mention structure such as code blocks.

---

### Q8. Why do you need spell correction?

**Cross-questions:**

* What spell-correction algorithm did you use?
* Why SymSpell?
* How does SymSpell work?
* What is the difference between SymSpell and BERT-based correction?
* What happens to technical terms?
* What happens to abbreviations?
* How do you prevent valid words from being "corrected"?

Your notes specifically identify this as a potential problem, with examples such as technical abbreviations being incorrectly modified. 

---

### Q9. How did you handle emojis?

**Follow-ups:**

* Why remove them?
* Why not use `emoji.demojize()`?
* What's the difference between removing an emoji and demojizing it?
* Can regex correctly handle complex emojis such as skin tones and joined emojis?
* Which approach did you finally choose?

---

# 3. Tokenization

### Q10. What is tokenization?

**Follow-ups:**

* Why is tokenization necessary?
* What's the difference between word and sentence tokenization?
* Why not simply use `split()`?
* What happens with punctuation?
* What happens with contractions?
* What happens with URLs?
* What happens with code?

### Q11. Why did you consider spaCy instead of simply using `split()`?

**Cross-question:**

> What exactly does a tokenizer give you that `split()` doesn't?

---

# 4. Stemming vs Lemmatization

This is an easy place for an interviewer to test whether you actually understand your project.

### Q12. Why would you use stemming?

**Follow-ups:**

* What is stemming?
* Give an example.
* What is a stem?
* Does a stem have to be a valid English word?
* Which stemmer did you use?
* Porter vs Snowball?

### Q13. How is lemmatization different from stemming?

**Cross-question:**

> If lemmatization produces better linguistic results, why not always use lemmatization?

Your notes specifically mention the trade-off that WordNet-based lemmatization requires dictionary lookup and is slower than stemming. 

---

# 5. Semantic Filtering

### Q14. What do you mean by semantic filtering?

**Follow-ups:**

* Why isn't normal text cleaning enough?
* How would embeddings help?
* What would you consider "useful" OCR text?
* What would you consider noise?
* Could keyword extraction solve this?
* What is the difference between keyword-based and embedding-based filtering?

---

# 6. Text Embeddings

This is likely one of the **most important chains**.

### Q15. Why generate text embeddings from OCR text?

**Follow-ups:**

* What exactly is an embedding?
* Why can't you just store the text?
* What model did you use?
* What dimensionality does your embedding have?
* What does each dimension represent?
* Why do semantically similar sentences have similar vectors?

### Q16. How would you compare two text embeddings?

**Follow-ups:**

* Cosine similarity or Euclidean distance?
* Why cosine similarity?
* What range does cosine similarity have?
* What happens if two vectors point in the same direction?
* What happens if they're orthogonal?
* Why does cosine similarity make sense for embeddings?

---

# 7. Image Embeddings / CLIP

### Q17. Why generate image embeddings if you already have OCR text?

This is a **very likely interviewer question**.

**Cross-questions:**

* What information does OCR lose?
* Give an example where two images have similar text but are visually different.
* What model did you use?
* Why CLIP?
* What is CLIP?
* How does CLIP generate an image embedding?
* Why 512 dimensions?
* Could you use another embedding model?

Your documentation specifies a **512-dimensional image embedding**, using OpenAI CLIP or another embedding model. 

---

# 8. SQLite Design

### Q18. Why did you choose SQLite?

**Follow-ups:**

* Why not PostgreSQL?
* Why not MongoDB?
* How much data can SQLite handle?
* What happens when you have lakhs of images?
* Why store embeddings as BLOBs?
* How do you retrieve them?
* What indexes do you have?
* Why store the cluster centroid?
* What is `cluster_id` used for?

Your documented schema contains image ID, name, path, embedding, OCR text, cluster ID and cluster centroid. 

---

# 9. Full-Text Search

### Q19. How does your text search work?

**Follow-ups:**

* Why SQLite FTS?
* What is an inverted index?
* How does FTS differ from `LIKE '%keyword%'`?
* How does multi-word search work?
* How do AND/OR queries work?
* How does ranking work?
* What is prefix search?
* What happens if the user searches for a synonym rather than the exact word?

This naturally leads to:

### Q20. What is the limitation of your SQLite FTS search?

Expected direction:

```text
"machine learning"
```

and

```text
"ML"
```

may not necessarily be treated as semantically equivalent.

Which leads directly to:

### Q21. How would you solve that?

→ **Semantic search.**

---

# 10. Hybrid Search

### Q22. Why did you build a hybrid search instead of using only semantic search?

**Follow-ups:**

* What does SQLite FTS give you?
* What does FAISS give you?
* Why combine them?
* Which search happens first?
* How do you combine the results?
* How would you rank the final results?
* What happens if FTS finds a highly relevant result that FAISS doesn't?
* What happens if FAISS finds something that FTS completely misses?

Your notes describe the intended approach as using SQLite FTS to obtain favorable text-search results and FAISS for additional similarity-based results. 

---

# 11. FAISS

### Q23. Why FAISS?

**Cross-questions:**

* Why not calculate cosine similarity against every image?
* What is the complexity of brute-force similarity search?
* Why does it become problematic at scale?
* What is FAISS actually doing?
* What is an index?
* What are approximate nearest-neighbor searches?
* What is the trade-off between speed and accuracy?
* How would you update the FAISS index when a new image arrives?

---

# 12. Clustering

### Q24. Why do you need clustering?

**Follow-ups:**

* What does a cluster represent in The Watcher?
* What data are you clustering?
* Image embeddings or text embeddings?
* Why?
* How do you determine whether two screenshots belong to the same cluster?

Your latest notes specifically say that you were moving toward **text-embedding-based clustering**. 

---

### Q25. Why HDBSCAN?

This can become a serious ML cross-question.

**Follow-ups:**

* How is HDBSCAN different from K-Means?
* How is it different from DBSCAN?
* Do you need to specify the number of clusters?
* What does `min_cluster_size` mean?
* What does `min_samples` mean?
* What does `cluster_selection_method` do?
* What does `cluster_selection_epsilon` do?
* How does HDBSCAN handle noise?
* Why is noise handling useful for your screenshots?

Your notes list HDBSCAN with cosine distance and these parameters. 

---

# 13. Incremental Clustering

This is another **very good interviewer chain** because your documentation explicitly discusses it.

### Q26. What happens when a new image arrives after clustering has already been performed?

Your documented idea is:

```text
New image
 ↓
Embedding
 ↓
Find nearest images
 ↓
KNN voting
 ↓
Determine best-fit cluster
```



Then expect:

### Q27. Why not simply run HDBSCAN again?

### Q28. Why use KNN for assigning a new image?

### Q29. What happens if the new image doesn't actually belong to any existing cluster?

### Q30. How do you know when your clusters have become stale?

Your notes propose periodically reclustering because clusters can become unstable over time, approximately every **500–1000 images**. 

---

# 14. The Really Important Cross-Question Chain

An interviewer could easily go:

> **You said you use embeddings. Why?**

↓

> What is an embedding?

↓

> How is an embedding generated?

↓

> What model are you using?

↓

> What is the dimensionality?

↓

> How do you compare two embeddings?

↓

> Why cosine similarity?

↓

> Why not Euclidean distance?

↓

> If you compare every image against every other image, what's the complexity?

↓

> How do you solve that?

↓

> FAISS.

↓

> What is FAISS?

↓

> Is FAISS exact or approximate?

↓

> What happens when a new image arrives?

↓

> How do you assign it to a cluster?

↓

> Why KNN?

↓

> When do you recluster?

**You should be able to answer this entire chain without getting stuck.**

---

# 15. OCR-Specific Cross-Question Chain

Another likely chain:

> **Why is OCR necessary?**

↓

> Why Tesseract?

↓

> What kind of errors does OCR produce?

↓

> How do you clean those errors?

↓

> Why lowercase?

↓

> Why remove punctuation?

↓

> Why remove stopwords?

↓

> What are stopwords?

↓

> Why stemming/lemmatization?

↓

> Why spell correction?

↓

> Doesn't spell correction sometimes change valid technical words?

↓

> How did you handle that?

↓

> What happens if OCR extracts code?

↓

> Should you apply the same preprocessing to code and normal English?

This is a **much more realistic interview preparation format** than memorizing isolated definitions.

---

# 16. System Design Cross-Questions

Finally, expect the interviewer to leave the ML details and attack the actual system.

### Q31. What happens if 10,000 screenshots arrive simultaneously?

* Can your server process them?
* Is your pipeline synchronous?
* Where is parallelism used?
* What is the bottleneck?
* CPU or GPU?
* Would you introduce a queue?
* How would you scale it?

### Q32. What happens if OCR takes 2 seconds per image?

### Q33. What happens if embedding generation takes 1 second?

### Q34. Which parts can run concurrently?

### Q35. What happens if your server crashes halfway through processing an image?

### Q36. How do you avoid processing the same screenshot twice?

### Q37. What happens when SQLite becomes the bottleneck?

### Q38. How would you redesign this if you had **10 million images** instead of lakhs?

---

## What I would prioritize for your interview

If this project is on your resume, I would prepare these **10 areas extremely well**:

1. **Complete architecture**
2. **OCR + Tesseract**
3. **Your cleaning pipeline and why each step exists**
4. **Image embeddings + CLIP**
5. **Text embeddings**
6. **Cosine similarity**
7. **FAISS**
8. **SQLite FTS + hybrid search**
9. **HDBSCAN + KNN incremental clustering**
10. **Why you made each design decision**

The key is not merely knowing *what* each technology is. The interviewer will repeatedly ask:

> **"Why did you choose this?" → "Why not X?" → "What happens if Y?"**

That's where you need to be strongest.
