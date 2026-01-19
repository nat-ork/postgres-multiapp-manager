<h1 align="center">postgres-multiapp-manager</h1>

<p align="center">
  Manage multiple PostgreSQL applications on one server with confidence.
</p>

<p align="center">
  <strong>Isolated clusters · Mandatory backups · Safe PITR</strong>
</p>

<p align="center">
  <em>No cloud services. No magic. Just PostgreSQL done properly.</em>
</p>

---

## 🚀 What This Is

**postgres-multiapp-manager** provides a small set of opinionated CLI tools  
to safely run **multiple isolated PostgreSQL applications** on a single  
Ubuntu server.

Each app gets its own:
- PostgreSQL cluster
- backup timeline
- recovery history

Failures stay isolated.  
Restores are predictable.

---

## ❓ Why This Exists

Most self-hosted setups start like this:

- One PostgreSQL server  
- Multiple apps  
- Shared databases  
- Shared users  
- No real backups  
- No safe rollback  

That works… **until it doesn’t**.

When something goes wrong:
- backups are incomplete
- restores are risky
- rollback becomes guesswork

This project enforces a stricter, production-safe model instead.

---

## 🧠 The Core Model

> **One app = one PostgreSQL cluster**

Each application runs in a fully isolated cluster with:

- its own data directory  
- its own port  
- its own WAL stream  
- its own pgBackRest stanza  
- independent backups and restores  

This makes **failure isolation, backups, and recovery predictable**.

---

## 🧩 Core Concepts

### 1️⃣ One App = One Cluster

Each application runs in its **own PostgreSQL cluster**,  
not just its own database.

Why this matters:
- PITR works at the **cluster level**
- Restores never affect other apps
- No cross-app blast radius

---

### 2️⃣ pgBackRest Is Mandatory

All clusters are created with:

- WAL archiving enabled  
- A dedicated pgBackRest stanza  
- A first full backup taken automatically  

PITR is always possible.  
There is no “we’ll add backups later”.

---

### 3️⃣ PITR Is Explicit & Safe

Point-in-Time Restore always:

- stops the cluster  
- preserves current data  
- restores to an exact timestamp  
- promotes the cluster  
- leaves old data intact for rollback  

No silent data loss.  
No half-restored clusters.

---

## 🧰 What This Repo Provides

### `initcluster`

Creates a **production-ready PostgreSQL cluster** for one app.

It guarantees:
- consistent naming
- correct WAL configuration
- pgBackRest stanza creation
- first full backup
- PITR readiness

---

### `pitr`

Performs a **safe Point-in-Time Restore** for one app cluster.

It guarantees:
- correct cluster selection
- correct port targeting
- restore + promotion
- no cluster left in recovery
- old data preserved automatically

---

## 📦 Requirements

This project is intentionally opinionated.

Supported environment:
- Ubuntu / Debian-based Linux
- PostgreSQL installed via `apt`
- `pg_createcluster` available
- pgBackRest installed
- `systemd`
- `sudo` access

Not designed for:
- Docker-only workflows
- RHEL or non-Debian distros
- Managed cloud PostgreSQL

---

## ⚙️ Installation

Clone the repository:
```bash
git clone https://github.com/nat-ork/postgres-multiapp-manager
cd postgres-multiapp-manager



Copy scripts into your PATH:

sudo cp initcluster /usr/local/bin/initcluster
sudo cp pitr /usr/local/bin/pitr


Make them executable:

sudo chmod +x /usr/local/bin/initcluster
sudo chmod +x /usr/local/bin/pitr


Verify:

initcluster --help
pitr --help

▶️ Usage
Create a new app cluster
sudo initcluster -a app1 -p 5432


This will:

create a PostgreSQL cluster named app1

bind it to port 5432

enable WAL archiving

register a pgBackRest stanza

take the first full backup

make the cluster PITR-ready

Create another app on the same server
sudo initcluster -a app2 -p 5433


Each app is fully isolated.

⏪ Performing a Point-in-Time Restore (PITR)
Example scenario

Data deleted at 2026-01-19 10:45 UTC

Restore target: 2026-01-19 10:40 UTC

Run:

sudo pitr -a app1 -t "2026-01-19 10:40:00+00"


What happens:

clear confirmation prompt

cluster is stopped

current data directory renamed to old-app1-<timestamp>

restore from pgBackRest

cluster started and promoted

recovery state verified

🛡 Safety Guarantees

These tools explicitly prevent:

restoring the wrong cluster

mixing clusters on the same port

running PITR while another cluster is in recovery

silent data loss

half-restored clusters

If something is unsafe, the command fails loudly.

🧹 Removing a Cluster

To permanently delete a cluster:

sudo pg_dropcluster 16 app1 --stop


This removes:

data directory

configuration

systemd service

⚠️ Do not delete directories manually.

⚖️ Known Design Decisions (Intentional)

PITR is cluster-wide, not database-specific

PostgreSQL version is hard-coded in scripts

pgBackRest is required

Ubuntu tooling is assumed

These choices keep the system:

predictable

debuggable

production-safe

👥 Who This Is For

Indie founders running multiple apps on one server

Self-hosted production setups

Cost-conscious teams avoiding managed databases

Developers who want real backups, not hope

🚫 Who This Is NOT For

Docker-only workflows

Managed cloud PostgreSQL users

Multi-tenant shared DB designs

Anyone wanting “magic” without understanding Postgres

🧠 Final Mental Model

initcluster creates a universe
pitr rewinds that universe

Each app lives — and fails — independently.