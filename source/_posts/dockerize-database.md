title: Dockerize Database
date: 2017-05-01 22:29:11
tags:
- Note
- Docker
- MongoDB
- PostgresSQL
- RethinkDB
- Redis
- MySQL

---
## Dockerize Database
　　繼上一篇dockerize MySQL之後，我心裡就很想把一些其他的資料庫補一補，畢竟身為一個後端工程師，玩過幾種資料庫也是合情合理吧，這邊所提及的資料庫基本上都是我工作上有接觸到的，就用這篇來記錄一下吧。
  
### MongoDB
　　要起一個MongoDB的語法其實相當簡單
	
    docker run -d -p 27017:27017 --name mongo mongo
另外如果你突然想要使用mongoshell但是又不想安裝MongoDB，也可以直接透過image中的shell來執行，語法參考如下

	# link to docker image (name: mongo)
	docker run -it --link mongo:mongo mongo bash -c 'mongo --host mongo'
    
    # link to ip
    docker run -it mongo bash -c 'mongo --host 127.0.0.1'

當初我要起MongoDB的原因主要是因為需要一個測試的資料庫，所以這邊也順便補上匯入資料的方法

	# restore to docker image (name: mongo)
    # --drop, -j, --noIndexRestore (not require)
	docker run -it --link mongo:mongo -v /data:/tmp mongo bash -c 'mongorestore --host mongo -j 1 --noIndexRestore --db 'test' --drop /tmp'
    
基本上在MongoDB我需要的操作就大概是上面這些。

### PostgresSQL
　　PostgresSQL是我最近這幾個禮拜才開始用的資料庫，一個歷史久遠且功能強大的資料庫，不囉嗦先來個起手式
 	
    docker run --name postgres -e POSTGRES_PASSWORD=0000 -p 5432:5432 -d postgres
另外也常會用到psql去執行一些*.sql，不過因為要執行的檔案一般都滿多的，所以我就沒有像MongoDB一樣用bash -c下去執行，而是直接進去執行他的bash
	
    # link to docker image (name: postgres)
    docker run -it --link postgres:db -v /sql/:/tmp/ postgres bash
這邊順便簡單紀錄一下我用到psql語法

	# connect to database
	psql -h target_host -U postres_user -d target_db
    
    # connect to database without password prompt
    # set env is the only way
	PGPASSWORD=0000 psql -h target_host -U postres_user -d target_db
    
    # exec sql file
	psql -h target_host -U postres_user -d target_db -f /tmp/file.sql
再來就是很常會用到匯出/匯入

	# dump
    pg_dump -Fc -v -f /tmp/file.dump -U postres_user -h db_host target_db
    
    # restore
    pg_restore -a -d target_db -h db_host -U postres_user -Fc /tmp/file.dump
另外我還有用到PostgresSQL的benchmark工具，由於pgbench基本上就可以寫一篇了吧，所以我這邊就簡單提一下怎麼執行就好，如果需要詳細測試參數可以參考[官方文件](https://www.postgresql.org/docs/9.5/static/pgbench.html)以及[Postgres Wiki](https://wiki.postgresql.org/wiki/Pgbenchtesting)

	# pgbench mark
    # initial test data
    PGPASSWORD=0000 pgbench host=130.211.247.227 -U postgres -i -s 70 bench
    
    # run with test data
    PGPASSWORD=0000 pgbench host=130.211.247.227 -U postgres -v -c 4 -j 2  -T 60 bench
以上就是目前我有用到PostgresSQL相關的指令。

### RethinkDB
　　雖說RethinkDB已經是夕陽資料庫了，但是就是這樣才更值得起一個docker container免得汙染我的電腦，不囉嗦直接複製貼上

	docker run --name rethinkdb -p 8080:8080 -p 28015:28015 -p 29015:29015 -d rethinkdb
基本上大部分的使用都是透過RethinkDB本身所提供的介面來操作，所以使用者就直接連上localhost:8080即可，另外28015是用來給sdk連線用，雖說他本身的介面已經提供滿強大的功能了，不過就是沒有資料的匯出/匯入，而且官方image內部的工具還不能執行!!!所以我只好請求[Open Source](https://github.com/petecoop/rethinkdb-driver)支援，這邊就一併附上吧

	docker run --rm --link rethinkdb:rethinkdb -v /dump.tar.gz:/tmp/dump.tar.gz petecoop/rethinkdb-driver rethinkdb-restoer --force -c rethinkdb /tmp/dump.tar.gzs

RethinkDB基本上就這樣吧，我也不想再過多著墨。

### Redis
　　其實要起一個Redis也就跟上面差不多，只是把image改個名字即可
  
  	docker run --name redis -d redis
不過我這邊大部分是想要用redis-cli，所以就順便寫一下

	# link to docker image (name: redis)
    docker -it redis bash -c 'redis-cli --host redis'
    
    # exec bash then call redis-cli
    docker -it redis bash
我對Redis的需求就沒有像上面那資料庫還需要匯入/匯出什麼的，所以這不就特別介紹了。

### MySQL & phpmyadmin
　　最後就加上之前介紹，我這邊就不多做解釋
  
    docker run --name mysql -e MYSQL_ROOT_PASSWORD=0000 -d mysql
    docker run --name myadmin -d --link mysql:db -p 8080:80 phpmyadmin/phpmyadmin
    
以上就是我目前有再用資料庫，以後如果有在玩新的資料庫再依依更新好了。
