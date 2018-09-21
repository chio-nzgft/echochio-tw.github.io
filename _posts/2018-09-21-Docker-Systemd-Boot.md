---
layout: post
title: Docker-Systemd-Boot
date: 2018-09-21
tags: docker
---

#  Docker Containers on Boot with Systemd
```
$ sudo cat>>postgres.service<<EOF
[Unit]
Description=PostgreSQL container
Requires=docker.service
After=docker.service
[Service]
Restart=on-failure
RestartSec=10
ExecStartPre=-/usr/bin/docker stop postgres
ExecStartPre=-/usr/bin/docker rm postgres
ExecStart=/usr/bin/docker run --name postgres postgres:alpine
ExecStop=/usr/bin/docker stop postgres
[Install]
WantedBy=multi-user.target
EOF
```

```
$ sudo cp postgres.service /etc/systemd/system
$ sudo systemctl enable /etc/systemd/system/postgres.service
$ sudo systemctl start postgres.service
$ journalctl --no-pager -u postgres.service
```
```
-- Logs begin at Sun 2017-03-19 23:02:17 UTC, end at Fri 2018-09-21 06:34:33 UTC. --
Sep 21 06:32:52 6142f68ab944 systemd[1]: Starting PostgreSQL container...
Sep 21 06:32:52 6142f68ab944 docker[1205]: Error response from daemon: No such container: postgres
Sep 21 06:32:52 6142f68ab944 docker[1208]: Error response from daemon: No such container: postgres
Sep 21 06:32:52 6142f68ab944 systemd[1]: Started PostgreSQL container.
Sep 21 06:32:52 6142f68ab944 docker[1213]: The files belonging to this database system will be owned by user "postgres".
Sep 21 06:32:52 6142f68ab944 docker[1213]: This user must also own the server process.
Sep 21 06:32:52 6142f68ab944 docker[1213]: The database cluster will be initialized with locale "en_US.utf8".
Sep 21 06:32:52 6142f68ab944 docker[1213]: The default database encoding has accordingly been set to "UTF8".
Sep 21 06:32:52 6142f68ab944 docker[1213]: The default text search configuration will be setto "english".
Sep 21 06:32:52 6142f68ab944 docker[1213]: Data page checksums are disabled.
Sep 21 06:32:52 6142f68ab944 docker[1213]: fixing permissions on existing directory /var/lib/postgresql/data ... ok
Sep 21 06:32:52 6142f68ab944 docker[1213]: creating subdirectories ... ok
Sep 21 06:32:52 6142f68ab944 docker[1213]: selecting default max_connections ... 100
Sep 21 06:32:52 6142f68ab944 docker[1213]: selecting default shared_buffers ... 128MB
Sep 21 06:32:52 6142f68ab944 docker[1213]: selecting dynamic shared memory implementation ... posix
Sep 21 06:32:52 6142f68ab944 docker[1213]: creating configuration files ... ok
Sep 21 06:32:52 6142f68ab944 docker[1213]: running bootstrap script ... ok
Sep 21 06:32:52 6142f68ab944 docker[1213]: performing post-bootstrap initialization ... sh: locale: not found
Sep 21 06:32:52 6142f68ab944 docker[1213]: 2018-09-21 06:32:52.997 UTC [25] WARNING:  no usable system locales were found
Sep 21 06:32:53 6142f68ab944 docker[1213]: ok
Sep 21 06:32:53 6142f68ab944 docker[1213]: syncing data to disk ... ok
Sep 21 06:32:53 6142f68ab944 docker[1213]: Success. You can now start the database server using:
Sep 21 06:32:53 6142f68ab944 docker[1213]:     pg_ctl -D /var/lib/postgresql/data -l logfilestart
Sep 21 06:32:53 6142f68ab944 docker[1213]: WARNING: enabling "trust" authentication for local connections
Sep 21 06:32:53 6142f68ab944 docker[1213]: You can change this by editing pg_hba.conf or using the option -A, or
Sep 21 06:32:53 6142f68ab944 docker[1213]: --auth-local and --auth-host, the next time you run initdb.
Sep 21 06:32:53 6142f68ab944 docker[1213]: ****************************************************
Sep 21 06:32:53 6142f68ab944 docker[1213]: WARNING: No password has been set for the database.
Sep 21 06:32:53 6142f68ab944 docker[1213]:          This will allow anyone with access to the
Sep 21 06:32:53 6142f68ab944 docker[1213]:          Postgres port to access your database. In
Sep 21 06:32:53 6142f68ab944 docker[1213]:          Docker's default configuration, this is
Sep 21 06:32:53 6142f68ab944 docker[1213]:          effectively any other container on the same
Sep 21 06:32:53 6142f68ab944 docker[1213]:          system.
Sep 21 06:32:53 6142f68ab944 docker[1213]:          Use "-e POSTGRES_PASSWORD=password" to set
Sep 21 06:32:53 6142f68ab944 docker[1213]:          it in "docker run".
Sep 21 06:32:53 6142f68ab944 docker[1213]: ****************************************************
Sep 21 06:32:53 6142f68ab944 docker[1213]: waiting for server to start....2018-09-21 06:32:53.875 UTC [30] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.887 UTC [31] LOG:  database system was shut down at 2018-09-21 06:32:53 UTC
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.889 UTC [30] LOG:  database system is ready to accept connections
Sep 21 06:32:53 6142f68ab944 docker[1213]:  done
Sep 21 06:32:53 6142f68ab944 docker[1213]: server started
Sep 21 06:32:53 6142f68ab944 docker[1213]: /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.970 UTC [30] LOG:  received fast shutdown request
Sep 21 06:32:53 6142f68ab944 docker[1213]: waiting for server to shut down....2018-09-21 06:32:53.970 UTC [30] LOG:  aborting any active transactions
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.971 UTC [30] LOG:  worker process: logical replication launcher (PID 37) exited with exit code 1
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.972 UTC [32] LOG:  shutting down
Sep 21 06:32:53 6142f68ab944 docker[1213]: 2018-09-21 06:32:53.982 UTC [30] LOG:  database system is shut down
Sep 21 06:32:54 6142f68ab944 docker[1213]:  done
Sep 21 06:32:54 6142f68ab944 docker[1213]: server stopped
Sep 21 06:32:54 6142f68ab944 docker[1213]: PostgreSQL init process complete; ready for startup.
Sep 21 06:32:54 6142f68ab944 docker[1213]: 2018-09-21 06:32:54.090 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
Sep 21 06:32:54 6142f68ab944 docker[1213]: 2018-09-21 06:32:54.090 UTC [1] LOG:  listening on IPv6 address "::", port 5432
Sep 21 06:32:54 6142f68ab944 docker[1213]: 2018-09-21 06:32:54.093 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
Sep 21 06:32:54 6142f68ab944 docker[1213]: 2018-09-21 06:32:54.108 UTC [39] LOG:  database system was shut down at 2018-09-21 06:32:53 UTC
Sep 21 06:32:54 6142f68ab944 docker[1213]: 2018-09-21 06:32:54.109 UTC [1] LOG:  database system is ready to accept connections
```
