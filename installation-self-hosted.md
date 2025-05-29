---
description: Install a self-hosted Burla instance inside your private cloud.
---

# Installation: Self-Hosted

{% hint style="info" %}
Self-Hosted Burla is currently exclusive to Google Cloud.

We fully intend to support AWS, Azure, and on-prem deployments, but don't yet.\
We offer [fully-managed Burla deployments](installation-fully-managed.md) for those not on GCP.
{% endhint %}

### Quick version:

* Simply run `pip install burla` then run `burla install`&#x20;
* If you're missing anything the command will tell you what to do!

### Instructions:

**Ensure `gcloud` is setup and installed:**\
If you haven't, [install the gcloud CLI](https://cloud.google.com/sdk/docs/install), and [login using application-default credentials](https://cloud.google.com/docs/authentication/set-up-adc-local-dev-environment).\
Also, ensure `gcloud` is pointing at the project you wish to install Burla inside:

* To view your current project run: `gcloud config get project`
* To change your current project run: `gcloud config set project <desired-project-id>`

**Then install Burla with:**

1. `pip install burla`
2. `burla install`

**That's it!**

Burla install requires that your user account have permission to run the following commands:

* `gcloud services enable ...`
* `gcloud compute firewall-rules create ...`
* `gcloud secrets create ...`
* `gcloud firestore databases create ...`
* `gcloud run deploy ...`

If you're missing any permissions, `burla install` will tell you which ones you need.\
For exact permissions including a custom IAM role, see the [CLI documentation](CLI-Reference.md#burla-install) for `burla install`.

**Next steps:**

1. Run `burla dashboard` to login to your new cluster dashboard.\
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





***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
