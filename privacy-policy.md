---
description: 'Effective Date: 1/16/26'
hidden: true
---

# Managed Service Privacy Policy

This Privacy Policy explains how Burla, Inc. (“Burla”, “we”, “us”) collects, uses, stores, and protects information when you use **burla.dev** and Burla’s cloud-based software platform (the “Service”).

***

#### 1. Scope

This Privacy Policy applies to information collected through the Service, including user authentication, managed compute instances, storage, billing, and operational metadata.

***

#### 2. Information We Collect

**2.1 Account Information**

When you authenticate using Google OAuth, Burla collects:

* Name
* Email address

This information is used solely for account identification, authentication, and access control.

***

**2.2 Service, Job, and Storage Data**

When you use the Service to run workloads, Burla collects limited information necessary to operate and support the Service, including:

* Job output logs
* Run duration
* Docker image reference
* Identity of the user who initiated the job
* Inputs submitted to the job
* Machine and execution metadata

Burla also stores data that users intentionally add to Burla’s network-attached filesystem. This data:

* Is stored in a dedicated Google Cloud Storage bucket
* Resides within the customer’s private, isolated Google Cloud environment
* Is logically isolated from other customers

When a user deletes data through the Service, Burla deletes the corresponding data from storage without undue delay.

If a customer cancels the Service, Burla retains stored filesystem data for thirty (30) days to allow the customer to download or retrieve their data. After this period, the data is permanently deleted.

***

**2.3 Billing Information**

Payments are processed by third-party payment processors such as Stripe. Burla does not store full payment card numbers.

Burla may receive and store limited billing-related records, such as subscription status, invoices, and payment confirmations, for accounting, tax, and compliance purposes.

***

#### 3. Information We Do Not Store

Burla may process customer code to execute jobs but does not retain customer code in persistent storage after execution.

Jobs are executed on ephemeral virtual machines. Data stored on these machines does not persist after the machines are terminated.

Any data explicitly written by the customer to external or customer-controlled storage systems is retained according to the configuration and policies of those systems and is not controlled by Burla.

***

#### 4. Managed Instances and Infrastructure

Each customer is provided a managed instance hosted within Burla’s virtual private cloud (VPC) on the Google Cloud Platform. Managed instances are logically isolated from one another.

All customer data stored by Burla is encrypted at rest and stored in private databases and storage systems hosted on Google Cloud.

***

#### 5. Access to Managed Instances

Burla personnel do not access customer managed instances as part of normal operations.

Access to a managed instance may occur only to diagnose or resolve service disruptions or to provide customer-requested support.

Burla will obtain customer consent prior to accessing a managed instance, except where access is required to address urgent service availability or security issues.

***

#### 6. How We Use Information

Burla uses collected information to:

* Provide, operate, and maintain the Service
* Authenticate users and manage access controls
* Execute jobs and manage storage
* Monitor system performance and reliability
* Provide customer support
* Communicate service-related updates
* Comply with legal obligations

Burla does not use customer data for advertising or unrelated analytics.

***

#### 7. Data Retention

Burla retains personal information, service metadata, and stored customer data only for as long as necessary to provide the Service, maintain system integrity and security, and meet legal, accounting, and compliance obligations.

***

#### 8. Sharing of Information

Burla does not sell personal information.

Burla may share information only with service providers that support operation of the Service, including cloud infrastructure and payment processors, or when required by law, regulation, or valid legal process.

***

#### 9. Security

Burla implements reasonable technical and organizational measures designed to protect information from unauthorized access, loss, misuse, or disclosure.

No method of transmission or storage is completely secure, and Burla cannot guarantee absolute security.

***

#### 10. Your Rights

You may contact Burla at any time to request deletion of personal information, including name and email address, or to request deletion or minimization of certain job metadata, where feasible.

Requests will be handled in accordance with applicable law and operational requirements.

***

#### 11. Changes to This Policy

Burla may update this Privacy Policy from time to time. Updates will be effective upon posting, with the effective date revised accordingly.

***

#### 12. Contact Information

If you have any questions or concerns regarding this Privacy Policy, please contact:

**jake@burla.dev**

