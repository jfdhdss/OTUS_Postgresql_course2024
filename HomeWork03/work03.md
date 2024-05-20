1. Созда VM в YC
2. Обновил пакеты
	```
	sudo apt update && sudo apt upgrade -y -q
	```
3. Установил postgresql 15
	```
	sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
	```
4. Проверил, запущен кластер или нет
	```
	kda@otus-home-wor3:~$ sudo -u postgres pg_lsclusters
	Ver Cluster Port Status Owner    Data directory              Log file
	15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
	```
5. Подключился к postgresql. Установил пароль user postgres
	```
	kda@otus-home-wor3:~$ sudo -u postgres psql
	psql (15.7 (Ubuntu 15.7-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# alter user postgres password 'postgres';
	ALTER ROLE
	```
6. Создал таблицу и вставил туда данные.
	```
	postgres=# create table test(c1 text);
	CREATE TABLE
	postgres=# insert into test values('1');
	INSERT 0 1
	postgres=# insert into test values('2');
	INSERT 0 1
	postgres-# \q
	```
7. Остановил postgresql. Тут было предупреждение, но кластер остановился. В итоге правильная команда для Ubuntu - sudo systemctl stop postgresql@15-main?
	```
	kda@otus-home-wor3:~$ sudo -u postgres pg_ctlcluster 15 main stop
	Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
	  sudo systemctl stop postgresql@15-main
	kda@otus-home-wor3:~$ sudo -u postgres pg_lsclusters
	Ver Cluster Port Status Owner    Data directory              Log file
	15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
	```
	
8. Создал в YC диск и подключил его к ВМ.
	```
	kda@otus-home-wor3:~$ sudo lsblk
	NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	vda    252:0    0  20G  0 disk
	├─vda1 252:1    0   1M  0 part
	└─vda2 252:2    0  20G  0 part /
	vdb    252:16   0   5G  0 disk
	kda@otus-home-wor3:~$
	```
9. Создадим раздел на подключенном диске
	```
	kda@otus-home-wor3:~$ sudo fdisk /dev/vdb

	Welcome to fdisk (util-linux 2.34).
	Changes will remain in memory only, until you decide to write them.
	Be careful before using the write command.

	Device does not contain a recognized partition table.
	Created a new DOS disklabel with disk identifier 0xf728de06.

	Command (m for help): m

	Help:

	  DOS (MBR)
	   a   toggle a bootable flag
	   b   edit nested BSD disklabel
	   c   toggle the dos compatibility flag

	  Generic
	   d   delete a partition
	   F   list free unpartitioned space
	   l   list known partition types
	   n   add a new partition
	   p   print the partition table
	   t   change a partition type
	   v   verify the partition table
	   i   print information about a partition

	  Misc
	   m   print this menu
	   u   change display/entry units
	   x   extra functionality (experts only)

	  Script
	   I   load disk layout from sfdisk script file
	   O   dump disk layout to sfdisk script file

	  Save & Exit
	   w   write table to disk and exit
	   q   quit without saving changes

	  Create a new label
	   g   create a new empty GPT partition table
	   G   create a new empty SGI (IRIX) partition table
	   o   create a new empty DOS partition table
	   s   create a new empty Sun partition table


	Command (m for help): n
	Partition type
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended (container for logical partitions)
	Select (default p): p
	Partition number (1-4, default 1):
	First sector (2048-10485759, default 2048):
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

	Created a new partition 1 of type 'Linux' and of size 5 GiB.

	Command (m for help): p
	Disk /dev/vdb: 5 GiB, 5368709120 bytes, 10485760 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 4096 bytes
	I/O size (minimum/optimal): 4096 bytes / 4096 bytes
	Disklabel type: dos
	Disk identifier: 0xf728de06

	Device     Boot Start      End  Sectors Size Id Type
	/dev/vdb1        2048 10485759 10483712   5G 83 Linux

	Command (m for help): w
	The partition table has been altered.
	Calling ioctl() to re-read partition table.
	Syncing disks.

	kda@otus-home-wor3:~$ sudo fdisk -l
	Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 4096 bytes
	I/O size (minimum/optimal): 4096 bytes / 4096 bytes
	Disklabel type: gpt
	Disk identifier: 25637092-3FEB-41CF-A2BB-20E068C29F21

	Device     Start      End  Sectors Size Type
	/dev/vda1   2048     4095     2048   1M BIOS boot
	/dev/vda2   4096 41943006 41938911  20G Linux filesystem


	Disk /dev/vdb: 5 GiB, 5368709120 bytes, 10485760 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 4096 bytes
	I/O size (minimum/optimal): 4096 bytes / 4096 bytes
	Disklabel type: dos
	Disk identifier: 0xf728de06

	Device     Boot Start      End  Sectors Size Id Type
	/dev/vdb1        2048 10485759 10483712   5G 83 Linux
	
	kda@otus-home-wor3:~$ sudo lsblk
	NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	vda    252:0    0  20G  0 disk
	├─vda1 252:1    0   1M  0 part
	└─vda2 252:2    0  20G  0 part /
	vdb    252:16   0   5G  0 disk
	└─vdb1 252:17   0   5G  0 part
	
	kda@otus-home-wor3:~$ sudo mkfs.ext4 /dev/vdb1
	mke2fs 1.45.5 (07-Jan-2020)
	Creating filesystem with 1310464 4k blocks and 327680 inodes
	Filesystem UUID: d62b5044-5363-4551-8578-606b71967cca
	Superblock backups stored on blocks:
			32768, 98304, 163840, 229376, 294912, 819200, 884736

	Allocating group tables: done
	Writing inode tables: done
	Creating journal (16384 blocks): done
	Writing superblocks and filesystem accounting information: done
	```
10. Создал католог и смонтировал туда диск
	```
	kda@otus-home-wor3:~$ sudo mkdir /mnt/vdb1
	kda@otus-home-wor3:~$ sudo mount /dev/vdb1 /mnt/vdb1
	kda@otus-home-wor3:~$ df -h
	Filesystem      Size  Used Avail Use% Mounted on
	udev            956M     0  956M   0% /dev
	tmpfs           198M  816K  197M   1% /run
	/dev/vda2        20G  2.8G   16G  15% /
	tmpfs           986M     0  986M   0% /dev/shm
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           986M     0  986M   0% /sys/fs/cgroup
	tmpfs           198M     0  198M   0% /run/user/1000
	/dev/vdb1       4.9G   24K  4.6G   1% /mnt/vdb1
	```
11. Прописываем монтирование в fstab.
	```
	kda@otus-home-wor3:~$ sudo blkid
	/dev/vda2: UUID="be2c7c06-cc2b-4d4b-96c6-e3700932b129" TYPE="ext4" PARTUUID="634e05c7-9520-4e9b-bcdb-3329603f1b2c"
	/dev/vda1: PARTUUID="c111767e-981d-4e57-85a0-ac66da561517"
	/dev/vdb1: UUID="d62b5044-5363-4551-8578-606b71967cca" TYPE="ext4" PARTUUID="f728de06-01"
	kda@otus-home-wor3:~$ sudo cp /etc/fstab /etc/fstab_old
	kda@otus-home-wor3:~$ sudo vim /etc/fstab
	kda@otus-home-wor3:~$ sudo cat /etc/fstab
	# /etc/fstab: static file system information.
	#
	# Use 'blkid' to print the universally unique identifier for a
	# device; this may be used with UUID= as a more robust way to name devices
	# that works even if disks are added and removed. See fstab(5).
	#
	# <file system> <mount point>   <type>  <options>       <dump>  <pass>
	# / was on /dev/vda2 during installation
	UUID=be2c7c06-cc2b-4d4b-96c6-e3700932b129 /               ext4    errors=remount-ro 0       1
	UUID=d62b5044-5363-4551-8578-606b71967cca /mnt/vdb1     ext4    defaults 0 0
	```
12. Перегрузил ВМ и убедился что диск смонтирован.
	```
	kda@otus-home-wor3:~$ df -h
	Filesystem      Size  Used Avail Use% Mounted on
	udev            956M     0  956M   0% /dev
	tmpfs           198M  808K  197M   1% /run
	/dev/vda2        20G  2.8G   16G  15% /
	tmpfs           986M  1.1M  985M   1% /dev/shm
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           986M     0  986M   0% /sys/fs/cgroup
	/dev/vdb1       4.9G   24K  4.6G   1% /mnt/vdb1
	tmpfs           198M     0  198M   0% /run/user/1000
	```
13. Сменил владельца 
	```
	kda@otus-home-wor3:~$ sudo chown -R postgres:postgres /mnt/vdb1
	kda@otus-home-wor3:~$ cd /mnt/
	kda@otus-home-wor3:/mnt$ ls -la
	total 12
	drwxr-xr-x  3 root     root     4096 May 20 08:55 .
	drwxr-xr-x 18 root     root     4096 May 20 09:04 ..
	drw
	xr-xr-x  3 postgres postgres 4096 May 20 08:53 vdb1
	```
14. Переместил каталог /var/lib/postgresql/15 в /mnt/vdb1/data. При запуске ошибка. Каталог /var/lib/postgresql/15/main пустой. А в конфиге он прописан /var/lib/postgresql/15/main
	```
	kda@otus-home-wor3:/var/lib/postgresql/15$ sudo mv /var/lib/postgresql/15 /mnt/vdb1/data
	kda@otus-home-wor3:/var/lib/postgresql/15$ sudo -u postgres pg_ctlcluster 15 main start
	Error: /var/lib/postgresql/15/main is not accessible or does not exist
	```
15. В конец файла /etc/postgresql/15/main/postgresql.conf добавляем строчку с параметром data_directory = '/mnt/vdb1/data/main' и запускаем кластер. Кластер запущен.
	```
	kda@otus-home-wor3:/var/lib/postgresql/15$ sudo vim /etc/postgresql/15/main/postgresql.conf
	kda@otus-home-wor3:/var/lib/postgresql/15$ sudo -u postgres pg_ctlcluster 15 main start
	Warning: the cluster will not be running as a systemd service. Consider using systemctl:
	  sudo systemctl start postgresql@15-main
	kda@otus-home-wor3:/var/lib/postgresql/15$ sudo pg_lsclusters
	Ver Cluster Port Status Owner    Data directory      Log file
	15  main    5432 online postgres /mnt/vdb1/data/main /var/log/postgresql/postgresql-15-main.log
	```
16. Подключаемся к кластеру и проверяем нашу таблицу
	```
	kda@otus-home-wor3:~$ sudo pg_lsclusters
	Ver Cluster Port Status Owner    Data directory      Log file
	15  main    5432 online postgres /mnt/vdb1/data/main /var/log/postgresql/postgresql-15-main.log
	kda@otus-home-wor3:~$ psql -h 158.160.167.24 -U postgres
	Password for user postgres:
	psql (15.7 (Ubuntu 15.7-1.pgdg20.04+1))
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	Type "help" for help.

	postgres=# \l
													 List of databases
	   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
	-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
	 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
	 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	(3 rows)

	postgres=# \dt
			List of relations
	 Schema | Name | Type  |  Owner
	--------+------+-------+----------
	 public | test | table | postgres
	(1 row)

	postgres=# select * from test;
	 c1
	----
	 1
	 2
	(2 rows)

	postgres=#
	```
Задание со звездочкой.

1. Останавливаем кластер на первой машине и отмантируем диск
	```
	kda@otus-home-wor3:~$ sudo pg_lsclusters
	Ver Cluster Port Status Owner    Data directory      Log file
	15  main    5432 down   postgres /mnt/vdb1/data/main /var/log/postgresql/postgresql-15-main.log
	kda@otus-home-wor3:~$ df -h
	Filesystem      Size  Used Avail Use% Mounted on
	udev            956M     0  956M   0% /dev
	tmpfs           198M  800K  197M   1% /run
	/dev/vda2        20G  2.8G   16G  15% /
	tmpfs           986M     0  986M   0% /dev/shm
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           986M     0  986M   0% /sys/fs/cgroup
	/dev/vdb1       4.9G   39M  4.6G   1% /mnt/vdb1
	tmpfs           198M     0  198M   0% /run/user/1000
	kda@otus-home-wor3:~$ umount /mnt/vdb1
	kda@otus-home-wor3:~$ sudo umount /mnt/vdb1
	kda@otus-home-wor3:~$ df -h
	Filesystem      Size  Used Avail Use% Mounted on
	udev            956M     0  956M   0% /dev
	tmpfs           198M  800K  197M   1% /run
	/dev/vda2        20G  2.8G   16G  15% /
	tmpfs           986M     0  986M   0% /dev/shm
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           986M     0  986M   0% /sys/fs/cgroup
	tmpfs           198M     0  198M   0% /run/user/1000
	kda@otus-home-wor3:~$
	```
2. Создаем вторую ВМ и ставим туда postgresql 15. Развертывание аналогично что и выше. Проверяем, что после установки кластер работает
	```
	kda@home-work03-01:~$ sudo pg_lsclusters
	Ver Cluster Port Status Owner    Data directory              Log file
	15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
	```
3. 	Подключился и проверил что бд пустая.
	```
	kda@home-work03-01:~$ sudo -u postgres psql
	psql (15.7 (Ubuntu 15.7-1.pgdg20.04+1))
	Type "help" for help.

	postgres=# alter user postgres password 'postgres';
	ALTER ROLE
	postgres=# \q
	kda@home-work03-01:~$ sudo psql -h 158.160.153.0 -U postgres
	Password for user postgres:
	psql (15.7 (Ubuntu 15.7-1.pgdg20.04+1))
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	Type "help" for help.

	postgres=# \l
													 List of databases
	   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
	-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
	 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
	 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	(3 rows)

	postgres=# \dt
	Did not find any relations.
	postgres=#
	```
4. Остановил кластер на второй ВМ и удалил каталог /var/lib/postgresql
	```
	kda@home-work03-01:~$ sudo rm -rf /var/lib/postgresql
	```
5. Отсоединил диск от первой ВМ и подсоединил ко второй ВМ
6. После подключения диска вижу такую картину
	```
	kda@home-work03-01:~$ sudo lsblk
	NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	vda    252:0    0  20G  0 disk
	├─vda1 252:1    0   1M  0 part
	└─vda2 252:2    0  20G  0 part /
	vdb    252:16   0   5G  0 disk
	└─vdb1 252:17   0   5G  0 part
	```
7. Создал каталог и смонтировал туда диск. Проверил владельца у каталога
	```
	kda@home-work03-01:~$ sudo mkdir /mnt/vdb1
	kda@home-work03-01:~$ sudo mount /dev/vdb1 /mnt/vdb1
	kda@home-work03-01:~$ df -h
	Filesystem      Size  Used Avail Use% Mounted on
	udev            956M     0  956M   0% /dev
	tmpfs           198M  812K  197M   1% /run
	/dev/vda2        20G  2.8G   16G  15% /
	tmpfs           986M  1.1M  985M   1% /dev/shm
	tmpfs           5.0M     0  5.0M   0% /run/lock
	tmpfs           986M     0  986M   0% /sys/fs/cgroup
	tmpfs           198M     0  198M   0% /run/user/1000
	/dev/vdb1       4.9G   39M  4.6G   1% /mnt/vdb1
	kda@home-work03-01:~$ ls -la /mnt/
	total 12
	drwxr-xr-x  3 root     root     4096 May 20 10:56 .
	drwxr-xr-x 18 root     root     4096 May 20 10:37 ..
	drwxr-xr-x  4 postgres postgres 4096 May 20 10:06 vdb1
	```
8. 	В конец файла /etc/postgresql/15/main/postgresql.conf добавляем строчку с параметром data_directory = '/mnt/vdb1/data/main' и запускаем кластер
	```
	kda@home-work03-01:~$ sudo vim /etc/postgresql/15/main/postgresql.conf
	kda@home-work03-01:~$ sudo pg_ctlcluster 15 main start
	kda@home-work03-01:~$ sudo pg_ctlcluster 15 main status
	pg_ctl: server is running (PID: 6577)
	/usr/lib/postgresql/15/bin/postgres "-D" "/mnt/vdb1/data/main" "-c" "config_file=/etc/postgresql/15/main/postgresql.conf"
	```
9. Проверяем нашу таблицу. Таблица на месте.
	```
	kda@home-work03-01:~$ sudo psql -h 158.160.153.0 -U postgres
	Password for user postgres:
	psql (15.7 (Ubuntu 15.7-1.pgdg20.04+1))
	SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
	Type "help" for help.

	postgres=# \l
													 List of databases
	   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
	-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
	 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
	 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
			   |          |          |             |             |            |                 | postgres=CTc/postgres
	(3 rows)

	postgres=# \dt
			List of relations
	 Schema | Name | Type  |  Owner
	--------+------+-------+----------
	 public | test | table | postgres
	(1 row)

	postgres=# select * from test
	postgres-# ;
	 c1
	----
	 1
	 2
	(2 rows)

	postgres=#
	```

11. Добавляем наш смонтированный диск в fstab(делаем аналогично как и выше).




	


	
	
