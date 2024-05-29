1. Создал ВМ в YC и развернул там postgresql.
2. Создал тестовую базу.
	```
	kda@otus-home-work04:/etc/postgresql/16/main$ sudo psql -h 158.160.173.79 -U postgres
	Password for user postgres:
	psql (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	Type "help" for help.

	postgres-# \l
														   List of databases
	   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges
	-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
	 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           |
	 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
			   |          |          |                 |             |             |            |           | postgres=CTc/postgres
	(3 rows)

	postgres=# create database test;
	CREATE DATABASE
	```
3. Инициализирова pgbench
	```
	kda@otus-home-work04:/etc/postgresql/16/main$ sudo su postgres
	postgres@otus-home-work04:/etc/postgresql/16/main$ pgbench -i test
	dropping old tables...
	NOTICE:  table "pgbench_accounts" does not exist, skipping
	NOTICE:  table "pgbench_branches" does not exist, skipping
	NOTICE:  table "pgbench_history" does not exist, skipping
	NOTICE:  table "pgbench_tellers" does not exist, skipping
	creating tables...
	generating data (client-side)...
	100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)
	vacuuming...
	creating primary keys...
	done in 0.60 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.37 s, vacuum 0.03 s, primary keys 0.20 s).
	postgres@otus-home-work04:/etc/postgresql/16/main$
	```
4. Запустил pgbench
	```
	postgres@otus-home-work04:/etc/postgresql/16/main$ pgbench -c 50 -j 2 -P 10 -T 60 test
	pgbench (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	starting vacuum...end.
	progress: 10.0 s, 683.8 tps, lat 71.998 ms stddev 83.358, 0 failed
	progress: 20.0 s, 671.6 tps, lat 74.567 ms stddev 80.993, 0 failed
	progress: 30.0 s, 595.3 tps, lat 83.950 ms stddev 113.193, 0 failed
	progress: 40.0 s, 649.9 tps, lat 76.734 ms stddev 88.813, 0 failed
	progress: 50.0 s, 669.5 tps, lat 74.797 ms stddev 84.310, 0 failed
	progress: 60.0 s, 630.1 tps, lat 79.346 ms stddev 92.796, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 50
	number of threads: 2
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 39052
	number of failed transactions: 0 (0.000%)
	latency average = 76.787 ms
	latency stddev = 90.801 ms
	initial connection time = 63.018 ms
	tps = 650.314008 (without initial connection time)
	postgres@otus-home-work04:/etc/postgresql/16/main$
	```
5. На сайте pgtune сконфигурировал настройки в зависимости от характеристик ВМ
	![pgtune](https://raw.githubusercontent.com/jfdhdss/OTUS_Postgresql_course2024/main/pic/pgtune.png)
6. Создал файл с настройками и применил их(перезапустил postgresql)
	```
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$ nano pgtune.conf
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$ cat pgtune.conf
	# DB Version: 16
	# OS Type: linux
	# DB Type: mixed
	# Total Memory (RAM): 4 GB
	# CPUs num: 2
	# Connections num: 100
	# Data Storage: hdd

	max_connections = 100
	shared_buffers = 1GB
	effective_cache_size = 3GB
	maintenance_work_mem = 256MB
	checkpoint_completion_target = 0.9
	wal_buffers = 16MB
	default_statistics_target = 100
	random_page_cost = 4
	effective_io_concurrency = 2
	work_mem = 2621kB
	huge_pages = off
	min_wal_size = 1GB
	max_wal_size = 4GB
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$ exit
	exit
	kda@otus-home-work04:/etc/postgresql/16/main/conf.d$ sudo pg_ctlcluster 16 main restart
	kda@otus-home-work04:/etc/postgresql/16/main/conf.d$ sudo pg_lsclusters
	Ver Cluster Port Status Owner    Data directory              Log file
	16  main    5432 online postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log
	```
7. Запустил pgbench с новыми настройками. Увелечение производительности не значительное. было tps = 650.314008 стало tps = 650.809025 транзакций в секунду 
	```
	kda@otus-home-work04:/etc/postgresql/16/main/conf.d$ sudo su postgres
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$ pgbench -c 50 -j 2 -P 10 -T 60 test
	pgbench (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	starting vacuum...end.
	progress: 10.0 s, 660.7 tps, lat 74.504 ms stddev 78.396, 0 failed
	progress: 20.0 s, 618.9 tps, lat 80.773 ms stddev 94.045, 0 failed
	progress: 30.0 s, 636.5 tps, lat 78.707 ms stddev 93.714, 0 failed
	progress: 40.0 s, 686.0 tps, lat 72.756 ms stddev 75.801, 0 failed
	progress: 50.0 s, 632.7 tps, lat 78.054 ms stddev 89.750, 0 failed
	progress: 60.0 s, 669.2 tps, lat 75.793 ms stddev 85.589, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 50
	number of threads: 2
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 39090
	number of failed transactions: 0 (0.000%)
	latency average = 76.724 ms
	latency stddev = 86.313 ms
	initial connection time = 61.436 ms
	tps = 650.809025 (without initial connection time)
	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$
	```
8. Установил следующие параметры и снова запустил pgbench
	```
	kda@otus-home-work04:/etc/postgresql/16/main/conf.d$ cat pgtune.conf
	# DB Version: 16
	# OS Type: linux
	# DB Type: mixed
	# Total Memory (RAM): 4 GB
	# CPUs num: 2
	# Connections num: 100
	# Data Storage: hdd

	max_connections = 100
	shared_buffers = 2GB
	effective_cache_size = 4GB
	maintenance_work_mem = 320MB
	checkpoint_completion_target = 0.9
	wal_buffers = 36MB
	default_statistics_target = 100
	random_page_cost = 4
	effective_io_concurrency = 2
	work_mem = 128MB
	huge_pages = off
	min_wal_size = 1GB
	max_wal_size = 4GB

	postgres@otus-home-work04:/etc/postgresql/16/main/conf.d$ pgbench -c 50 -j 2 -P 10 -T 60 test
	pgbench (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	starting vacuum...end.
	progress: 10.0 s, 661.0 tps, lat 73.535 ms stddev 78.381, 0 failed
	progress: 20.0 s, 579.8 tps, lat 87.398 ms stddev 108.069, 0 failed
	progress: 30.0 s, 694.2 tps, lat 71.979 ms stddev 76.421, 0 failed
	progress: 40.0 s, 674.9 tps, lat 73.060 ms stddev 81.101, 0 failed
	progress: 50.0 s, 628.5 tps, lat 80.576 ms stddev 93.479, 0 failed
	progress: 60.0 s, 670.0 tps, lat 74.404 ms stddev 87.539, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 50
	number of threads: 2
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 39133
	number of failed transactions: 0 (0.000%)
	latency average = 76.623 ms
	latency stddev = 87.810 ms
	initial connection time = 65.998 ms
	tps = 651.558267 (without initial connection time)
	```
	Результат стал лучше tps = 651.558267 против tps = 650.314008
	
9. Установил парметр synchronous_commit = off на дефолтных параметрах postgre. Результат прям очень хорошоий), 595 против 2319 транзакци в секунду.
```
	postgres@work04:/home/kda$ pgbench -c 50 -j 2 -P 10 -T 60 test
	pgbench (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	starting vacuum...end.
	progress: 10.0 s, 652.1 tps, lat 75.403 ms stddev 81.900, 0 failed
	progress: 20.0 s, 561.7 tps, lat 88.117 ms stddev 102.172, 0 failed
	progress: 30.0 s, 580.6 tps, lat 86.985 ms stddev 121.097, 0 failed
	progress: 40.0 s, 597.8 tps, lat 83.352 ms stddev 97.031, 0 failed
	progress: 50.0 s, 605.1 tps, lat 82.776 ms stddev 108.348, 0 failed
	progress: 60.0 s, 570.7 tps, lat 87.795 ms stddev 122.035, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 50
	number of threads: 2
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 35730
	number of failed transactions: 0 (0.000%)
	latency average = 83.935 ms
	latency stddev = 105.980 ms
	initial connection time = 60.967 ms
	tps = 595.010564 (without initial connection time)


	postgres@work04:/home/kda$ pgbench -c 50 -j 2 -P 10 -T 60 test
	pgbench (16.3 (Ubuntu 16.3-1.pgdg20.04+1))
	starting vacuum...end.
	progress: 10.0 s, 2286.9 tps, lat 21.624 ms stddev 17.283, 0 failed
	progress: 20.0 s, 2318.2 tps, lat 21.510 ms stddev 18.655, 0 failed
	progress: 30.0 s, 2331.7 tps, lat 21.380 ms stddev 17.667, 0 failed
	progress: 40.0 s, 2345.7 tps, lat 21.253 ms stddev 17.812, 0 failed
	progress: 50.0 s, 2310.4 tps, lat 21.580 ms stddev 17.995, 0 failed
	progress: 60.0 s, 2323.3 tps, lat 21.451 ms stddev 17.606, 0 failed
	transaction type: <builtin: TPC-B (sort of)>
	scaling factor: 1
	query mode: simple
	number of clients: 50
	number of threads: 2
	maximum number of tries: 1
	duration: 60 s
	number of transactions actually processed: 139212
	number of failed transactions: 0 (0.000%)
	latency average = 21.478 ms
	latency stddev = 17.858 ms
	initial connection time = 60.058 ms
	tps = 2319.446644 (without initial connection time)
```


