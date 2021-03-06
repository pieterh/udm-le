# Let's Encrypt for Ubiquiti UbiOS firmwares

## Overview

This should work on UbiOS based firmware versions 1.7.0 onwards. This includes:

* UniFi Dream Machine
* UniFi Dream Machine Pro

This script supports issuing LetsEncrypt certificates via DNS using [Lego](https://go-acme.github.io/lego/).

Out of the box, it has support for AWS Route53 and Cloudflare DNS providers, and with a bit of work you could
get it working with any of the supported [Lego DNS Providers](https://go-acme.github.io/lego/dns/).

## Installation

1. Copy the contents of this repo to your device at `/mnt/data/udm-le`.
2. Edit `udm-le.env` and tweak variables to meet your needs.
3. Run `/mnt/data/udm-le/udm-le.sh initial`. This will handle your initial certificate generation and setup a cron task at `/etc/cron.d/udm-le` to attempt certificate renewal each morning at 0300.

## Persistance

On firmware updates, the cron file (`/etc/cron.d/udm-le`) gets removed, so if you'd like for this to persist between upgrades, I suggest so you install boostchicken's [on-boot-script](https://github.com/boostchicken/udm-utilities/tree/master/on-boot-script) package.

This script is setup such that if it determines that on-boot-script is enabled, it will set up an additional script at `/mnt/data/on_boot.d/99-udm-le.sh` which will attempt certificate renewal shortly after a reboot (and subsequently set the cron back up again).

## DNS Providers

### AWS Route53

AWS Route53 DNS challenge can use configuration and authentication values easily through shared credentials and configuration files [as described here](https://go-acme.github.io/lego/dns/route53/). This script will check for and include these files during the initial certificate generation and subsequent renewals. Ensure that `route53` is set for `DNS_PROVIDER` in `udm-le.env`, create a new directory called `.aws` in `/mnt/data/udm-le` and add `credentials` and `config` files as required for your authentication. See the [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) for more information. Currently only the `default` profile is supported.

### GCP Cloud DNS

GCP Cloud DNS can be configured by establishing a service account with the role [`roles/dns.admin`](https://cloud.google.com/iam/docs/understanding-roles#dns-roles) and exporting a [service account key](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) for that service account. Ensure that `gcloud` is set for `DNS_PROVIDER` in `udm-le.env`, and `GCE_SERVICE_ACCOUNT_FILE` references the path to the service account key (e.g. `./root/.gcloud/my_service_account.json`) . Create a new directory called `.gcloud` in `/mnt/data/udm-le` and add the service account file.

### Cloudflare

In your Cloudflare account settings, create an API token with the following permissions:

* Zone > Zone > Read
* Zone > DNS > Edit

Once you have your token generated, add the value to `udm-le.env`.
