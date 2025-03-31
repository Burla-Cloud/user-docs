---
description: Instructions to install Burla inside your private cloud.
---

# Installation

#### TLDR:

* Simply run `pip install burla` then run `burla install`&#x20;
* If you're missing anything the command will tell you what to do!

### Burla is currently GCP only.

We fully intend to support AWS, Azure, and on-prem deployments, but don't yet.\
E-mail [jake@burla.dev](mailto:jake@burla.dev) if you're an AWS/Azure shop and are really interested in Burla,

[Private managed Burla clusters](installation.md#not-on-gcp-use-a-private-managed-burla-cluster-instead) are available if you're not on GCP and want to use Burla now.

### How to install Burla in GCP:

#### First, ensure `gcloud` is installed and configured correctly:

If you haven't yet, [install the gcloud CLI](https://cloud.google.com/sdk/docs/install), and [login using application-default credentials](https://cloud.google.com/docs/authentication/set-up-adc-local-dev-environment).\
Also, ensure `gcloud` is pointing at the project you wish to install Burla inside:

* To view your current project run: `gcloud config get project`
* To change your current project run: `gcloud config set project <desired-project-id>`

#### Then deploy Burla with:

1. `pip install burla`
2. `burla install`

**That's it!**

Burla install requires that your user account have permission to run the following commands:

* gcloud run deploy ...
* gcloud firestore databases create ...
* gcloud compute firewall-rules create ...
* gcloud services enable ...

If you're missing any permissions, `burla install` will tell you which ones you still need.

Once installed, point your client at your new burla cluster by setting the enviroinment variable `BURLA_API_URL` to the URL of the cloud run service that was just deployed.\
The `burla install` command will print this URL as well a short quickstart when finished.

### Not on GCP? Use a private managed Burla cluster instead

Private deployments are fully managed by us, and are available at your own custom `.burla.dev` domain. If you're interested a private, fully-managed Burla deployment please email me at [jake@burla.dev](https://app.gitbook.com/u/vjhGohhUhsQhYKnFjO0y1B7Ajh82) or schedule a meeting here: [cal.com/jakez](https://cal.com/jakez)

#### Pricing:

We charge a flat 1.5x multiple of our underlying compute cost from your deployment.\
If you spend $100 on compute, we will bill you $150.





***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
