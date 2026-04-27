# Cluster all arXiv abstracts before naming extinct topics

The compromised version embeds recent ML papers and calls it a trend study. The real version embeds 2,710,783 arXiv abstracts, clusters the whole corpus, and then asks which topics peaked, faded, or appeared recently. If the corpus starts after your favorite field got big, the experiment is already biased.

This demo uses the arXiv metadata snapshot, MiniLM embeddings via fastembed, MiniBatchKMeans, and FAISS.

## what we built

Stage 0 streams the JSONL snapshot and writes 10,000-paper Parquet shards to `/workspace/shared/arxiv-fossils/raw/`.

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

Each map worker embeds one raw shard and writes a vector shard.

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

## how the pipeline works

The reduce loads every vector shard, clusters a sample, predicts labels for all papers, and builds a nearest-neighbor index.

```python
km = MiniBatchKMeans(n_clusters=400, random_state=42, batch_size=16384, max_iter=80, n_init=1)
km.fit(fit_vecs)
labels = km.predict(vecs)

index = faiss.IndexFlatIP(vecs.shape[1])
index.add(vecs.astype("float32"))
```

## why this demo is interesting

Topic history is a full-corpus problem. If you embed only recent papers, every topic looks alive. If you embed only one field, cross-field outliers disappear. The real run lets old high-energy physics clusters, pandemic SIR bursts, and LLM evaluation clusters sit in the same vector space before any label is attached.

The reduce stage is where the scientific question happens. KMeans gives rough topic neighborhoods. Year histograms decide which clusters peaked and faded. FAISS nearest-neighbor distance finds the loneliest paper. Burla does not make those choices for you; it makes the full set of choices cheap enough to run.

## how to build your version

Use the full corpus if your question is historical. Shard raw text first, embed in map workers, and do the expensive global operations in one large reduce worker. Pin ONNX threads to one per worker so parallelism comes from workers, not oversubscribed threads.

## practical notes from the build

The main trick is separating local decisions from global ones. Embedding is embarrassingly parallel because each abstract stands alone. Clustering and nearest-neighbor search are global because every paper competes with every other paper. The pipeline reflects that: many map workers write vectors, one larger reducer loads the vectors and makes corpus-level decisions.

For a reader building a similar system, resist the urge to classify papers during the map stage. The worker does not know whether a topic is extinct, emergent, or lonely. It only knows how to produce a vector and metadata. The label comes later, after the full corpus is visible.

## why Burla fits

Burla removes the embedding worker pool, shared shard storage, reducer machine selection, and queue logic. You can mix many small embedding workers with one high-memory reducer.


## ways to adapt it

Use the same pattern for patents, PubMed abstracts, legal opinions, support tickets, or internal docs. The map stage should produce vectors and metadata. The reduce stage should do the global math: clustering, nearest neighbors, time histograms, or drift scores. Keep labels downstream of the full vector set so early workers do not make corpus-level claims from local shards.

## what the small corpus misses

The real run finds the lonely Norway financial-statements paper and old braneworld clusters fading relative to the full archive. The compromised recent-ML sample cannot see extinction because it removed history.
