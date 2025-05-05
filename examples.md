# Examples

## Quickstart:

{% hint style="info" %}
Hello! As you may be aware Burla is still a very new project, and we really don't want anyone to have a bad experience. We REALLY recommend you [schedule a quick call with us](http://cal.com/jakez), or [sign up here](https://www.burla.dev/copy-of-demo-1), so we can provice as much help as possible!
{% endhint %}

_This currently assumes you have a google cloud account and are setup with the `gcloud` CLI_

1. Run `pip install Burla`
2. Run `burla install`
3. Follow the link to your cluster dashboard and start the cluster
4. Run `burla login`, to authenticate with any google account.
5. Run the following code, or something similar!

```python
from burla import remote_parallel_map

my_arguments = [1, 2, 3, 4]

def my_function(my_argument: int):
    print(f"Running on remote computer #{my_argument} in the cloud!")
    return my_argument * 2
    
results = remote_parallel_map(my_function, my_arguments)

print(f"return values: {list(results)}")
```

## Other Examples:



<table data-card-size="large" data-column-title-hidden data-view="cards" data-full-width="false"><thead><tr><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td>Analyze every notebook on kaggle.com</td><td><a href=".gitbook/assets/Google_Colaboratory_SVG_Logo.svg.png">Google_Colaboratory_SVG_Logo.svg.png</a></td><td><a href="https://colab.research.google.com/drive/1A8reU23sdN8HRvaOuDlPunZz_XJ36rN6?usp=sharing">https://colab.research.google.com/drive/1A8reU23sdN8HRvaOuDlPunZz_XJ36rN6?usp=sharing</a></td></tr><tr><td>IDATs to PGEN Biotech Pipeline Demo</td><td><a href=".gitbook/assets/Google_Colaboratory_SVG_Logo.svg.png">Google_Colaboratory_SVG_Logo.svg.png</a></td><td><a href="https://colab.research.google.com/drive/1Qza09HuIC8ZC8O7IO4erNlbo_chvfu0a?usp=sharing">https://colab.research.google.com/drive/1Qza09HuIC8ZC8O7IO4erNlbo_chvfu0a?usp=sharing</a></td></tr></tbody></table>









***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
