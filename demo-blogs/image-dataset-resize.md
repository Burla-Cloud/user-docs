# Resize the whole image corpus before training on it

The compromised version resizes a few folders, starts training, and discovers later that half the corpus has EXIF rotations, corrupt PNGs, or odd aspect ratios. The real preprocessing job touches every image and writes the exact sizes the model will consume.

This demo resizes 5,000,000 images from S3 into 256, 512, and 1024 pixel variants using Pillow on thousands of Burla workers.

## what we built

The client lists source keys and batches them into 1,000-image chunks.

```python
import boto3

keys = []
paginator = boto3.client("s3").get_paginator("list_objects_v2")
for page in paginator.paginate(Bucket="my-photos", Prefix="originals/"):
    keys += [obj["Key"] for obj in page.get("Contents", []) if obj["Key"].lower().endswith((".jpg", ".jpeg", ".png"))]

chunks = [keys[i:i + 1000] for i in range(0, len(keys), 1000)]
```

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

## how the pipeline works

Results stream back while workers write images directly to S3.

```python
from burla import remote_parallel_map

for chunk_result in remote_parallel_map(
    resize_chunk, chunks, func_cpu=1, func_ram=4, generator=True, grow=True,
):
    for row in chunk_result:
        f.write(json.dumps(row) + "\n")
```

## why this demo is interesting

Image preprocessing is easy to underestimate because a preview folder looks fine. At corpus scale, the failures are mundane and expensive: portrait images with EXIF rotation, CMYK JPEGs, giant PNGs, truncated files, odd file extensions, and source buckets that throttle a single client. A compromised run smooths over exactly the cases that later crash training.

The output report is as useful as the resized images. It gives you input dimensions, success flags, and error strings by key. That report becomes the training manifest, the retry list, and the evidence that your model did not silently skip a biased slice of the corpus.

## how to build your version

Choose a chunk size that keeps network and CPU balanced. Put all output writes inside the worker, and stream a report to JSONL. Add your model-specific preprocessing: center crop, pad, WebP conversion, EXIF stripping, or train/val folder routing.

## why Burla fits

Burla removes Lambda fan-out limits, Batch images, manual queues, and a self-managed worker fleet. You get thousands of VMs reading and writing S3 with a normal Python function.

## what the folder sample misses

The compromised subset misses the corrupt images and weird dimensions that break training. The real run builds the manifest that tells you exactly which images are usable.
