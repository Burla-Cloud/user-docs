---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# CLI-Reference

## CLI-Reference

Burla's CLI contains the following commands:

* [`burla install`](CLI-Reference.md#burla-install) Install a self-hosted Burla instance in your Google Cloud project.
* [`burla login`](CLI-Reference.md#burla-login) Authenticate your machine.
* [`burla dashboard`](CLI-Reference.md#burla-dashboard) Open / login to the Burla dashboard in your default browser.

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla install`

Install a self-hosted Burla instance in your Google Cloud project.\
Running `burla install` multiple times will update the existing installation with the latest version.

#### **Description**

Installs Burla inside the Google Cloud project that your [gcloud CLI](https://cloud.google.com/sdk/gcloud) is currently pointing to.\
For a more user-friendly installation guide see: [Installation: Self-Hosted](installation-self-hosted.md)

To view your current project run: `gcloud config get project`\
To change your current project run: `gcloud config set project <desired-project-id>`&#x20;

#### **Prerequisites:**

* Have the [gcloud CLI](https://cloud.google.com/sdk/gcloud) installed ([how do I install the gcloud CLI?](https://cloud.google.com/sdk/docs/install)).
* Be logged in to the [gcloud CLI](https://cloud.google.com/sdk/gcloud) ([how do I log in?](https://cloud.google.com/sdk/docs/authorizing#user-account))\
  (`gcloud auth login`, `gcloud auth application-default login`)
* Have a Google Cloud user account with at least the minimum required permissions to install Burla.\
  (Just run `burla install`, if you're missing any it will tell you which ones!)

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

The easiest way to ensure you have all these permissions is to have one of these roles:

* Project Owner (`roles/owner`)
* Project Editor (`roles/editor`)

</details>

**Here is exactly what happens when `burla install` is run:**

We encourage you to check out [\_install.py](https://github.com/Burla-Cloud/burla/blob/main/client/src/burla/_install.py) in the client for even more specific details.

<details>

<summary>Exact installation operations</summary>

1. &#x20;Required services are enabled (if they are not already enabled):
   * `gcloud services enable` is called on:
     * `compute.googleapis.com`
     * `run.googleapis.com`
     * `firestore.googleapis.com`
     * `cloudresourcemanager.googleapis.com`
     * `secretmanager.googleapis.com`
2. Port 8080 is opened on any GCE VM having the tag "burla-cluster-node":
   * `gcloud compute firewall-rules create burla-cluster-node-firewall \`\
     `--action=ALLOW --rules=tcp:8080 --target-tags=burla-cluster-node ...`
3. &#x20;A secret is created that's used to encrypt auth cookies in the dashboard and identify this instance.
   * `gcloud secrets describe burla-cluster-id-token`&#x20;
   * `gcloud secrets create burla-cluster-id-token ...`&#x20;
4. A Google Cloud Firestore database is created (stores information displayed in the dashboard)
   * `gcloud firestore databases create --database=burla ...`
5. &#x20;The main-service (dashboard) is deployed on Google Cloud Run:
   * `gcloud run deploy burla-main-service \` \
     `--image=burlacloud/main-service:latest ...`
   * `gcloud run services update-traffic burla-main-service --to-latest ...`

</details>

#### **After Installing:**

`burla install` prints the following:

<pre><code>Success! To view your new dashboard run `burla dashboard`
Quickstart:
<strong>  1. Start your cluster by hitting "⏻ Start" in the dashboard.
</strong>  2. Import and call `remote_parallel_map`!
</code></pre>

### `burla login`

Ensures only authorized users can run interact with your Burla instance.

#### **Description**

Launches the "sign in with google" page in your default web browser.\
This gives **our** backend access to **only your email and name** according to your google account.\
See our [privacy-policy](privacy-policy.md) to learn how we protect this information.

This is used to ensure that only people you have explicitly authorized have access to your Burla instance.

Once signed-in successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating system's recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

This token is refreshed each time the `burla login` or `burla dashboard` authorization flow is completed.

### `burla dashboard`&#x20;

Launch and login to the burla dashboard associated with the current Google Cloud project.

#### **Description**

Runs the same OAuth authorization flow used in the `burla login` command, but redirects the user to their current project's Burla dashboard, instead of the [login success](https://docs.burla.dev/auth-success) page.

The current project's Burla dashboard URL is discovered using the following command:\
`gcloud run services describe burla-main-service ...`&#x20;

When redirecting to this dashboard the client attaches an authentication cookie identifying the user to the dashboard. Only explicitly authorized users are allowed to view the dashboard.

Like the `burla login` command, this command also updates local authorization credentials stored in `burla_credentials.json`, see the [login command](CLI-Reference.md#burla-login) documentation for more info on these credentials.





***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
