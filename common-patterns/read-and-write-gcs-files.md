---

description: Write and read files in GCS through /workspace/shared.
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false

---

# Read and Write GCS Files

In Burla, `/workspace/shared` is like a shared folder that is connected to your Google Cloud Storage bucket.

- Save a file to `/workspace/shared/...` and it shows up in your bucket.
- Read a file from `/workspace/shared/...` and it is read from your bucket.

## Before you run this

1. Install Burla: `pip install burla`
2. Log in: `burla login`
3. Start your cluster in the Burla dashboard

## Step 1: Write files (saves to GCS)

```python
from pathlib import Path

from burla import remote_parallel_map


def write_text_file(path_and_text):
    file_path, text = path_and_text
    Path(file_path).write_text(text)
    return file_path


files_to_write = [
    ("/workspace/shared/hello.txt", "hello\n"),
    ("/workspace/shared/goodbye.txt", "goodbye\n"),
]

remote_parallel_map(write_text_file, files_to_write)
```

In the Burla dashboard → **Filesystem**, you should see:

- `/workspace/shared/hello.txt`
- `/workspace/shared/goodbye.txt`

Put your screenshot here:

![Filesystem shows the new files](<PUT_YOUR_FILESYSTEM_SCREENSHOT_PATH_HERE>)

## Step 2: Read files back (comes from GCS)

```python
from pathlib import Path

from burla import remote_parallel_map


def read_text_file(file_path):
    return {"path": file_path, "text": Path(file_path).read_text()}


files_to_read = [
    "/workspace/shared/hello.txt",
    "/workspace/shared/goodbye.txt",
]

results = remote_parallel_map(read_text_file, files_to_read)
print(results)
```

You should see a list with the file paths and their text (the words `hello` and `goodbye`).

Put your screenshot of the printed output here:

![Printed output after reading](<PUT_YOUR_OUTPUT_SCREENSHOT_PATH_HERE>)
