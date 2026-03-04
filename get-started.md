# Get Started

There are two ways to host Burla:

1. **In your cloud.**\
   Burla is open-source, and can be deployed with one command (currently Google-Cloud only).\
   [Click here](get-started.md#quickstart-self-hosted) to get started with self-hosted Burla.
2. **In our cloud.**\
   We recommend everyone try Burla in our cloud before self-hosting to make sure it's right for you.\
   [Click here](get-started.md#quickstart-managed-runs-in-our-cloud) to get started with managed Burla. It's free to try out, and only takes 2 minutes!

&#x20;

***

## Quickstart: (Managed) (Runs in our Cloud)

We recommend everyone try Burla in our cloud before self-hosting to make sure it's right for you.\
Burla is free to try out, and only takes 2 minutes to get started. Here's how:

<a href="https://login.burla.dev/signup" class="button primary">Sign Up / Log In</a>

1. ☝️ Sign up using your Google or Microsoft account.
2. Click the **`⏻ Start`** button to boot some computers.
3. Run the example in [this Google Colab notebook](https://colab.research.google.com/drive/1bR8Gpa85gqJi7_9uKdcJDX9_WG0tuVmG?usp=sharing) using 1,000 CPU's!

***

## Quickstart: (Self-Hosted) (Runs in your Cloud)

{% hint style="info" %}
Self-Hosted Burla is currently exclusive to Google Cloud.\
We fully intend to support AWS, Azure, and on-prem deployments, but don't yet.

Please reach out anytime if you get stuck! ([jake@burla.dev](https://app.gitbook.com/u/vjhGohhUhsQhYKnFjO0y1B7Ajh82))
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

See the [install docs](API-Reference.md#burla-install) for more info regarding permissions.

### 3. Start a machine and run some code!

1. Use the **Login** button on this website to get to your new cluster dashboard.
2. Hit the **⏻ Start** button in the dashboard to turn the cluster on.\
   By default this starts one 4-CPU node. If inactive for >5 minutes this node will shut itself off.
3. While booting, run `burla login` to connect your local machine to your cluster.
4. Run the example below!

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3]) 
```

&#x20;

***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
