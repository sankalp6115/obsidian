We save a lot of images as resources and screenshots from various places like instagram, youtube, reddit etc, but when it comes to referring them in future, we cant search for the screenshot, it is basically lost in the large sea of screenshots, and it would lie there indefinitely until removed permanently. 
To solve this i have made a image analysis and clustering system that clusters similar screenshots together and shows them in a organized UI, where you can search by semantic meaning, exact word match, and visuals. 

Here you can see all images in a unified interface and be able to easily find out the screenshot that was lost, just by knowing what that screenshot could have contained, the exact or surrounding text, how it looked etc which is much easier to figure out.

The system:
I have devised a client-server architecture where a central server takes in all the screenshots from all the connected clients, accumulates in an organized manner, runs a complete python pipeline on it to get required info from the images and retains them in a SQLite DB.

Now this retained data is fetched from server, and shown on a UI, this data shown is clustered using semantic meaning matching and visual matching.

### Stage1 - Data Accumulation
This stage involves uploading screenshots from client devices to central server.
This can be done via several techniques:

syncthing: Overall best syncing service between two devices, open-source and free to use, this service can be setup to periodically sync , sync-on-demand and sync in regular intervals. Gives inbuilt features of syncing. direct machine-to-machine connection via public discovery server, can work even when devices are not on same network.

rsync: Linux command that implements memory system in syncing , i.e. it remembers what it synced/copied last time so it does not repeat it again and again.

scp: Linux command that just copies files/folders from one folder to other

rclone: application that is very useful for syncing with cloud services. compatibility with 70+ cloud services

-> fswatch: this command watches certain folder , looks for changes like file addition, removal, rename, etc and performs specified action.

-> cronjob: cronjobs are background tasks that are programmed to take place periodically on their own, they can be configured to run, weekly,monthly,on a day, hourly and minutely.
Server can setup a simple cronjob to ssh into client machines , and copy the screenshot folders from them into server.

These are most viable ways of data accumulation for our system
The most viable in these is syncthing as it gives most features well-built, with robustness, no LAN needed.

The next viable solution will be usage of rclone, (which internally also uses rsync)
based on condition , it can also be the most viable solution as it offers cloud storage, so clients can  run a background service or an app which periodically syncs directly to the cloud storage , so it is write only, (wont be given read/sync permission), and the server can periodically read/sync  (read-only permission, no writing allowed) from cloud service, this way i could make a single source of truth, the cloud service.

the syncing will periodically run to sync the latest screenshots, to the server,
during syncing process , fswatch also runs and continuosly watches for changes, and when a new image is found, it will put the image into processing queue, wait for some time for more images to come to make a complete batch process, and when a complete batch is made, it will be sent to the py pipeline to process.

The outputs of the pipeline are saved in the sqlitedb.
### Stage 2 - OCR Pipeline
The OCR process uses an OCR Engine like Tesseract, PaddleOCR, EasyOCR, docTR, Datalab Marker.
Right now system is only being tested on tesseract , it already gives pretty good results.

This complete process takes OCR text and cleans it using simple and NLP techniques.
Below i am dexribing whole process:

- validation - validatiing what is received from ocr is valid string
```python
def validation(sentence):
    return sentence if isinstance(sentence, str) else ""
```

- lowercase - lowercasing whole text to normalize
- emoji_filter - 
	two approaches:
	- emoji library - more extensive control over emojis
	- regex - unicode matching of emojis, might not work on complex combinations of emojis unicode.
```python
EMOJI_PATTERN = regex.compile(r'\p{Emoji}(?<!\d)')

import emoji
clean_text = emoji.replace_emoji(text, replace="")
```
Demojize can help preserve  the meaning of emoji in text while also removing emoji
```python
import emoji
text = "I love this! ❤️🔥"
print(emoji.demojize(text))
# I love this! :red_heart::fire:
```

- remove punctuation - 
	normalization of words , eg heeyyyy -> hey
	using simple dsa algo
	
	punctuation removed using python translate methods
	```python
	PUNCT_DELETION_CHARS = "".join(c for c in string.punctuation if c not in ALLOWED_PUNCT)
	PUNCT_TRANSLATION_TABLE = str.maketrans("", "", PUNCT_DELETION_CHARS)
	def remove_punctuation(sentence):
	    if not isinstance(sentence, str):
	        return ""
	    return sentence.translate(PUNCT_TRANSLATION_TABLE)
	```

- filter valid tokens- 
	some words are required to be not replaced like 30kbps, 40kg and soon

- spelling correction- 
	spelling correction using python symspellpy, it uses a dictionary of word and weight that allow the wrong spelling to be corrected via a distance algorithm.

- special character remover-
	removes words with too many special characters, 
	occurs mainly due to bad extraction by OCR
	eg. &^%193hel

- unique words- 
	extracts unique words from the sentence


Proposed pipeline sequence: 
String validation -> lowercase -> emoji_filter -> normalize -> is_valid_token -> correct_spelling -> special_character_removal

The pipeline is still having confusing steps, if time, repair it.
Some steps must be combinable.

Then the steps are defined in an array, which runs the whole process in a sequential manner to clean the whole text .

### Stage 3
text embedding pipeline
this pipeline simply used SentenceTransformer from huggingface and returns a 384 dimension embedding of the text

image embedding pipeline
this pipeline uses open_clip -> all-MiniLM-L6-v2 model to return a 512 dimn embedding for the image.

these both models run in another independent fastapi server, exposing endpoints where hitting a post request returns the response as the embedding,  avoiding the loading of model for every image processing

### Stage 4
batch ingestion
collects output of pipeline and inserts a batch into DB instead of reducing num of operations in DB, increasing efficiency, by avoiding opening and closing DB connections repetitively.

Till here, data accumulation, preprocessing, processing is complete.

Furthur we do ML techniques to cluster and categorize the images.

### Stage 5 - Storage & Database Schema
The processed output, metadata, text embeddings, and image embeddings are retained in a central SQLite database (`watcher.db`).

- Main Table (`images`):
```sql
CREATE TABLE IF NOT EXISTS images (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    extension TEXT,
    size_bytes INTEGER,
    modified_time TIMESTAMP,
    path TEXT NOT NULL UNIQUE,
    cleaned_text TEXT,
    text_embedding BLOB,
    image_embedding BLOB,
    cluster_id INTEGER,
    text_embedding_model TEXT,
    image_embedding_model TEXT
)
```
  - `id`: unique SHA-256 hash generated from image file binary content.
  - `text_embedding`: 384-dimension vector stored as binary float32 BLOB (`all-MiniLM-L6-v2`).
  - `image_embedding`: 512-dimension vector stored as binary float32 BLOB (`openclip-vit-b-32-laion2b_s34b_b79k`).

- Full-Text Search Virtual Table (`images_fts`):
  SQLite has built-in FTS5 engine (`images_fts`) indexing `cleaned_text` with unindexed `id`.
```sql
CREATE VIRTUAL TABLE IF NOT EXISTS images_fts USING fts5(
    id UNINDEXED,
    cleaned_text
)
```

- Automatic Synchronization Triggers:
  Triggers (`images_ai`, `images_ad`, `images_au`) keep the `images_fts` index updated automatically whenever records are inserted, deleted, or updated in the main `images` table.


### Stage 6 - Search Systems (FTS5, Vector & Hybrid RRF)

- Full-Text Search (FTS5):
  Fast exact word-level search using SQLite FTS5 engine scored by BM25 ranking (`bm25(images_fts)`).
  Supports multi-word queries, AND/OR logic, prefix searching, and ranking.

- Vector / Semantic Search:
  Query text is sent to `/embed-text` endpoint to get a 384-dim vector.
  Cosine similarity is calculated between query embedding and stored `text_embedding` BLOBs:
```python
def calculate_cosine_similarity(a, b):
    dot_product = np.dot(a, b)
    normA = np.linalg.norm(a)
    normB = np.linalg.norm(b)
    if normA == 0 or normB == 0:
        return 0.0
    return dot_product / (normA * normB)
```

- Hybrid Search with Reciprocal Rank Fusion (RRF):
  Combines keyword match results from FTS search and semantic results from Vector search.
  Reranks using Reciprocal Rank Fusion (RRF):
  $$\text{RRF Score} = \frac{1}{k + \text{rank}}$$
  (using default parameter $k = 60$).
```python
def reciprocal_rank_fusion(fts_results, vector_results, k=60):
    rrf_scores = {}
    for rank, item in enumerate(fts_results):
        img_id = item["id"]
        rrf_scores[img_id] = rrf_scores.get(img_id, 0.0) + 1.0 / (k + rank + 1)
    for rank, item in enumerate(vector_results):
        img_id = item["id"]
        rrf_scores[img_id] = rrf_scores.get(img_id, 0.0) + 1.0 / (k + rank + 1)
    return sorted(rrf_scores.items(), key=lambda x: x[1], reverse=True)
```


### Stage 7 - Clustering & Categorization Pipeline

1. Feature Fusion & Combined Distance Matrix:
   - Reads image and text embeddings from SQLite database.
   - Computes pairwise cosine distance matrix for image embeddings (`dist_img`) and text embeddings (`dist_text`).
   - Combines them using weighted distance matrix:
     ```python
     distance_matrix = (IMG_WEIGHT * dist_img) + (TEXT_WEIGHT * dist_text)
     ```
     where `IMG_WEIGHT = 0.50` and `TEXT_WEIGHT = 0.50`.

2. Density-Based Clustering (HDBSCAN):
   - Uses `sklearn.cluster.HDBSCAN` with parameters:
     - `min_samples = 2`
     - `min_cluster_size = 4`
     - `metric = 'precomputed'`
     - `cluster_selection_method = 'eom'`
     - `cluster_selection_epsilon = 0.25`
   - Groups images into dense clusters. Unassigned outlier images receive label `-1`.

3. Noise Reassignment Pipeline (2nd Stage Centroid Re-alignment):
   - Calculates cluster centroids (`centroids_img`, `centroids_text`) by taking mean embeddings of points in each valid cluster.
   - For all noise images (`label == -1`), calculates combined weighted cosine distance to all cluster centroids.
   - Reassigns noise image to nearest cluster if minimum distance $\le \text{DISTANCE\_THRESHOLD}$ (where `DISTANCE_THRESHOLD = 0.42`).
   - Updates assigned `cluster_id` back to SQLite DB via `update_cluster_ids`.

4. Threshold Decider & Distribution Analysis (`threshold_decider.py`):
   - Computes cosine distance of each image in a cluster to its cluster centroid.
   - Generates Histogram and Kernel Density Estimation (KDE) plot with vertical mean line saved as `distances_histogram.png` to analyze distance distribution and determine threshold.

5. Dimensionality Reduction & Review Utilities:
   - `visualizer.py`: uses `umap.UMAP(n_components=2, random_state=42)` for 2D embedding visualization.
   - `cluster_review.py`: copies cluster sample images to a local folder (`cluster_review/`) for manual quality auditing.


### Stage 8 - Server Infrastructure & Frontend UI

- Independent Embedding Server (`embedding_server.py` on Port 8000):
  - FastAPI server running embedding models continuously in memory (auto device selection for CUDA / MPS / CPU).
  - `/embed-text`: `SentenceTransformer("all-MiniLM-L6-v2")` returning 384-dim list.
  - `/embed-image`: `open_clip` (`ViT-B-32`, `laion2b_s34b_b79k`) returning normalized 512-dim list.

- Main Backend API Server (`main.py` on Port 1000):
  - FastAPI app with CORS middleware enabled.
  - GET `/`: health check endpoint.
  - GET `/images`: returns list of screenshot filenames in screenshot directory.
  - GET `/image/{filename}`: serves individual image files.
  - GET `/image/search/{query}`: returns FTS text search results.
  - GET `/image/search_id/{img_id}`: returns image metadata by ID.
  - POST `/ingest`: accepts `IngestRequest` containing image path, runs ingestion pipeline, and saves record to SQLite.
  - GET `/get-clusters`: returns images ordered by `cluster_id`.

- Continuous Ingestion Watcher (`backend/bash/watcher.sh`):
  - Uses `fswatch` to monitor local screenshots directory for new files.
  - Automatically sends POST requests with JSON payload to `/ingest` endpoint when new screenshots appear.

- Frontend UI (`frontend/`):
  - Built using React and Vite.
  - Fetches image data from `/get-clusters` API endpoint.
  - Groups images by `cluster_id` and renders gallery cards visually organized by cluster (e.g. Cluster 0, Cluster 1, and "Noise / Unclustered" for `-1`).
  - Search bar component allows interactive search and filtering across screenshot collections.


### System Flow & Reclustering Strategy (from System Design Notes)

- Baseline Accumulation:
  - Initial run creates metadata, OCR text, and embeddings for all client screenshots and populates SQLite.
  - Clusters are formed using HDBSCAN and saved to DB.

- Incremental Ingestion for New Screenshots:
  - New image arrives -> OCR & embeddings extracted.
  - Calculates cosine similarity / FAISS distance against existing cluster embeddings.
  - Runs k-nearest neighbors (KNN) voting / min-heap distance check to assign new image to best fit cluster.

- Periodic Full Reclustering:
  - Cluster centroids become unstable over time as new images arrive.
  - System requires running full re-clustering periodically (every 500-1000 images) to maintain high cluster quality and avoid centroid drift.



### Future steps:
- separation of media per-user, so each user can have their own images clustered.
- multi-tenant system
- making each component in the end-to-end pipeline sklearn fit-transform compatible so sklearn pipeline can be used.



![](Assets/Screenshot%20analyser.pdf)