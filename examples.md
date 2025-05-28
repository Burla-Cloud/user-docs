# Examples

## Quickstart:

{% hint style="info" %}
This currently assumes you have a Google Cloud account and have setup the `gcloud` CLI.\
For more information, see our installation guides in the right-hand sidebar.
{% endhint %}

1. Run `pip install Burla`
2. Run `burla install` (GCP only)
3. Run `burla dashboard` to login to your new cluster dashboard.
4. Hit the **⏻ Start** button in your dashboard to turn the cluster on.
5. Run the code below!

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

## Other Examples:



<table data-card-size="large" data-column-title-hidden data-view="cards" data-full-width="false"><thead><tr><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td>Analyze every notebook on kaggle.com</td><td><a href=".gitbook/assets/Google_Colaboratory_SVG_Logo.svg.png">Google_Colaboratory_SVG_Logo.svg.png</a></td><td><a href="https://colab.research.google.com/drive/1A8reU23sdN8HRvaOuDlPunZz_XJ36rN6?usp=sharing">https://colab.research.google.com/drive/1A8reU23sdN8HRvaOuDlPunZz_XJ36rN6?usp=sharing</a></td></tr><tr><td>IDATs to PGEN Biotech Pipeline Demo</td><td><a href=".gitbook/assets/Google_Colaboratory_SVG_Logo.svg.png">Google_Colaboratory_SVG_Logo.svg.png</a></td><td><a href="https://colab.research.google.com/drive/1Qza09HuIC8ZC8O7IO4erNlbo_chvfu0a?usp=sharing">https://colab.research.google.com/drive/1Qza09HuIC8ZC8O7IO4erNlbo_chvfu0a?usp=sharing</a></td></tr></tbody></table>









***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
