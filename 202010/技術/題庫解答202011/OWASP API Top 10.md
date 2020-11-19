# OWASP API Top 10
```
https://secbuzzer.co/post/221
https://secbuzzer.co/post/222
https://secbuzzer.co/post/223
```
# 什麼是API？
```
API (Application Programming Interface) 是位於客戶端 (Client) 與伺服器端 (Server) 中間的介面或通訊協定
其目的為更加簡化客戶端軟體的構建，也常被形容為客戶端與伺服器端之間的合約。

以下歸納 API 與一般網頁上的差異：
1.邏輯使用從後端移到前端 (通常會伴隨著一些安全性弱點)
2.資料更多的使用 proxy
3.程式解析模組在客戶端而不是伺服器端 
4.用戶狀態通常由客戶端維護和監測
5.在 HTTP 請求當中會包含更多參數

由於 API 與一般網頁上特性的不同，因此下列弱點在 API 中會比較少見：
1.SQL injection
2.CSRF
3.Path Manipulation
```
# OWASP APT Top 10弱點剖析
```
API 1：無效的對象層級授權 (Broken Object Level Authorization)
API 2：無效的認證與授權(Broken Authentication)
API 3：過多資訊洩漏 (Excessive Data Exposure)
API 4：缺乏資源及速率的限制 (Lack of Resources & Rate Limiting)
API 5：無效函式權限控管 (Broken Function Level Authorization)
API 6：跨域指派 (Mass Assignment)
API 7：安全性配置錯誤 (Security Misconfiguration)
API 8：惡意指令注入 (Injection)
API 9：資產管理不當 (Improper Assets Management)
API 10：記錄及監控不足 (Insufficient Logging & Monitoring)
```
### API 1：無效的對象層級授權 (Broken Object Level Authorization)
```
可以透過修改請求當中的目標 ID 將非授權的帳號存取甚至修改到其他帳號的資料
在 API-based 的應用程式中相當常見
伺服器端注重在客戶端發送過來的參數，而不會充分地去追蹤客戶端的狀態

對象層級授權是實行於 Code Level 的驗證機制
Server 端應該要去檢查登入的使用者是否有權限修改或訪問該使用者欲處理的資料
否則可能會導致資訊洩漏、資料遺失或資料被竄改等，甚至整個帳戶被移轉或盜用

建議的預防措施：
1.依據使用者政策與等級制度來建立認證、授權機制
2.不使用從 Client 端傳來的 ID 資訊，改成使用存取在 session object 中的 ID
3.使用授權機制來檢查，確認是否登入的使用者有權使用其要求的動作
4.使用隨機且不可預測的值當作 ID
5.對授權機制進行測試及評估
```
### API 2：無效的認證與授權(Broken Authentication)
```
使用者透過 API 存取資料時會須經過認證機制進行驗證
加上軟體工程師對認證的界線與如何使用有許多的誤解
所以認證機制是一個明確的攻擊目標
認證缺乏保護機制或錯誤的使用認證也會發生問題

下列狀況代表 API 不安全：
1.允許憑證填充攻擊 (可能導致攻擊者拿到使用者的正確帳號與密碼)
2.允許針對同一使用者帳號進行暴力攻擊，並且沒有驗證碼或帳號鎖定的機制
3.允許使用者採用簡易的密碼
4.在 URL 當中含有機密資訊，像是 tokens 或密碼
5.沒有針對 tokens 的驗證
6.允許簡易的 JSON Web Token 並且沒有驗證到期日
7.使用簡易加密的金鑰或 API Keys

建議的預防措施：
1.了解整個 API 認證的流程
2.了解認證機制的用法，例如：OAuth 不是認證機制，也不是 API keys，是屬於授權機制
3.不要亂改認證機制、token 的生成方式及密碼的儲存方式，建議遵循NIST SP800-63標準
4.登入頁面、忘記密碼或其他具有身分驗證機制的功能，建議要具備防暴力破解、速率限制或帳號鎖定的功能，以防止針對特定用戶的暴力行為
5.建議使用多重因素認證(Multi-Factor Authentication)
6.建議定期宣導使用者資安意識，避免使用弱密碼，亦可從Server端強制設定要求使用者密碼長度或強健程度
7.API金鑰不應使用在使用者身分認證，僅適用於授權機制
```
### API 3：過多資訊洩漏 (Excessive Data Exposure)
```
API 因設計問題而洩漏、回傳過多的資訊
最主要的原因是現在許多網頁或應用程式都只依靠 client 端做資料的過濾
所以攻擊者只要簡單側錄封包，便能獲得所有資訊，造成用戶隱私外洩等狀況

該弱點無法被自動化工具偵測
工具難以區分哪些資料是合法且可公開，或為敏感資料不可外洩的情形。

建議的預防措施：
1.不使用 Client 端做敏感資料的過濾及分析
2.確保 API 的回應只包含合法資料
3.明確定義固定模式或文法，讓所有 API 回應的資料、錯誤訊息等，完全遵照定義之模式或文法
```
### API 4：缺乏資源及速率的限制 (Lack of Resources & Rate Limiting)
```
發生在 API 沒有設定資源大小或速率限制時
容易遭受 DDoS 攻擊，影響到 Server 的性能。
若 API 無需認證且可以同時接收多個請求，亦未使用 rate limiting 防護機制時，也容易產生此項弱點。

下列任一項限制未落實，便易遭受攻擊的 API：
1.執行 timeout
2.記憶體分配最大空間
3.File descriptors 的數量
4.Process 的數量
5.Request payload 的大小
6.Request 的數量
7.一個 Request 中所回傳的每頁記錄數量

建議的預防措施：
1.使用 Docker，因為在 Docker 中可以很簡單的限制 memory、CPU及processes 等
2.設定 Client 能再次呼叫 API 的時間限制
3.對於登入等敏感操作要使用 rate limits
4.添加伺服器端的驗證，特別是能控制回應數量等的參數
5.針對所有傳給 API 的參數、payload等加上 Maximum size 的限制
6.若 API 允許壓縮檔，在解壓縮前檢查壓縮比
```
### API 5：無效函式權限控管 (Broken Function Level Authorization)
```
攻擊者能透過合法 API 來取得未經授權的資料
並且越有結構、架構性的資料越好取得
因為可以透過簡單的預測，去修改 API 路徑與其值
通常具有管理者權限的功能會是攻擊者的重要目標。
目前發現此問題的最好方法為針對整個授權機制執行深入分析。

建議的預防措施：
1.拒絕在默認情況下所有來自Client端的存取，並明確制定使用者帳戶可以訪問功能項目
2.確保 Administrative Controllers 與Administrative Functions 都有執行基於 User Group/Role的授權檢查機制
```
### API 6：跨域指派 (Mass Assignment)
```
API 自動將從 Client 傳來的參數轉換成物件中的參數
沒有考慮到物件中有敏感參數，或使用者不能修改的參數 (例如：is_admin, user.cash, article.created_time)。
要完成此攻擊手法需要對商業邏輯、物件關係及 API 結構有一定的了解。
此攻擊可能產生權限提升、資料竄改與安全機制被繞過等攻擊。

建議的預防措施：
1.避免使用自動將輸入值綁定到物件參數的功能
2.建立白名單列出能被 client 修改的參數
3.內部建立黑名單列出不能被 client 修改的參數
4.建議預先定義，或執行讓輸入資料有固定的格式或模式
```
### API 7：安全性配置錯誤 (Security Misconfiguration)
```
API stack 的每一層都有可能發生此安全性配置錯誤的問題，從網路層一直到應用層都有可能
是可以被自動化工具所檢測到的
此問題可能會導致敏感資訊，甚至是系統的詳細資訊等機敏資料外流，以致整個系統被攻擊者控制。

API 滿足以下任何條件便是有漏洞的 API:
1.應用程式間沒有適當的安全加固，或是其在雲服務的權限配置是錯誤的
2.沒有更新到最新的 security patches 或是系統版本太舊
3.有不必要的功能被啟用，像是 HTTP verbs
4.沒有傳輸層安全性協定 (Transport Layer Security, TLS)
5.安全指令並沒有送到客戶端，像是 Security Headers
6.沒有設定跨來源資源共用 (Cross-origin Resource Sharing, CORS) 的政策或設置錯誤
7.沒有設定跨站請求偽造 (Cross-site request forgery, CSRF) 的政策或設置錯誤
8.錯誤訊息當中包含了 Stack traces 或是其他敏感資訊

建議的預防措施，一個正常 API 的 life cycle 應該要包含以下幾點：
1.重複的加固處理，可以讓適當的鎖定環境更簡單更快的部署
2.檢查且更新整個 API stack的配置，包含編排的檔案、 API 元件及雲服務 (e.g. S3 bucket permissions)
3.在 API 訪問靜態資產 (ex. image) 時要有安全的通訊渠道
4.建議要有個自動化的 process 不斷地來評估整個環境配置及設定的效用/效力
5.為了避免攻擊者從回應獲得例外追蹤或其他的敏感資料，如果可以的話，針對 API 的回應定義一個模式或樣式，其中也包含錯誤回應。
```
### API 8：惡意指令注入 (Injection)
```
攻擊者將惡意指令插入其他正常指令中
可能是透過直接輸入、參數等方法
在 SQL queries、LDAP queries、NoSQL queries、OS commands、XML parsers、ORM 當中是非常常見的一種攻擊手法
攻擊者可以用 Scanners 或 Fuzzers 來進行此種攻擊
很容易透過檢查程式碼來發現
也可能會導致資料洩漏、DoS 或甚至整台主機被接管、控制等嚴重狀況

什麼樣的 API 會有此種漏洞?
1.從 Client 傳過來的資料或從外部系統取得的資料沒有經過驗證、過濾或消毒
2.資料直接使用或是直接連結到 SQL queries、NoSQL queries、LDAP queries

建議的預防措施：
1.讓資料跟命令與 queries 分離
2.透過單一、可靠且有持續維護的 library 來做資料驗證
3.特殊字元應該要使用特殊的語法進行轉義
4.一個好的 API 應該要提供參數化的接口
5.限制所有 API回應資料的數量，避免大量洩漏資料
6.利用足夠的過濾器來對傳進 API 的資料進行驗證並且只允許有效的輸入參數
7.針對 API 的回應制定固定格式或模式
8.所有參數的資料類別及模式都要嚴格、明確定義
```
### API 9：資產管理不當 (Improper Assets Management)
```
攻擊者可以透過舊的 API 版本進入到系統當中來獲取資料或進行其他惡意行為
在舊的 API 版本當中，可能有許多未修補的漏洞，不用去面對現在最先進的安全機制
過期的文件也會讓系統端去尋找漏洞更加困難。

若缺乏資產庫存及退休的策略會導致舊的、有漏洞的系統持續在運作，進而造成敏感資訊洩漏等問題
現在也有許多不必要的 API 主機被暴露
因為現代的概念像是 microservices，都讓應用程式可以更簡單的配置及獨立。

以下情況代表 API 有此弱點:
1.API 的目的及目標不明確
2.API 的資訊不明確，像是 API 的執行環境、誰有權限呼叫 API、API 版本、API 收集且處理甚麼資料、data flow 是甚麼等等
3.沒有 API document，或是其 document 沒有更新
4.沒有每個 API 版本的退休計畫
5.主機的目錄遺失或過期
6.舊版的 API 未修補但卻持續運行

建議的預防措施：
1.對於每個 API，詳細列出所有 API 主機及 document 的重要部分，像是 API 環境、誰用有網路的存取權限、API 版本等
2.記錄綜合服務及 document 的重要資訊，像是甚麼 data 被交換，還有 data 敏感度等
3.記錄 API 的細節，像是 errors、rate limiting 等
4.透過公開標準自動化生成 API documentation
5.對於所有版本的API 接採取額外的保護措施，像是防火牆等
6.避免使用 production 與 non-production API 這樣的部署；如果真的無法避免，這些節點也都要採取與 production 相同的安全標準
7.當新的 API 版本有安全性的更新時，對舊版本進行執行風險評估決定對於舊版本的行動
```
### API 10：記錄及監控不足 (Insufficient Logging & Monitoring)
```
 API 沒有或缺乏任何 logging 及 monitoring，我們就幾乎不可能去追查任何可疑活動且即時回應
攻擊者也會有更充足的時間來進行攻擊
因為沒有任何的監視機制，也更難去發現攻擊者。

以下情況則代表 API 有此弱點:
1.沒有任何 Logs 或 Logging level 設定錯誤，又或是 Logs 太簡略並沒有包含詳細內容
2.Logs 的公正性不被保證
3.Logs 並沒有連續的在記錄
4.API 的基礎設施並沒有處於持續監控的狀態

本文建議的預防措施：
1.記錄所有失敗的認證嘗試、被拒絕的 access、輸入認證錯誤等
2.Logs 應該要有固定格式，並且其中要包含足夠的資訊可鑑定是否為惡意行為
3.Logs 要被當成敏感資訊來處理，且其公正性也要被保證
4.配置一個監控系統來持續監視基礎設施、網路以及 API functions
5.使用安全資訊及事件管理系統 (SIEM) 來統計且管理所有 API stack 跟主機
6.配置自定義的 dashboards 及警報，並且啟用針對可以行動的偵測，能夠更早回報
```