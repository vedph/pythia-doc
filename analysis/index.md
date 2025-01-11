---
title: Analysis
layout: default
parent: Home
nav_order: 6
---

# Analysis

- [analysis process](analysis)
- [software components](components)
- [integrating UDPipe](udp)
  - [simple example](example)
  - [simple example dump: Catullus](example-dump-1)
  - [simple example dump: Horatius](example-dump-2)

The analysis process collects, preprocesses and indexes the input documents, as specified by the provided Pythia pipeline configuration (which is a simple JSON document).

## Configuring Analysis

To setup your system to build a Pythia index (which in the end is just a standard relational database), you need:

- ▶️ access to a [PostgreSQL](https://www.postgresql.org) service. Typically you can do this via a Docker image in your host, unless you already have PostgreSQL installed locally, or access to some external PostgreSQL server.

>💡 If you need to setup Docker, you can find quick instructions for your OS at <https://vedph.github.io/cadmus-doc/deploy/docker.html>.

- ▶️ the [Pythia CLI tool](../cli). You can 🌐 download its latest version from <https://github.com/vedph/pythia/releases>: just pick the version for your OS among `linux`, `osx` (MacOS), and `win` (Windows), and unzip it into a folder of your choice.

>⚠️ If you are on Linux or MacOS, remember to mark the `pythia` tool file as executable with a command like `chmod +x pythia`.

You will then have to provide your own documents to be indexed, and a Pythia configuration profile (which is just a JSON document) for them.

>⚠️ If the indexing process takes time, and you want to be periodically notified via email on its progress, you should provide into environment variables a pair of public/private MailJet account API keys: please see the [CLI documentation](../cli#index-command) for more.

### Using Docker-Based PostgreSQL

To launch a PostgreSQL service without installing it, any up-to-date PostgreSQL image is fine. You can easily run a container like this (in this sample, I created a folder in my drive at `c:\data\pgsql` to host data outside the container):

```sh
docker run --volume postgresData://c/data/pgsql -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=postgres -d postgres
```

💡 For newcomers, here is how you can quickly dump and restore a database using PostgreSQL client tools:

- ▶️ **backup** database to file (adjust path to dump file accordingly):

```bash
pg_dump --username=postgres -f c:/users/dfusi/desktop/pythia.sql pythia
```

- ▶️ **restore** database from file (adjust path to dump file accordingly):

```bash
psql -U postgres -d pythia -f c:/users/dfusi/desktop/pythia.sql
```

>If you want to directly access the database you can use any tool like e.g. [DBeaver](https://dbeaver.io).
