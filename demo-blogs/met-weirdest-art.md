# Finding the weirdest art at the Met

The Met Museum has a large open-access collection, but browsing it is still catalog-first: department, artist, culture, date. This demo asks a visual question instead. Which public-domain artworks are most isolated in CLIP space, and which pairs look nearly identical despite coming from different departments, centuries, or cultures?

The pipeline joins enhanced Met metadata with a community CRDImages URL map, fetches web-size JPEGs from `images.metmuseum.org/CRDImages/`, embeds them with the `Qdrant/clip-ViT-B-32-vision` ONNX model through `fastembed`, and reduces the vector shards with FAISS.

## What The Job Does

The compromised version embeds a few thousand Met thumbnails and sorts nearest neighbors. It produces fun images, but the result is fragile. Visual outliers are only meaningful when they are outliers against most of the collection.

The real version works over about 213,000 artworks with both metadata and direct CDN image URLs. Stage 0 downloads `BetterMetObjects.csv` and `met-openaccess-images.csv`, joins on `object_id`, swaps `/original/` image paths to `/web-large/`, and writes `objects.parquet` under `/workspace/shared/met-weirdest/`. The map stage fans out batches of object IDs. Each worker loads the Parquet, fetches images with a 16-thread HTTP pool, thumbnails them, embeds them, and writes one vector Parquet shard.

The reduce stage loads every vector shard, normalizes the matrix, builds a FAISS index, ranks isolation by kth-nearest-neighbor similarity, and finds high-similarity pairs that are not obvious duplicates.

## The Code Shape

The worker is a mix of network IO and CPU inference. It fetches many images concurrently, then runs CLIP locally on the worker.

```python
def fetch_and_embed(batch: list[int]) -> str:
    import requests
    from PIL import Image
    from concurrent.futures import ThreadPoolExecutor, as_completed

    rows = objs.reindex(batch).dropna(subset=["crd_urlpath"]).reset_index(drop=True)
    session = requests.Session()
    session.headers.update({"User-Agent": USER_AGENT, "Accept": "image/*"})

    with ThreadPoolExecutor(max_workers=HTTP_THREADS) as ex:
        futures = [ex.submit(_fetch, w) for w in work]
        for fut in as_completed(futures):
            oid, data = fut.result()
            results[oid] = data

    model = _get_clip()
    vecs = np.asarray(list(model.embed(images, batch_size=CLIP_BATCH)), dtype="float32")
    norms = np.linalg.norm(vecs, axis=1, keepdims=True)
    norms = np.where(norms < 1e-12, 1.0, norms)
    vecs = vecs / norms
    pq.write_table(out_tbl, str(out_path))
    return str(out_path)
```

The dispatch is bounded because the Met CDN can tolerate parallel reads, but not infinite pressure:

```python
vec_paths_raw = list(remote_parallel_map(
    fetch_and_embed,
    batches,
    func_cpu=1,
    func_ram=4,
    max_parallelism=max_workers,
))
```

The reducer runs larger because FAISS needs the full vector matrix:

```python
[results_dir] = list(remote_parallel_map(
    reduce_met,
    [vec_paths],
    func_cpu=16,
    func_ram=64,
))
```

## Why It Matters

This is a good shape for visual dataset audits. The expensive part is not one model call. It is fetching hundreds of thousands of images, surviving bad URLs and CDN behavior, writing durable vector shards, and doing a global nearest-neighbor pass.

Burla is useful here because the code stays close to the data problem: download, embed, reduce. The artifact is a ranked set of visual outliers and hidden twins, backed by CLIP vectors and FAISS rather than a hand-picked gallery.
