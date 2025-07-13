---
description: Burla can be self-hosted, or used as a managed service.
---

# Installation

## How to Self-Host Burla:

Install a self-hosted Burla instance inside your private cloud.

{% hint style="info" %}
Self-Hosted Burla is currently exclusive to Google Cloud.

We fully intend to support AWS, Azure, and on-prem deployments, but don't yet.\
We offer [fully-managed Burla deployments](broken-reference) for those not on GCP.
{% endhint %}

### Quick version:

* Simply run `pip install burla` then run `burla install`&#x20;
* If you're missing anything the command will tell you what to do!

### Instructions:

**Ensure `gcloud` is setup and installed:**\
If you haven't, [install the gcloud CLI](https://cloud.google.com/sdk/docs/install), and [login using application-default credentials](https://cloud.google.com/docs/authentication/set-up-adc-local-dev-environment).\
Also, ensure `gcloud` is pointing at the project you wish to install Burla inside:

* To view your current gcloud project run: `gcloud config get project`
* To change your current gcloud project run: `gcloud config set project <NEW-PROJECT-ID>`

**Then install Burla with:**

1. `pip install burla`
2. `burla install`&#x20;

**That's it!**

Burla install requires that your user account have permission to run the following commands:

* `gcloud services enable ...`
* `gcloud compute firewall-rules create ...`
* `gcloud secrets create ...`
* `gcloud firestore databases create ...`
* `gcloud run deploy ...`

If you're missing any permissions, `burla install` will tell you which ones you need!\
To see the exact required IAM permissions, check out the [CLI documentation](broken-reference) for `burla install`.

**Next steps:**

1. Run `burla login` to login to your new cluster dashboard.\
   You will need to login using the same email you used to authenticate `gcloud`. This ensures that only you the installer are allowed to access your new self-hosted Burla instance. To add other users, simpy add their email to the list of authorized users in the settings tab.
2. Hit the **⏻ Start** button in your dashboard to turn the cluster on.\
   By default this will start one 4-CPU node. If inactive for >5 minutes this node will shut itself off.
3. Run the example!

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```



## How to use our managed service:

How to use Burla without a Google Cloud account.

{% hint style="info" %}
Currently, Fully-Managed deployments are manually created on an individual basis.

We fully intend to make this process 100% self-serve, but haven't yet.\
For now, simply email us!
{% endhint %}

### Instructions:

1. E-Mail [jake@burla.dev](https://app.gitbook.com/u/vjhGohhUhsQhYKnFjO0y1B7Ajh82) or schedule a meeting here: [cal.com/jakez](https://cal.com/jakez)
2. I'll reply as quickly as possible with further instructions!

### FAQ:

**Security:**

Managed Burla deployments are created in separate projects each in a separate VPC.\
By default Burla deployments allow access only to those in the authorized-user list in your settings tab, which means we can't actually observe your instance without explicitly manually modifying your database.

**How do I pay?**

We piggyback off of existing Google Cloud billing infrastructure to track costs coming from your instance (Google Cloud project) and simply foreward you the bill through Stripe Billing.

**How much does it cost?**

We charge a simple 2x multiple whatever charges originate from your instance according to Google Cloud billing, this equates to about $0.08 per cpu-hour.

***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
