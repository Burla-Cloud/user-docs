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

#### Burla CLI Reference

**Description**

Burla's CLI provides the ability to:

* authenticate with our backend using the command: `burla login`.
* Install burla in inside your Google Cloud project.

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla login`

Required. Ensures only authorized users can run commands on your cluster.

**Description**

Launches the "sign in with google" page in your default web browser.\
This gives our backend access to **only your email and name** from your google account.\
See our [privacy-policy](privacy-policy.md) to learn how we protect this information.

Once signed-in successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating system's recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

This token is refreshed each time the `burla login` authorization flow is completed.

### `burla install`

Currently, burla only be installed in Google Cloud.

**Description**

Installs burla inside the google cloud project that `gcloud` is currently pointing to.

To view your current project run: `gcloud config get project`\
To change your current project run: `gcloud config set project <desired-project-id>`

When run, `burla install` enables the following google cloud services:

* Compute Engine
* Cloud Run
* Firestore
* Cloud Resource Manager

In order to install burla you will need a Google Cloud account with user or admin level permission on all of these resources. If a permission is missing the error should tell you which permission you'll need.

After enabling, `burla install` runs three `gcloud` commands in the background that:

* Open port 8080 to any VM's having the tag `burla-cluster-node`
* Create a new firestore database called `burla`
* Deploys the latest [Burla-main-service](https://hub.docker.com/repository/docker/jakezuliani/burla_main_service/general) image to google cloud run.

Once installed, simply point your client at the new burla cluster by setting the enviroinment variable `BURLA_API_URL` to the url of the cloud run service that was just deployed. `burla install` will print this url as well a short quickstart when finished.





***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
