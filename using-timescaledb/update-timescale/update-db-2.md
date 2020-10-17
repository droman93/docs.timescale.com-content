# Updating TimescaleDB to 2.0 [](update)

Use these instructions to update TimescaleDB 1.3+ to TimescaleDB 2.0

### Notice of breaking changes from TimescalDB 1.3+
TimescaleDB 2.0 supports **in-place updates** just like previous releases. During the update, scripts will automatically configure
updated features to work as expected with TimescaleDB 2.0.

Because this is our first major version release in two years, however, we’re providing additional guidance to help you ensure the update completes successfully and everything is configured as expected (and optimized for your workload). In particular, settings related to [Continuous Aggregates][caggs], [compression][compression], and [data retention][retension] have been modified to provide greater configuration transparency and flexibility, therefore we highly recommend verifying that these settings were migrated correctly.

Before completing the upgrade, we encourage you to read [Changes in TimescaleDB 2.0][changes-in-ts2] for a more detailed look at the major changes in TimescaleDB 2.0 and how they impact the way your applications and scripts interact with the API.

### Prerequisites
#### PostgreSQL Compatibility
**TimescaleDB 2.0 is not compatible with PostgreSQL 9.6 or 10**. If your current PostgreSQL installation is not at least version 11, please upgrade PostgreSQL first. Whenever possible, prefer the most recent supported version, PostgreSQL 12.

>:TIP: Please see our Updating Postgres guide for help.

#### Fix Continuous Aggregate Errors Before Upgrading
Before starting the upgrade to TimescaleDB 2.0, **we highly recommend checking the database log for errors related to failed retention policies that were occuring in TimescaleDB 1.x** and then either remove them or update them to be compatible with existing continuous aggregates. Any remaining retention policies that are still incompatible with the 'ignore_invalidation_older_than' setting will automatically be disabled during the upgrade and a notice provided.

>:TIP:Read more about changes to continuous aggregates and data retension policies [here][retention-cagg-changes]


### Update TimescaleDB

Software upgrades use PostgreSQL's `ALTER EXTENSION` support to update to the
latest version. TimescaleDB supports having different extension
versions on different databases within the same PostgreSQL instance. This
allows you to update extensions independently on different databases. The
upgrade process involves three-steps:

1. We recommend that you perform a [backup][] of your database via `pg_dump`.
1. [Install][] the latest version of the TimescaleDB extension.
1. Execute the following `psql` command inside any database that you want to
   update:

```sql
ALTER EXTENSION timescaledb UPDATE;
```

>:WARNING: When executing `ALTER EXTENSION`, you should connect using `psql`
with the `-X` flag to prevent any `.psqlrc` commands from accidentally
triggering the load of a previous TimescaleDB version on session startup. 
It must also be the first command you execute in the session. 
<!-- -->

This will upgrade TimescaleDB to the latest installed version, even if you
are several versions behind.

After executing the command, the psql `\dx` command should show the latest version:

```sql
\dx timescaledb

    Name     | Version |   Schema   |                             Description
-------------+---------+------------+---------------------------------------------------------------------
 timescaledb | x.y.z   | public     | Enables scalable inserts and complex queries for time-series data
(1 row)
```

### Example: Migrating docker installations [](update-docker)

As a more concrete example, the following steps should be taken with a docker
installation to upgrade to the latest TimescaleDB version, while
retaining data across the updates.

The following instructions assume that your docker instance is named
`timescaledb`. If not, replace this name with the one you use in the subsequent
commands.

#### Step 1: Pull new image [](update-docker-1)
Install the latest TimescaleDB image:

```bash
docker pull timescale/timescaledb:latest-pg12
```
>:TIP: If you are using PostgreSQL 11 images, use the tag `latest-pg11`.

#### Step 2: Determine mount point used by old container [](update-docker-2)
As you'll want to restart the new docker image pointing to a mount point
that contains the previous version's data, we first need to determine
the current mount point.

There are two types of mounts. To find which mount type your old container is
using you can run the following command:
```bash
docker inspect timescaledb --format='{{range .Mounts }}{{.Type}}{{end}}'
```
This command will return either `volume` or `bind`, corresponding
to the two options below.

1. [Volumes][volumes] -- to get the current volume name use:
```bash
$ docker inspect timescaledb --format='{{range .Mounts }}{{.Name}}{{end}}'
069ba64815f0c26783b81a5f0ca813227fde8491f429cf77ed9a5ae3536c0b2c
```

2. [Bind-mounts][bind-mounts] -- to get the current mount path use:
```bash
$ docker inspect timescaledb --format='{{range .Mounts }}{{.Source}}{{end}}'
/path/to/data
```

#### Step 3: Stop old container [](update-docker-3)
If the container is currently running, stop and remove it in order to connect
the new one.

```bash
docker stop timescaledb
docker rm timescaledb
```

#### Step 4: Start new container [](update-docker-4)
Launch a new container with the updated docker image, but pointing to
the existing mount point. This will again differ by mount type.

1. For volume mounts you can use:
```bash
docker run -v 069ba64815f0c26783b81a5f0ca813227fde8491f429cf77ed9a5ae3536c0b2c:/var/lib/postgresql/data -d --name timescaledb -p 5432:5432 timescale/timescaledb
```

2. If using bind-mounts, you need to run:
```bash
docker run -v /path/to/data:/var/lib/postgresql/data -d --name timescaledb -p 5432:5432 timescale/timescaledb
```


#### Step 5: Run ALTER EXTENSION [](update-docker-5)
Finally, connect to this instance via `psql` (with the `-X` flag) and execute the `ALTER` command
as above in order to update the extension to the latest version:

```bash
docker exec -it timescaledb psql -U postgres -X

# within the PostgreSQL instance
ALTER EXTENSION timescaledb UPDATE;
```

You can then run the `\dx` command to make sure you have the
latest version of TimescaleDB installed.

[upgrade-pg]: /using-timescaledb/update-timescale/upgrade-pg
[update-db-1]: /using-timescaledb/update-timescale/update-db-1
[update-db-2]: /using-timescaledb/update-timescale/update-db-2
[pg_upgrade]: https://www.postgresql.org/docs/current/static/pgupgrade.html
[backup]: /using-timescaledb/backup
[Install]: /getting-started/installation
[telemetry]: /using-timescaledb/telemetry
[volumes]: https://docs.docker.com/engine/admin/volumes/volumes/
[bind-mounts]: https://docs.docker.com/engine/admin/volumes/bind-mounts/
[caggs]: /using-timescaledb/continuous-aggregates
[compression]: /using-timescaledb/compression
[retension]: /using-timescaledb/data-retension
[retention-cagg-changes]: /getting-started/changes-in-timescaledb-2#retention-and-caggs
[changes-in-ts2]: /getting-started/changes-in-timescaledb-2