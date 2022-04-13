# Rueidis 問題記錄

> 以下為所有請教問題的記錄

## 2022年1月27日

### 問題

當我看到 Redis 6 時，我在想一個問題當一個資料在 Redis 被改變時，是 Redis 會主動馬上通知我呢？還是我還要再去問 Redis ？看起來 Redis 不會主動通知我去更新 Cache，還要自已跑去問再去更新 Cache

 <img src="../assets/image-20220309215214038.png" alt="image-20220309215214038" style="zoom:100%;" />

### 解答

如果有用到server-assisted client side caching 的話，Redis 會主動通知你喔圖片圈起來的部分是想問什麼問題呢？

### 問題

紅色圈起來的地方，像是程式碼主動去問 Redis，那如果不是，能否指出 Redis 會主動通知 Client 的程式碼位置？謝謝

### 解答

紅色圈起來的部分是程式在被動的讀取 redis 傳過來的訊息處理 redis invalidation notification 的地方是在裡面一點 [https://github.com/....../f957b83a911fc6....../pipe.go......](https://github.com/rueian/rueidis/blob/f957b83a911fc695a0a6137171d1a54e8a4724ce/pipe.go?fbclid=IwAR1uyznrRrjcTeuJiAaRJr7dEL_efS3U6hKj-G3r9lnOKh7ieqp6IbjRTRE#L271)

## 2022年2月3日

### 問題

pipe 的優點在程式碼裡可以看得到，DoMulti 函式可以處理向 Redis 傳送多條訊息，再一次讀取多條回應的結果請教一下，我有看到您把 pipe 的連線用 Close() 函式關閉，pipe 的連線不就無法再繼續接收到 Redis 的新訊息了，那您是如何恢復 pipe 的連線的？如果能的話，能否配合程式碼說明，謝謝

<img src="../assets/image-20220309220454983.png" alt="image-20220309220454983" style="zoom:100%;" />

### 解答

走到那邊的話，的確是沒有要讀取之後的訊息了重新連線的部分可以看這裡 [https://github.com/....../02cd0ba16fe89a4....../mux.go......](https://github.com/rueian/rueidis/blob/02cd0ba16fe89a4635d93a7a1f7b05e4d3fb6be7/mux.go?fbclid=IwAR0dG2Jhzb9PT2cdhgZA1IW5ICAFm2V3GLsRmtgXl4fv8zZObUztz4drXKU#L89)

## 2022年2月10日

### 問題

這一段程式碼就可以看出是環狀結構環狀的數量為 2 的 n 次方，比如 2^3 = 88 的二進制為 1000環狀的數量減一的值為 77 的二進制為 01111000 和 0111 用 & 操作為 0今天值只要一直增加，加到 8 時，用 & 操作，就直接跳回原點 0就為環狀結構請教一下，PutOne 和 PutMulti 兩個函式中，是否可以用 PutMulti 函式去取代 PutOne 函式的功能，可能您為了能節省效能，所以另外在多寫出 PutOne 函式，還是說 PutOne 和 PutMulti 有不同的意義和用途？謝謝

<img src="../assets/image-20220309220829597.png" alt="image-20220309220829597" style="zoom:100%;" /> 

### 解答

我也想用 PutMulti 取代 PutOne，不過還沒找到好方法另外你看的這份有點舊了，建議更新一下

## 2022年2月16日

### 問題

請教一個問題？大部份 sync.WaitGroup 的 Done 會在 Wait 的前面請問為何這裡程式碼，為何 Wait 會在 Done 的前面，我是認為 _pipe() 會在不同的地方被執行，但細節我目前不清楚？是否能說明運作流程？謝謝

<img src="../assets/image-20220309221250899.png" alt="image-20220309221250899" style="zoom:100%;" /> 

### 解答

是的，_pipe 同時間會被很多地方呼叫，但我希望同時只有一個能呼叫到 wireFn。因此有拿到 WaitGroup 的就等待結果即可可以參考官方的 singleflight pkg

### 版務人員提供參考資料

可以參考這個解釋比較詳細[https://xnum.github.io/2018/11/syncmap-loadorstore](https://xnum.github.io/2018/11/syncmap-loadorstore?fbclid=IwAR0uCsXXgFhLKd4CZ4ohf_QOakmTVmcntztmPHG2T_l8f3OqpIuE3HKKPRk)

## 2022年2月23日

請教一個問題，我去看程式的流程，當使用 Pipe 時，最後要寫入 EOF 的訊息作為結束，代表沒有資料要傳送了p.background() 會傳入 Pipe EOF 訊息，但在比較後面，而且有 if 判斷式，代表不一定會執行如果 p.background() 不及時執行的話，那不就代表 writeCmd(p.w, cmd.Commands()) 和 syncRead(p.r) 會發生互相等待，那整個 syncDo 函式不會停止我主要是想要請教 EOF 在程式碼裡是如何傳送和處理的？謝謝

<img src="../assets/image-20220309221926308.png" alt="image-20220309221926308" style="zoom:100%;" />

我預期 EOF 的寫入位置在這裡

<img src="../assets/image-20220309222216001.png" alt="image-20220309222216001" style="zoom:100%;" /> 

### 解答

可能你有什麼誤會，應該沒有 EOF 要寫入

### 問題

我一開始還覺得奇怪，把 mockClient.Close() 註解起來，整個程式永遠不會停止，進行互相等待的死結，後來想想這是合理的因為要告知 Pipe 沒有更多的資料要傳了，執行 Close() 函式時，(應該同時也傳入 EOF)我是想要請教您，這方面的問題，您是如何處理的？如果防止 mockClient 和 mockServer 互相等待的死結？沒有告知對方EOF的訊息，程式不會停的

<img src="../assets/image-20220309222356943.png" alt="image-20220309222356943" style="zoom:100%;" /> 

### 解答

喔，原來你說是 Go 的 io.EOF，這只是 Golang 用來表示一個 read stream 已經讀完了的訊息。而一個 tcp stream 什麼時候才會讀完？就是讀到對方送給你的 FIN 包的時候，這時候 Golang 會返回 io.EOF 給你。而你用的 io.ReadAll 的行為就是要把整個 stream 讀完，也就是讀到 io.EOF 才結束。在 Redis client 中 tcp stream 是重複使用的，不會用 io.ReadAll 來讀取

## 2022年3月2日

### 問題

我這樣理解，您看對不對？如果要在圖上找 pipe 物件的位置，我認為pipe 類實現 wire 接口，wire 接口可以讀寫 client side cache，因為這接口有 DoCache 函式再上一層的 conn 接口可以重置連線，因為這接口有 Overwrite 函式最後 mux 類為所有功能的大集合，就把它當成主要的控制端Client 接口為用戶介面，Client 接口馬上直接連接到 mux 控制端，就可以控制整個程式了這樣子理解對嗎？謝謝

<img src="../assets/image-20220309222617175.png" alt="image-20220309222617175" style="zoom:100%;" />

### 解答

是的，大致上就是這樣。我的區分方式是 pipe 負責一個 tcp connection。mux 則是負責多個 pipe。 

## 2022年3月17日

### 問題

請教您有關 Cache 的問題，我是覺得看您的程式碼，每看一次會有不同的啟發當 Linked List 用在 Cache 時，我是覺得 hit 數量最多的資料會往前移動但為何在下圖中，反而是往後移動？謝謝

<img src="../assets/image-20220329015537478.png" alt="image-20220329015537478" style="zoom:100%;" /> 

這是另一個版本的 Cache，是往前移動的，和您的程式碼不同

<img src="../assets/image-20220329020753735.png" alt="image-20220329020753735" style="zoom:80%;" /> 

### 解答

單純只是我選擇 list 後面放要留比較久的。要改成放前面也是可以

### 回應

了解，謝謝您，後來再去看程式碼，發現程式清除數據是從頭開始清除了，所以往後移動數據就可以留久一點

<img src="../assets/image-20220329021737097.png" alt="image-20220329021737097" style="zoom:100%;" /> 

## 2022年3月24日

### 問題

這是一個比較沒有變化的 LRU cache 的組件圖
您的 cache 很有趣，變化比較多，之後再畫成另一張組件圖作比較
想請教您，兩個問題
1 您的 cache 有使用 channel 作等待，cache 已經有 sync.RWMutex 多讀單寫的上鎖，
再加一個 channel 做更進一步的控制，讓有些讀取可以進行必要的等待，
我是覺得很有趣，我這樣理解正確嗎？
2 moveThreshold 臨界值的用途為？我是想不透
謝謝

<img src="../assets/image-20220414004906895.png" alt="image-20220414004906895" style="zoom:100%;" />

補上程式碼的圖片

<img src="../assets/image-20220414005013211.png" alt="image-20220414005013211" style="zoom:80%;" /> 

### 解答

1. 對，RWMutex 是為了取出 entry。取出後 entry 本身就用 channel 做 singleflight
2. moveThreshold 只是為了增加使用 RLock 的機會而已

### 回應

關於 single flight 的部份我還要在更仔細的看程式碼了，看看能不能再學習到更多技巧。
後來我再去查，程式不會把 "快取的擊中次數" 重置為零
代表只是單純想讓 "擊中次數超過臨界值的資料" 將更有機會留在快取裡。
謝謝您的引導和解答

<img src="../assets/image-20220414005258672.png" alt="image-20220414005258672" style="zoom:80%;" /> 

## 2022年4月7日

### 問題

我也把您上星期所說的改正如下圖，並用紫色來標記

也找到您提到 cache missing 的部份，但還是看不懂，請教您一下，關於這段程式碼
} else if entry != nil {
return newResult(entry.Wait(), nil)

我對這小段程式碼的理解為 當 LRU cache 有資料，再進行 single fligt 的等待

我想請教的是，LRU cache 都己經有資料了，為何還要等待，就直接回傳不就好了？

謝謝您

<img src="../assets/image-20220414003928521.png" alt="image-20220414003928521" style="zoom:100%;" />

### 解答

第一個分支的 v.typ != 0 才是真的 cache hit

那個 entry 就是之前提到的 singleflight placeholder，有拿到這個 placeholder 的就要等待
沒拿到 placeholder 的就創建一個並且往下走真的發出 request
