# Resize the whole image corpus before training on it

In this example we:

* List 5,000,000 source images from S3.
* Resize each image into 256, 512, and 1024 pixel variants.
* Stream a manifest while workers write outputs back to S3.

A preview folder always looks fine. The full corpus is where the EXIF rotations, corrupt PNGs, CMYK JPEGs, and odd aspect ratios live.

### Step 1: Chunk the image keys

The client lists source keys and batches them into 1,000-image chunks.

```python
import boto3

keys = []
paginator = boto3.client("s3").get_paginator("list_objects_v2")
for page in paginator.paginate(Bucket="my-photos", Prefix="originals/"):
    keys += [obj["Key"] for obj in page.get("Contents", []) if obj["Key"].lower().endswith((".jpg", ".jpeg", ".png"))]

chunks = [keys[i:i + 1000] for i in range(0, len(keys), 1000)]
```

### Step 2: Resize inside the worker

The worker opens each image, fixes EXIF orientation, writes every target size, and returns a small report.

```python
def resize_chunk(image_keys: list[str]) -> list[dict]:
    import io, os, boto3
    from PIL import Image, ImageOps

    s3 = boto3.client("s3")
    out = []
    for key in image_keys:
        body = s3.get_object(Bucket="my-photos", Key=key)["Body"].read()
        img = ImageOps.exif_transpose(Image.open(io.BytesIO(body))).convert("RGB")
        stem = os.path.splitext(os.path.basename(key))[0]
        for size in [256, 512, 1024]:
            resized = img.copy()
            resized.thumbnail((size, size), Image.Resampling.LANCZOS)
            buf = io.BytesIO()
            resized.save(buf, format="JPEG", quality=85, optimize=True, progressive=True)
            s3.put_object(Bucket="my-photos-resized", Key=f"resized/{size}/{stem}.jpg", Body=buf.getvalue())
        out.append({"key": key, "orig_w": img.size[0], "orig_h": img.size[1], "ok": True})
    return out
```

### Step 3: Stream the manifest

Workers write images directly to S3. The client writes the report as chunks finish.

```python
from burla import remote_parallel_map

for chunk_result in remote_parallel_map(
    resize_chunk,
    chunks,
    func_cpu=1,
    func_ram=4,
    generator=True,
    grow=True,
):
    for row in chunk_result:
        f.write(json.dumps(row) + "\n")
```

### What's the point?

The resized images are only half the result. The manifest tells you which files worked, what dimensions they had, and which ones need a retry.

If I were about to train on this dataset, I would want that manifest before training starts. Otherwise the model can silently skip the weird slice of the corpus, and you only find out later when the training data looks cleaner than reality.
