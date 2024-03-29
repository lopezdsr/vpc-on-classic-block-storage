---

Copyright:
  years: 2019
lastupdated: "2019-06-14"

keywords: block storage, IBM Cloud, VPC, virtual private cloud, Key Protect, encryption, key management, Hyper Protect Crypto Services, HPCS, volume, data storage, virtual server instance, instance

subcollection: vpc-on-classic-block-storage


---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:important: .important}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}


# Creating block storage volumes with customer managed encryption
{: #block-storage-encryption}

By default, {{site.data.keyword.block_storage_is_short}} boot and data volumes are encrypted with IBM-managed encryption. You can also create customer-managed encrypted volumes by using a supported key management service to create or import your customer root key. Customer-managed encryption is an option for boot and data volumes created during instance provisioning.  You can also specify customer-managed encryption when you create a standalone data volume.  

## Key managenment servcies for block storage volumes
{: #key-mgt-services-for-storage}

Two key management services are available, {{site.data.keyword.keymanagementserviceshort}} and {{site.data.keyword.hscrypto}} (available in certain [regions](/docs/services/hs-crypto?topic=hs-crypto-regions#regions)). {{site.data.keyword.cloud_notm}} data centers provide a dedicated hardware security module (HSM) to protect your keys. The following table provides information about each service.

| Key Management Service | HSM Encryption Certification |
| ----- | ----- |
| [{{site.data.keyword.keymanagementserviceshort}}](/docs/services/key-protect/concepts?topic=key-protect-getting-started-tutorial#getting-started-tutorial) | FIPS 140-2 *Level 2* compliance |
| [{{site.data.keyword.hscrypto}}](/docs/services/hs-crypto?topic=hs-crypto-get-started#get-started) | FIPS 140-2 *Level 4* compliance |

## Prerequisites
{: #custom-managed-vol-prereqs}

To create block storage volumes with customer-managed encryption, you must first provision a key management service and create or import your customer root key.
You must also authorize access between Cloud Block Storage and the key management service. When you've completed these prerequisites, you can start creating block storage volumes that use customer-managed encryption.

The following steps are specific to {{site.data.keyword.keymanagementserviceshort}}, but the general flow also applies to {{site.data.keyword.hscrypto}}.  If you're using {{site.data.keyword.hscrypto}}, see the [{{site.data.keyword.hscrypto}} information](/docs/services/hs-crypto?topic=hs-crypto-get-started#get-started) for corresponding instructions.
{:note}

1. Provision the [{{site.data.keyword.keymanagementserviceshort}}](/docs/services/key-protect?topic=key-protect-provision#provision) service.

   Provisioning a new {{site.data.keyword.keymanagementserviceshort}} service instance ensures that it includes the most recent updates that are required for customer managed encryption of block storage volumes. {{site.data.keyword.keymanagementserviceshort}} instances created in 2019 include the extension that is required to support customer managed encryption.
   {: tip}

2. [Create](/docs/services/key-protect?topic=key-protect-create-root-keys#create-root-keys) or
[import](/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys) a customer root key (CRK) in
{{site.data.keyword.keymanagementservicelong_notm}}.
3. From IBM {{site.data.keyword.iamshort}} (IAM), [authorize access](/docs/iam?topic=iam-serviceauth#serviceauth) between **Cloud Block Storage** (source service) and **{{site.data.keyword.keymanagementserviceshort}}** (target service).

## Creating customer managed encrypted data volumes using the UI
{: #data-vol-encryption-ui}

You can specify customer-managed encryption when you create a new block storage volume during instance provisioning or as a standalone volume.

To create an encrypted volume when you create a virtual server instance, see [Creating virtual server instances with customer-managed encryption](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-instance-creating-instances-byok).

To specify customer-managed encryption when creating a standalone volume, follow these steps:

1. In {{site.data.keyword.cloud_notm}} console, navigate to **Menu icon ![Menu icon](../../icons/icon_hamburger.svg) > VPC Infrastructure > Storage > Block storage volumes**.
A list of all block storage volumes displays.
1. Select **New volume**. On the New block storage volume page, update the fields in the Encryption section.
1. On the **New block storage volume** page, update the fields in the **Encryption** section. See the following table for more information. When your changes are complete, click **Create Volume**.

| Field | Value |
| ----- | ----- |
| Encryption | _Provider managed_ is the default encryption mode. To use customer-managed encryption, select a key management service ({{site.data.keyword.keymanagementserviceshort}} or {{site.data.keyword.hscrypto}}). The key management service instance should include the customer root key that you want to use for customer managed encryption. |
| Encryption service instance | If you have multiple key management service instances provisioned in your account, select the one that includes the customer root key that you want to use for customer-managed encryption. |
| Key name | Select the data encryption key within the {{site.data.keyword.keymanagementserviceshort}} instance that you want to use for encrypting the volume. |
| Key ID | Displays the key ID that is associated with the data encryption key that you selected. |
{: caption="Table 1. Values for customer managed encryption" caption-side="top"}

## Creating customer-managed encrypted data volumes using the CLI
{: #data-vol-encryption-cli}

To create a block storage volume with customer-managed encryption using the CLI, specify the ```ibmcloud is volume-create``` command with the `--encryption-key` parameter:

```bash
ibmcloud is volume-create VOLUME_NAME PROFILE_NAME ZONE_NAME [--encryption-key ENCRYPTION_KEY] [--capacity CAPACITY] [--iops IOPS] [--resource-group-id RESOURCE_GROUP_ID | --resource-group-name RESOURCE_GROUP_NAME] [--json]
```

The `--encryption-key` parameter takes the CRN of the root key. Obtain the CRN of the root key in your key management service instance. For information, see the [virtual server instance documentation on customer-managed encryption](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-creating-instances-byok#provision-byok-cli). For information on creating block storage volumes using the CLI, see  [Creating block storage volumes using the CLI](/docs/vpc-on-classic-block-storage?topic=vpc-on-classic-block-storage-creating-block-storage-cli).

The following example shows a new volume created with customer-managed encryption.

```bash
$ ibmcloud is volume-create demo_volume custom us-south-1 --iops 1000 --encryption-key abccorp-kp-vpc-2 5437644a-c4b1-447f-9646-b1a2a4df61382
Creating volume demovolume in resource group Default under account VPC 01 as user rtuser1@mycompany.com...
ID                                      933c8781-f7f5-4a8f-8a2d-3bfc711788ee
Name                                    demo_volume
Capacity                                100
IOPS                                    1000
Profile                                 custom
Encryption Key                          crn:v1:bluemix:public:kms:us-south:a/8d65fb1cf5e99e86dd7229ddef9e5b7b:b1abf7c5-381d-4f34-836e-5db7193250bc:key:fd57250e-908c-4785-a8a5-1f53176bcd2f
Encryption                              customer_managed
Status                                  pending
Resource Group                          Default(dbb12715c2a22f2bb60df4ffd4a543f2)
Created                                 2018-09-20 10:09:28
Zone                                    us-south-1
Volume Attachment Instance Reference    none
```
{:screen}

You can also create volumes with customer-managed encryption during instance provisioning.  For information, see [Using the CLI to provision instances and volumes with customer managed encryption](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-creating-instances-byok#provision-byok-cli).

## Editing boot volumes to use customer-managed encryption using the UI

When you create an instance from the UI, you can specify customer-managed encryption by editing the boot volume properties. For information, see [Provisioning virtual server instances with volumes that use customer managed encryption](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-storage#provision-byok-ui).
