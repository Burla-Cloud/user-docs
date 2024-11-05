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

Currently the only purpose of Burla's CLI is to provide the ability to authenticate with Burla's cloud, using the command: `burla login.`

The global arg `--help` can be placed after any command or command group to see CLI documentation.

***

### `burla login`

**Authenticate with Burla cloud.**

**Description**

Obtains access credentials for your user account via a web-based (OAuth2) authorization flow.\
When this command completes successfully, an auth-token is saved in the text file `burla_credentials.json`. This file is stored in your operating systems recommended user data directory which is determined using the [appdirs](https://github.com/ActiveState/appdirs) python library.

This auth-token is refreshed each time the `burla login` authorization flow is completed.







***

Questions?\
[Schedule a call with us](https://cal.com/jakez/burla/), or email **jake@burla.dev**. We're always happy to talk.
