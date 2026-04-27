# Let CLIP compare every Met image, then look for impossible twins

The compromised art demo embeds a gallery sample and finds obvious visual matches. The real version fetches every Met Open Access artwork with a usable image, embeds about 192,000 pieces, and searches across centuries and departments. A sample would mostly find same-room neighbors.

This demo asks a plain question: which artworks look alike even though the metadata says they should not?

## what we built

Discovery joins Met object metadata to CRDImages paths, then batches object ids. The image URL is deterministic, so workers can fetch from the CDN without the rate-limited API.

```python
CRD_IMAGE_BASE = "https://images.metmuseum.org/CRDImages/"
OBJECTS_PATH = Path("/workspace/shared/met-weirdest/objects.parquet")

ids = df["object_id"].tolist()
random.Random(42).shuffle(ids)
batches = [ids[i:i + batch_size] for i in range(0, len(ids), batch_size)]
```

Each map worker fetches images with a thread pool, thumbnails them, embeds with CLIP, normalizes, and writes a vector Parquet shard.

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

## how the pipeline works

The reducer loads vector shards, builds a FAISS cosine index, and filters matches by time gap, culture, and department.

```python
model = _get_clip()
vecs = np.asarray(list(model.embed(images, batch_size=CLIP_BATCH)), dtype="float32")
vecs = vecs / np.maximum(np.linalg.norm(vecs, axis=1, keepdims=True), 1e-12)

index = faiss.IndexFlatIP(CLIP_DIM)
index.add(vecs.astype("float32"))
```

## why this demo is interesting

Museum search usually starts with metadata: artist, period, department, object type. This demo asks what happens when you ignore that and let image geometry speak first. The answer is not art history truth. It is visual coincidence at scale, which is interesting precisely because the metadata says the pairs are unrelated.

The full run matters because nearest-neighbor search is competitive. A gallery sample might find a nice pair, but it cannot tell you whether that pair is actually unusual among 192,000 images. The FAISS reduce makes each candidate compete against the whole museum image corpus before it gets into the final gallery.

## how to build your version

Use stable image URLs when you can. Keep fetch concurrency inside each worker and worker concurrency outside with `max_parallelism`. Write metadata next to vectors so the reduce can filter matches without another API call.

## practical notes from the build

The useful engineering choice was to avoid the Met API during the hot path. The API is fine for discovery, but the image fetch should hit stable CDN URLs with enough metadata already joined into `objects.parquet`. That lets workers fetch and embed without another metadata round trip.

The reduce filter matters as much as CLIP. Raw nearest neighbors would produce many same-period, same-object-type matches. The demo is interesting because candidates have to cross time, culture, or department boundaries. If you build your own version, write the exclusion rules before browsing the results, or you will tune them around the pairs you already like.

## why Burla fits

Burla removes the image worker fleet, shared vector storage, reducer setup, and CDN-friendly dispatch. It lets you keep the whole experiment as one Python script with map and reduce stages.


## ways to adapt it

The same method works for product catalogs, satellite chips, microscopy images, fashion archives, or museum collections with stable image URLs. The map worker should fetch, normalize, embed, and write vectors with enough metadata for filtering. The reduce should encode the interesting constraint: cross-century, cross-brand, cross-region, rare defect, or near-duplicate. The constraint is what turns nearest neighbors into a discovery tool.

## what the gallery sample misses

The real run found a 19th-century silverware case and a Bronze Age Cypriot dagger blade that CLIP sees as visual twins. The compromised gallery sample would have found bowls next to bowls and missed the strange cross-century pair.
