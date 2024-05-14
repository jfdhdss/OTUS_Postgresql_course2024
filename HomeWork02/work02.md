1. Создал в yc виртуальную машину

2. Обновил список пакетов
	```
    kda@otus-db-postgre-work02:~$ sudo apt update
    ```

3. Установил набор пакетов, необходимый для доступа к репозиториям по протоколу HTTPS:
	-	apt-transport-https — активирует передачу файлов и данных через https;
	-	ca-сertificates — активирует проверку сертификаты безопасности;
	-	curl — утилита для обращения к веб-ресурсам;
	-	software-properties-common — активирует возможность использования скриптов для управления программным обеспечением.
	```
    kda@otus-db-postgre-work02:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ```
	
4. Добавил ключ GPG для официального репозитория Docker
	```
    kda@otus-db-postgre-work02:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	OK

    ```

5. Добавил репозиторий Docker в источники APT:
	```
    kda@otus-db-postgre-work02:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
	Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
	Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease
	Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease
	Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
	Get:5 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
	Get:6 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [43.0 kB]
	Fetched 101 kB in 1s (169 kB/s)
	Reading package lists... Done
	kda@otus-db-postgre-work02:~$
    ```

6. Обновил пакеты еще раз
	```
    kda@otus-db-postgre-work02:~$ sudo apt update
	Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
	Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease
	Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease
	Hit:4 https://download.docker.com/linux/ubuntu focal InRelease
	Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease
	Reading package lists... Done
	Building dependency tree
	Reading state information... Done
	3 packages can be upgraded. Run 'apt list --upgradable' to see them.
	kda@otus-db-postgre-work02:~$

    ```

7. Проверил, что пакеты будут браться из репозитория docker а не ubuntu
	```
    kda@otus-db-postgre-work02:~$ apt-cache policy docker-ce
	docker-ce:
		Installed: (none)
		Candidate: 5:26.1.2-1~ubuntu.20.04~focal
	Version table:
		5:26.1.2-1~ubuntu.20.04~focal 500
			500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
		5:26.1.1-1~ubuntu.20.04~focal 500

    ```

8. Установил docker
	```
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
	kda@otus-db-postgre-work02:~$ sudo systemctl status docker
	● docker.service - Docker Application Container Engine
		Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset>
		Active: active (running) since Tue 2024-05-14 11:53:48 UTC; 1min 18s ago
	TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
	Main PID: 3002 (dockerd)
      Tasks: 8
     Memory: 30.4M
     CGroup: /system.slice/docker.service
             └─3002 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/cont>

	May 14 11:53:47 otus-db-postgre-work02 systemd[1]: Starting Docker Application >
	May 14 11:53:47 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:47>
	May 14 11:53:47 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:47>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 dockerd[3002]: time="2024-05-14T11:53:48>
	May 14 11:53:48 otus-db-postgre-work02 systemd[1]: Started Docker Application C>
	lines 1-21/21 (END)

    ```

9. Создал каталог /var/lib/postgres

	```
   kda@otus-db-postgre-work02:~$ sudo mkdir /var/lib/postgres

    ```

10. Добавил себя в группу docker, чтобы не использовать sudo
	```
    kda@otus-db-postgre-work02:~$ sudo usermod -aG docker ${USER}
    ```

11. Извлек публичный образ postgres	
	```
    kda@otus-db-postgre-work02:~$ docker pull postgres:14
	14: Pulling from library/postgres
	b0a0cf830b12: Pull complete
	2f87c12e2712: Pull complete
	e6a43afda61d: Pull complete
	8cedd55c42f4: Pull complete
	a2bc7fe683cb: Pull complete
	0a45c24c5b42: Pull complete
	bd35783d3353: Pull complete
	d89e99066c8c: Pull complete
	24bb6b836d34: Pull complete
	f6a828f17546: Pull complete
	ab4316b5e1ff: Pull complete
	566ec9b17686: Pull complete
	828cc9d6e31e: Pull complete
	63879be3750f: Pull complete
	Digest: sha256:f5e7b2f87ec217f44490f35d83cc97c1ab3748e88b641dbf87d16b8b99a46edd
	Status: Downloaded newer image for postgres:14
	docker.io/library/postgres:14

    ```

12. Развернул контейнер. Я правильно указал каталог, который нужно прокинуть /var/lib/postgresql/data или можно все прокинуть /var/lib/postgresql/? Как правильней?	
	```
    docker run --name kda-work02-docker -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgre -e POSTGRES_DB=work02 -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
    ```
13. Пытаюсь сделать контейнер с клиентом и подключиться к контейнеру с postgres. Получаю ошибку, Не может отрезолвить имя
	```
	docker run -it postgres:14 psql -U postgre -h kda-work02-docker -p 5432
	psql: error: could not translate host name "kda-work02-docker" to address: Name or service not known
	```
14. Убил контейнер с postgres.
	```
	kda@otus-db-postgre-work02:~$ docker rm -f kda-work02-docker
	kda-work02-docker
	kda@otus-db-postgre-work02:~$ docker ps
	CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
	```
15. Создал сеть для docker
	```
	kda@otus-db-postgre-work02:~$ docker network create work
	cf5bca2f85f3b758c87820337af7458c446932a9492e6ecd968310383eea5f35
	kda@otus-db-postgre-work02:~$ docker network ls
	NETWORK ID     NAME      DRIVER    SCOPE
	6c14254b969d   bridge    bridge    local
	bbaf86746810   host      host      local
	9126dea52147   none      null      local
	cf5bca2f85f3   work      bridge    local
	```
16. Заново развернул контейнер
	```
    docker run --name kda-work02-docker -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgre -e POSTGRES_DB=work02 -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data --network work postgres:14
    kda@otus-db-postgre-work02:~$ docker ps
	CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
	e3ec7e29095c   postgres:14   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   kda-work02-docker

	```
17. Сделал контейнер с клиентом и подключаемся. Первый раз не указал бд к какой подключаться и получил ошибку.	
	```
	kda@otus-db-postgre-work02:~$ docker run -it --network work postgres:14 psql -U postgre -h kda-work02-docker -p 5432
	Password for user postgre:
	psql: error: connection to server at "kda-work02-docker" (172.18.0.2), port 5432 failed: FATAL:  database "postgre" does not exist
	
	kda@otus-db-postgre-work02:~$ docker run -it --network work postgres:14 psql -U postgre work02  -h kda-work02-docker -p 5432
	Password for user postgre:
	psql (14.12 (Debian 14.12-1.pgdg120+1))
	Type "help" for help.

	work02=#

	```
18. Создал таблицу и вставил несколько строк
	```
	work02=# \dt
	Did not find any relations.
	work02=# create table persons(id serial, first_name text, second_name text);
	CREATE TABLE
	work02=# insert into persons(first_name, second_name) values('petr', 'petrov');
	INSERT 0 1
	work02=# insert into persons(first_name, second_name) values('sergey', 'sergeev');
	INSERT 0 1
	work02=# select * from persons;
	 id | first_name | second_name
	----+------------+-------------
	  1 | petr       | petrov
	  2 | sergey     | sergeev
	(2 rows)

	work02=# \dt
			 List of relations
	 Schema |  Name   | Type  |  Owner
	--------+---------+-------+---------
	 public | persons | table | postgre
	(1 row)

	```
19. Подключился со своей машины к серверу с помощью pgadmin4
	(connect from pgadmin)[https://github.com/jfdhdss/OTUS_Postgresql_course2024/blob/main/pic/pgadmin.png]
	
20. Удалил контайнер. Как правильно останавливать базу данных в контейнере, чтобы не потерять данные?
	```
	kda@otus-db-postgre-work02:~$ docker rm -f kda-work02-docker
	kda-work02-docker
	kda@otus-db-postgre-work02:~$ docker ps
	CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
	
	```
21. Создал контейнер заново
	```
	kda@otus-db-postgre-work02:~$ docker run --name kda-work02-docker -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgre -e POSTGRES_DB=work02 -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data --network work postgres:14
	542b702fd2ba351d49be03a7fbc5d8341d809fb191b8bd6028b995eda94e5a58
	kda@otus-db-postgre-work02:~$ docker ps
	CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
	542b702fd2ba   postgres:14   "docker-entrypoint.s…"   6 seconds ago   Up 5 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   kda-work02-docker

	```
22. Подключился котейнером с клиентом к серверу и проверил данные. Данные остались после удаления.
	```
	kda@otus-db-postgre-work02:~$ docker run -it --network work postgres:14 psql -U postgre work02  -h kda-work02-docker -p 5432
	Password for user postgre:
	psql (14.12 (Debian 14.12-1.pgdg120+1))
	Type "help" for help.
	
	work02=# \dt
			List of relations
	Schema |  Name   | Type  |  Owner
	--------+---------+-------+---------
	public | persons | table | postgre
	(1 row)
	
	work02=# select * from persons
	work02-#
	work02-#
	work02-# ;
	id | first_name | second_name
	----+------------+-------------
	1 | petr       | petrov
	2 | sergey     | sergeev
	(2 rows)

	```


