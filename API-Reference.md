---
layout:
  width: default
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: false
---

# API/CLI Reference

## API-Reference

### `burla.remote_parallel_map`

Run any python function on many remote computers at the same time.\
This is the only function in the Burla python package.

```python
remote_parallel_map(
  function_,
  inputs,
  func_cpu=1,
  func_ram=4,
  background=False,
  spinner=True,
  generator=False,
  max_parallelism=None,
)
```

Run provided `function_` on each item in `inputs` at the same time, each on a separate CPU. If more inputs are provided than there are available CPU's, they are queued and processed sequentially on each worker. `remote_parallel_map` can reliably queue millions of inputs.

While running:

* If the provided `function_` raises an exception, the exception, including stack trace, is re-raised on the client machine in a way that looks like it was running locally.
* Your print statements (anything written to stdout/stderr) are streamed back to your local machine, appearing like they would have if running the same code locally.

When finished `remote_parallel_map` returns a list of objects returned by each `function_` call.

| **Parameters**    |                                                                                                                                                                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**          | **Description**                                                                                                                                                                                                              |
| `function_`       | <p><code>Callable</code></p><p>Any python function that is &#x3C;100MB (1M lines) when pickled. </p>                                                                                                                         |
| `inputs`          | <p><code>List[Any]</code></p><p>List of elements passable to <code>function_</code>.<br>Tuples are unpacked into *args by default.</p>                                                                                       |
| `func_cpu`        | <p><code>int</code></p><p>(Optional) Number of CPU's made available to each running instance of <code>function_</code>. Max possible value is determined by your cluster machine type. Defaults to 1.</p>                    |
| `func_ram`        | <p><code>int</code></p><p>(Optional) Amount of RAM (GB) made available to each running instance of <code>function_</code>. Max possible value is determined by your cluster machine type. Defaults to 4.</p>                 |
| `detach`          | <p><code>bool</code><br>(Optional) Job will continue to run independently on the cluster if stopped locally. Detached jobs can run in the background for at least a week (as far as we tested).</p>                          |
| `spinner`         | <p><code>bool</code></p><p>(Optional) Set to <code>False</code> to hide the status indicator/spinner.</p>                                                                                                                    |
| `generator`       | <p><code>bool</code><br>(Optional) Set to <code>True</code> to return a <code>Generator</code> instead of a <code>List</code>. The generator will yield outputs as they are produced, instead of all at once at the end.</p> |
| `max_parallelism` | <p><code>int</code></p><p>(Optional) Maximum number of <code>function_</code> instances allowed to be running at the same time.</p>                                                                                          |



| **Returns**           |                                                                                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Type**              | **Description**                                                                                                                                                                 |
| `List` or `Generator` | List of objects returned by `function_` in no particular order. If `Generator=True`, returns generator yielding objects returned by `function_` in the order they are produced. |

***

&#x20;

## CLI-Reference

Burla's CLI contains the following commands:

* [`burla install`](API-Reference.md#burla-install)  Deploy self-hosted Burla in your Google Cloud project.
* [`burla login`](API-Reference.md#burla-login)  Connect your computer to the cluster you last logged into in the browser.

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla install`

Deploy a self-hosted Burla instance in your current Google Cloud Project.\
Running `burla install` multiple times will update the existing installation with the latest version.

**Description:**

Installs Burla inside the Google Cloud project that your [gcloud CLI](https://cloud.google.com/sdk/gcloud) is currently pointing to.\
For a more user-friendly installation guide see: [Installation: Self-Hosted](get-started.md)

To view your current gcloud project run: `gcloud config get project`\
To change your current gcloud project run: `gcloud config set project <desired-project-id>`&#x20;

**Prerequisites:**

* Have the [gcloud CLI](https://cloud.google.com/sdk/gcloud) installed ([how do I install the gcloud CLI?](https://cloud.google.com/sdk/docs/install)).
* Be logged in to the [gcloud CLI](https://cloud.google.com/sdk/gcloud) ([how do I log in?](https://cloud.google.com/sdk/docs/authorizing#user-account))\
  (`gcloud auth login` & `gcloud auth application-default login`)
* Have a Google Cloud user account with at least the minimum required permissions to install Burla.\
  Or: Just run `burla install`, if you're missing any permissions it will tell you which ones!

Here is the set of permissions you'll need to run `burla install`:\
Any of these three permission set's will work.

<details>

<summary>Simplest possible permissions</summary>

Burla can be installed with either of the following roles:

* Project Owner (`roles/owner`)
* Project Editor (`roles/editor`)

</details>

<details>

<summary>Service admin based permissions (more specific)</summary>

Burla can be installed by users having the following generic roles:

1. Service Usage Admin (`roles/serviceusage.serviceUsageAdmin`)
2. Cloud Run Admin (`roles/run.admin`)
3. Compute Network Admin (`roles/compute.networkAdmin`)
4. Secret Manager Admin (`roles/secretmanager.admin`)
5. Service Account Admin (`roles/iam.serviceAccountAdmin`)
6. Service Account Key Admin (`roles/iam.serviceAccountKeyAdmin`)
7. Project IAM Admin (`roles/resourcemanager.projectIamAdmin`)
8. Firestore / Datastore Owner (`roles/datastore.owner`)

</details>

<details>

<summary>Exact minimum required permissions (very specific)</summary>

Here is an IAM role definition for this permission set:

```yaml
title: "Burla Installer"
description: "Minimum permissions required to install Burla"
stage: "GA"
includedPermissions:
  # Enable required APIs
  - serviceusage.services.enable
  - serviceusage.services.get
  - serviceusage.services.list

  # Read project number + set project IAM bindings
  - resourcemanager.projects.get
  - resourcemanager.projects.getIamPolicy
  - resourcemanager.projects.setIamPolicy

  # Firewall rule for cluster nodes
  - compute.firewalls.create
  - compute.firewalls.get
  - compute.firewalls.list

  # Create + manage bucket
  - storage.buckets.create
  - storage.buckets.get
  - storage.buckets.list
  - storage.buckets.update

  # Secret for cluster token + IAM bindings + versions
  - secretmanager.secrets.create
  - secretmanager.secrets.get
  - secretmanager.secrets.list
  - secretmanager.secrets.getIamPolicy
  - secretmanager.secrets.setIamPolicy
  - secretmanager.versions.add
  - secretmanager.versions.access

  # Create service accounts, grant them roles, rotate keys, and use them for Cloud Run deploy
  - iam.serviceAccounts.create
  - iam.serviceAccounts.get
  - iam.serviceAccounts.list
  - iam.serviceAccounts.getIamPolicy
  - iam.serviceAccounts.setIamPolicy
  - iam.serviceAccounts.actAs
  - iam.serviceAccountKeys.create
  - iam.serviceAccountKeys.list
  - iam.serviceAccountKeys.delete

  # Create Firestore database + write initial config doc via Firestore client
  - datastore.databases.create
  - datastore.databases.get
  - datastore.entities.create
  - datastore.entities.update
  - datastore.entities.get

  # Deploy and configure Cloud Run service
  - run.services.create
  - run.services.get
  - run.services.list
  - run.services.update
  - run.services.getIamPolicy
  - run.services.setIamPolicy
```

</details>

{% hint style="info" %}
On install, your Google account (the one you are currently logged in to `gcloud` with) is set as the only account authorized to access this new Burla deployment.
{% endhint %}

We encourage you to check out [\_install.py](https://github.com/Burla-Cloud/burla/blob/main/client/src/burla/_install.py) in the client for even more specific installation details.

***

&#x20;

### `burla login`

Connects your computer to the Burla cluster you most recently logged into in your browser.\
Authorizes your machine to call `remote_parallel_map` on this cluster.

**Description:**

Launches the "Authorize this Machine" page in your default web browser.

If there is no auth-cookie (you have not yet logged into the dashboard), throws simple error requesting you login to your cluster dashboard first.

When the "Authorize" button is hit, a new auth token is created and sent to your machine.&#x20;

This token is saved in the text file `burla_credentials.json`. This file is stored in your operating system's recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

This token is refreshed each time the `burla login` is run, or certain amount of time passes.

&#x20;

&#x20;

***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
