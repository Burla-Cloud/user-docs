---
description: How to use Burla without a Google Cloud account.
---

# Installation: Fully-Managed

{% hint style="info" %}
Currently, Fully-Managed deployments are manually created on an individual basis.

We fully intend to make this process 100% self-serve, but haven't yet.\
For now, simply email us!
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

