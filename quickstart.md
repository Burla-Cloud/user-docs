# Quickstart

{% hint style="info" %}
This currently assumes you have a Google Cloud account and have setup the `gcloud` CLI.\
For more information, see our installation guides in the sidebar on the left.
{% endhint %}

1. Run `pip install Burla`
2. Run `burla install` (Google Cloud only)
3. Run `burla dashboard` to login to your new cluster dashboard.
4. Hit the **⏻ Start** button in your dashboard to turn the cluster on.
5. Run the code below!

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on remote computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```
