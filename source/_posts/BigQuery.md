title: Intro & BigQuery
date: 2017-05-07 17:19:52
tags:
- BigQuery
- Google Cloud Platform
- GCP
- golang
---
### Intro
　　想想到meepshop也將在這個月底滿一年了，在這過去一年多內學了不少新東西，也玩了不少服務，不過主要都是以GCP(Google Cloud Platform)的服務為主，因此，接下來幾篇文章都會聚焦在這些服務上，也算是一個簡短的工作回顧吧，至於我有用過哪些服務呢(這邊以有用過SDK為主)，以下簡單列舉一下
  - BigQuery
  - Storage
  - Datastore
  - Spanner
  - SQL(Postgres)
  - PubSub
  - Networking(DNS, loadbalance etc...)   

以上這些服務基本上都是以golang作為開發主要語言，因此，之後的介紹也應該會是圍繞在如何操作SDK，並配合簡單範例以供參考，廢話不多說直接來進入主題吧。

### BigQuery
　　為什麼第一篇選擇介紹BigQuery呢？原因無它，因為她是我第一個用的GCP服務，簡單介紹一下什麼BigQuery，BigQuery是Google提供的一個服務，主要是用來統計大量的資料，特別的是它支援NoSQL的資料結構同時又能使用SQL like的語法查詢，因此可以說是相當強大，猶豫敝公司所使用的是NoSQL資料庫，因此，看到這些支援的特色整個就是很合胃口，所以公司就委託小弟我去簡單研究一下，在正式進入程式碼之前還有一個東西要提一下，就是Goolge的SDK貌似有兩種，一種是[google/google-api-go-client](https://github.com/GoogleCloudPlatform/google-cloud-go)，另一種則是[GoogleCloudPlatform/google-cloud-go](https://github.com/google/google-api-go-client)的版本，那麼這兩種有什麼差別呢？簡單來說google開頭的repos包含了所有google所提供的服務api-sdk，像是translate、drive以及GCP上面有提供的BigQuery、Datastore等等，而另外一個GoogleCloudPlatform開頭的，感覺比較像是將前面那一個SDK和一些語言本身的使用習慣wrap之後的SDK，因此，使用上相對會比較簡單，而在GCP的官方教學內容，大部分也都是以GoogleCloudPlatform為主，不過身為一個硬派玩家，只使用GoogleCloudPlatform的開發工具，有很多事情似乎辦不到，以BigQuery來說好了，如果你想要在程式裡面使用[UDF](https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions)的話，在GoogleCloudPlatform的SDK上是完全沒有辦法，不能的理由也非常簡單就是他根本沒有實作這個部分，因此，我一開始也是看官方推薦使用，但是在開發產品的時候才會發現，基本上要用的功能想要什麼就沒什麼，所以我這邊才會找到另外一個google-api-go-client的repos，不過如果你需要僅是非常基本的功能，那照著官方的文件應該就可以滿足你的需求，不過這邊主要還是要介紹比較底層SDK的使用方法，再來就簡單敘述一下怎麼使用這SDK吧。
  　
   這邊介紹一下整個範例程式的架構，簡單來說就是先建立一個BigQuery的client，再來就是到BigQuery建立一個要執行的Job並展示結果，詳細程式碼我最後會附上連結
```
func main() {
	// create a bigquery client
	jwtConfig, err := NewJWTConfig()
	if err != nil {
		log.Fatalln(err)
	}

	client := jwtConfig.Client(context.Background())
	bigqueryService, err := bigquery.New(client)
	if err != nil {
		log.Fatalln(err)
	}

	// create a jobsService
	jobsService = bigquery.NewJobsService(bigqueryService)

	// create an async job
	jobId, err := jobInsert(cmd)
	if err != nil {
		log.Fatalln(err)
	}

	// polling to check status
	err = jobDone(jobId)
	if err != nil {
		log.Fatalln(err)
	}

	// get result
	jobGetResult(jobId)
}
```
感覺上好像沒有什麼特別，不過和官方教學最大的差異是這裡執行的是一個非同步的Job，而同步和非同步的差別主要是在於執行時所能夠使用的資源和時間，因此如果你想要分析上百GB的資料，官方提供的方法是完全不管用的，因為在運算的時候你就會拿到timeout或是response too large的錯誤，這時候如果你看官方資料它會建議你開啟allow large results的選項，不過尷尬的是如果你要用這個功能就必須是一個非同步的Job，而官方的開發文件對這邊對非同步Job幾乎沒有什麼著墨，所以我這邊就當作是分享順便記錄方便之後使用吧。
　　要使用原生的api-sdk一開始就是需要建立一個認證連線，這邊我只用的是jwt的方法，配合各個服務所提供credentials來做一個認證，至於這個NewJWTConfig的方法也不是我發明的，而是直接參考GoogleCloudPlatform的認證方法在小改良一下，基本上GCP上面所有的服務如果要使用api-sdk這樣的方式都是通用的，有興趣各位在自己去爬爬原始碼吧，再來是要建立一個新的job
```
func jobInsert(cmd string) (string, error) {
	jobConfigQuery := bigquery.JobConfigurationQuery{
		Query: cmd,
		UserDefinedFunctionResources: []*bigquery.UserDefinedFunctionResource{&bigquery.UserDefinedFunctionResource{
			InlineCode: `
				function Calendar(row, emit){
					var startTime = new Date(row.start*1000), endTime = new Date(row.end*1000);
					while(endTime > startTime){
						emit({YEAR: startTime.getUTCFullYear(), MONTH: startTime.getUTCMonth()+1, Day: startTime.getUTCDate(), Hour: startTime.getUTCHours()});
						startTime.setTime(startTime.getTime()+60*60*1000)
					}
				}
				bigquery.defineFunction(
					'Calendar',                           // Name of the function exported to SQL
					['start', 'end'],                    // Names of input columns
					[{'name': 'YEAR', 'type': 'integer'},  // Output schema
					 {'name': 'MONTH', 'type': 'integer'},
					 {'name': 'Day', 'type': 'integer'},
					 {'name': 'Hour', 'type': 'integer'}
					],
					Calendar                       // Reference to JavaScript UDF
				);`,
		}},
	}
	jobConfig := bigquery.JobConfiguration{Query: &jobConfigQuery}

	result, err := jobsService.Insert(projectID, &bigquery.Job{Configuration: &jobConfig}).Do()
	if err != nil {
		return "", err
	}
	return result.JobReference.JobId, nil
}
```
要建立一個非同步的job最重要的事情就是要建立一個JobConfigurationQuery的條件，說真如果只憑GoDoc上面的說明要產生上面的程式根本是天方夜譚，我個人認為是不可能，因為所有api-sdk的文件都是機器產生出來的，所以教學範例也少的可憐，基本上能有這段程式碼，我是藉由google console所送出的封包，反向推敲才設定出來的結果，真的可以說是行行皆辛苦，如果你僅需要執行一個耗時很久查詢，而沒有需要UDF，基本上你可以直接把UserDefinedFunctionResources這東西整個拿掉，其實這整篇的重點有70%就是在這個JobConfigurationQuery的設置，至於後面的job的狀態和資料的讀取就請大家直接看原始碼吧，最後附上[範例程式](https://github.com/lbeeon/demo-bigquery)。