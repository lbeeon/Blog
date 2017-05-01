title: docker-mysql-phpmyadmin
date: 2017-03-26 22:00:07
tags:
- Note
- Docker

---
## Dockerize your mysql & phpmyadmin
　　最近email從遠方捎來的一個訊息，就是很久以前我在別人的Blog上的留言居然有人回復了!! [那篇文章](http://omarghader.github.io/docker-tutorial-phpmyadmin-and-mysql-server/)其實就是一個Docker的教學，教學內容簡單說就是分別起一個mysql和一個phpmyadmin的image並且能用phpmyadmin訪問，那篇的文章其實寫得還算OK，但是他用的image版本似乎不是官方的，而且照著弄也會有錯誤，而回應我的答案也是怪怪的感覺走歪掉(要我bash連進去mysql，再跑個sql改個帳號密碼什麼的)，所以開一篇記錄一下，怎麼用兩行指令就起一個mysql+phpmyadmin的服務。


    docker run --name mysql -e MYSQL_ROOT_PASSWORD=0000 -d mysql
    docker run --name myadmin -d --link mysql:db -p 8080:80 phpmyadmin/phpmyadmin


執行之後就可以直接用瀏覽器連上localhost:8080。其實這個問題我已經在我看完官方的文件後就已經解決了，不過如果臨時要我起一個可能還是要再查一下，所以乾脆開一篇記錄一下方便以後使用。　

**ps: root/0000**