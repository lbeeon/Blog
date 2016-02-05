title: 'C# To Python'
date: 2016-02-06 02:03:23
tags:
- C#
- Python
- MongoDB

---
## 隨手筆記

這幾個月我在公司的角色有了點改變，就是從後台開發被調去遊戲伺服器開發，在後台我是使用C#，而在遊戲伺服器則是使用Python，先聲明我完全沒寫過Python，不過上頭好像覺得我學什麼都很快，所以就調我過去支援，而兩邊開發最大的共同點就是都會使用MongoDB，而在C#開發的時候使用MongoDB似乎就沒有哪麼直覺，畢竟C#是強型別語言，而MongoDB又是支援Document type的NoSQL，因此每此在處理上就會有些不順手，不過這都不是重點，重點是在C#上使用MongoDB的習慣，好像也被我帶進Python...
這個話題就要從我review game server的程式碼開始說起，不過太長了我想還是直接講重點，就是我認為我們的code對schema的管控太鬆散，往往都是哪邊需要資料就直接在dict()內加一個key，這樣開發也沒有不好，不過個人是覺得當專案越來越大的時候，這樣的寫法可能就增加人為錯誤的發生，因此就讓我懷念起在C#開發時候對MongoDB schema的管理方式，一般而言，為了能夠讓資料讀回來放進List()內，都會先去宣告一個Class，再藉由這個Class為基礎去解析資料格式，因此所有的資料都依循宣告的資料型態以供使用，好處是，至少不會有人在某個地方偷塞一個key，你卻不知道，而這樣類似的寫法在nodejs也有提供，因此我就在想能不能提高Python對MongoDB Schema的管控，於是才有下面的程式碼，不過寫完後個人覺得完全沒屁用，唯一有用處的大概就是我知道怎麼使用 \_\_iter\_\_() ,next()的功能，因此還是紀錄一下這段沒有功用程式碼，
先示範一下一般新增一筆資料到MongoDB的最陽春寫法：
	
    if __name__ == '__main__':
    continue_period = 36000
    current_time = int(time.time())

    test_objectA =
    {
    	"id": str(current_time),
        "name": "diinben",
        "begin_time_ts": current_time,
        "end_time_ts": current_time + continue_period,
        "test_objectB":{
        	"num1": 50,
            "num2": 100
        }
    }
    
    conn = pymongo.MongoClient(host='localhost', port=27017)
    db = conn['TestDB']
    col = db['TestObjectA']
    ret = col.insert(test_objectA)
    print 'done: %s' % ret
    
大概類似這樣，直接產生一個dict()，然後把要key-value直接放進去，然後就新增，其實沒什麼太大問題，但是感覺上對於schema的管理就很鬆散，這樣的狀況下如果有人偷加key-value是很難用肉眼看出來的，因此我希望能改良成更有組織的程式碼，如下

    class MongoDBObject(JSONEncoder):
        def default(self, o):
            if isinstance(o, MongoDBObject):
                return o.__dict__
            else:
                raise TypeError(type(o) + " is not MongoDBObject")

        def __iter__(self):
            self.is_iter = True
            return self

        def next(self):
            if not self.is_iter :
                raise StopIteration
            self.is_iter = False
            docs = self.__dict__
            ret_dict = dict()
            for key in docs:
                if isinstance(docs[key], MongoDBObject):
                    ret_dict[key] = docs[key].__dict__
                else:
                    ret_dict[key] = docs[key]
            return ret_dict

    class TestObjectA(MongoDBObject):
        def __init__(self):
            pass

    class  TestObjectB(MongoDBObject):
        def __init__(self):
            pass

    if __name__ == '__main__':
        continue_period = 36000
        current_time = int(time.time())

        test_objectA = TestObjectA()
        
        test_objectA.id = str(current_time)
        test_objectA.name = "diinben"
        test_objectA.begin_time_ts = current_time
        test_objectA.end_time_ts = current_time + continue_period

        test_objectB = TestObjectB()
        test_objectB.num1 = 50
        test_objectB.num2 = 100
        test_objectA.test_objectB = test_objectB

        conn = pymongo.MongoClient(host='localhost', port=27017)
        db = conn['TestDB']
        col = db['TestObjectA']
        ret = col.insert(test_objectA)
        print 'done: %s' % ret
        
由於這邊我並沒有在TestObjectA和TestObjectB兩個Class內先定義參數，但是雛型可以看的出來，是可做的到類似C#的概念，不過就像我說的，這段程式碼沒什麼屁用，繞了一大圈寫了一堆程式碼，感覺好像也沒有直接使用dict()來的方便快速。