# 19.3. Starting the Database Server

Before anyone can access the database, you must start the database server. The database server program is called `postgres`. The `postgres` program must know where to find the data it is supposed to use. This is done with the `-D` option. Thus, the simplest way to start the server is:

```
$ postgres -D /usr/local/pgsql/data
```

which will leave the server running in the foreground. This must be done while logged into the PostgreSQL user account. Without `-D`, the server will try to use the data directory named by the environment variable `PGDATA`. If that variable is not provided either, it will fail.

Normally it is better to start `postgres` in the background. For this, use the usual Unix shell syntax:

```
$ postgres -D /usr/local/pgsql/data >logfile 2>&1 &
```

It is important to store the server's stdout and stderr output somewhere, as shown above. It will help for auditing purposes and to diagnose problems. (See [Section 24.3](https://www.postgresql.org/docs/12/logfile-maintenance.html) for a more thorough discussion of log file handling.)

The `postgres` program also takes a number of other command-line options. For more information, see the [postgres](https://www.postgresql.org/docs/12/app-postgres.html) reference page and [Chapter 19](https://www.postgresql.org/docs/12/runtime-config.html) below.

This shell syntax can get tedious quickly. Therefore the wrapper program [pg\_ctl](https://www.postgresql.org/docs/12/app-pg-ctl.html) is provided to simplify some tasks. For example:

```
pg_ctl start -l logfile
```

will start the server in the background and put the output into the named log file. The `-D` option has the same meaning here as for `postgres`. `pg_ctl` is also capable of stopping the server.

Normally, you will want to start the database server when the computer boots. Autostart scripts are operating-system-specific. There are a few distributed with PostgreSQL in the `contrib/start-scripts` directory. Installing one will require root privileges.

Different systems have different conventions for starting up daemons at boot time. Many systems have a file `/etc/rc.local` or `/etc/rc.d/rc.local`. Others use `init.d` or `rc.d` directories. Whatever you do, the server must be run by the PostgreSQL user account _and not by root_ or any other user. Therefore you probably should form your commands using `su postgres -c '...'`. For example:

```
su postgres -c 'pg_ctl start -D /usr/local/pgsql/data -l serverlog'
```

Here are a few more operating-system-specific suggestions. (In each case be sure to use the proper installation directory and user name where we show generic values.)

* For FreeBSD, look at the file `contrib/start-scripts/freebsd` in the PostgreSQL source distribution.
*   On OpenBSD, add the following lines to the file `/etc/rc.local`:

    ```
    if [ -x /usr/local/pgsql/bin/pg_ctl -a -x /usr/local/pgsql/bin/postgres ]; then
        su -l postgres -c '/usr/local/pgsql/bin/pg_ctl start -s -l /var/postgresql/log -D /usr/local/pgsql/data'
        echo -n ' postgresql'
    fi
    ```
*   On Linux systems either add

    ```
    /usr/local/pgsql/bin/pg_ctl start -l logfile -D /usr/local/pgsql/data
    ```

    to `/etc/rc.d/rc.local` or `/etc/rc.local` or look at the file `contrib/start-scripts/linux` in the PostgreSQL source distribution.

    When using systemd, you can use the following service unit file (e.g., at `/etc/systemd/system/postgresql.service`):

    ```
    [Unit]
    Description=PostgreSQL database server
    Documentation=man:postgres(1)

    [Service]
    Type=notify
    User=postgres
    ExecStart=/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
    ExecReload=/bin/kill -HUP $MAINPID
    KillMode=mixed
    KillSignal=SIGINT
    TimeoutSec=0

    [Install]
    WantedBy=multi-user.target
    ```

    Using `Type=notify` requires that the server binary was built with `configure --with-systemd`.

    Consider carefully the timeout setting. systemd has a default timeout of 90 seconds as of this writing and will kill a process that does not notify readiness within that time. But a PostgreSQL server that might have to perform crash recovery at startup could take much longer to become ready. The suggested value of 0 disables the timeout logic.
* On NetBSD, use either the FreeBSD or Linux start scripts, depending on preference.
*   On Solaris, create a file called `/etc/init.d/postgresql` that contains the following line:

    ```
    su - postgres -c "/usr/local/pgsql/bin/pg_ctl start -l logfile -D /usr/local/pgsql/data"
    ```

    Then, create a symbolic link to it in `/etc/rc3.d` as `S99postgresql`.

While the server is running, its PID is stored in the file `postmaster.pid` in the data directory. This is used to prevent multiple server instances from running in the same data directory and can also be used for shutting down the server.

## 18.3.1. Server Start-up Failures

There are several common reasons the server might fail to start. Check the server's log file, or start it by hand (without redirecting standard output or standard error) and see what error messages appear. Below we explain some of the most common error messages in more detail.

```
LOG:  could not bind IPv4 address "127.0.0.1": Address already in use
HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

This usually means just what it suggests: you tried to start another server on the same port where one is already running. However, if the kernel error message is not `Address already in use` or some variant of that, there might be a different problem. For example, trying to start a server on a reserved port number might draw something like:

```
$ postgres -p 666
LOG:  could not bind IPv4 address "127.0.0.1": Permission denied
HINT:  Is another postmaster already running on port 666? If not, wait a few seconds and retry.
FATAL:  could not create any TCP/IP sockets
```

A message like:

```
FATAL:  could not create shared memory segment: Invalid argument
DETAIL:  Failed system call was shmget(key=5440001, size=4011376640, 03600).
```

probably means your kernel's limit on the size of shared memory is smaller than the work area PostgreSQL is trying to create (4011376640 bytes in this example). Or it could mean that you do not have System-V-style shared memory support configured into your kernel at all. As a temporary workaround, you can try starting the server with a smaller-than-normal number of buffers ([shared\_buffers](https://www.postgresql.org/docs/12/runtime-config-resource.html#GUC-SHARED-BUFFERS)). You will eventually want to reconfigure your kernel to increase the allowed shared memory size. You might also see this message when trying to start multiple servers on the same machine, if their total space requested exceeds the kernel limit.

An error like:

```
FATAL:  could not create semaphores: No space left on device
DETAIL:  Failed system call was semget(5440126, 17, 03600).
```

does _not_ mean you've run out of disk space. It means your kernel's limit on the number of System V semaphores is smaller than the number PostgreSQL wants to create. As above, you might be able to work around the problem by starting the server with a reduced number of allowed connections ([max\_connections](https://www.postgresql.org/docs/12/runtime-config-connection.html#GUC-MAX-CONNECTIONS)), but you'll eventually want to increase the kernel limit.

If you get an “illegal system call” error, it is likely that shared memory or semaphores are not supported in your kernel at all. In that case your only option is to reconfigure the kernel to enable these features.

Details about configuring System V IPC facilities are given in [Section 18.4.1](https://www.postgresql.org/docs/12/kernel-resources.html#SYSVIPC).

## 18.3.2. Client Connection Problems

Although the error conditions possible on the client side are quite varied and application-dependent, a few of them might be directly related to how the server was started. Conditions other than those shown below should be documented with the respective client application.

```
psql: could not connect to server: Connection refused
        Is the server running on host "server.joe.com" and accepting
        TCP/IP connections on port 5432?
```

This is the generic “I couldn't find a server to talk to” failure. It looks like the above when TCP/IP communication is attempted. A common mistake is to forget to configure the server to allow TCP/IP connections.

Alternatively, you'll get this when attempting Unix-domain socket communication to a local server:

```
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

The last line is useful in verifying that the client is trying to connect to the right place. If there is in fact no server running there, the kernel error message will typically be either `Connection refused` or `No such file or directory`, as illustrated. (It is important to realize that `Connection refused` in this context does _not_ mean that the server got your connection request and rejected it. That case will produce a different message, as shown in [Section 20.15](https://www.postgresql.org/docs/12/client-authentication-problems.html).) Other error messages such as `Connection timed out` might indicate more fundamental problems, like lack of network connectivity.
