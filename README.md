# AWS IAM Policies for Seqera Platform

This repository provides AWS Identity and Access Management (IAM) policies for using the
Seqera Platform with AWS.

## Choose Your Setup:

There are two ways to configure Seqera Platform with AWS:

* **Seqera Forge (Recommended):** Let Seqera automatically create and manage the AWS Batch
  infrastructure for you. This is the simplest way to get started.
  * **Use the policies in the [`forge/`](./forge) directory.**
* **Manual Setup:** Manually create and manage your own AWS Batch compute environments and
  queues. This gives you more control over the underlying infrastructure.
  * **Use the policies in the [`launch/`](./launch) directory.**

In short: if you are new to Seqera Platform or prefer a simpler setup, use the `forge`
policy. If you have existing AWS Batch infrastructure or specific security requirements, the
`launch` policy might be a better fit.

For more details on each setup, see the `README.md` files in the respective directories.
