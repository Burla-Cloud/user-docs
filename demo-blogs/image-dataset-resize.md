# Resizing an image dataset in parallel

Image resize jobs are usually simple until there are millions of images. Then the slow parts are not the Pillow calls. They are listing object storage, downloading bytes, preserving EXIF orientation, writing several output sizes, and getting progress without one huge failure domain.

This demo reads original images from an S3 bucket, creates 256, 512, and 1024 pixel JPEG versions, and writes them to another bucket. It is deliberately plain: `boto3`, Pillow, chunked input lists, and `remote_parallel_map`.

## What The Job Does

The compromised version runs on a local directory or a few thousand S3 keys. That is enough to test resize quality and output naming. It does not test the real bottleneck: sustained S3 read and write throughput across a large bucket.

The real version paginates `my-photos/originals/`, filters JPEG and PNG keys, groups them into 1,000-key chunks, and sends each chunk to a worker. The worker downloads each image from `my-photos`, applies `ImageOps.exif_transpose`, converts to RGB, writes three progressive JPEGs to `my-photos-resized`, and returns one small status row per input image.

The driver streams chunk results with `generator=True` and writes `resize_report.jsonl` as each worker finishes. That is a better shape than waiting for every image status in memory.

## The Code Shape

The worker keeps all image IO inside the remote task. Only status rows return to the driver.

```python
def resize_chunk(image_keys: list[str]) -> list[dict]:
    import io
    import os
    import boto3
    from PIL import Image, ImageOps

    s3 = boto3.client("s3")
    out = []
    for key in image_keys:
        body = s3.get_object(Bucket="my-photos", Key=key)["Body"].read()
        img = Image.open(io.BytesIO(body))
        img = ImageOps.exif_transpose(img).convert("RGB")

        stem = os.path.splitext(os.path.basename(key))[0]
        for size in [256, 512, 1024]:
            resized = img.copy()
            resized.thumbnail((size, size), Image.Resampling.LANCZOS)
            buf = io.BytesIO()
            resized.save(buf, format="JPEG", quality=85, optimize=True, progressive=True)
            s3.put_object(
                Bucket="my-photos-resized",
                Key=f"resized/{size}/{stem}.jpg",
                Body=buf.getvalue(),
                ContentType="image/jpeg",
            )
        out.append({"key": key, "orig_w": w, "orig_h": h, "ok": True})
    return out
```

The Burla call is a streaming map:

```python
with open("resize_report.jsonl", "w") as f:
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

## Why It Matters

This is a common data-engineering chore. Teams often reach for a queue, workers, container images, and a retry dashboard. For a one-off or periodic resize, that can be more machinery than the job deserves.

Burla fits because the unit of work is obvious: a list of object keys. The source and destination are S3 buckets, the worker function is normal Pillow code, and the output report is JSONL. The cluster only supplies enough parallel IO and CPU to finish the backlog quickly.
