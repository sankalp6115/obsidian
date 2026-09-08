
This code compares embedding of new image to embeddings of all images of dataset, using cosiine similarity, and finds top 5 closest similar images, and then using KNN we can assign cluster to that image
```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

query = new_embedding.reshape(1, -1)
all_embeddings = np.array([...])  # load from DB

scores = cosine_similarity(query, all_embeddings)[0]

top_k_indices = scores.argsort()[-5:][::-1]
```
