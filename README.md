# Welcome

### Run any Python script on 1000 computers in 1 second.

With any hardware, in any docker container, self-hosted in your cloud.

{% embed url="https://www.youtube.com/watch?v=1HQkTL-7_VY" %}

#### Use Cases:

<table data-view="cards"><thead><tr><th align="center"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td align="center">Batch AI Inference</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.42.56 PM.png">Screenshot 2025-07-13 at 5.42.56 PM.png</a></td></tr><tr><td align="center">Data Prep</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.40.15 PM.png">Screenshot 2025-07-13 at 5.40.15 PM.png</a></td></tr><tr><td align="center">Computational Bio</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.42.46 PM.png">Screenshot 2025-07-13 at 5.42.46 PM.png</a></td></tr></tbody></table>

#### How it works:

Burla is a python package with only one function:

```python
from burla import remote_parallel_map

def my_function(my_input):
    print("I'm running on my own separate computer in the cloud!")
    
remote_parallel_map(my_function, [1, 2, 3])
```

With Burla, running code in the cloud feels the same as coding locally:

* Anything you print appears your local terminal.
* Exceptions thrown in your code are thrown on your local machine.
* Responses are pretty quick, you can call a million simple functions in a couple seconds.

See our [API-Reference](API-Reference.md) for a list of additional arguments.

#### Join our mailing list:

{% @formspree/formspree %}





***

Questions?\
[Schedule a call with us](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.\
Click [here](https://docs.burla.dev/privacy-policy) to view our privacy policy.



