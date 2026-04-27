# Let CLIP compare every Met image

In this example we:

* Fetch every Met Open Access artwork with a usable image.
* Embed about 192,000 images with CLIP.
* Use FAISS to find visual twins across time, culture, and department.

The question is deliberately weird: which artworks look alike even though the metadata says they should not?

### Step 1: Build the image queue

Discovery joins Met object metadata to CRDImages paths. After that the image URL is deterministic, so workers can fetch from the CDN instead of hammering the API.

```python
CRD_IMAGE_BASE = "https://images.metmuseum.org/CRDImages/"
OBJECTS_PATH = Path("/workspace/shared/met-weirdest/objects.parquet")

ids = df["object_id"].tolist()
random.Random(42).shuffle(ids)
batches = [ids[i:i + batch_size] for i in range(0, len(ids), batch_size)]
```

### Step 2: Fetch and embed

Each worker fetches images with a thread pool, thumbnails them, embeds with CLIP, and writes a vector shard.

```python
def fetch_and_embed(batch: list[int]) -> str:
    import requests
    from PIL import Image
    from concurrent.futures import ThreadPoolExecutor, as_completed

    objs = pd.read_parquet(OBJECTS_PATH).set_index("object_id", drop=False)
    rows = objs.reindex(batch).dropna(subset=["crd_urlpath"]).reset_index(drop=True)
    session = requests.Session()
    work = [(int(oid), CRD_IMAGE_BASE + str(path)) for oid, path in zip(rows["object_id"], rows["crd_urlpath"])]
    with ThreadPoolExecutor(max_workers=HTTP_THREADS) as ex:
        results = dict(f.result() for f in as_completed([ex.submit(_fetch, w) for w in work]))
```

### Step 3: Search the museum

The reducer loads vector shards, builds a FAISS cosine index, and filters out boring matches.

```python
model = _get_clip()
vecs = np.asarray(list(model.embed(images, batch_size=CLIP_BATCH)), dtype="float32")
vecs = vecs / np.maximum(np.linalg.norm(vecs, axis=1, keepdims=True), 1e-12)

index = faiss.IndexFlatIP(CLIP_DIM)
index.add(vecs.astype("float32"))
```

The filter matters. Raw nearest neighbors mostly find same-period, same-object-type matches. The fun pairs are the ones that cross boundaries.

### What's the point?

I would not call this art history truth. It is visual coincidence at scale, and that is exactly why it is fun.

A small gallery sample finds bowls next to bowls. The full run can find a 19th-century silverware case that looks weirdly like a Bronze Age Cypriot dagger blade. I would never trust that result from a tiny sample, because nearest-neighbor search is competitive. The candidate has to compete against the whole museum.
