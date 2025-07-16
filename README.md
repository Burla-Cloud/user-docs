# Welcome

### Run any Python script on 1000 computers in 1 second.

With any hardware, in any docker container, self-hosted in your cloud.

<figure><img src=".gitbook/assets/WebsiteDemo-ezgif.com-video-to-gif-converter.gif" alt=""><figcaption></figcaption></figure>

#### One function, endless possibilities:

<table data-view="cards"><thead><tr><th align="center"></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td align="center">Batch AI Inference</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.42.56 PM.png">Screenshot 2025-07-13 at 5.42.56 PM.png</a></td><td><a href="use-cases/">use-cases</a></td></tr><tr><td align="center">Data Pipelines</td><td><a href=".gitbook/assets/Screenshot 2025-07-16 at 11.41.14 AM.png">Screenshot 2025-07-16 at 11.41.14 AM.png</a></td><td></td></tr><tr><td align="center">Computational Bio</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.42.46 PM.png">Screenshot 2025-07-13 at 5.42.46 PM.png</a></td><td><a href="use-cases/">use-cases</a></td></tr><tr><td align="center">Remote Dev Environments</td><td><a href=".gitbook/assets/Screenshot 2025-07-16 at 12.27.14 PM.png">Screenshot 2025-07-16 at 12.27.14 PM.png</a></td><td></td></tr><tr><td align="center">Data Prep</td><td><a href=".gitbook/assets/Screenshot 2025-07-13 at 5.40.15 PM.png">Screenshot 2025-07-13 at 5.40.15 PM.png</a></td><td><a href="use-cases/">use-cases</a></td></tr><tr><td align="center">Background Task Queue</td><td><a href=".gitbook/assets/Screenshot 2025-07-16 at 11.48.03 AM.png">Screenshot 2025-07-16 at 11.48.03 AM.png</a></td><td></td></tr></tbody></table>

#### How it works:

Burla is an open-source platform for orchestrating parallel Python in the cloud.\
Our package only has one function:

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

#### Deploy in two commands:

Run `pip install burla` then run `burla install` and that's it! (**Currently Google Cloud only!**)

{% embed url="https://docs.burla.dev/getting-started#quickstart" %}

#### &#x20;Any Hardware, Any Container:

Run Python inside any Docker image, just paste a public (or private) URI.

<figure><img src=".gitbook/assets/ScreenRecording2025-07-16at3.46.31PM-ezgif.com-video-to-gif-converter.gif" alt=""><figcaption></figcaption></figure>

&#x20;

#### Stay up to date:

{% @formspree/formspree %}

&#x20;

***

Questions?\
[Schedule a call](http://cal.com/jakez/burla), or email **jake@burla.dev**. We're always happy to talk.
