---
title: "[Chroma DB] Change embeddings(using huggingface model)"
layout: post
date: '2023-12-20 22:31:30'
author: Edward Park
categories:
- Python
tags:
- VectorDB
cover: "/assets/instacode.png"
---

## Intro
[Chroma Docs](https://docs.trychroma.com/embeddings) <br>
- Chroma DB's default embedding model is `all-MiniLM-L6-v2`. But in languages other than English, better models exist.<br>
- Chroma DB supports huggingface models and usage is very simple.

## Code
**My Development Env**<br>
- Python: 3.9.13
- Chroma: 0.4.18

### Use Default Embedding Model
- It **may take a long time to download the model** when running it for the first time.

```Python
import chromadb
from chromadb.db.base import UniqueConstraintError
client = chromadb.PersistentClient(path="db/")  # data stored in 'db' folder
try:
    collection = client.create_collection(name='article')
except UniqueConstraintError:  # already exist collection
    collection = client.get_collection(name='article')
```

### Use Custom Embedding Model
- In this example, used [Huffon/sentence-klue-roberta-base](https://huggingface.co/Huffon/sentence-klue-roberta-base) model in huggingface.


```Python
import chromadb
from chromadb.db.base import UniqueConstraintError
from chromadb.utils import embedding_functions
client = chromadb.PersistentClient(path="db/")  # data stored in 'db' folder
em = embedding_functions.SentenceTransformerEmbeddingFunction(model_name="Huffon/sentence-klue-roberta-base")
try:
    collection = client.create_collection(name='article', embedding_function=em)
except UniqueConstraintError:  # already exist collection
    collection = client.get_collection(name='article', embedding_function=em)
```

### Test: Save/Query
```Python
# save
collection.add(
        documents = ['NAVER Corporation Earnings Surprise', 'Samgsung Corporation Earnings Surprise'],
        ids = ['naver1', 'samsung1']
    )
# query
results = collection.query(
    query_texts='SamSam',
    n_results=1
)
print(results)
```
