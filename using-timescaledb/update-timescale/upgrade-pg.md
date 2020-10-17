
# Upgrade PostgreSQL

If you are looking to upgrade the version of the **PostgreSQL instance** (e.g. from 10 to 12) rather than the version of the TimescaleDB extension, you have two choices. Either use [`pg_upgrade`][pg_upgrade] with the command:

 ```
 pg_upgrade -b oldbindir -B newbindir -d olddatadir -D newdatadir"
 ```

 or [backup][] and then restore into a new version of the instance.
