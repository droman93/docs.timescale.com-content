# Updating TimescaleDB versions [](update)

This section describes how to upgrade between different versions of
TimescaleDB. TimescaleDB supports **in-place updates**:
you don't need to dump and restore your data, and versions are published with
automated migration scripts that convert any internal state if necessary.

### TimescaleDB Release Compatability

TimescaleDB currently has two major release versions listed below. Please ensure that your version of
PostgreSQL is supported with the extension version you want to install or update.

 TimescaleDB Release |   Supported PostgreSQL Release
 --------------------|-------------------------------
 1.3 - 1.7.4         | 9.6, 10, 11, 12
 2.0                 | 11, 12

If you need to upgrade PostgreSQL first, please see [our documentation][upgrade-pg].

### Upgrade TimescaleDB

Use the instructions linked below or in the left menu based on the TimescaleDB extension version you are
attempting to update.

[TimescaleDB 1.x: Update software][update-db-1]

[TimescaleDB 2.0: Update software from TimescaleDB 1.3+][update-db-2]


[upgrade-pg]: /using-timescaledb/update-timescale/upgrade-pg
[update-db-1]: /using-timescaledb/update-timescale/update-db-1
[update-db-2]: /using-timescaledb/update-timescale/update-db-2
