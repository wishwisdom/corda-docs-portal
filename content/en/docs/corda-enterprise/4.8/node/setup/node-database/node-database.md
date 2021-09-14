---
date: '2020-04-07T12:00:00Z'
menu:
  corda-enterprise-4-8:
    identifier: corda-enterprise-4-8-corda-nodes-configuring-db
    name: "Corda node database"
    parent: corda-enterprise-4-8-corda-nodes-configuring
tags:
- node
- database
title: Corda node database
weight: 1
---

Corda Enterprise Nodes require a relational database for node and CorDapp data storage. This document provides an overview of:

* Supported databases
* Required database permissions
* Creating database schema objects
* Verifying the correct database schema version.

## Supported databases

Corda Enterprise supports the following commercial third-party databases:

* Azure SQL
* SQL Server
* Oracle
* PostgreSQL

For specific versions of supported databases, please see the [platform support matrix](../../../platform-support-matrix.md/)

## Database user permissions

Nodes connect to a database using a single database user and store data within a single database schema - a schema namespace. A database schema can only be shared between nodes that are part of the same hot-cold highly available deployment. Nodes can connect to the database with varying user permissions dependent on how the schema objects were defined. The permissions a node can use are:

* **restricted permissions**, granting the database user access to data manipulation language (DML) execution only (to manipulate data itself e.g. select/delete rows). A database administrator must create database schema objects before running the Corda node. This permission set is recommended for a Corda node in a production environment (including hot-cold-deployment).
* **administrative permissions**, granting the database user full access to the node database, such that it can execute both database definition language (DDL) statements (to define data structures/schema content) and DML queries (to manipulate data itself). This permission set is more permissive and should be used with caution in production environments. A node with full control of the database schema can create or upgrade schema objects automatically upon node startup, and is commonly used in development or testing scenarios.

Database setup for production systems is described in [Database schema setup](node-database-admin.md), and the recommended setup for development/testing environments are described in [Simplified database schema setup for development](node-database-developer.md).

## Database schema objects management

Database DDL scripts defining database tables (and other schema objects) are embedded inside the Corda `.jar` file or within CorDapp distributions. Corda and custom CorDapps are shipped without separate DDL scripts for each database vendor. Use the Corda Database Management Tool to create a DDL script for your database.

The Corda Database Management Tool outputs a compatible DDL script for the Corda release and the database which the tool was run against. A node may be configured to create database tables (and other schema objects) automatically upon startup (and subsequently update tables).

### DDL scripts

DDL scripts are defined in a cross-database syntax and grouped in changesets. At startup, nodes compare the list of change sets recorded in the database with the lists embedded inside the node and associated CorDapps. Depending on the outcome and the node configuration, the node will stop and report any differences or will create/update any missing database objects.

The node and the Corda Database Management Tool use the [Liquibase library/tool](http://www.liquibase.org) for versioning database schema changes.

Liquibase implements an automated, version-based, database migration framework with support for a large number of databases. It works by maintaining a list of applied changesets. A changeset represents any change to the database schema. Each changeset is stored the `DATABASECHANGELOG` table. This changelog table will be read every time a migration command is run to determine what changesets need to be executed. The database schema version is represented by this sum of executed changesets. Each changeset is a script written in xml, yml, or sql, and should never be modified once they have been executed.

Any changes or updates should be applied as new changesets. It is important that you have a good understanding of [Liquibase](https://www.thoughts-on-java.org/database-migration-with-liquibase-getting-started) for understanding how database migrations work in Corda.

### Default Corda node configuration

Nodes do not attempt database migration scripts at startup even if a new changeset is detected. If a new changeset is detected, the node stops and reports the discrepancy. This is to avoid data corruption. Use the Database Management Tool to bring the database to the correct version of the database schema.

Nodes can be configured to automatically run database migration scripts at startup by specifing the `initial registration` subcommand when starting the node. Schema migration can be performed using the `run-migration-script` subcommand of the [node command line](../../node-commandline.md/)

Nodes should only be configured to automatically update database schema when in development or testing environments to protect against data corruption.

### Database Management Tool

The Database Management Tool is distributed as a standalone `.jar` file named `tools-database-manager-${corda_version}.jar`, and should be used to control database schema changes in production
environments.

{{< warning >}}
H2 databases cannot be upgraded using the Database Management Tool.
{{< /warning >}}

You can review all available commands and options in the [Database Management Tool documentation](../../database-management-tool).

## Node database tables

By default, the node database has the following tables:


{{< table >}}

|Table name|Columns|
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|DATABASECHANGELOG|ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, EXECTYPE, MD5SUM, DESCRIPTTION, COMMENTS, TAG, LIQUIBASE, CONTEXTS, LABELS, DEPLOYMENT_ID|
|DATABASECHANGELOGLOCK|ID, LOCKED, LOCKGRANTED, LOCKEDBY|
|NODE_ATTACHMENTS|ATT_ID, CONTENT, FILENAME, INSERTION_DATE, UPLOADER, VERSION|
|NODE_ATTACHMENTS_CONTRACTS|ATT_ID, CONTRACT_CLASS_NAME|
|NODE_ATTACHMENTS_SIGNERS|ATT_ID, SIGNER|
|NODE_CHECKPOINTS|FLOW_ID, STATUS, COMPATIBLE, PROGRESS_STEP, FLOW_IO_REQUEST, TIMESTAMP|
|NODE_CHECKPOINT_BLOBS|FLOW_ID, CHECKPOINT_VALUE, FLOW_STATE, TIMESTAMP|
|NODE_FLOW_RESULTS|FLOW_ID, RESULT_VALUE, TIMESTAMP|
|NODE_FLOW_EXCEPTIONS|FLOW_ID, TYPE, EXCEPTION_MESSAGE, STACK_TRACE, EXCEPTION_VALUE, TIMESTAMP|
|NODE_FLOW_METADATA|FLOW_ID, INVOCATION_ID, FLOW_NAME, FLOW_IDENTIFIER, STARTED_TYPE, FLOW_PARAMETERS, CORDAPP_NAME, PLATFORM_VERSION, STARTED_BY, INVOCATION_TIME, START_TIME, FINISH_TIME|
|NODE_CONTRACT_UPGRADES|STATE_REF, CONTRACT_CLASS_NAME|
|NODE_CORDAPP_METADATA|CORDAPP_HASH, NAME, VENDOR, VERSION|
|NODE_CORDAPP_SIGNERS|CORDAPP_HASH, CORDAPP_SIGNERS|
|NODE_HASH_TO_KEY|PK_HASH, PUBLIC_KEY|
|NODE_IDENTITIES|PK_HASH, IDENTITY_VALUE|
|NODE_IDENTITIES_NO_CERT|PK_HASH, NAME|
|NODE_INFOS|NODE_INFO_ID, NODE_INFO_HASH, PLATFORM_VERSION, SERIAL|
|NODE_INFO_HOSTS|HOST_NAME, PORT, NODE_INFO_ID, HOSTS_ID|
|NODE_INFO_PARTY_CERT|PARTY_NAME, ISMAIN, OWNING_KEY_HASH, PARTY_CERT_BINARY|
|NODE_LINK_NODEINFO_PARTY|NODE_INFO_ID, PARTY_NAME|
|NODE_MESSAGE_IDS|MESSAGE_ID, INSERTION_TIME, SENDER, SEQUENCE_NUMBER|
|NODE_METERING_COMMANDS|ID, COMMAND_HASH, COMMAND_CLASS|
|NODE_METERING_CORDAPPS|ID, STACK_HASH, CORDAPP_HASH, POSITION|
|NODE_METERING_DATA|TIMESTAMP, SIGNING_ID, TRANSACTION_TYPE, CORDAPP_STACK_ID, COMMAND_ID, VERSION, COUNT, IS_COLLECTED|
|NODE_MUTUAL_EXCLUSION|MUTUAL_EXCLUSION_ID, MACHINE_NAME, PID, MUTUAL_EXCLUSION_TIMESTAMP, VERSION|
|NODE_NAMED_IDENTITIES|NAME, PK_HASH|
|NODE_NETWORK_PARAMETERS|HASH, EPOCH, PARAMETERS_BYTES, SIGNATURE_BYTES, CERT, PARENT_CERT_PATH|
|NODE_OUR_KEY_PAIRS|PUBLIC_KEY_HASH, PRIVATE_KEY, PUBLIC_KEY|
|NODE_PROPERTIES|PROPERTY_KEY, PROPERTY_VALUE|
|NODE_RPC_AUDIT_DATA|USERNAME, INTERFACE, ACTION, PARAMETERS, INVOCATIONTIME, INVOCATIONID, ALLOWED|
|NODE_SCHEDULED_STATES|OUTPUT_INDEX, TRANSACTION_ID, SCHEDULED_AT|
|NODE_TRANSACTIONS|TX_ID, TRANSACTION_VALUE, STATE_MACHINE_RUN_ID, STATUS, TIMESTAMP|
|PK_HASH_TO_EXT_ID_MAP|EXTERNAL_ID, PUBLIC_KEY_HASH|
|STATE_PARTY|OUTPUT_INDEX, TRANSACTION_ID, PUBLIC_KEY_HASH, X500_NAME|
|VAULT_FUNGIBLE_STATES|OUTPUT_INDEX, TRANSACTION_ID, ISSUER_NAME, ISSUER_REF, OWNER_NAME, QUANTITY|
|VAULT_FUNGIBLE_STATES_PARTS|OUTPUT_INDEX, TRANSACTION_ID, PARTICIPANTS|
|VAULT_LINEAR_STATES|OUTPUT_INDEX, TRANSACTION_ID, EXTERNAL_ID, UUID|
|VAULT_LINEAR_STATES_PARTS|OUTPUT_INDEX, TRANSACTION_ID, PARTICIPANTS|
|VAULT_STATES|OUTPUT_INDEX, TRANSACTION_ID, CONSUMED_TIMESTAMP, CONTRACT_STATE_CLASS_NAME, LOCK_ID, LOCK_TIMESTAMP, NOTARY_NAME, RECORDED_TIMESTAMP, STATE_STATUS, RELEVANCY_STATUS, CONSTRAINT_TYPE, CONSTRAINT_DATA|
|VAULT_TRANSACTION_NOTES|SEQ_NO, NOTE, TRANSACTION_ID|
|V_PKEY_HASH_EX_ID_MAP|PUBLIC_KEY_HASH, TRANSACTION_ID, OUTPUT_INDEX, EXTERNAL_ID|

{{< /table >}}

For more details, see [Database tables](node-database-tables.md).

The node database for a Simple Notary has additional tables:


{{< table >}}

|Table name|Columns|
|------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|NODE_NOTARY_COMMITTED_STATES|OUTPUT_INDEX, TRANSACTION_ID, CONSUMING_TRANSACTION_ID|
|NODE_NOTARY_COMMITTED_TXS|TRANSACTION_ID|
|NODE_NOTARY_REQUEST_LOG|ID, CONSUMING_TRANSACTION_ID, REQUESTING_PARTY_NAME, REQUEST_TIMESTAMP, REQUEST_SIGNATURE|

{{< /table >}}

The structure of the tables of JPA notaries are described at [Configuring a JPA notary backend](../../notary/installing-jpa.md#configuring-jpa-notary-backend).

The tables for other experimental notary implementations are not described here.


### Database Schema Migration Logging

Database migration logs for Corda internal tables follow a structured format
described in [Database Schema Migration Logging](../../node-database-migration-logging.md#database-schema-migration-logging).
