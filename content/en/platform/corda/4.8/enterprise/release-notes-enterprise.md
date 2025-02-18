---

date: '2021-07-01'

menu:
  corda-enterprise-4-8:
    identifier: corda-enterprise-4-8-release-notes
    name: "Release notes"
tags:
- release
- notes
- enterprise
title: Corda Enterprise release notes
weight: 1
---


# Corda Enterprise release notes

## Corda Enterprise 4.8.1

Corda Enterprise 4.8.1 is a patch release of Corda Enterprise that fixes a security vulnerability in Corda Enterprise 4.8.

### Upgrade recommendation

As a developer, you should upgrade to the [latest released version of Corda](../../../../../en/platform/corda/4.8/enterprise.html) as soon as possible. The latest Corda Enterprise release notes are on this page, and you can find the latest upgrade guide [here](../../../../../en/platform/corda/4.8/enterprise/upgrading-index.md).

As a node operator, you should upgrade to the [latest released version of Corda](../../../../../en/platform/corda/4.8/enterprise.html) if the fixed issues listed below are relevant to your work.

### Fixed issues

In this patch release:

* Support for Oracle 19C database has been validated.
* A fix has been introduced to reduce memory consumption during batched transaction resolution of large backchains.
* Support has been introduced for RedHat Enterprise Linux 8.x in Corda 4.8.1.
* Performance verification of the HA Notary with CockroachDB 20.2.8 has been carried out - performance is not worse in comparison to the HA Notary configured on CockroachDB 20.1.6.
* Support for PostgreSQL 13.3 for node databases has been added.
* Hibernate ORM has been updated to version to 5.4.32 to remove a security concern.
* The [Node management console](../../../../../en/platform/corda/4.8/enterprise/node/management-console.html#node-management-console) configuration has been updated. Configuration is now set in `node.management.plugin.middleware`, no longer `node.admin.middleware`.
* The [Flow management console](../../../../../en/platform/corda/4.8/enterprise/node/node-flow-management-console.html#flow-management-console) configuration has been updated. Configuration is now set in `flow.management.plugin.middleware`, no longer `flow.admin.middleware`.
* LedgerGraph has been updated to version 1.2.2. This upgrade minimizes memory footprint, and is not a functional change.



## Corda Enterprise 4.8 release overview

Corda Enterprise 4.8, released on April 21st 2021, includes several new features, enhancements, and fixes.

* The [notary database now supports Oracle database version 19c](#notary-database-support-update).
* You can use Azure-managed identities to authenticate [Azure Key Vault HSM](#azure-managed-identities-authentication)s.
* You can configure metrics to use [time-window reservoirs](#time-window-metrics-gathering) for data collection.
* Additional metrics have been added for [tracking notary latency](#additional-notary-metrics).
* Confidential identities support has been added via [Utimaco and Gemalto Luna HSMs](platform-support-matrix.html#hardware-security-modules-hsm).

{{< note >}}
This page only describes functionality specific to Corda Enterprise 4.8. However, as a Corda Enterprise customer, you can also make full use of the features available as part of the Corda open source releases.

See the [Corda open source release notes](../../../../../en/platform/corda/4.8/open-source/release-notes.md) for information about new features, enhancements, and fixes shipped as part of Corda 4.8.
{{< /note >}}

{{< note >}}
You can use states and CorDapps valid in Corda 3.0 and above with Corda 4.8 and Corda Enterprise 4.8.


For the commitment Corda makes to wire and API stability, see [API stability guarantees](../../../../../en/platform/corda/4.8/enterprise/cordapps/api-stability-guarantees.md).
{{< /note >}}

## Long-term support release

[Corda 4.8](../../../../../en/platform/corda/4.8/open-source/release-notes.md) and Corda Enterprise 4.8 are our long-term support (LTS) platform versions.

R3 provides LTS for this release for 30 months starting April 21st 2021. This is 6 months longer than the support periods for previous releases, giving Corda customers extra time to plan for the next upgrade.

## Platform version change

Corda 4.8 uses platform version 10.

For more information about platform versions, see [Versioning](../../../../../en/platform/corda/4.8/enterprise/cordapps/versioning.md).

## New features and enhancements

### Notary database support update

The [JPA notary](../../../../../en/platform/corda/4.8/enterprise/notary/installing-jpa.md) now supports [Oracle DB version 19c](../../../../../en/platform/corda/4.8/enterprise/platform-support-matrix.html#jpa-notary-databases). This database is supported until April 30th 2027.

### Azure managed identities authentication

If you use an Azure Key Vault HSM with Corda Enterprise, you can now use an existing Azure Managed Identities service as authentication.

See [Using an HSM with Corda Enterprise](../../../../../en/platform/corda/4.8/enterprise/node/operating/cryptoservice-configuration.html#azure-keyvault) for more information.

### Time-window metrics gathering

You can now configure timer and histogram metrics to use time-window data gathering. Time-window data gathering collects all data points for a given time window, allowing outlying data points to be properly represented.

See [Node metrics](../../../../../en/platform/corda/4.8/enterprise/node/operating/monitoring-and-logging/node-metrics.md) for more information.

### Additional notary metrics

You can use `StartupQueueTime` and `BatchSignLatency` metrics to help calculate notary latency and assess notary worker performance across a notary cluster.

* `StartupQueueTime` represents the time a flow has been queued before starting, in milliseconds.
* `BatchSignLatency` represents the time elapsed during a batch signature, in milliseconds.

See [Monitoring Notary Latency](../../../../../en/platform/corda/4.8/enterprise/notary/faq/notary-latency-monitoring.md) for more information.


## Fixed issues

Corda Enterprise 4.8 fixes:

* A security issue that affects notary systems that use the JPA notary implementation in an HA configuration and when the notary backing database has been set up using the Corda database management tool. The new version of the Corda [database management tool](../../../../../en/platform/corda/4.8/enterprise/database-management-tool.md) must be re-run for the fix to take effect.
* Several issues that cause memory leaks. As a result, we have added a new node configuration field - `enableURLConnectionCache` - and we have modified the `attachmentClassLoaderCacheSize` node configuration field. See the [node configuration fields page](../../../../../en/platform/corda/4.8/enterprise/node/setup/corda-configuration-fields.html#enterpriseconfiguration) for details.
* An issue where the node is unable to resolve transaction chains that contain states or contracts that it did not relate to installed CorDapps.
* Flow state, invocation source, and suspension source filters breaking in the node GUI.
* Transaction verification being performed outside of the attachments class loader.
* HA utilities not logging messages that state that the master key is not needed when using a native mode HSM.
* HA utilities not logging information about the `freshIdentitiesConfiguration`.
* Log messages incorrectly stating that a confidential identity key has been created.
* An issue that causes the node to hang if shut down using `SIGTERM`.
* Attachment presence cache containing the attachment contents.
* The Corda Firewall throwing an error when retrieving version information.
* HA utilities creating erroneous logs when using confidential identities.

## Known issues

* An issue with the Oracle 12c database that causes the JDBC driver to hang if blocked by an empty entropy pool.

{{< note >}}
This issue is specific to Corda Enterprise 4.8. Known issues relating to other versions of Corda Enterprise are listed in the release notes for each version.
{{< /note >}}
