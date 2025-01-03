---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# CLI-Reference

## CLI-Reference

#### Burla CLI Reference

**Description**

Currently the only purpose of Burla's CLI is to provide the ability to authenticate with our backend using the command: `burla login`.

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla login`

(currently required for all users, requires a google account)

**Description**

Launches the "sign in with google" page in your default web browser.\
This gives our backend access only to your email and name from your google account.\
See our [privacy-policy](privacy-policy.md) to learn how we protect this information.

Once signed-in successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating systems recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

We currently require login because the client is hardcoded to only call our free public cluster ([cluster.burla.dev](https://cluster.burla.dev)) and we want to track who is doing what there in order to prevent abuse.\
This requirement will probably change once we add the ability to self-host Burla.

This token is refreshed each time the `burla login` authorization flow is completed.







***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
