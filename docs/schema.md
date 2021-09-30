# Chaingraph Schema

This document describes the database schema used by Chaingraph, including the rationale behind various design decisions.

## Overview

Chaingraph stores all chain data in a Postgres SQL database, maintaining consistency between a set of trusted nodes and the Chaingraph database state.

## Data Model

This section details the data model used by Chaingraph, including tables, columns, computed fields, relationships, and indexes.

> This section is derived from the comments/documentation which are built into the SQL schema. Changes to this section should be applied in the database via [SQL migration](../images/hasura/hasura-data/migrations/).

### [TODO: insert exported GraphQL docs]

<!-- TODO: export schema from SQL COMMENTs/Hasura -->

## Mutability

The Chaingraph data model reduces the base table sizes by de-duplicating many common fields.

For example, where other indexers might store `fee_satoshis` in each `transaction` row, Chaingraph uses a stored procedure to compute each `fee_satoshis` value when requested. For applications which commonly require this field, the `transaction_fee_satoshis_index` can be created to effectively pre-compute this value for all transactions. This strategy offers the best of both worlds: applications which don't require fast aggregation of this field save `~2.5GB`, while applications which require fast aggregation can still get equivalent performance to an in-table `fee_satoshis` column.

This field de-duplication strategy can significantly reduce database storage requirements, but to achieve good performance, it requires a strict immutability policy across many tables: if – for example – a single `output` record is deleted, the `transaction` record it references will begin to misreport its `fee_satoshis` value, since the `transaction_fee_satoshis` method subtracts the sum of output values from the sum of input values.

For this reason, the database schema includes triggers which serve to prevent various child records from being deleted without also deleting the parent record. These limitations are important for avoiding data corruption: applications which attempt to delete or mutate records should carefully evaluate if those strategies compromise the integrity of the Chaingraph database.

<!-- TODO: additional triggers to prevent corruption via deletions -->

## Approximate Space Usage

**Last update: March 2, 2021.**

This section provides space usage estimates for all tables and indexes. Statistics are taken from a deployment with one trusted node, syncing only the BCH chain.

Multi-node, multi-chain deployments share common history, so only divergent blocks require additional block space, though independent transaction validation and block acceptance histories may be stored for each node.

[TODO: copy from pgHero]