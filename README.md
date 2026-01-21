# postgres-multiapp-manager

postgres-multiapp-manager is a set of administrative command-line tools for managing
multiple isolated PostgreSQL application clusters on a single Linux host.

The project enforces a strict operational model focused on **isolation, backups,
deterministic recovery, and safe branching**, using native PostgreSQL tooling and
pgBackRest.

---

## Overview

postgres-multiapp-manager provides a controlled way to operate PostgreSQL in
self-hosted environments where multiple applications share a server but must not
share failure domains.

Each application is provisioned as a **fully independent PostgreSQL cluster**, with
its own:

- data directory
- network port
- WAL stream
- pgBackRest stanza
- backup and recovery history

Clusters never share state.  
All destructive operations are explicit and guarded.

---

## Operational Model

### One Application = One Cluster

Each application is deployed as a separate PostgreSQL cluster, not merely as a
database within a shared cluster.

This design ensures:

- point-in-time recovery operates correctly
- failures are isolated
- restores never affect other applications
- backups and WAL streams are independent

---

### Mandatory Backups

All clusters are created with:

- WAL archiving enabled
- a dedicated pgBackRest stanza
- an initial full backup taken automatically

Backups are not optional.  
There is no supported mode without pgBackRest.

---

### Cluster-Wide Recovery

All recovery operations operate at the cluster level.

A restore:

- stops the cluster
- preserves the existing data directory
- restores from pgBackRest
- verifies recovery state
- promotes the cluster when required

Partial or database-level restores are intentionally unsupported.

---

### Explicit Branching

Branches are implemented as **independent PostgreSQL clusters** derived from a parent.

A branch:

- is restored from the parent’s backup
- has its own data directory, port, and pgBackRest stanza
- is writable immediately after creation
- can be safely destroyed or resynced

Parent clusters are never modified during branch operations.

---

## Provided Tools

### `initcluster`

Initializes a new PostgreSQL application cluster.

Responsibilities:
- create a new PostgreSQL cluster
- assign a unique port
- configure WAL archiving
- register a pgBackRest stanza
- take an initial full backup
- ensure PITR readiness

---

### `pitr`

Performs a point-in-time restore for an application cluster.

Responsibilities:
- validate cluster state
- stop the cluster
- preserve current data
- restore to an exact timestamp
- verify recovery state
- promote the cluster if required

---

### `pgbranch`

Manages PostgreSQL cluster branches.

**Branch creation**
- interactive parent selection
- deterministic branch naming
- isolated cluster creation
- restore from parent backup
- independent pgBackRest configuration

**Branch synchronization**
- one-way sync from parent to branch
- branch data preserved before restore
- parent cluster is never touched

---

### `pgbackup`

Triggers manual backups and optional off-host export.

Responsibilities:
- detect active PostgreSQL clusters
- run pgBackRest backups for each cluster
- optionally export the pgBackRest repository to an external location
- never assume repository paths

---

## System Requirements

### Supported Platforms
- Ubuntu or Debian-based Linux
- PostgreSQL installed via `apt`
- `pg_createcluster` tooling available
- pgBackRest installed and configured
- systemd-based service management
- sudo access

### Unsupported Environments
- Docker-only workflows
- Managed cloud PostgreSQL
- Shared multi-tenant clusters
- Non-Debian distributions

---

## Installation

Clone the repository:

```bash
git clone https://github.com/nat-ork/postgres-multiapp-manager
cd postgres-multiapp-manager


Install the tools:

sudo cp initcluster /usr/local/bin/initcluster
sudo cp pitr        /usr/local/bin/pitr
sudo cp pgbranch    /usr/local/bin/pgbranch
sudo cp pgbackup    /usr/local/bin/pgbackup


Set executable permissions:

sudo chmod +x /usr/local/bin/initcluster
sudo chmod +x /usr/local/bin/pitr
sudo chmod +x /usr/local/bin/pgbranch
sudo chmod +x /usr/local/bin/pgbackup


Verify installation:

initcluster --help
pitr --help
pgbranch --help
pgbackup --help

Usage
Creating a Cluster
sudo initcluster -a app1 -p 5432


This creates a fully isolated PostgreSQL cluster named app1 and makes it
immediately PITR-capable.

Point-in-Time Restore
sudo pitr -a app1 -t "2026-01-19 10:40:00+00"


The existing data directory is preserved and recovery is verified before promotion.

Creating a Branch
sudo pgbranch -c


Branches are created interactively and are immediately writable.

Synchronizing a Branch
sudo pgbranch -b app1_dev sync


This operation replaces the branch state with the parent state while preserving
existing branch data.

Manual Backup and Export
pgbackup


Export backups to external storage:

pgbackup -path /mnt/external/pgbackups

Removing a Cluster
sudo pg_dropcluster 16 app1 --stop


This removes the cluster configuration and data directory.
Backups are retained unless explicitly deleted.

Safety Guarantees

The tools are designed to prevent:

accidental cross-cluster restores

port collisions

restores during active recovery

bidirectional or reversed sync operations

silent data loss

partially restored clusters

Unsafe operations fail explicitly.

Design Constraints

The following decisions are intentional:

recovery is cluster-wide

PostgreSQL version is fixed

pgBackRest is mandatory

Ubuntu tooling is assumed

These constraints prioritize predictability and correctness over flexibility.

Conceptual Summary
initcluster     → creates a cluster
pgbranch        → derives or resynchronizes a cluster
pitr            → restores a cluster
pgbackup        → preserves cluster state
pg_dropcluster  → removes a cluster


Each cluster operates as an independent PostgreSQL system with no shared failure
domain.