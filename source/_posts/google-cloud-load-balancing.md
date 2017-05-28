title: Google Cloud Load Balancing
date: 2017-05-28 22:34:33
tags:
- Google Cloud Platform
- GCP
- golang
- Load Balancing

---
### Google Cloud Load Balancing
　　延續著上一篇的步調，這次來談談Google Load Balancinig，至於為什麼我會需要操作或是說用程式來控制就讓我娓娓道來，我想有使用過雲端服務的人應該都大概知道Load Balancer的功用，簡單說Load Balancer就是能夠讓你將網站或是服務乘載量大幅提高的工具，不過前提是你的程式也要能支援這樣擴展性，那我這次被指派的任務是什麼呢？簡單說就是要幫我們的Load Balancer提供Https的Ssl憑證，並且還要定期去更新這個憑證，由於這種服務的控制正常流程都是上Google Cloud Console點一點就結束了，所以官方提供的[api/compute](https://godoc.org/google.golang.org/api/compute/v1)感覺上就是基於協定由程式產生的程式碼，所以使用起來真的是相當不直覺也沒有太多的說明，基本上可以當作是這個SDK幫你發一個POST到Compute Engine API，至於POST要提供的參數、呼叫API的先後順序大概只能用通靈的方法得知吧，因此這篇就簡單紀錄一下怎麼用這個SDK以及呼叫API的順序。
　　如上面所說的，Google提供的SDK相當的不方便，因此我自己就把這個SDK簡單的wrap，這個wrapper同時也實作了polling的功能，用過雲端服務的人基本上都知道，服務的開關不像是通電一樣說來就來，基本上都是需要一小段時間服務才會開始正常運作，因此如果你再不對的時間的要去存取一個還在準的資源(Resource)，那就會有很大的機會遇到一些怪怪的錯誤，尤其是像Load Balancer這種服務，如果你去Google Cloud Console上操作可能會覺得只有一個指令，但是如果用SDK操作那麼你最少會需要存取5~7種資源
  
	1. Backend Service
	2. Url Map
	3. Target Proxy
	4. HTTPS Target Proxy
	5. Global Address
	6. Forwardig Rule
	7. Certificate
  
這些條列的資源在Load Balancing的進階選項基本上都能看到，同時上面的順序也就是要產生一個Load Balancer所需要操作API的順序，至於各個資源各自所代表的意思和功能大家在自己上去文件看吧。
　　到此萬事都具備了，再來就是要操作SDK了，這邊簡單接紹一下我的[gcp-loadbalancing](https://github.com/lbeeon/gcloud-load-balancing)，由於[api/compute](https://godoc.org/google.golang.org/api/compute/v1)所涵蓋的功能超級多，所以我這邊也只將會用的進行處理，所提供的功能如下
  
* GetBackendService func(name string) (*compute.BackendService, error)

* GetUrlMap func(name string)(*compute.UrlMap, error)

* GetTargetHttpProxy func(name string)(*compute.TargetHttpProxy, error)

* GetTargetHttpsProxy func(name string)(*compute.TargetHttpsProxy, error)

* GetGlobalAddress func(name string) (*compute.Address, error)

* GetForwardRule func(name string) (*compute.ForwardingRule, error)

* GetSslCertificate func(name string)(*compute.SslCertificate, error)

* InsertUrlMap func(name string, defaultService string)(*compute.Operation, error)

* InsertTargetHttpProxy func(name string, urlMap string)(*compute.Operation, error)

* InsertTargetHttpsProxy func(name string, urlMap string, sslCert []string)(*compute.Operation, error)

* InsertTcpGlobalForwardRule func(name string, ipAddress string, targetProxy string, portRange string) (*compute.Operation, error)

* InsertGlobalAddress func(name string)(*compute.Operation, error)

* InsertCertificate func (name string, certificate string, privateKey string) (*compute.Operation, error)

* SetTargetHttpsProxySslCert func (name string, sslCert []string)(*compute.Operation, error)

基本上Get開頭就是去讀取資源的名稱、狀態或是其他詳細資訊，Insert開頭就是建立一個相對應的資源，不囉嗦直接看程式碼
  ```
import lbt "github.com/lbeeon/gcp-loadbalancing"

tools := lbt.NewLoadBalanceTools(projectID string)

// get backend service info
backendServ, err := tools.GetBackendService("service_name")

// polling result automatically
op, err := tools.InsertCertificate("new_cert", "certificate","privateKey")

op, err = tools.SetTargetHttpsProxySslCert("new_cert", []string{"https://www.googleapis.com/compute/v1/projects/xxxxxxxx-xxxxxx-xxxxx/global/sslCertificates/xxxxxxxx"})
  ```
第一行是載入要使用的package，再來就是要建立一個可以操作API的物件，這邊要提醒一下要操作API都是要提供Credentials的，那麼在這邊我的程式會和類似BigQuery SDK一樣自動去抓GOOGLE_APPLICATION_CREDENTIALS環境變數，我想有在使用GCP的人大概都知道這是官方強烈建議要設定的環境變數，因此用這個當作標準我想是很合情合理的一件事，不過要建立一個API的物件還是要提供一個ProjectID的參數，這邊的設計基本上也是想要依循其他SDK的標準，有了這個物件之後，再來就可以簡單的試用了，像是用Get的方法去看一下Backend Service的狀況阿，或是建立一個新的憑證，又或者是幫Https Target Proxy設定一個憑證都可以簡單的操作了。
　　當然如果你有一些自許或自尊什麼作祟的話那建議你可以去用官方提供的[api/compute](https://godoc.org/google.golang.org/api/compute/v1)，因為我也不保證我提供的程式碼完全沒有問題XD，不過在使用過這些時間以來我是沒有遇到什麼問題就是了，以上就是簡單紀錄一下怎麼使用SDK操作Google Load Balancing。
  