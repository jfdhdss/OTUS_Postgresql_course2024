1. Создал ВМ и установил Postgresql
2. Установил pg_probackup 
```
	kda@otus-home-work04:~$ sudo apt install gpg wget && wget -qO - https://repo.postgrespro.ru/pg_probackup/keys/GPG-KEY-PG-PROBACKUP |
	\ sudo tee /etc/apt/trusted.gpg.d/pg_probackup.asc
	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	gpg is already the newest version (2.2.19-3ubuntu2.2).
	wget is already the newest version (1.20.3-1ubuntu2).
	kda@otus-home-work04:~$ . /etc/os-release
	kda@otus-home-work04:~$ echo "deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb $VERSION_CODENAME main-$VERSION_CODENAME" | \
	> sudo tee /etc/apt/sources.list.d/pg_probackup.list
	deb [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb focal main-focal
	kda@otus-home-work04:~$ echo "deb-src [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb $VERSION_CODENAME main-$VERSION_CODENAME" | \
	> sudo tee -a /etc/apt/sources.list.d/pg_probackup.list
	deb-src [arch=amd64] https://repo.postgrespro.ru/pg_probackup/deb focal main-focal
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo apt install pg-probackup-16
	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	The following NEW packages will be installed:
	  pg-probackup-16
	0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
	Need to get 201 kB of archives.
	After this operation, 547 kB of additional disk space will be used.
	Get:1 https://repo.postgrespro.ru/pg_probackup/deb focal/main-focal amd64 pg-probackup-16 amd64 2.5.15-1.ac92457c2d1cfe43fced5b1167b5c90ecdc24cbe.focal [201 kB]
	Fetched 201 kB in 0s (2,723 kB/s)
	Selecting previously unselected package pg-probackup-16.
	(Reading database ... 107437 files and directories currently installed.)
	Preparing to unpack .../pg-probackup-16_2.5.15-1.ac92457c2d1cfe43fced5b1167b5c90ecdc24cbe.focal_amd64.deb ...
	Unpacking pg-probackup-16 (2.5.15-1.ac92457c2d1cfe43fced5b1167b5c90ecdc24cbe.focal) ...
	Setting up pg-probackup-16 (2.5.15-1.ac92457c2d1cfe43fced5b1167b5c90ecdc24cbe.focal) ...

```
3. Создал каталог для бэкапа
```
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo rm -rf /home/backups && sudo mkdir /home/backups && sudo chmod 777 /home/backups
```
4. Инициализируем каталог
```
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo su postgres
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ pg_probackup-16 init -B /home/backups
	INFO: Backup catalog '/home/backups' successfully initialized
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ cd /home/backups
	postgres@otus-home-work04:/home/backups$ ls -la
	total 16
	drwxrwxrwx 4 root     root     4096 May 24 07:42 .
	drwxr-xr-x 4 root     root     4096 May 24 07:40 ..
	drwx------ 2 postgres postgres 4096 May 24 07:42 backups
	drwx------ 2 postgres postgres 4096 May 24 07:42 wal
	postgres@otus-home-work04:/home/backups$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# show data_directory;
		   data_directory
	-----------------------------
	 /var/lib/postgresql/16/main
	(1 row)

	postgres=# \q
	postgres@otus-home-work04:/home/backups$ pg_probackup-16 add-instance --instance 'main' -D /var/lib/postgresql/16/main -B /home/backups
	INFO: Instance 'main' successfully initialized

```
5. Создал бд и таблицу. Вставил туда данные.
```
	postgres@otus-home-work04:/home/backups$ psql -c "CREATE DATABASE otus;"
	CREATE DATABASE
	postgres@otus-home-work04:/home/backups$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# \l
														   List of databases
	   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 otus      | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	(4 rows)

	postgres=# CREATE TABLE test(i int);
	CREATE TABLE
	postgres=# INSERT INTO test VALUES (10), (20), (30);
	INSERT 0 3
	postgres=# SELECT * FROM test;
	 i
	----
	 10
	 20
	 30
	(3 rows)
```
6. Сделал full бэкап
```
	postgres@otus-home-work04:/home/backups$ pg_probackup-16 backup --instance 'main' -b FULL --stream --temp-slot -B /home/backups
	INFO: Backup start, pg_probackup version: 2.5.15, instance: main, backup ID: SDZCFB, backup mode: FULL, wal mode: STREAM, remote: false, compress-algorithm: none, compress-level: 1
	WARNING: This PostgreSQL instance was initialized without data block checksums. pg_probackup have no way to detect data block corruption without them. Reinitialize PGDATA with option '--data-checksums'.
	WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_probackup under superuser.
	INFO: Database backup start
	INFO: wait for pg_backup_start()
	INFO: Wait for WAL segment /home/backups/backups/main/SDZCFB/database/pg_wal/000000010000000000000002 to be streamed
	INFO: PGDATA size: 29MB
	INFO: Current Start LSN: 0/2000110, TLI: 1
	INFO: Start transferring data files
	INFO: Data files are transferred, time elapsed: 0
	INFO: wait for pg_stop_backup()
	INFO: pg_stop backup() successfully executed
	INFO: stop_lsn: 0/2000250
	INFO: Getting the Recovery Time from WAL
	INFO: Syncing backup files to disk
	INFO: Backup files are synced, time elapsed: 6s
	INFO: Validating backup SDZCFB
	INFO: Backup SDZCFB data files are valid
	INFO: Backup SDZCFB resident size: 45MB
	INFO: Backup SDZCFB completed
	postgres@otus-home-work04:/home/backups/backups/main/SDZCFB/database$ pg_probackup-16 show -B /home/backups

	BACKUP INSTANCE 'main'
	================================================================================================================================
	 Instance  Version  ID      Recovery Time           Mode  WAL Mode  TLI  Time  Data   WAL  Zratio  Start LSN  Stop LSN   Status
	================================================================================================================================
	 main      16       SDZCFB  2024-05-24 07:50:01+00  FULL  STREAM    1/0   17s  29MB  16MB    1.00  0/2000110  0/2000250  OK

```
7. Вставил а таблицу новую строку. Сделал инкрементный бэкап.
```
	postgres=# INSERT INTO test VALUES (4);
	INSERT 0 1
	postgres=# select *
	postgres-# from test;
	 i
	----
	 10
	 20
	 30
	  4
	(4 rows)
	postgres@otus-home-work04:/home/backups/backups/main/SDZCFB/database$ pg_probackup-16 backup --instance 'main' -b DELTA --stream --temp-slot -B /home/backups
	INFO: Backup start, pg_probackup version: 2.5.15, instance: main, backup ID: SDZCQL, backup mode: DELTA, wal mode: STREAM, remote: false, compress-algorithm: none, compress-level: 1
	WARNING: This PostgreSQL instance was initialized without data block checksums. pg_probackup have no way to detect data block corruption without them. Reinitialize PGDATA with option '--data-checksums'.
	WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_probackup under superuser.
	INFO: Database backup start
	INFO: wait for pg_backup_start()
	INFO: Parent backup: SDZCFB
	INFO: Wait for WAL segment /home/backups/backups/main/SDZCQL/database/pg_wal/000000010000000000000004 to be streamed
	INFO: PGDATA size: 29MB
	INFO: Current Start LSN: 0/4000028, TLI: 1
	INFO: Parent Start LSN: 0/2000110, TLI: 1
	INFO: Start transferring data files
	INFO: Data files are transferred, time elapsed: 0
	INFO: wait for pg_stop_backup()
	INFO: pg_stop backup() successfully executed
	INFO: stop_lsn: 0/40001A0
	INFO: Getting the Recovery Time from WAL
	INFO: Syncing backup files to disk
	INFO: Backup files are synced, time elapsed: 0
	INFO: Validating backup SDZCQL
	INFO: Backup SDZCQL data files are valid
	INFO: Backup SDZCQL resident size: 17MB
	INFO: Backup SDZCQL completed
	postgres@otus-home-work04:/home/backups/backups/main/SDZCFB/database$ pg_probackup-16 show -B /home/backups

	BACKUP INSTANCE 'main'
	===================================================================================================================================
	 Instance  Version  ID      Recovery Time           Mode   WAL Mode  TLI  Time    Data   WAL  Zratio  Start LSN  Stop LSN   Status
	===================================================================================================================================
	 main      16       SDZCQL  2024-05-24 07:56:46+00  DELTA  STREAM    1/1   10s  1327kB  16MB    1.00  0/4000028  0/40001A0  OK
	 main      16       SDZCFB  2024-05-24 07:50:01+00  FULL   STREAM    1/0   17s    29MB  16MB    1.00  0/2000110  0/2000250  OK


```
8. Остановил postgres и очищаем данные
```
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo rm -rf /var/lib/postgresql/16/main/
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo ls -la /var/lib/postgresql/16/main
	ls: cannot access '/var/lib/postgresql/16/main': No such file or directory

```	
9. Восстанавливае данные из FULL бэкапа.
```
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo su postgres
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ pg_probackup-16 show -B /home/backups

	BACKUP INSTANCE 'main'
	===================================================================================================================================
	 Instance  Version  ID      Recovery Time           Mode   WAL Mode  TLI  Time    Data   WAL  Zratio  Start LSN  Stop LSN   Status
	===================================================================================================================================
	 main      16       SDZCQL  2024-05-24 07:56:46+00  DELTA  STREAM    1/1   10s  1327kB  16MB    1.00  0/4000028  0/40001A0  OK
	 main      16       SDZCFB  2024-05-24 07:50:01+00  FULL   STREAM    1/0   17s    29MB  16MB    1.00  0/2000110  0/2000250  OK
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ pg_probackup-16 restore --instance 'main' -i 'SDZCFB' -D /var/lib/postgresql/16/main/ -B /home/backups
	INFO: Validating backup SDZCFB
	INFO: Backup SDZCFB data files are valid
	INFO: Backup SDZCFB WAL segments are valid
	INFO: Backup SDZCFB is valid.
	INFO: Restoring the database from backup SDZCFB
	INFO: Start restoring backup files. PGDATA size: 45MB
	INFO: Backup files are restored. Transfered bytes: 45MB, time elapsed: 0
	INFO: Restore incremental ratio (less is better): 100% (45MB/45MB)
	INFO: Syncing restored files to disk
	INFO: Restored backup files are synced, time elapsed: 7s
	INFO: Restore of backup SDZCFB completed.
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ exit
	exit
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo pg_ctlcluster 16 main start
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo su postgres
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# \l
														   List of databases
	   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 otus      | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	(4 rows)

	postgres=# select * from test;
	 i
	----
	 10
	 20
	 30
	(3 rows)

	postgres=#
```	
9. Восстанавливаем инкрементный бэкап
```
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo pg_ctlcluster 16 main stop
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo rm -rf /var/lib/postgresql/16/main/
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo su postgres
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ pg_probackup-16 restore --instance 'main' -i 'SDZCQL' -D /var/lib/postgresql/16/main/ -B /home/backups
	INFO: Validating parents for backup SDZCQL
	INFO: Validating backup SDZCFB
	INFO: Backup SDZCFB data files are valid
	INFO: Validating backup SDZCQL
	INFO: Backup SDZCQL data files are valid
	INFO: Backup SDZCQL WAL segments are valid
	INFO: Backup SDZCQL is valid.
	INFO: Restoring the database from backup SDZCQL
	INFO: Start restoring backup files. PGDATA size: 45MB
	INFO: Backup files are restored. Transfered bytes: 45MB, time elapsed: 0
	INFO: Restore incremental ratio (less is better): 100% (45MB/45MB)
	INFO: Syncing restored files to disk
	INFO: Restored backup files are synced, time elapsed: 6s
	INFO: Restore of backup SDZCQL completed.
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ exit
	exit
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo pg_ctlcluster 16 main start
	kda@otus-home-work04:/etc/apt/trusted.gpg.d$ sudo su postgres
	postgres@otus-home-work04:/etc/apt/trusted.gpg.d$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# select * from test;
	 i
	----
	 10
	 20
	 30
	  4
	(4 rows)

	postgres=#
```
10. Настроить хранение старше 7 дней и не больше 2 полных копий
```
	kda@otus-home-work04:~$ sudo su postgres
	postgres@otus-home-work04:/home/kda$ pg_probackup-16 delete --instance 'main' --delete-expired --retention-window=7 --retention-redundancy=2 -B /home/backups
	INFO: Evaluate backups by retention
	INFO: Backup SDZCQL, mode: DELTA, status: OK. Redundancy: 1/2, Time Window: 0d/7d. Active
	INFO: Backup SDZCFB, mode: FULL, status: OK. Redundancy: 1/2, Time Window: 0d/7d. Active
	INFO: There are no backups to merge by retention policy
	INFO: There are no backups to delete by retention policy
	INFO: There is no WAL to purge by retention policy
	postgres@otus-home-work04:/home/kda$ psql -c 'alter system set archive_mode = on'
	ALTER SYSTEM
	postgres@otus-home-work04:/home/kda$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# alter system set archive_command = 'pg_probackup-16 archive-push -B /home/backups/ --instance=main --wal-file-path=%p --wal-file-name=%f --compress';
	ALTER SYSTEM
	postgres=# \q
	postgres@otus-home-work04:/home/kda$ exit
	exit
	kda@otus-home-work04:~$ sudo pg_ctlcluster 16 main restart
	kda@otus-home-work04:~$ sudo postgres
	sudo: postgres: command not found
	kda@otus-home-work04:~$ sudo su postgres
	postgres@otus-home-work04:/home/kda$ psql -c 'show archive_mode'
	 archive_mode
	--------------
	 on
	(1 row)

	postgres@otus-home-work04:/home/kda$ pg_probackup-16 backup --instance 'main' -b FULL --stream --temp-slot -B /home/backups
	INFO: Backup start, pg_probackup version: 2.5.15, instance: main, backup ID: SDZKP4, backup mode: FULL, wal mode: STREAM, remote: false, compress-algorithm: none, compress-level: 1
	WARNING: This PostgreSQL instance was initialized without data block checksums. pg_probackup have no way to detect data block corruption without them. Reinitialize PGDATA with option '--data-checksums'.
	WARNING: Current PostgreSQL role is superuser. It is not recommended to run pg_probackup under superuser.
	INFO: Database backup start
	INFO: wait for pg_backup_start()
	INFO: Wait for WAL segment /home/backups/backups/main/SDZKP4/database/pg_wal/000000010000000000000006 to be streamed
	INFO: PGDATA size: 29MB
	INFO: Current Start LSN: 0/6000060, TLI: 1
	INFO: Start transferring data files
	INFO: Data files are transferred, time elapsed: 0
	INFO: wait for pg_stop_backup()
	INFO: pg_stop backup() successfully executed
	INFO: stop_lsn: 0/60001A0
	INFO: Getting the Recovery Time from WAL
	INFO: Syncing backup files to disk
	INFO: Backup files are synced, time elapsed: 6s
	INFO: Validating backup SDZKP4
	INFO: Backup SDZKP4 data files are valid
	INFO: Backup SDZKP4 resident size: 45MB
	INFO: Backup SDZKP4 completed
	postgres@otus-home-work04:/home/kda$ psql otus -c "insert into test values (10);"
```
11. Восстанавливаем данные на определенную точку во времени.
```
	kda@otus-home-work04:~$ sudo pg_ctlcluster 16 main stop
	kda@otus-home-work04:~$ sudo rm -rf /var/lib/postgresql/16/main/
	kda@otus-home-work04:~$ sudo su postgres
	postgres@otus-home-work04:/home/kda$ pg_probackup-16 restore --instance 'main' -D /var/lib/postgresql/16/main/ -B /home/backups --recovery-target-time="2024-05-24 14:00:00+00"
	INFO: Validating backup SDZKP4
	INFO: Backup SDZKP4 data files are valid
	WARNING: Thread [1]: Could not read WAL record at 0/8000000
	ERROR: Thread [1]: WAL segment "/home/backups/wal/main/000000010000000000000008" is absent
	WARNING: Recovery can be done up to time 2024-05-24 11:05:15+00, xid 754 and LSN 0/743E6F8
	ERROR: Not enough WAL records to time 2024-05-24 14:00:00+00
	postgres@otus-home-work04:/home/kda$ date
	Fri 24 May 2024 11:08:03 AM UTC
	postgres@otus-home-work04:/home/kda$ pg_probackup-16 restore --instance 'main' -D /var/lib/postgresql/16/main/ -B /home/backups --recovery-target-time="2024-05-24 11:00:00+00"
	INFO: Validating backup SDZKP4
	INFO: Backup SDZKP4 data files are valid
	INFO: Backup validation completed successfully on time 2024-05-24 11:04:26+00, xid 752 and LSN 0/7428A58
	INFO: Backup SDZKP4 is valid.
	INFO: Restoring the database from backup SDZKP4
	INFO: Start restoring backup files. PGDATA size: 45MB
	INFO: Backup files are restored. Transfered bytes: 45MB, time elapsed: 0
	INFO: Restore incremental ratio (less is better): 100% (45MB/45MB)
	INFO: Syncing restored files to disk
	INFO: Restored backup files are synced, time elapsed: 7s
	INFO: Restore of backup SDZKP4 completed.
	postgres@otus-home-work04:/home/kda$ psql
	psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
			Is the server running locally and accepting connections on that socket?
	postgres@otus-home-work04:/home/kda$ exit
	exit
	kda@otus-home-work04:~$ sudo pg_ctlcluster 16 main start
	kda@otus-home-work04:~$ sudo su postgres
	postgres@otus-home-work04:/home/kda$ psql
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# \l
															   List of databases
		   Name        |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-------------------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 postgres          | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 readme_to_recover | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0         | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
					   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1         | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
					   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	(4 rows)

	postgres=#
```	

