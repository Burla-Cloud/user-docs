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

(currently required for all users, requires a google account)

**Description**

Launches the "sign in with google" page in your default web browser.\
This gives our backend access only to your email and name from your google account.\
See our [privacy-policy](privacy-policy.md) to learn how we protect this information.

Once signed-in successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating systems recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

We currently require login because the client is hardcoded to only call our public demo cluster ([cluster.burla.dev](https://cluster.burla.dev)) and we want to track who is doing what there in order to prevent abuse.\
This requirement will probably change once we add the ability to easily self-host Burla.

This token is refreshed each time the `burla login` authorization flow is completed.

### `burla install`

(Currently, burla only works with google cloud)

**Description**

Installs burla inside the google cloud project that `gcloud` is currently pointing to.

To view this run `gcloud config get project`\
To modify this run `gcloud config set project <desired-project-id>`

When run, `burla install` enables the following google cloud services:

* Compute Engine
* Cloud Run
* Firestore
* Cloud Resource Manager

In order to install burla you will need a Google Cloud account with user or admin level permission on all of these resources. If a permission is missing the error should tell you which permission you'll need.

After enabling, three gcloud commands are run that:

* Open port 8080 to any VM's having the tag `burla-cluster-node`
* Creates a new firestore database called `burla`
* Deploys [the latest Burla-main-service image](https://hub.docker.com/repository/docker/jakezuliani/burla_main_service/general) to google cloud run.

Once installed this command will print a short quicksrt guide:





***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
