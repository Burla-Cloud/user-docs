# API/CLI Reference

## API-Reference

### `burla.remote_parallel_map`

Run any python function on many remote computers at the same time.

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

When finished `remote_parallel_map` returns a list of values returned by each `function_` call.

| **Parameters**    |                                                                                                                                                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**          | **Description**                                                                                                                                                                                                   |
| `function_`       | <p><code>Callable</code></p><p>Python function. <strong>Must have single input argument</strong>, eg: <code>function_(inputs[0])</code> does not raise an exception.</p>                                          |
| `inputs`          | <p><code>List[Any]</code></p><p>Iterable of elements passable to <code>function_</code>.</p>                                                                                                                      |
| `func_cpu`        | <p><code>int</code></p><p>(Optional) Number of CPU's made available to each running instance of <code>function_</code>. Max possible value is determined by your cluster machine type.</p>                        |
| `func_ram`        | <p><code>int</code></p><p>(Optional) Amount of RAM (GB) made available to each running instance of <code>function_</code>. Max possible value is determined by your cluster machine type.</p>                     |
| `background`      | <p><code>bool</code><br>(Optional) <code>remote_parallel_map</code> returns as soon as your inputs and function have been uploaded. The job then continues to run independently in the background until done.</p> |
| `spinner`         | <p><code>bool</code></p><p>(Optional) Set to <code>False</code> to hide the status indicator/spinner.</p>                                                                                                         |
| `generator`       | <p><code>bool</code><br>(Optional) Set to <code>True</code> to return a <code>Generator</code> instead of a <code>List</code>. The generator will yield outputs as they are produced, instead of all at once.</p> |
| `max_parallelism` | <p><code>int</code></p><p>(Optional) Maximum number of <code>function_</code> instances allowed to be running at the same time.</p>                                                                               |



| **Returns**           |                                                                                                                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Type**              | **Description**                                                                                                                                                                 |
| `List` or `Generator` | List of objects returned by `function_` in no particular order. If `Generator=True`, returns generator yielding objects returned by `function_` in the order they are produced. |

***

&#x20;

## CLI-Reference

Burla's CLI contains the following commands:

* [`burla install`](API-Reference.md#burla-install) Install a self-hosted Burla instance in your Google Cloud project.
* [`burla login`](API-Reference.md#burla-login) Authenticate your machine.

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla install`

Install a self-hosted Burla instance in your Google Cloud project.\
Running `burla install` multiple times will update the existing installation with the latest version.

**Description:**

Installs Burla inside the Google Cloud project that your [gcloud CLI](https://cloud.google.com/sdk/gcloud) is currently pointing to.\
For a more user-friendly installation guide see: [Installation: Self-Hosted](getting-started.md)

To view your current gcloud project run: `gcloud config get project`\
To change your current gcloud project run: `gcloud config set project <desired-project-id>`&#x20;

**Prerequisites:**

* Have the [gcloud CLI](https://cloud.google.com/sdk/gcloud) installed ([how do I install the gcloud CLI?](https://cloud.google.com/sdk/docs/install)).
* Be logged in to the [gcloud CLI](https://cloud.google.com/sdk/gcloud) ([how do I log in?](https://cloud.google.com/sdk/docs/authorizing#user-account))\
  (`gcloud auth login` & `gcloud auth application-default login`)
* Have a Google Cloud user account with at least the minimum required permissions to install Burla.\
  Or: Just run `burla install`, if you're missing any permissions it will tell you which ones!

Here are three sets of permissions, each of which would authorize somebody to run `burla install`:

<details>

<summary>Exact minimum required permissions</summary>

1. Service Usage API (`serviceusage.googleapis.com`):
   * `serviceusage.services.enable` for enabling:
     * `compute.googleapis.com`
     * `run.googleapis.com`
     * `firestore.googleapis.com`
     * `cloudresourcemanager.googleapis.com`
     * `secretmanager.googleapis.com`
2. Compute Engine API (`compute.googleapis.com`):
   * `compute.firewalls.create`
   * `compute.firewalls.get` (to check if firewall rule exists)
   * `compute.networks.updatePolicy`
3. Secret Manager API (`secretmanager.googleapis.com`):
   * `secretmanager.secrets.create`
   * `secretmanager.secrets.get`
   * `secretmanager.versions.add`
4. Firestore API (`firestore.googleapis.com`):
   * `datastore.databases.create`
   * `datastore.databases.get`
   * `datastore.documents.create`
   * `datastore.documents.write`
5. Cloud Run API (`run.googleapis.com`):
   * `run.services.create`
   * `run.services.update`
   * `run.services.get`
   * `run.services.setIamPolicy` (for --allow-unauthenticated flag)

Here is an IAM role definition for this permission set:

```yaml
title: "Burla Installation Role"
description: "Minimum permissions needed to install Burla"
stage: "GA"
includedPermissions:
- serviceusage.services.enable
- compute.firewalls.create
- compute.firewalls.get
- compute.networks.updatePolicy
- secretmanager.secrets.create
- secretmanager.secrets.get
- secretmanager.versions.add
- datastore.databases.create
- datastore.databases.get
- datastore.documents.create
- datastore.documents.write
- run.services.create
- run.services.update
- run.services.get
- run.services.setIamPolicy
```

</details>

<details>

<summary>Generic role based permissions (less complicated)</summary>

Burla can be installed by users having the following generic roles:

1. Service Usage Admin (`roles/serviceusage.serviceUsageAdmin`)
2. Cloud Run Admin (`roles/run.admin`)
3. Compute Network Admin (`roles/compute.networkAdmin`)
4. Secret Manager Admin (`roles/secretmanager.admin`)
5. Firestore Database Admin (`roles/datastore.owner`)

</details>

<details>

<summary>Simplest possible permissions</summary>

Burla can be installed with either of the following roles:

* Project Owner (`roles/owner`)
* Project Editor (`roles/editor`)

</details>

**Here is exactly what happens when `burla install` is run:**

{% hint style="info" %}
On install, your Google account (the one you are currently logged in to `gcloud` with) is set as the only account authorized to access this new Burla deployment.

To access the deployment you will need prove your identity by signing in to this same Google account through a Google sign-in page. Burla follows the standard Google OAuth 2.0 authorization flow.
{% endhint %}

<details>

<summary>Exact installation operations</summary>

1. &#x20;Required services are enabled (if they are not already enabled):
   * `gcloud services enable` is called on:
     * `compute.googleapis.com`
     * `run.googleapis.com`
     * `firestore.googleapis.com`
     * `cloudresourcemanager.googleapis.com`
     * `secretmanager.googleapis.com`
2. Port `8080` is opened on any GCE VM having the tag `burla-cluster-node`:
   * `gcloud compute firewall-rules create burla-cluster-node-firewall \`\
     `--action=ALLOW --rules=tcp:8080 --target-tags=burla-cluster-node ...`
3. &#x20;A secret is created that's used to encrypt auth cookies in the dashboard:
   * `gcloud secrets describe burla-cluster-id-token`&#x20;
   * `gcloud secrets create burla-cluster-id-token ...`&#x20;
4. A Google Cloud Firestore database is created:\
   (manages information displayed in the dashboard)
   * `gcloud firestore databases create --database=burla ...`&#x20;
5. Your Google account (that you are currently logged in to `gcloud` with) is set as the only account authorized to access this new Burla deployment.
   * This account is discovered using the following command:\
     `gcloud auth list --filter=status:ACTIVE --format="value(account)"`
   * Once set, this cannot be changed by other users running `burla install` again.
6. &#x20;The main-service (dashboard) is deployed on Google Cloud Run:
   * `gcloud run deploy burla-main-service \` \
     `--image=burlacloud/main-service:latest ...`
   * `gcloud run services update-traffic burla-main-service --to-latest ...`
7. Thats it!

</details>

We encourage you to check out [\_install.py](https://github.com/Burla-Cloud/burla/blob/main/client/src/burla/_install.py) in the client for even more specific installation details.

**After Installing:**

`burla install` prints the following:

```
Success! To view your new dashboard run `burla login`
Quickstart:
  1. Start your cluster by hitting "⏻ Start" in the dashboard.
  2. Import and call `remote_parallel_map`!
```

***

&#x20;

### `burla login`

Authenticates the current machine through a Google OAuth consent screen.\
Allows you to call `remote_parallel_map` on Burla deployments where you're authorized to do so.

**Description:**

Launches the "sign in with google" page in your default web browser.\
This gives **our** backend access to **only your email and name** according to your google account.\
See our [privacy-policy](privacy-policy.md) to learn how we protect this information.

This is used to ensure that only people you have explicitly authorized have access to your Burla instance.

Once signed-in successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating system's recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

This token is refreshed each time the `burla login`authorization flow is completed.

&#x20;

&#x20;

***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
