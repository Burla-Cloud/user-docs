# Cluster all arXiv abstracts before naming extinct topics

In this example we:

* Embed 2,710,783 arXiv abstracts.
* Cluster the whole corpus with MiniBatchKMeans.
* Use FAISS to find lonely papers and topic clusters that faded over time.

If the question is historical, a recent-paper sample is almost useless. It starts after half the history already happened.

### Step 1: Shard the metadata

The arXiv snapshot is JSONL. We stream it once and write 10,000-paper Parquet shards into the shared folder.

```python
def flush(records, idx):
    out = RAW_DIR / f"shard_{idx:05d}.parquet"
    tbl = pa.table({
        "id": [r.get("id", "") for r in records],
        "title": [" ".join((r.get("title") or "").split()) for r in records],
        "abstract": [" ".join((r.get("abstract") or "").split()) for r in records],
        "categories": [r.get("categories", "") or "" for r in records],
        "created": [_extract_created(r) for r in records],
    })
    pq.write_table(tbl, str(out))
    return str(out)
```

### Step 2: Embed each shard

Each worker reads one raw shard, embeds title plus abstract, normalizes the vectors, and writes another Parquet shard.

```python
def embed_shard(raw_path: str) -> str:
    tbl = pq.read_table(raw_path)
    texts = [f"{t}\n{a}" for t, a in zip(tbl.column("title").to_pylist(), tbl.column("abstract").to_pylist())]
    model = _get_model()
    vecs = np.asarray(list(model.embed(texts, batch_size=EMBED_BATCH)), dtype="float32")
    vecs = vecs / np.maximum(np.linalg.norm(vecs, axis=1, keepdims=True), 1e-12)
    out_path = VEC_DIR / f"{Path(raw_path).stem}.parquet"
    pq.write_table(tbl.append_column("vector", pa.array(vecs.tolist(), type=pa.list_(pa.float32(), 384))), str(out_path))
    return str(out_path)
```

### Step 3: Reduce the whole corpus

The reduce worker loads the vector shards, clusters a sample, predicts labels for all papers, and builds a nearest-neighbor index.

```python
km = MiniBatchKMeans(n_clusters=400, random_state=42, batch_size=16384, max_iter=80, n_init=1)
km.fit(fit_vecs)
labels = km.predict(vecs)

index = faiss.IndexFlatIP(vecs.shape[1])
index.add(vecs.astype("float32"))
```

### What's the point?

The worker cannot know whether a topic is extinct. It only sees one shard. The label comes later, once the whole archive is visible.

That is the useful shape here: many workers produce vectors, then one bigger worker makes the global decision. If I were doing this for patents, PubMed abstracts, legal opinions, or internal docs, I would keep the same split.
