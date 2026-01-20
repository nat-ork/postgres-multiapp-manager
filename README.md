<h1 align="center">postgres-multiapp-manager</h1>

<p align="center">
  Manage multiple PostgreSQL applications on one server with confidence.
</p>

<p align="center">
  <strong>Isolated clusters · Mandatory backups · Safe PITR · Safe Branching</strong>
</p>

<p align="center">
  <em>No cloud services. No magic. Just PostgreSQL done properly.</em>
</p>

---

## 🚀 What This Is

**postgres-multiapp-manager** provides a small set of opinionated CLI tools  
to safely run **multiple isolated PostgreSQL applications** on a single  
Ubuntu / Debian server.

Each app gets its own:

- PostgreSQL cluster  
- backup timeline  
- recovery history  
- optional child branches (for staging / testing)

Failures stay isolated.  
Restores are predictable.  
Branching is safe and explicit.

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
- independent backups, restores, and branches  

This makes **failure isolation, backups, recovery, and branching predictable**.

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
- promotes only if required  
- leaves old data intact for rollback  

No silent data loss.  
No half-restored clusters.

---

### 4️⃣ Branching Is Explicit & Isolated

Branches are **full PostgreSQL clusters**, not replicas.

- One-way derived from a parent cluster
- Parent is never modified
- Each branch has:
  - its own data directory
  - its own port
  - its own pgBackRest stanza
  - its own PITR timeline

Branching is **safe, destructive only to the branch, and reversible**.

---

## 🧰 What This Repo Provides

### `initcluster`

Creates a **production-ready PostgreSQL cluster** for one app.

Guarantees:

- consistent naming
- correct WAL configuration
- pgBackRest stanza creation
- first full backup
- PITR readiness

---

### `pitr`

Performs a **safe Point-in-Time Restore** for one app cluster.

Guarantees:

- correct cluster selection
- correct port targeting
- restore to an exact timestamp
- promotion only when required
- recovery state verification
- old data preserved automatically

---

### `pgbranch`

Manages **safe PostgreSQL cluster branching**.

#### Branch creation
- interactive parent selection
- automatic child naming (`app_dev`)
- new isolated cluster creation
- restore from parent via pgBackRest
- new stanza + first backup
- branch always ends writable

#### Branch sync
- one-way sync: **parent → branch**
- parent is never touched
- branch data preserved before sync
- restore from parent
- success explicitly verified

---

## 📦 Requirements

This project is intentionally opinionated.

### Supported environment

- Ubuntu / Debian-based Linux
- PostgreSQL installed via `apt`
- `pg_createcluster` available
- pgBackRest installed
- `systemd`
- `sudo` access

### Not designed for

- Docker-only workflows  
- RHEL or non-Debian distros  
- Managed cloud PostgreSQL  
- Multi-tenant shared DB designs  

---

## ⚙️ Installation

### 1️⃣ Clone the repository

```bash
git clone https://github.com/nat-ork/postgres-multiapp-manager
cd postgres-multiapp-manager


2️⃣ Install the CLI tools

Copy scripts into your PATH:

sudo cp initcluster /usr/local/bin/initcluster
sudo cp pitr        /usr/local/bin/pitr
sudo cp pgbranch    /usr/local/bin/pgbranch


Make them executable:

sudo chmod +x /usr/local/bin/initcluster
sudo chmod +x /usr/local/bin/pitr
sudo chmod +x /usr/local/bin/pgbranch

3️⃣ Verify installation
initcluster --help
pitr --help
pgbranch --help

▶️ Usage
Create a new app cluster
sudo initcluster -a app1 -p 5432


What this does:

creates a PostgreSQL cluster named app1

binds it to port 5432

enables WAL archiving

registers a pgBackRest stanza

takes the first full backup

makes the cluster PITR-ready

Create another app on the same server:

sudo initcluster -a app2 -p 5433


Each app is fully isolated.

⏪ Perform a Point-in-Time Restore (PITR)

Scenario

Data deleted at: 2026-01-19 10:45 UTC

Restore target: 2026-01-19 10:40 UTC

Run:

sudo pitr -a app1 -t "2026-01-19 10:40:00+00"


What happens:

confirmation prompt

cluster stopped

current data preserved as old-app1-<timestamp>

restore from pgBackRest

cluster started

recovery state verified

🌱 Create a branch (staging / dev)
sudo pgbranch -c


Flow:

select parent cluster interactively

enter branch name (e.g. dev)

branch created as app1_dev

new isolated cluster is created

branch has its own backups and PITR

🔄 Sync a branch with its parent
sudo pgbranch -b app1_dev sync


What happens:

branch is stopped

old branch data preserved

parent data restored into branch

branch started and verified writable

parent is never modified

🧹 Removing a Cluster

To permanently delete a cluster:

sudo pg_dropcluster 16 app1 --stop


This removes:

data directory

configuration

systemd service

pgBackRest backups are not deleted.

⚠️ Do not delete directories manually.

🛡 Safety Guarantees

These tools explicitly prevent:

restoring the wrong cluster

mixing clusters on the same port

running PITR while another cluster is in recovery

syncing data in the wrong direction

silent data loss

half-restored clusters

If something is unsafe, the command fails loudly.

⚖️ Known Design Decisions (Intentional)

PITR is cluster-wide, not database-specific

PostgreSQL version is hard-coded

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
initcluster → creates a universe
pgbranch    → forks or syncs universes
pitr        → rewinds a universe
pg_dropcluster → destroys a universe


Each app lives — and fails — independently.