title: Golang Intro
date: 2016-08-31 22:41:33
tags:
- golang

---
　　到公司也將近三個月了，是時候紀錄一下這陣子所學的新東西，不過我想也只是其中很小一部份而已，這三個月中大概我花了兩個多月的時間學習golang，因此這篇就決定簡單介紹一下這個滿新的語言，以及一些簡單的環境配置。　
  
　　Go又稱作golang是google開發的一種程式語言，根據[wiki](https://zh.wikipedia.org/wiki/Go)上的資料我才發現，這語言是2009年11月才推出，難怪stackoverflow上的參考資料真的是少之又少，在開始之前先來個結論好了，我認為golang這個語言的語法使用上並不算太難，由於是很新的語言因此很容易感受到，在語法設計上有很多語言的影子，例如C的mina,pointer，類似python的range寫法，像javascrpit不用分號的結尾等等，另外最特別的是channel的功能，在很多情境下能發揮出意想不到的效果，而golang的學習曲線在初期並不算是困難，但是在要進入開發產品的階段時可能會有一段陣痛期，過了之後就會覺得其實寫golang滿爽的，因此，爾後如果再開發小的micro service時，golang可能會是我偏好的一個語言，接下來就先簡單的介紹golang的環境安裝以及基本語法。 
  
  要玩golang當然要先安裝一下環境，可以直接到golang官網[https://golang.org/dl/](https://golang.org/dl/) 下載安裝即可，目前windows, osx, linux三個版本我都裝過，沒什麼太大問題，安裝完之後可以先開一個terminal鍵入go version如過安裝成功應該就換看到版本訊息，在windows環境下的話安裝後還要到環境變數中增加路徑安裝才算完成，到這邊golang大概裝好8成了，再來就是要設定GOPATH，GOPATH這個東西有點討厭，也有點難解釋，如果你有寫過node的話大概可以理解成global的node_modules，不過還是不太一樣就是了，有機會在好好解釋一下，不過就算沒有設定GOPATH，只要你有看到go version基本上就具備執行*.go的環境了，再來就是來一下golang的起手式，先建立一個C:\workspace\hello.go的檔案，將下列程式碼貼上 
  
    package main

    import "fmt"

    func main() {
        fmt.Printf("hello, world\n")
    }
再到terminal下面執行<strong>go run C:\workspace\hello.go</strong>應該就可以看到令人興奮的"hello, world"。 

　　golang這個語言有很多眉眉角角，例如一個檔案的第一行一定是"package XXX"，而且如果是一個程式的進入點的話一定要將package的名稱設定為main，而在這個進入點的程式內一定要有一個func main()，這寫法就很跟C是一樣的，再來簡單介紹一下golang的一些參數宣告和函式的寫法，先看以下範例

    package main

    import "fmt"

    func main(){
        var x int
        x = 2
        y := 3
        add, min := AddAndMinus(x, y)
        fmt.Printf("x:%v, y:%v, x+y:%v, x-y:%v", x, y, add, min )
    }

    func AddAndMinus(x int, y int) (int, int){
        return x+y, x-y
    }

前面兩行和第一個範例相同不解釋，在main裡面我定義了兩個變數x, y，不過兩個宣告的方式有點不太一樣，以x來說是先宣告x的型態為int，之後在給予初始值，而y的型態和初始值則同時賦予，由於golang是個強行別語言，所以在變數在宣告後就無法在轉型了，即便是Int32和Int64在一般情況下也不能直接操作，試想如果所有的資料都要先定義型態之後再賦予值在開發上會有多麼不便，用這些int, bool的基本型態可能覺得還好，但是要是調用第三方的lib就頭大了，因此，golang可以再賦予值的同時給予型態，用法就是在等號左邊加上冒號，上面的y, add, min都是用相同的初始方式，再來就是要介紹func定義，先看AddAndMinus的寫法，func是函式定義的開頭，再來是接函式的名稱，函示名稱後面第一個括號式要傳入的參數，在這個例子即是傳入兩個int型態的變數，接著後面的這個括號則是要回傳的值，有寫過python的人應該很熟悉，沒錯！！golang支援多值回傳，讓人寫起來輕鬆寫意，將上面的程式貼到hello.go，在執行一次<strong>go run C:\workspace\hello.go</strong>，應該就可以看到"x:2, y:3, x+y:5, x-y:-1"的結果。

　　說實在golang還有很多小細節，一篇文章有點難把我兩個月學的東西介紹完，例如：pointer, interfce(型態), struct, channel等等很多東西，有機會再來一一介紹，以下分享一些我們CTO介紹我看的學習資源
  
  
  * [https://learnxinyminutes.com/docs/go/](https://learnxinyminutes.com/docs/go/)
  * [https://www.golang-book.com/books/intro](https://www.golang-book.com/books/intro)
  * [https://gobyexample.com/](https://gobyexample.com/) 

編輯器目前則是vscode和atom在支援上較完整， 

這篇就先這樣吧。(to be continue...