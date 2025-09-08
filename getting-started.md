---
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

# Getting Started

## Quickstart (Managed Burla instance)

{% hint style="info" %}
Don't have a managed instance? Send me an email and we'll get you one today! (jake@burla.dev)

You should have received an email with a link to your custom deployment.\
Something like: `https://<yourname>.burla.dev`&#x20;
{% endhint %}

1. Navigate to your custom link in the browser, and sign in using the email we've authorized.
2. Hit the **⏻ Start** button to boot 1000 CPUs! (should take 1-2 min)
3. While booting, run the following in your local terminal:
   1. `pip install Burla`
   2. `burla login`  (connects your computer to the cluster)
4. Once booted, run some code!

```python
# Each call to `compute_square` runs in parallel in it's own separate contianer.
# That's why it finishes quickly even though each function call takes ~1 second.

from time import sleep
from burla import remote_parallel_map

def compute_square(x):

    sleep(1)  # <- pretend this is some intense math!

    print(f"Squaring {x} on a separate computer in the cloud!")
    return x * x

squared_numbers = remote_parallel_map(compute_square, list(range(1000)))
```

5. Celebrate 🎉🎉🎉🎉\
   You just ran Python code on 1000 CPU's in 1000 separate containers.\
   That's not something many people know how to do!

***

## Installation (self-hosted)

{% hint style="info" %}
Self-Hosted Burla is currently exclusive to Google Cloud.

We fully intend to support AWS, Azure, and on-prem deployments, but don't yet.\
We offer [fully-managed Burla deployments](getting-started.md#use-burla-running-in-our-cloud-fully-managed) for those not on GCP.
{% endhint %}

### 1. Ensure `gcloud` is setup and installed:

If you haven't, [install the gcloud CLI](https://cloud.google.com/sdk/docs/install), and [login using application-default credentials](https://cloud.google.com/docs/authentication/set-up-adc-local-dev-environment).

Ensure `gcloud` is pointing at the project you wish to install Burla inside:

* To view your current gcloud project run: `gcloud config get project`
* To change your current gcloud project run: `gcloud config set project <NEW-PROJECT-ID>`

### 2. Run the `burla install` command:

Run `pip install burla` then run `burla install`.

<details>

<summary>What permissions does my Google Cloud account need to run <code>burla install</code> ?</summary>

{% hint style="info" %}
If you don't have permissions, run the command anyway, and it will tell you which ones you need!
{% endhint %}

To run `burla install` you'll need permission to run these `gcloud` commands:

* `gcloud services enable ...`
* `gcloud compute firewall-rules create ...`
* `gcloud secrets create ...`
* `gcloud firestore databases create ...`
* `gcloud run deploy ...`

I've listed the **exact required permissions** for the `burla install` command [in it's CLI doc](API-Reference.md#prerequisites).

</details>

### 3. Start a machine and run the quickstart!

1. Open your new cluster dashboard!
   * Run the command: [`burla login`](API-Reference.md#burla-login) \
     This command will open your dashboard in the default browser. Please login using the same account you used to authenticate `gcloud`, this restriction ensures only you the installer can access your new dashboard.
2. Hit the **⏻ Start** button in the dashboard to turn the cluster on.\
   By default this starts one 4-CPU node. If inactive for >5 minutes this node will shut itself off.
3. Run the example!

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

&#x20;

***

## Installation (fully-managed)

How to use Burla without a Google Cloud account.

{% hint style="info" %}
Fully-Managed deployments are manually created on an individual basis.

Email us (jake@burla.dev)&#x20;
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

&#x20;

&#x20;

***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
